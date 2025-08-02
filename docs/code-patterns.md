## CODE_PATTERNS.md

```markdown
# Code Patterns and Examples

## Creating a New Overlay

### Basic Overlay Structure
```cpp
// OverlayExample.h
#pragma once
#include "Overlay.h"
#include "Config.h"
#include "iracing.h"

class OverlayExample : public Overlay
{
public:
    OverlayExample() : Overlay("OverlayExample") {}

protected:
    virtual void onEnable() override;
    virtual void onDisable() override;
    virtual void onUpdate() override;
    virtual void onConfigChanged() override;
    virtual void onSessionChanged() override;
    virtual float2 getDefaultSize() override { return float2(400, 200); }
    
    // Optional overrides
    virtual bool canEnableWhileNotDriving() const override { return false; }
    virtual bool canEnableWhileDisconnected() const override { return false; }

private:
    // Rendering resources
    Microsoft::WRL::ComPtr<IDWriteTextFormat> m_textFormat;
    Microsoft::WRL::ComPtr<IDWriteTextFormat> m_textFormatSmall;
    TextCache m_text;
    
    // State variables
    float m_lastValue = 0.0f;
    std::vector<float2> m_dataPoints;
};
```
Implementation Template
```cpp
// OverlayExample.cpp
#include "OverlayExample.h"

void OverlayExample::onEnable()
{
    // Initialize resources when overlay is shown
    onConfigChanged();
}

void OverlayExample::onDisable()
{
    // Clean up resources
    m_text.reset();
    m_dataPoints.clear();
}

void OverlayExample::onConfigChanged()
{
    // Reload configuration and recreate resources
    m_text.reset(m_dwriteFactory.Get());
    
    const std::string font = g_cfg.getString(m_name, "font", "Arial");
    const float fontSize = g_cfg.getFloat(m_name, "font_size", 16.0f);
    
    HRCHECK(m_dwriteFactory->CreateTextFormat(
        toWide(font).c_str(), 
        NULL,
        DWRITE_FONT_WEIGHT_NORMAL,
        DWRITE_FONT_STYLE_NORMAL,
        DWRITE_FONT_STRETCH_NORMAL,
        fontSize,
        L"en-us",
        &m_textFormat
    ));
    
    m_textFormat->SetTextAlignment(DWRITE_TEXT_ALIGNMENT_CENTER);
    m_textFormat->SetParagraphAlignment(DWRITE_PARAGRAPH_ALIGNMENT_CENTER);
}

void OverlayExample::onSessionChanged()
{
    // Handle session transitions
    m_dataPoints.clear();
    m_lastValue = 0.0f;
}

void OverlayExample::onUpdate()
{
    const float w = (float)m_width;
    const float h = (float)m_height;
    
    // Get telemetry data
    if (!ir_Speed.isValid())
        return;
        
    float speed = ir_Speed.getFloat() * 2.237f; // Convert to mph
    
    // Start rendering
    m_renderTarget->BeginDraw();
    
    // Clear background (handled by base class)
    
    // Draw content
    const float4 textColor = g_cfg.getFloat4(m_name, "text_color", float4(1,1,1,1));
    m_brush->SetColor(textColor);
    
    wchar_t text[256];
    swprintf_s(text, L"Speed: %.1f mph", speed);
    
    m_text.render(
        m_renderTarget.Get(),
        text,
        m_textFormat.Get(),
        10.0f,          // left margin
        w - 10.0f,      // right margin  
        h / 2.0f,       // vertical center
        m_brush.Get(),
        DWRITE_TEXT_ALIGNMENT_CENTER
    );
    
    m_renderTarget->EndDraw();
}
```
### Common Rendering Patterns
## Drawing Text with Caching
```cpp
// Use TextCache for performance
m_text.render(
    m_renderTarget.Get(),
    L"Hello World",
    m_textFormat.Get(),
    x,                              // left bound
    x + width,                      // right bound
    y + height/2,                   // vertical center
    m_brush.Get(),
    DWRITE_TEXT_ALIGNMENT_CENTER    // or _LEADING, _TRAILING
);
```
### Drawing Shapes
```cpp
// Filled Rectangle
D2D1_RECT_F rect = {x, y, x + width, y + height};
m_brush->SetColor(fillColor);
m_renderTarget->FillRectangle(&rect, m_brush.Get());

// Outlined Rectangle
m_brush->SetColor(strokeColor);
m_renderTarget->DrawRectangle(&rect, m_brush.Get(), strokeWidth);

