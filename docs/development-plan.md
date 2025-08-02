# iRon Overlay Extension Development Plan

## Project Overview

iRon is a lightweight overlay system for iRacing that provides real-time telemetry displays during races. This document outlines the technical architecture and planned extensions for enhanced competitor information, lap timing, and vehicle dynamics overlays.

## Current Technical Architecture

### Core Technologies
- **Language**: C++ (Visual Studio 2022)
- **Graphics API**: DirectX 11 / Direct2D for hardware-accelerated rendering
- **UI Framework**: Direct2D with DirectWrite for text rendering
- **Composition**: DirectX Composition API for transparent overlays
- **Telemetry Source**: iRacing SDK (iRSDK) for real-time data access
- **Configuration**: JSON-based configuration system with live reloading
- **Platform**: Windows-only (leverages Windows-specific APIs)

### Project Structure
```
iRon-overlay/
├── main.cpp                 # Application entry point, hotkey handling, overlay management
├── iracing.h/.cpp          # iRSDK wrapper and telemetry data structures
├── Overlay.h/.cpp          # Base class for all overlays (DirectX setup, window management)
├── Config.h/.cpp           # JSON configuration system with file watching
├── util.h                  # Utility functions and data structures
├── irsdk/                  # iRacing SDK files
│   ├── irsdk_client.h/.cpp # Core SDK functionality
│   ├── irsdk_defines.h     # SDK constants and structures
│   ├── irsdk_utils.cpp     # SDK utility functions
│   └── yaml_parser.h/.cpp  # YAML session data parser
└── Overlay*.h              # Individual overlay implementations
    ├── OverlayRelative.h   # Competitor positions + minimap
    ├── OverlayDDU.h        # Dashboard with fuel calculator
    ├── OverlayInputs.h     # Throttle/brake/steering graphs
    ├── OverlayStandings.h  # Full field standings with ratings
    ├── OverlayCover.h      # UI element masking
    └── OverlayDebug.h      # Development/debugging overlay
```

### Key Design Patterns

#### Overlay Architecture
All overlays inherit from the base `Overlay` class which provides:
- DirectX device and render target management
- Window positioning and sizing with live editing
- Configuration change handling
- Enable/disable state management
- UI edit mode for repositioning overlays

#### Data Flow
1. **iRSDK Connection**: `ir_tick()` maintains connection to iRacing and updates telemetry
2. **Data Processing**: Each overlay processes relevant telemetry data in `onUpdate()`
3. **Rendering**: Direct2D renders graphics to transparent windows
4. **Composition**: DirectX Composition blends overlays over the game

#### Configuration System
- JSON-based configuration stored in `config.json`
- Live file watching for instant config changes
- Per-overlay settings with sensible defaults
- Hotkey customization for all functions

### iRSDK Data Access

The iRacing SDK provides extensive telemetry data at 60Hz including:

#### Session Information (YAML format, updated periodically)
- Driver information (names, ratings, car numbers)
- Session details (type, length, weather)
- Track information and car specifications

#### Live Telemetry Variables (60Hz binary data)
- **Vehicle Dynamics**: Position, velocity, acceleration, orientation
- **Engine Data**: RPM, temperatures, pressures, fuel level
- **Tire Data**: Pressures, temperatures, wear across all four tires
- **Suspension**: Shock positions and velocities
- **Inputs**: Steering, throttle, brake, clutch, gear position
- **Track Position**: Lap distance, lap times, sector times
- **Competitors**: Position data for all cars in session (up to 64 cars)
- **Race State**: Flags, penalties, pit status, session timing

## Planned Extensions

### Phase 1: Enhanced Competitor Information

#### 1. Real-time Competitor Tracker (`OverlayCompetitors.h`)
**Purpose**: Enhanced situational awareness of nearby competitors
**Key Features**:
- Live competitor positions in configurable radius around player
- Speed differential indicators relative to player car
- Car class identification with color coding
- Predicted overtaking scenarios and defensive positioning alerts
- Radar-style or list view display options

**Technical Implementation**:
- Use `ir_CarIdxLapDistPct[]` for precise track positioning
- Calculate relative positions using track geometry
- Implement speed differential via `ir_Speed` vs `ir_CarIdxEstTime[]`
- Add collision prediction algorithms for safety alerts

