# Advanced Bathroom Occupancy with Voice Query & Manual Override Protection

## Overview
This Home Assistant blueprint creates an intelligent bathroom lighting system that combines motion detection, door sensors, voice confirmation, and manual override protection to accurately determine room occupancy. It prevents lights from being turned off while someone is still in the bathroom, even when motion isn't detected, and gracefully handles manual light switch operations.

## What It Does
The automation intelligently manages bathroom lighting by:
- **Automatically turning lights ON** when motion is detected or the door opens
- **Using voice confirmation** before turning lights OFF to prevent leaving someone in the dark
- **Tracking occupancy state** to avoid unnecessary voice queries
- **Handling manual light operations** with intelligent safeguards and confirmations
- **Preventing automation conflicts** when users manually control the lights
- **Maintaining synchronization** between manual and automatic control

## How It Works

### Phase 1: Occupancy Detection (Lights ON)
The system turns lights ON when either:
- **Motion is detected** in the bathroom
- **Door is opened** (someone entering)
- **Light is manually turned ON** (safeguard assumes occupancy)

When this happens:
1. Sets the occupancy helper to "ON"
2. Turns the bathroom light ON (if not already on)

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

### Phase 3: Manual Override Protection (NEW!)
The system monitors manual light operations and responds intelligently:

#### Manual Light Turn-ON Safeguards:
When someone manually turns the light ON:
1. **Waits for settling period** (configurable delay)
2. **Checks occupancy state**: If automation thinks room is empty
3. **Updates occupancy tracking**: Assumes someone is present
4. **Synchronizes system state**: Prevents future conflicts

#### Manual Light Turn-OFF Safeguards:
When someone manually turns the light OFF:
1. **Waits for settling period** (configurable delay)
2. **Evaluates current conditions**: Checks motion and door sensors
3. **If sensors indicate occupancy**:
   - Asks: "Light was turned off manually. Should I turn it back on? Say yes or no."
   - **"Yes"**: Restores light + confirms action
   - **"No"**: Respects choice + updates occupancy state
   - **No response**: Assumes intentional, updates state accordingly
4. **If no sensor activity**: Assumes intentional turn-off, updates occupancy state

### Voice Commands Recognized
The system understands various natural responses:

**Occupancy Confirmation:**
- "yes", "yeah", "yep", "still here", "occupied"

**Empty Confirmation:**
- "no", "nope", "empty", "not occupied", "nobody here"

**Manual Override Responses:**
- **Restore light**: "yes", "yeah", "yep", "turn it on", "turn on"
- **Keep off**: "no", "nope", "keep it off", "leave it off"

## Required Setup

### 1. Hardware Requirements
- **Motion sensor** in the bathroom (binary_sensor with motion device_class)
- **Door sensor** on the bathroom door (binary_sensor with door device_class)
- **Media player/Speaker** for voice announcements
- **Light or switch** to control (with manual switch capability)