// Rounded Rectangle
D2D1_ROUNDED_RECT roundedRect = {
    {x, y, x + width, y + height},
    radius, radius
};
m_renderTarget->FillRoundedRectangle(&roundedRect, m_brush.Get());

// Line
m_renderTarget->DrawLine(
    D2D1::Point2F(x1, y1),
    D2D1::Point2F(x2, y2),
    m_brush.Get(),
    strokeWidth
);
``` 
Creating Path Geometry (for complex shapes)
```cpp
// Create a path for a graph line
Microsoft::WRL::ComPtr<ID2D1PathGeometry> pathGeometry;
Microsoft::WRL::ComPtr<ID2D1GeometrySink> sink;

m_d2dFactory->CreatePathGeometry(&pathGeometry);
pathGeometry->Open(&sink);

sink->BeginFigure(
    D2D1::Point2F(points[0].x, points[0].y),
    D2D1_FIGURE_BEGIN_HOLLOW  // or D2D1_FIGURE_BEGIN_FILLED
);

for (size_t i = 1; i < points.size(); ++i) {
    sink->AddLine(D2D1::Point2F(points[i].x, points[i].y));
}

sink->EndFigure(D2D1_FIGURE_END_OPEN);  // or D2D1_FIGURE_END_CLOSED
sink->Close();

// Draw the path
m_renderTarget->DrawGeometry(pathGeometry.Get(), m_brush.Get(), 2.0f);
```
## Gradient Fills
```cpp
// Create gradient stops
D2D1_GRADIENT_STOP gradientStops[2];
gradientStops[0].color = D2D1::ColorF(D2D1::ColorF::Green);
gradientStops[0].position = 0.0f;
gradientStops[1].color = D2D1::ColorF(D2D1::ColorF::Red);
gradientStops[1].position = 1.0f;

// Create gradient stop collection
Microsoft::WRL::ComPtr<ID2D1GradientStopCollection> gradientStopCollection;
m_renderTarget->CreateGradientStopCollection(
    gradientStops,
    2,
    &gradientStopCollection
);

// Create linear gradient brush
Microsoft::WRL::ComPtr<ID2D1LinearGradientBrush> gradientBrush;
m_renderTarget->CreateLinearGradientBrush(
    D2D1::LinearGradientBrushProperties(
        D2D1::Point2F(x, y),
        D2D1::Point2F(x + width, y)
    ),
    gradientStopCollection.Get(),
    &gradientBrush
);

// Use gradient brush
m_renderTarget->FillRectangle(&rect, gradientBrush.Get());
```
### Configuration Patterns
## Reading Config Values
```cpp
// With defaults
float fontSize = g_cfg.getFloat(m_name, "font_size", 16.0f);
bool showSpeed = g_cfg.getBool(m_name, "show_speed", true);
float4 color = g_cfg.getFloat4(m_name, "text_color", float4(1,1,1,1));
std::string units = g_cfg.getString(m_name, "units", "mph");

// Check and provide defaults in one place
void OverlayExample::onConfigChanged()
{
    m_fontSize = g_cfg.getFloat(m_name, "font_size", 16.0f);
    m_textColor = g_cfg.getFloat4(m_name, "text_color", float4(1,1,1,1));
    m_showGrid = g_cfg.getBool(m_name, "show_grid", false);
    
    // Validate ranges
    m_fontSize = std::clamp(m_fontSize, 8.0f, 48.0f);
}
```
## Saving Config Values
```cpp
// Save user preferences
g_cfg.setBool(m_name, "enabled", isEnabled);
g_cfg.setInt(m_name, "display_mode", m_currentMode);
g_cfg.save();
```
### Data Collection Patterns
## Building Historical Data
```cpp
class OverlayGraph : public Overlay
{
private:
    static constexpr int MAX_SAMPLES = 300;  // 5 seconds at 60Hz
    std::vector<float> m_samples;
    
    void collectSample(float value)
    {
        m_samples.push_back(value);
        
        // Keep buffer size limited
        if (m_samples.size() > MAX_SAMPLES) {
            m_samples.erase(m_samples.begin());
        }
    }
    
    void onUpdate()
    {
        // Collect new sample
        if (ir_Speed.isValid()) {
            collectSample(ir_Speed.getFloat());
        }
        
        // Render graph from samples
        renderGraph();
    }
};
```
### Tracking Session Changes
```cpp
class OverlayLapTracker : public Overlay
{
private:
    int m_lastLap = -1;
    std::vector<float> m_lapTimes;
    
