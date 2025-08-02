## FEATURE_SPECS.md

```markdown
# Feature Specifications

## Track Map Overlay

### Requirements
- Display 2D top-down view of track layout
- Show all cars as colored dots/triangles
- Highlight player's car distinctly
- Support fixed and rotating views
- Show car numbers on hover/option
- Configurable size and zoom level
- Mini-map and full-map modes

### Technical Approach
1. **Track Learning Mode**
   - Collect car positions during first lap
   - Store points when ir_CarIdxLapDistPct changes
   - Build spline or connected segments
   - Save learned tracks to file for reuse

2. **Coordinate Transformation**
   - Map ir_CarIdxLapDistPct (0-1) to track coordinates
   - For road courses: use collected X,Y positions
   - For ovals: consider elliptical approximation
   - Support rotation around player car

3. **Rendering**
   - Draw track outline with ID2D1PathGeometry
   - Render cars as triangles pointing in direction
   - Use different colors for: player, class, lapped/lapping
   - Optional: sector lines, pit entry/exit

### Config Options
```json
{
  "OverlayTrackMap": {
    "enabled": true,
    "mode": "fixed", // "fixed" or "rotating"
    "zoom": 1.0,
    "show_car_numbers": true,
    "car_size": 10,
    "track_color": [0.3, 0.3, 0.3, 1.0],
    "player_color": [1.0, 1.0, 0.0, 1.0],
    "same_class_color": [1.0, 1.0, 1.0, 1.0],
    "other_class_color": [0.5, 0.5, 0.5, 1.0]
  }
}
```
### Enhanced Input Display
## Requirements

Current speed (with units)
Gear indicator
Session time (remaining/elapsed)
Brake bias display
Brake pressure indicator
Lap counter (current/total)
Optional: Water/Oil temps

## Layout Design
┌─────────────────────────┐
│ SPEED: 156 mph   GEAR: 4│
│ TIME: 23:45 / 45:00     │
│ LAP: 34/120  BIAS: 54.5%│
├─────────────────────────┤
│ [Throttle Graph]        │
│ [Brake Graph]           │
│ [Steering Graph]        │
└─────────────────────────┘
## Implementation Notes

Place text info above existing graphs
Use consistent font sizes
Color-code gear (red near redline)
Flash lap counter on new lap
Make units configurable (mph/kph)

### Stint/Pitstop Information Overlay
## Requirements

Current stint duration (time & laps)
Fuel level and consumption rate
Estimated laps until empty
Last pit stop (lap & duration)
Average lap time this stint
Tire age indicator

## Display Format
STINT INFO
Time: 23:45  Laps: 18
Fuel: 45.3L  Rate: 2.8L/lap
Laps Remaining: ~16

Last Stop: L23 (13.4s)
Avg Lap: 1:34.567
Tire Age: 18 laps
Calculations
```cpp
// Stint time
float stintTime = ir_SessionTime.getFloat() - lastPitExitTime;

// Fuel per lap (moving average)
float fuelPerLap = fuelUsedThisStint / lapsThisStint;

// Laps remaining
float lapsRemaining = ir_FuelLevel.getFloat() / fuelPerLap;
```
### Tire Information in Standings
## Requirements

Show tire compound for each car
Display tire age (laps since change)
Highlight fresh tires (<3 laps old)
Different indicators for different compounds

## Display Integration
Add columns to existing standings:

Compound indicator (S/M/H/Wet or color)
Age in laps
Fresh tire highlight

## Visual Design

Use single letter: S(oft), M(edium), H(ard), W(et)
Color code by compound
Bold/highlight for fresh tires
Gray out for very old tires (>30 laps)

### Delta Bar Overlay
## Requirements

Visual bar showing time delta
Support multiple comparison modes
Color gradient (green=faster, red=slower)
Numerical delta display
Smooth transitions

## Comparison Modes

vs Personal Best Lap
vs Session Best Lap
vs Optimal Lap
vs Last Lap
vs Specific Driver

## Visual Design
┌─────────────────────────────┐
│ ████████████|████           │
│        -0.234s              │
└─────────────────────────────┘
## Implementation
```cpp
// Bar width calculation
float maxDelta = 2.0f; // +/- 2 seconds
float delta = ir_LapDeltaToBestLap.getFloat();
float barPosition = 0.5f + (delta / maxDelta) * 0.5f;
barPosition = std::clamp(barPosition, 0.0f, 1.0f);

// Color interpolation
float4 color;
if (delta < 0) {
    // Faster - green
    color = float4(0, 1, 0, 1);
} else if (delta < 0.5f) {
    // Slightly slower - yellow
    color = float4(1, 1, 0, 1);
} else {
    // Much slower - red
    color = float4(1, 0, 0, 1);
}
```
### Config Options
```json
{
  "OverlayDeltaBar": {
    "enabled": true,
    "comparison_mode": "best_lap",
    "bar_height": 30,
    "show_numerical": true,
    "max_delta": 2.0,
    "faster_color": [0.0, 1.0, 0.0, 1.0],
    "slower_color": [1.0, 0.0, 0.0, 1.0]
  }
}
```
### General Implementation Guidelines
## Error Handling

Always check ir_session.driverCarIdx >= 0
Verify telemetry variables are valid
Handle session transitions gracefully
Provide fallback values for missing data

## Performance Considerations

Cache calculations between frames
Use dirty flags for expensive operations
Limit text recreations
Pool geometry objects

## User Experience

All colors configurable
Consistent hotkey patterns
Save overlay positions
Smooth animations/transitions
Clear visual hierarchy