#### 2. Battle Tracker Overlay (`OverlayBattle.h`)
**Purpose**: Focus on immediate competitive battles
**Key Features**:
- Dedicated view of closest competitors (±3 positions)
- Real-time gap timing with trend indicators
- Sector-by-sector comparison with immediate rivals
- Side-by-side detection and wheel-to-wheel alerts

**Technical Implementation**:
- Filter competitor data to positions ±3 from player
- Use `ir_CarIdxF2Time[]` for gap calculations
- Implement trend analysis over time windows
- Add proximity detection using position and speed vectors

### Phase 2: Advanced Timing & Performance

#### 3. Lap Analysis Overlay (`OverlayLapAnalysis.h`)
**Purpose**: Real-time performance analysis and optimization
**Key Features**:
- Sector-by-sector timing comparison vs personal/session best
- Live delta display with color-coded performance indicators
- Lap time prediction based on current sector performance
- Theoretical vs actual lap time analysis
- Purple/green sector indicators for personal/session records

**Technical Implementation**:
- Track sector splits using `ir_LapDist` and track sector boundaries
- Store and compare against `ir_LapBestLapTime` and session records
- Implement predictive algorithms for lap time estimation
- Use `ir_LapDeltaToBestLap` and similar delta variables

#### 4. Session Performance Overlay (`OverlaySessionPerf.h`)
**Purpose**: Long-term performance and consistency tracking
**Key Features**:
- Lap time consistency analysis (standard deviation tracking)
- Tire degradation visualization over stint length
- Fuel consumption efficiency per lap with trend analysis
- Temperature trend monitoring (tires, engine, track)
- Performance degradation alerts

**Technical Implementation**:
- Maintain rolling windows of performance data
- Use tire temperature/wear data from `ir_LFtempCL`, `ir_RFwearL`, etc.
- Track fuel consumption via `ir_FuelLevel` over time
- Implement statistical analysis for consistency metrics

### Phase 3: Enhanced Input & Vehicle Dynamics

#### 5. Advanced Inputs Overlay (Enhanced `OverlayInputs.h`)
**Purpose**: Comprehensive input analysis and technique improvement
**Additional Features**:
- Clutch and handbrake input visualization
- Force feedback strength indicators
- Input smoothness and consistency analysis
- Trail braking technique visualization
- Steering angle vs speed correlation analysis

**Technical Implementation**:
- Add `ir_Clutch`, `ir_HandbrakeRaw` to existing input graphs
- Implement smoothness algorithms using input derivatives
- Add FFB visualization using `ir_SteeringWheelTorque`
- Create correlation analysis between inputs and lap time

#### 6. Vehicle Dynamics Overlay (`OverlayDynamics.h`)
**Purpose**: Advanced vehicle dynamics monitoring for setup optimization
**Key Features**:
- Real-time G-force display (lateral and longitudinal)
- Tire slip angle indicators for all four corners
- Suspension travel and deflection visualization
- Weight transfer analysis during braking/acceleration
- Aerodynamic efficiency metrics and balance indicators

**Technical Implementation**:
- Use `ir_LatAccel`, `ir_LongAccel` for G-force calculations
- Calculate slip angles using velocity vectors and steering input
- Visualize suspension data from `ir_LFSHshockDefl`, etc.
- Implement aerodynamic calculations from speed and acceleration data

### Phase 4: Strategic Information

#### 7. Pit Strategy Overlay (`OverlayPitStrategy.h`)
**Purpose**: Optimize pit stop timing and strategy decisions
**Key Features**:
- Optimal pit window calculator based on fuel and tire wear
- Fuel requirements vs remaining laps/time
- Tire wear projection and optimal change timing
- Competitor pit status tracking and strategic implications
- Weather-dependent strategy adjustments

**Technical Implementation**:
- Extend existing fuel calculation logic from DDU overlay
- Use tire wear data to predict optimal change points
- Track competitor pit stops via `ir_CarIdxOnPitRoad[]`
- Implement strategic algorithms for different race scenarios

#### 8. Track Conditions Overlay (`OverlayConditions.h`)
**Purpose**: Environmental awareness for setup and strategy optimization
**Key Features**:
- Real-time track temperature evolution tracking
- Grip level indicators by track section
- Wind direction and speed with lap time impact analysis
- Optimal racing line suggestions based on current conditions