    void onUpdate()
    {
        int currentLap = ir_Lap.getInt();
        
        // Detect new lap
        if (currentLap > m_lastLap && m_lastLap >= 0) {
            float lastLapTime = ir_LapLastLapTime.getFloat();
            if (lastLapTime > 0) {
                m_lapTimes.push_back(lastLapTime);
                onNewLap(lastLapTime);
            }
        }
        
        m_lastLap = currentLap;
    }
    
    void onSessionChanged()
    {
        // Reset tracking for new session
        m_lastLap = -1;
        m_lapTimes.clear();
    }
};
```
### Performance Patterns
## Dirty Flag Pattern
```cpp
class OverlayExpensive : public Overlay
{
private:
    bool m_geometryDirty = true;
    Microsoft::WRL::ComPtr<ID2D1PathGeometry> m_cachedGeometry;
    
    void onConfigChanged()
    {
        m_geometryDirty = true;
    }
    
    void rebuildGeometry()
    {
        // Expensive geometry creation
        m_cachedGeometry.Reset();
        // ... create new geometry ...
        m_geometryDirty = false;
    }
    
    void onUpdate()
    {
        if (m_geometryDirty) {
            rebuildGeometry();
        }
        
        // Use cached geometry
        if (m_cachedGeometry) {
            m_renderTarget->DrawGeometry(m_cachedGeometry.Get(), m_brush.Get());
        }
    }
};
```
## Frame Skipping for Heavy Operations
```cpp
class OverlayHeavy : public Overlay
{
private:
    int m_frameCounter = 0;
    static constexpr int UPDATE_INTERVAL = 6;  // Update every 6 frames (~10Hz)
    
    void onUpdate()
    {
        m_frameCounter++;
        
        // Do heavy calculations
               // Do heavy calculations only every N frames
       if (m_frameCounter % UPDATE_INTERVAL == 0) {
           performHeavyCalculations();
       }
       
       // Always render using cached results
       renderCachedData();
   }
};
```
### Error Handling Patterns
## Safe Telemetry Access
```cpp
void onUpdate()
{
    // Check connection
    if (!ir_session.driverCarIdx >= 0) {
        // Not in car, show placeholder
        renderPlaceholder("Waiting for session...");
        return;
    }
    
    // Check data validity
    if (!ir_Speed.isValid()) {
        return;  // Skip frame if data not ready
    }
    
    // Safe array access
    int carIdx = ir_session.driverCarIdx;
    if (carIdx >= 0 && carIdx < IR_MAX_CARS) {
        float lapPct = ir_CarIdxLapDistPct.getFloat(carIdx);
        // Use lapPct safely
    }
}
```
## Resource Creation Error Handling
```cpp
void createResources()
{
    HRESULT hr = m_dwriteFactory->CreateTextFormat(
        L"Arial",
        NULL,
        DWRITE_FONT_WEIGHT_NORMAL,
        DWRITE_FONT_STYLE_NORMAL,
        DWRITE_FONT_STRETCH_NORMAL,
        16.0f,
        L"en-us",
        &m_textFormat
    );
    
    if (FAILED(hr)) {
        // Fall back to default font
        hr = m_dwriteFactory->CreateTextFormat(
            L"Segoe UI",  // System font
            NULL,
            DWRITE_FONT_WEIGHT_NORMAL,
            DWRITE_FONT_STYLE_NORMAL,
            DWRITE_FONT_STRETCH_NORMAL,
            16.0f,
            L"en-us",
            &m_textFormat
        );
        HRCHECK(hr);  // This will exit if system font fails
    }
}
```
### Layout Patterns
Dynamic Column Layout
```cpp
void layoutColumns()
{
    ColumnLayout layout;
    
    // Define columns with optional width
    layout.add(COLUMN_POSITION, 40.0f, 5.0f);     // Fixed width
    layout.add(COLUMN_NAME, 0.0f, 10.0f);         // Auto width
    layout.add(COLUMN_DELTA, 80.0f, 5.0f);        // Fixed width
    layout.add(COLUMN_SPEED, 60.0f, 5.0f, 10.0f); // Different padding
    
    // Calculate actual positions
    layout.layout(m_width);
    
    // Use in rendering
    const ColumnLayout::Column* nameCol = layout.get(COLUMN_NAME);
    if (nameCol) {
        m_text.render(
            m_renderTarget.Get(),
            driverName,
            m_textFormat.Get(),
            nameCol->textL,
            nameCol->textR,
            yPos,
            m_brush.Get(),
            DWRITE_TEXT_ALIGNMENT_LEADING
        );
    }
}
```
## Responsive Sizing
```cpp
void onUpdate()
{
    const float w = (float)m_width;
    const float h = (float)m_height;
    
    // Scale elements based on window size
    float fontSize = std::max(12.0f, h / 20.0f);
    float margin = w * 0.05f;  // 5% margin
    float elementHeight = h / 10.0f;
    
    // Adjust layout for very small windows
    if (w < 200 || h < 150) {
        fontSize = 10.0f;
        margin = 5.0f;
        // Simplified layout for tiny window
    }
}
```
### Animation Patterns
## Smooth Transitions
```cpp
class OverlayAnimated : public Overlay
{
private:
    float m_currentValue = 0.0f;
    float m_targetValue = 0.0f;
    static constexpr float SMOOTH_FACTOR = 0.1f;
    
