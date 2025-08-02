## TODO.md

```markdown
# Implementation Plan

## Phase 1: Track Map Overlay
- [ ] Create OverlayTrackMap class structure
- [ ] Implement track point collection system
  - [ ] Store points during first lap
  - [ ] Detect lap completion for saving
  - [ ] Handle different track types (oval vs road)
- [ ] Build coordinate transformation
  - [ ] Map lap percentage to X,Y coordinates
  - [ ] Support fixed and rotating views
  - [ ] Implement zoom functionality
- [ ] Add car rendering
  - [ ] Draw track outline
  - [ ] Render cars as triangles/dots
  - [ ] Color code by class/position
  - [ ] Add car number labels (optional)
- [ ] Implement track persistence
  - [ ] Save learned tracks to file
  - [ ] Load existing track data
  - [ ] Handle track config changes
- [ ] Add configuration options
  - [ ] View mode (fixed/rotating)
  - [ ] Zoom level
  - [ ] Car sizes and colors
  - [ ] Show/hide options
- [ ] Testing and optimization
  - [ ] Test on various track types
  - [ ] Optimize rendering performance
  - [ ] Handle edge cases

## Phase 2: Enhanced Input Display
- [ ] Modify OverlayInputs layout
  - [ ] Add header section for text info
  - [ ] Reorganize existing graphs
- [ ] Add speed display
  - [ ] Read ir_Speed telemetry
  - [ ] Implement unit conversion (mph/kph)
  - [ ] Make units configurable
- [ ] Add gear indicator
  - [ ] Display current gear
  - [ ] Color code near redline
  - [ ] Handle reverse/neutral
- [ ] Add session information
  - [ ] Time remaining/elapsed toggle
  - [ ] Format time display nicely
  - [ ] Handle unlimited sessions
- [ ] Add lap counter
  - [ ] Current lap / total laps
  - [ ] Flash on new lap
  - [ ] Handle practice/qual sessions
- [ ] Add brake bias display
  - [ ] Show current bias percentage
  - [ ] Update in real-time
- [ ] Configuration updates
  - [ ] Font sizes for new elements
  - [ ] Show/hide each element
  - [ ] Color customization

## Phase 3: Stint/Pitstop Information
- [ ] Create OverlayStintInfo class
- [ ] Implement stint tracking
  - [ ] Detect pit entry/exit
  - [ ] Track stint start time/lap
  - [ ] Calculate stint duration
- [ ] Add fuel tracking
  - [ ] Monitor fuel level changes
  - [ ] Calculate consumption rate
  - [ ] Moving average for accuracy
- [ ] Estimate remaining laps
  - [ ] Based on fuel consumption
  - [ ] Account for fuel saving
  - [ ] Show confidence level
- [ ] Track pit stops
  - [ ] Record pit stop lap
  - [ ] Measure pit stop duration
  - [ ] Store historical data
- [ ] Add tire age tracking
  - [ ] Detect tire changes
  - [ ] Count laps on tires
  - [ ] Show compound if available
- [ ] Create clean UI layout
  - [ ] Organized information display
  - [ ] Clear section headers
  - [ ] Appropriate font sizes

## Phase 4: Tire Info in Standings
- [ ] Analyze OverlayStandings structure
  - [ ] Understand current column system
  - [ ] Plan integration approach
- [ ] Add tire compound column
  - [ ] Read ir_CarIdxTireCompound
  - [ ] Map to display characters
  - [ ] Add to column layout
- [ ] Add tire age column
  - [ ] Track tire changes per car
  - [ ] Calculate laps since change
  - [ ] Handle missing data gracefully
- [ ] Implement visual indicators
  - [ ] Fresh tire highlighting
  - [ ] Color coding by compound
  - [ ] Age-based transparency
- [ ] Update configuration
  - [ ] Toggle tire info display
  - [ ] Customize colors
  - [ ] Column width adjustments

## Phase 5: Delta Bar Overlay
- [ ] Create OverlayDeltaBar class
- [ ] Implement delta data collection
  - [ ] Read delta telemetry values
  - [ ] Handle invalid data states
  - [ ] Support multiple delta types
- [ ] Build comparison system
  - [ ] vs Best lap
  - [ ] vs Session best
  - [ ] vs Optimal lap
  - [ ] vs Last lap
  - [ ] Mode switching
- [ ] Design visual bar
  - [ ] Calculate bar position
  - [ ] Implement color gradient
  - [ ] Smooth transitions
- [ ] Add numerical display
  - [ ] Format delta time
  - [ ] Position text clearly
  - [ ] Optional hide/show
- [ ] Create configuration
  - [ ] Bar dimensions
  - [ ] Color settings
  - [ ] Delta range limits
  - [ ] Display options

## Phase 6: Integration and Testing
- [ ] Add all overlays to main.cpp
- [ ] Configure hotkeys for new overlays
- [ ] Test overlay interactions
  - [ ] Performance with all active
  - [ ] Window management
  - [ ] Config save/load
- [ ] Performance optimization
  - [ ] Profile rendering times
  - [ ] Optimize heavy operations
  - [ ] Implement frame skipping if needed
- [ ] Edge case testing
  - [ ] Session transitions
  - [ ] Disconnections
  - [ ] Missing telemetry data
- [ ] User experience polish
  - [ ] Consistent styling
  - [ ] Smooth animations
  - [ ] Clear visual hierarchy

## Phase 7: Documentation and Release
- [ ] Update README with new features
- [ ] Document configuration options
- [ ] Create example config entries
- [ ] Add screenshots of new overlays
- [ ] Write troubleshooting guide
- [ ] Test on different systems
- [ ] Package for release

## Future Enhancements (Post-Release)
- [ ] Weather information overlay
- [ ] Relative lap times in standings
- [ ] Sector time comparisons
- [ ] Multi-class enhancements
- [ ] Replay mode support
- [ ] Track map auto-learning improvements
- [ ] Performance graphs (temps, pressures)
- [ ] Customizable dashboards
- [ ] Touch/tablet support
- [ ] Export session data

## Known Issues to Address
- [ ] Text rendering performance at high resolutions
- [ ] Memory usage with long sessions
- [ ] Config file corruption handling
- [ ] Multiple monitor support
- [ ] DPI scaling issues

## Testing Checklist
- [ ] All track types (oval, road, dirt)
- [ ] Different session types (practice, qual, race)
- [ ] Various car counts (2-60 cars)
- [ ] Long endurance sessions
- [ ] Quick session switches
- [ ] Config file modifications while running
- [ ] Performance mode (30Hz)
- [ ] Minimize/restore behavior
- [ ] Hotkey conflicts