### 2. Home Assistant Requirements
- **Voice Assistant** configured (Home Assistant's built-in conversation or external)
- **TTS (Text-to-Speech)** service configured
- **Input Boolean Helper** created for occupancy tracking
- **Conversation integration** enabled for voice recognition

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
| **Manual Override Delay** | Wait after manual operation | 10 seconds | 5-60 seconds |
| **Enable Manual Safeguards** | Toggle manual protection | Enabled | Recommended to keep enabled |
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

### Scenario 3: Manual Override - Turn Off While Occupied
1. Person in bathroom → Light manually turned OFF
2. Motion sensor still active → System detects conflict
3. Voice query: "Light was turned off manually. Should I turn it back on?"
4. Person says "no" → System respects choice, updates occupancy state
5. System confirms: "Okay, keeping the light off"

### Scenario 4: Manual Override - Turn On When Empty
1. Room appears empty → Person manually turns light ON
2. System detects manual operation → Assumes occupancy
3. Updates internal state → No voice query needed
4. Future automation behavior adjusted accordingly

### Scenario 5: Manual Turn-Off with Confirmation
1. Person using bathroom → Accidentally turns light OFF
2. Motion still detected → System asks about restoring light
3. Person says "yes" → Light restored immediately
4. System confirms: "Turning the light back on"

### Scenario 6: Intentional Manual Turn-Off
1. Person in bathroom → Manually turns light OFF
2. No motion detected → System assumes intentional
3. Updates occupancy state silently → No voice query

## Technical Details

### Automation Mode
- **Mode**: Single
- **Max Exceeded**: Silent
- This prevents multiple instances from running simultaneously and conflicts

### State Management
The automation uses the input_boolean helper to track occupancy state:
- **ON**: Room is occupied, lights should be on
- **OFF**: Room is empty, lights can be off

Manual operations update this state to maintain synchronization between manual and automatic control.

### Trigger Priority & Handling
The automation responds to triggers in this priority:
1. **Motion/Door opening** → Immediate lights ON + occupancy tracking
2. **Manual light operations** → Safeguard evaluation + state synchronization
3. **Door closing** → Potential exit sequence (if occupied)
4. **Motion clearing** → Alternative exit sequence (if door closed and occupied)

### Manual Override Logic
- **Turn-ON safeguards**: Assume occupancy, update tracking
- **Turn-OFF safeguards**: Evaluate sensors, ask for confirmation if needed
- **Configurable delays**: Prevent race conditions between manual and automatic actions
- **Voice confirmation**: Respect user intent while maintaining safety

### Error Handling
- **Voice recognition timeouts** are handled gracefully
- **Multiple manual operations** don't cause conflicts due to single mode
- **Sensor failures** won't leave lights stuck in wrong state
- **Media player unavailable** won't prevent basic lighting functions
- **Manual override conflicts** are resolved through intelligent questioning

## Troubleshooting

### Voice Commands Not Working
- Verify voice assistant is properly configured
- Check media player volume and connectivity
- Test TTS service independently
- Ensure conversation integration is enabled
- Check automation traces for voice recognition events

### Manual Overrides Not Detected
- Verify light entity is correct in configuration
- Check that manual switch actually changes the entity state
- Increase Manual Override Delay if switches are slow
- Disable/re-enable Manual Safeguards setting

### Lights Not Turning On
- Check motion sensor state in Developer Tools
- Verify door sensor is working
- Confirm light entity is correct and controllable
- Check if Manual Safeguards are interfering

### Voice Query Too Frequent
- Increase Motion Clear Delay setting
- Check for motion sensor false positives
- Verify door sensor isn't bouncing between states
- Check manual override settings aren't conflicting

### Automation Conflicts with Manual Use
- Ensure Manual Override Delay is appropriate for your switches
- Check that Enable Manual Safeguards is turned ON
- Verify voice responses are being recognized correctly
- Review automation traces during manual operations

### Lights Turn Off Despite Occupancy
- Check voice command recognition in automation traces
- Verify No Response Timeout is adequate for your household
- Test media player audibility in bathroom
- Ensure manual overrides aren't interfering

## Advanced Usage

### Multiple Bathrooms
Create separate instances of this blueprint for each bathroom, each with its own:
- Sensors (motion, door)
- Light control
- Input boolean helper
- Media player (can be shared if within range)

### Integration with Other Automations
The occupancy helper can be used by other automations:
- Ventilation fan control
- Humidity management
- Security systems
- Energy monitoring

**Important**: Don't let other automations modify the occupancy helper directly, as this will interfere with the manual override protection.

### Customization Options
You can modify the blueprint for:
- **Different voice messages**: Edit TTS service calls
- **Additional voice commands**: Expand conversation command lists
- **Different timeout periods**: Adjust for household needs
- **Multiple light entities**: Modify to control light groups

### Smart Switch Integration
Works with:
- **Physical wall switches**: Manual operations detected via entity state changes
- **Smart switches**: Paddle/button press detection
- **Voice assistants**: "Turn off bathroom light" commands
- **Mobile apps**: Manual control through HA interface

The manual override protection makes this automation suitable for real-world use where family members will inevitably use manual light switches, ensuring the system adapts intelligently to human behavior while maintaining its core safety features.
