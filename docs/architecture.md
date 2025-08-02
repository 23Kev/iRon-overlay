# iRon Architecture Overview

## Core Components

### Base Overlay Class Pattern
Every overlay inherits from the `Overlay` base class (Overlay.h/cpp) which provides:
- Window creation and management (borderless, topmost, transparent)
- Direct2D/DirectWrite rendering setup
- Position/size persistence
- Config change handling
- UI edit mode for repositioning

### Config System
- `Config` class (Config.h/cpp) manages JSON configuration
- Auto-reloads on file changes
- Provides typed accessors: `getBool()`, `getInt()`, `getFloat()`, `getFloat4()`, `getString()`
- Saves window positions and user preferences

### Rendering Pipeline
- **Direct2D**: Hardware-accelerated 2D graphics
- **DirectWrite**: Text rendering
- **DXGI**: Swap chain management for transparency
- **Windows Composition**: Layered windows with alpha blending
- Triple-buffered at 60Hz (or 30Hz in performance mode)

### iRacing SDK Integration
- `irsdkCVar` class wraps telemetry variables
- `ir_tick()` updates connection and session data
- Session data parsed from YAML string
- Telemetry data updated at 60Hz

### Main Loop and Hotkey Handling
- Windows message pump processes hotkeys
- Each overlay can have toggle hotkey
- UI edit mode (Alt+J by default) for positioning
- Frame-based update distribution in performance mode

## Adding New Overlays

### Step 1: Create Header File
```cpp
// OverlayExample.h
#pragma once
#include "Overlay.h"

class OverlayExample : public Overlay
{
public:
    OverlayExample() : Overlay("OverlayExample") {}

protected:
    virtual void onEnable();
    virtual void onDisable();
    virtual void onUpdate();
    virtual void onConfigChanged();
    virtual void onSessionChanged();
    virtual float2 getDefaultSize() { return float2(400, 300); }
    
private:
    // State variables
    Microsoft::WRL::ComPtr<IDWriteTextFormat> m_textFormat;
    TextCache m_text;
};
```
### Step 2: Implement Core Methods
```cpp
void OverlayExample::onEnable()
{
    // Called when overlay is shown
    onConfigChanged(); // Load fonts/settings
}

void OverlayExample::onConfigChanged()
{
    // Reload configuration
    const float fontSize = g_cfg.getFloat(m_name, "font_size", 16.0f);
    HRCHECK(m_dwriteFactory->CreateTextFormat(...));
}

void OverlayExample::onUpdate()
{
    // Render overlay content
    m_renderTarget->BeginDraw();
    // ... rendering code ...
    m_renderTarget->EndDraw();
}

```
### Step 3: Add to Main.cpp
```cpp
overlays.push_back(new OverlayExample());

```
### Step 4: Add hotkey support (optional)
```cpp
// In main.cpp
enum class Hotkey {
    // ... existing hotkeys ...
    Example
};

// Register hotkey
if (parseHotkey(g_cfg.getString("OverlayExample", "toggle_hotkey", "ctrl-5"), &mod, &vk))
    RegisterHotKey(NULL, (int)Hotkey::Example, mod, vk);
```
### Rendering Best Practices
Performance Tips

1. Cache text layouts using TextCache class
2. Minimize state changes between draw calls
3. Use geometry instancing for repeated shapes
4. Enable performance mode for 30Hz updates on slower systems

### Common Patterns
## Drawing Text
```cpp
cppm_brush->SetColor(g_cfg.getFloat4(m_name, "text_color", float4(1,1,1,1)));
m_text.render(m_renderTarget.Get(), L"Hello World", 
              m_textFormat.Get(), x, x+width, y, 
              m_brush.Get(), DWRITE_TEXT_ALIGNMENT_CENTER);
```
## Drawing Shapes
```cpp 
// Rectangle
D2D1_RECT_F rect = {x, y, x+width, y+height};
m_brush->SetColor(color);
m_renderTarget->FillRectangle(&rect, m_brush.Get());

// Rounded Rectangle
D2D1_ROUNDED_RECT rr = {{x, y, x+w, y+h}, cornerRadius, cornerRadius};
m_renderTarget->DrawRoundedRectangle(&rr, m_brush.Get(), strokeWidth);
```
## Creating Path Geometry
```cpp
Microsoft::WRL::ComPtr<ID2D1PathGeometry1> path;
Microsoft::WRL::ComPtr<ID2D1GeometrySink> sink;
m_d2dFactory->CreatePathGeometry(&path);
path->Open(&sink);
sink->BeginFigure(startPoint, D2D1_FIGURE_BEGIN_FILLED);
sink->AddLine(nextPoint);
// ... more points ...
sink->EndFigure(D2D1_FIGURE_END_CLOSED);
sink->Close();
```
## Color Configuration
All colors use float4 (RGBA) with values 0-1:

```cpp 
float4 color = g_cfg.getFloat4(m_name, "background_color", float4(0,0,0,0.7f));
```
## Safe Rendering
Always check for valid data before rendering:
```cpp
if (ir_session.driverCarIdx >= 0 && ir_Speed.isValid()) {
    // Safe to use speed data
}
