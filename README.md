# Advanced Bathroom Occupancy with Voice Query

## Overview
This Home Assistant blueprint creates an intelligent bathroom lighting system that combines motion detection, door sensors, and voice confirmation to accurately determine room occupancy. It prevents lights from being turned off while someone is still in the bathroom, even when motion isn't detected.

## What It Does
The automation intelligently manages bathroom lighting by:
- **Automatically turning lights ON** when motion is detected or the door opens
- **Using voice confirmation** before turning lights OFF to prevent leaving someone in the dark
- **Tracking occupancy state** to avoid unnecessary voice queries
- **Handling edge cases** like people sitting still or using phones (no motion detected)

## How It Works

### Phase 1: Occupancy Detection (Lights ON)
The system turns lights ON when either:
- **Motion is detected** in the bathroom
- **Door is opened** (someone entering)

When this happens:
1. Sets the occupancy helper to "ON"
2. Turns the bathroom light ON

### Phase 2: Exit Detection & Voice Confirmation (Lights OFF)
The system initiates the exit sequence when:
- **Door closes** (potential exit)
- **Motion clears** after the configured delay (alternative trigger)

When this happens, the system:
1. **Waits 5 seconds** for any immediate motion
2. **Checks for ongoing motion:**
   - If motion is still active, waits for it to clear (up to 5 minutes)
   - If no motion, proceeds immediately
3. **Asks via TTS**: "Is someone still in the bathroom? Please say yes or no."
4. **Listens for response** for the configured timeout period
5. **Takes action based on response:**
   - **"Yes" response**: Keeps lights on, confirms with "Okay, keeping the light on"
   - **"No" response**: Turns off lights, confirms with "Turning off the bathroom light"
   - **No response**: Assumes empty, turns off lights silently

### Voice Commands Recognized
The system understands various natural responses:

**Affirmative (Keep lights on):**
- "yes", "yeah", "yep", "still here", "occupied"

**Negative (Turn off lights):**
- "no", "nope", "empty", "not occupied", "nobody here"

## Required Setup

### 1. Hardware Requirements
- **Motion sensor** in the bathroom (binary_sensor with motion device_class)
- **Door sensor** on the bathroom door (binary_sensor with door device_class)
- **Media player/Speaker** for voice announcements
- **Light or switch** to control

### 2. Home Assistant Requirements
- **Voice Assistant** configured (Home Assistant's built-in conversation or external)
- **TTS (Text-to-Speech)** service configured
- **Input Boolean Helper** created for occupancy tracking

### 3. Creating the Input Boolean Helper
1. Go to **Settings** → **Devices & Services** → **Helpers**
2. Click **"+ Create Helper"**
3. Select **"Toggle"**
4. Name it something like "Bathroom Occupied"
5. Save it

## Configuration Options

| Setting | Description | Default | Notes |
|---------|-------------|---------|-------|
| **Motion Sensor** | Bathroom motion detector | Required | Must be binary_sensor with motion device_class |
| **Door Sensor** | Bathroom door sensor | Required | Must be binary_sensor with door device_class |
| **Light** | Light/switch to control | Required | Can be light or switch entity |
| **Occupancy Helper** | Input boolean for state tracking | Required | Dedicated helper, not used elsewhere |
| **Media Player** | Speaker for announcements | Required | Any media_player entity |
| **No Response Timeout** | Voice response wait time | 2 minutes | 1-30 minutes |
| **Motion Clear Delay** | Wait time after motion stops | 30 seconds | 5-300 seconds |
| **Voice Assistant** | Conversation entity | home_assistant | Usually default is fine |

## Behavior Examples

### Scenario 1: Normal Exit
1. Person enters → Motion detected → Lights ON
2. Person leaves → Door closes → 5-second delay
3. No motion detected → Voice query: "Is someone still in the bathroom?"
4. No response after 2 minutes → Lights OFF

### Scenario 2: Someone Still Inside
1. Person enters → Motion detected → Lights ON
2. Door closes (person still inside, sitting still)
3. No motion for 30 seconds → Voice query triggered
4. Person responds "yes" → Lights stay ON
5. System responds "Okay, keeping the light on"

### Scenario 3: Motion While Using
1. Person in bathroom, using phone → Motion stops for 30+ seconds
2. Door still closed → Voice query: "Is someone still in the bathroom?"
3. Person says "still here" → Lights stay ON

### Scenario 4: Quick Re-entry
1. Person exits → Door closes → Voice query initiated
2. Person re-enters → Motion detected → Cancels voice query, keeps lights ON

## Technical Details

### Automation Mode
- **Mode**: Single
- **Max Exceeded**: Silent
- This prevents multiple instances from running simultaneously

### State Management
The automation uses the input_boolean helper to track occupancy state:
- **ON**: Room is occupied, lights should be on
- **OFF**: Room is empty, lights can be off

This prevents unnecessary voice queries when the room is already known to be empty.

### Trigger Priority
The automation responds to triggers in this priority:
1. **Motion/Door opening** → Immediate lights ON
2. **Door closing** → Potential exit sequence (if occupied)
3. **Motion clearing** → Alternative exit sequence (if door closed and occupied)

### Error Handling
- **Voice recognition timeouts** are handled gracefully
- **Multiple motion events** don't cause conflicts
- **Door sensor failures** won't leave lights stuck on
- **Media player unavailable** won't prevent basic motion lighting

## Troubleshooting

### Voice Commands Not Working
- Verify voice assistant is properly configured
- Check media player volume and connectivity
- Test TTS service independently
- Ensure conversation integration is enabled

### Lights Not Turning On
- Check motion sensor state in Developer Tools
- Verify door sensor is working
- Confirm light entity is correct and controllable

### Voice Query Too Frequent
- Increase Motion Clear Delay setting
- Check for motion sensor false positives
- Verify door sensor isn't bouncing between states

### Lights Turn Off Despite Occupancy
- Check voice command recognition in automation traces
- Verify No Response Timeout is adequate
- Test media player audibility in bathroom

## Advanced Usage

### Multiple Bathrooms
Create separate instances of this blueprint for each bathroom, each with its own:
- Sensors
- Light control
- Input boolean helper
- Media player (can be shared)

### Integration with Other Automations
The occupancy helper can be used by other automations:
- Ventilation fan control
- Humidity management
- Security systems

### Customization
You can modify the voice messages by editing the blueprint's TTS service calls to match your preferred language or phrasing.