    void onUpdate()
    {
        // Get new target
        if (ir_Speed.isValid()) {
            m_targetValue = ir_Speed.getFloat();
        }
        
        // Smooth interpolation
        m_currentValue += (m_targetValue - m_currentValue) * SMOOTH_FACTOR;
        
        // Use smoothed value for display
        renderSpeed(m_currentValue);
    }
};
```
## Flash/Pulse Effects
```cpp
class OverlayAlerts : public Overlay
{
private:
    float m_flashTimer = 0.0f;
    bool m_isFlashing = false;
    
    void onUpdate()
    {
        // Check for condition
        if (shouldShowAlert()) {
            m_isFlashing = true;
            m_flashTimer = 1.0f;
        }
        
        // Update flash
        if (m_isFlashing) {
            m_flashTimer -= 0.016f;  // ~60Hz
            if (m_flashTimer <= 0) {
                m_isFlashing = false;
            }
            
            // Calculate flash alpha
            float alpha = (sin(m_flashTimer * 10.0f) + 1.0f) * 0.5f;
            m_brush->SetColor(float4(1, 0, 0, alpha));
            
            // Draw alert
            renderAlert();
        }
    }
};
```
### Integration with iRacing Session
## Handling Session States
```cpp
void onUpdate()
{
    // Check session state
    int state = ir_SessionState.getInt();
    
    switch (state) {
        case irsdk_StateGetInCar:
            renderMessage("Get in car");
            break;
            
        case irsdk_StateWarmup:
            renderMessage("Warmup");
            break;
            
        case irsdk_StateParadeLaps:
            renderMessage("Pace lap");
            break;
            
        case irsdk_StateRacing:
            // Normal operation
            renderRaceData();
            break;
            
        case irsdk_StateCheckered:
            renderMessage("Checkered flag!");
            break;
    }
}
```
## Detecting Race Events
```cpp
void checkForEvents()
{
    static int lastSessionFlags = 0;
    int currentFlags = ir_SessionFlags.getInt();
    
    // Yellow flag started
    if (!(lastSessionFlags & irsdk_yellow) && (currentFlags & irsdk_yellow)) {
        onYellowFlag();
    }
    
    // Green flag
    if (!(lastSessionFlags & irsdk_green) && (currentFlags & irsdk_green)) {
        onGreenFlag();
    }
    
    // Checkered flag
    if (!(lastSessionFlags & irsdk_checkered) && (currentFlags & irsdk_checkered)) {
        onCheckeredFlag();
    }
    
    lastSessionFlags = currentFlags;
}
### Memory Management Patterns
## Vector Preallocation
```cpp
class OverlayTrackMap : public Overlay
{
private:
    std::vector<float2> m_trackPoints;
    
    void onEnable()
    {
        // Preallocate for typical track
        m_trackPoints.reserve(1000);
    }
    
    void collectTrackPoint(float2 point)
    {
        // Avoid reallocation during collection
        if (m_trackPoints.size() < m_trackPoints.capacity()) {
            m_trackPoints.push_back(point);
        }
    }
};
```
## Resource Pooling
```cpp
class OverlayMultiGraph : public Overlay
{
private:
    struct GraphResources {
        Microsoft::WRL::ComPtr<ID2D1PathGeometry> geometry;
        bool inUse = false;
    };
    
    std::vector<GraphResources> m_graphPool;
    
    GraphResources* getGraphResources()
    {
        // Find unused resource
        for (auto& res : m_graphPool) {
            if (!res.inUse) {
                res.inUse = true;
                return &res;
            }
        }
        
        // Create new if needed
        m_graphPool.emplace_back();
        m_graphPool.back().inUse = true;
        return &m_graphPool.back();
    }
    
    void releaseAllResources()
    {
        for (auto& res : m_graphPool) {
            res.inUse = false;
        }
    }
};