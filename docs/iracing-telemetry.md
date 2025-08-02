## IRACING_TELEMETRY.md

```markdown
# iRacing Telemetry Reference

## Session Information
- `ir_SessionTime` - Seconds since session start (double)
- `ir_SessionTimeRemain` - Seconds until session ends (double)
- `ir_SessionTimeTotal` - Total session length in seconds (double)
- `ir_SessionTimeOfDay` - Time of day in seconds (float)
- `ir_SessionNum` - Current session number (int)
- `ir_SessionState` - Session state (irsdk_SessionState enum)
- `ir_SessionFlags` - Session flags bitfield (yellow, green, checkered, etc.)
- `ir_SessionLapsRemain` - Laps remaining in session (int)
- `ir_SessionLapsRemainEx` - Improved laps remaining (int)

## Driver Inputs
- `ir_Throttle` - Throttle position 0-1 (float)
- `ir_Brake` - Brake position 0-1 (float)
- `ir_Clutch` - Clutch position 0-1 (float)
- `ir_SteeringWheelAngle` - Steering angle in radians (float)
- `ir_SteeringWheelAngleMax` - Maximum steering angle (float)
- `ir_Gear` - Current gear: -1=reverse, 0=neutral, 1+=gear (int)
- `ir_RPM` - Engine RPM (float)

## Vehicle State
- `ir_Speed` - Speed in m/s (float)
- `ir_VelocityX/Y/Z` - Velocity components in m/s (float)
- `ir_Yaw` - Yaw orientation in radians (float)
- `ir_Pitch` - Pitch orientation in radians (float)
- `ir_Roll` - Roll orientation in radians (float)
- `ir_VertAccel` - Vertical acceleration in m/s² (float)
- `ir_LatAccel` - Lateral acceleration in m/s² (float)
- `ir_LongAccel` - Longitudinal acceleration in m/s² (float)

## Lap Information
- `ir_Lap` - Current lap number (int)
- `ir_LapCompleted` - Laps completed (int)
- `ir_LapDist` - Distance traveled this lap in meters (float)
- `ir_LapDistPct` - Percentage of lap completed 0-1 (float)
- `ir_LapBestLapTime` - Personal best lap time (float)
- `ir_LapLastLapTime` - Last completed lap time (float)
- `ir_LapCurrentLapTime` - Current lap time estimate (float)
- `ir_LapDeltaToBestLap` - Delta to personal best (float)
- `ir_LapDeltaToBestLap_OK` - Delta data is valid (bool)
- `ir_LapDeltaToSessionBestLap` - Delta to session best (float)

## Fuel & Pit Information
- `ir_FuelLevel` - Fuel remaining in liters (float)
- `ir_FuelLevelPct` - Fuel remaining percentage (float)
- `ir_FuelUsePerHour` - Fuel consumption rate kg/h (float)
- `ir_OnPitRoad` - Car is on pit road (bool)
- `ir_PitSvFlags` - Pit service flags (bitfield)
- `ir_PitSvFuel` - Fuel to add in liters (float)
- `ir_dcBrakeBias` - Brake bias adjustment (float)

## Tire Information
- `ir_LFtempCL/CM/CR` - Left Front temps: Left/Middle/Right (float, °C)
- `ir_RFtempCL/CM/CR` - Right Front temps (float, °C)
- `ir_LRtempCL/CM/CR` - Left Rear temps (float, °C)
- `ir_RRtempCL/CM/CR` - Right Rear temps (float, °C)
- `ir_LFwearL/M/R` - Left Front wear percentage (float)
- `ir_RFwearL/M/R` - Right Front wear percentage (float)
- `ir_LRwearL/M/R` - Left Rear wear percentage (float)
- `ir_RRwearL/M/R` - Right Rear wear percentage (float)
- `ir_PlayerTireCompound` - Current tire compound (int)

## Per-Car Arrays (indexed by CarIdx, max 64 cars)
- `ir_CarIdxLap[idx]` - Lap count for car (int)
- `ir_CarIdxLapCompleted[idx]` - Completed laps (int)
- `ir_CarIdxLapDistPct[idx]` - Position around lap 0-1 (float)
- `ir_CarIdxPosition[idx]` - Overall position (int)
- `ir_CarIdxClassPosition[idx]` - Class position (int)
- `ir_CarIdxOnPitRoad[idx]` - Car on pit road (bool)
- `ir_CarIdxF2Time[idx]` - Time behind leader (float)
- `ir_CarIdxEstTime[idx]` - Estimated lap time (float)
- `ir_CarIdxLastLapTime[idx]` - Last lap time (float)
- `ir_CarIdxBestLapTime[idx]` - Best lap time (float)
- `ir_CarIdxTireCompound[idx]` - Tire compound (int)
- `ir_CarIdxSteer[idx]` - Steering input (float)
- `ir_CarIdxRPM[idx]` - Engine RPM (float)
- `ir_CarIdxGear[idx]` - Current gear (int)

## Useful Calculations

### Speed Conversions
```cpp
float mph = ir_Speed.getFloat() * 2.237f;
float kph = ir_Speed.getFloat() * 3.6f;
```
### Fuel Calculations
```cpp // Fuel used per lap (liters)
float fuelPerLap = (ir_FuelUsePerHour.getFloat() / 3600.0f) * lastLapTime;

// Laps remaining on fuel
float lapsOnFuel = ir_FuelLevel.getFloat() / fuelPerLap;
```
## Time Formatting
```cpp 
// Use provided formatLaptime() function
std::string lapStr = formatLaptime(seconds);
```
## Brake Bias
```cpp
// Brake bias is typically 45-65%
float brakeBias = ir_dcBrakeBias.getFloat() * 100.0f;
```
## Track Position
```cpp 
// Convert lap percentage to approximate track position
float trackLength = ir_session.trackLength; // if available
float distanceOnTrack = ir_CarIdxLapDistPct.getFloat(carIdx) * trackLength;
```
## Session States
```cpp
enum irsdk_SessionState {
    irsdk_StateInvalid,
    irsdk_StateGetInCar,
    irsdk_StateWarmup,
    irsdk_StateParadeLaps,
    irsdk_StateRacing,
    irsdk_StateCheckered,
    irsdk_StateCoolDown
};
```
### Important Notes

- All temperatures are in Celsius
- All distances are in meters
- All speeds are in m/s unless noted
- Fuel is in liters for volume, kg/h for consumption rate
- Time values are in seconds
- Angular values are in radians