**Technical Implementation**:
- Monitor `ir_TrackTempCrew`, `ir_AirTemp` over time
- Use `ir_WindVel`, `ir_WindDir` for wind analysis
- Correlate environmental data with lap time performance
- Implement track section analysis using position data

### Phase 5: Race Management

#### 9. Race Director Overlay (`OverlayRaceDirector.h`)
**Purpose**: Comprehensive race situation awareness
**Key Features**:
- Detailed flag status information and implications
- Incident probability and safety car prediction
- DRS/Push-to-Pass availability and optimal usage timing
- Penalty tracking and driving standards notifications

**Technical Implementation**:
- Monitor `ir_SessionFlags` for comprehensive flag information
- Use `ir_PushToPass`, `ir_CarIdxP2P_Status[]` for P2P tracking
- Implement incident detection algorithms
- Add penalty tracking via session state changes

#### 10. Spotter Integration Overlay (`OverlaySpotter.h`)
**Purpose**: Enhanced spatial awareness and communication
**Key Features**:
- Visual spotter information for car positioning
- Radio communication status and team coordination
- Automated emergency situation detection and alerts
- Integration with team strategy communications

**Technical Implementation**:
- Use `ir_CarLeftRight` and positional data for spotter info
- Monitor `ir_RadioTransmitCarIdx` for communication tracking
- Implement automated safety alerts based on proximity and speed
- Add visual indicators for radio communication status

## Implementation Strategy

### Development Approach
1. **Incremental Development**: Implement overlays progressively to maintain stability
2. **Pattern Consistency**: Follow established `Overlay` base class patterns
3. **Performance Optimization**: Utilize 30Hz mode for complex calculations
4. **Configuration Extension**: Expand JSON config system for new features
5. **Hotkey Integration**: Add new toggles following existing hotkey patterns

### Technical Considerations

#### Performance Optimization
- Use the existing 30Hz performance mode for computationally intensive overlays
- Implement data caching to avoid redundant calculations
- Optimize DirectX rendering calls for complex visualizations
- Consider background thread processing for heavy analytics

#### Memory Management
- Follow existing RAII patterns with Microsoft::WRL::ComPtr
- Implement proper cleanup in overlay destructors
- Cache frequently accessed telemetry data to reduce SDK calls
- Use efficient data structures for historical data storage

#### Configuration Management
- Extend existing JSON schema for new overlay settings
- Maintain backward compatibility with existing configurations
- Implement validation for new configuration parameters
- Provide sensible defaults for all new features

### Development Priority
1. **OverlayLapAnalysis** - Builds on existing timing infrastructure
2. **Enhanced OverlayInputs** - Extends current input visualization
3. **OverlayCompetitors** - Provides immediate competitive value
4. **OverlayPitStrategy** - Strategic enhancement for race management
5. **Remaining overlays** - Based on user feedback and racing priorities

## Technical Dependencies

### Build Requirements
- Visual Studio 2022 (Community Edition sufficient)
- Windows 10/11 SDK
- DirectX development libraries (included with Windows SDK)

### Runtime Requirements
- Windows 10/11 (DirectX 11+ support required)
- iRacing simulation software
- DirectX runtime (typically pre-installed)

### External Libraries
- **picojson.h** - Lightweight JSON parsing (header-only)
- **iRSDK** - iRacing telemetry SDK (included in project)
- **No additional dependencies** - Maintains project's lightweight philosophy

## Future Considerations

### Potential Enhancements
- **Multi-monitor support** - Extend overlay positioning to multiple displays
- **VR integration** - Adapt overlays for virtual reality racing setups
- **Network features** - Team telemetry sharing and comparison
- **AI insights** - Machine learning for optimal line and setup suggestions
- **Mobile companion** - Smartphone app for pit crew functionality

### Scalability Considerations
- Modular overlay architecture supports unlimited additions
- Configuration system can accommodate extensive customization
- iRSDK provides comprehensive data access for any racing scenario
- DirectX rendering pipeline scales well with modern GPUs

This development plan provides a comprehensive roadmap for transforming iRon into a professional-grade racing telemetry suite while maintaining its core principles of simplicity, performance, and reliability.