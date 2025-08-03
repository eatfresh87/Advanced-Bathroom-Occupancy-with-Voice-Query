# Advanced Bathroom Occupancy with Dual Voice Confirmation & Manual Override Protection

## Overview
This Home Assistant blueprint creates an intelligent bathroom lighting system that combines motion detection, door sensors, dual voice confirmation, and manual override protection to accurately determine room occupancy. It prevents lights from being turned off while someone is still in the bathroom through a thoughtful two-question system that accounts for real-world situations like brushing teeth, washing face, or having water in eyes/mouth.

## What It Does
The automation intelligently manages bathroom lighting by:
- **Automatically turning lights ON** when motion is detected or the door opens
- **Using dual voice confirmation** with two separate questions before turning lights OFF
- **Providing advance warning** about the second question to reduce user pressure
- **Tracking occupancy state** to avoid unnecessary voice queries
- **Handling manual light operations** with intelligent safeguards and confirmations
- **Preventing automation conflicts** when users manually control the lights
- **Maintaining synchronization** between manual and automatic control
- **Accommodating real-world scenarios** where users can't immediately respond

## How It Works

### Phase 1: Occupancy Detection (Lights ON)
The system turns lights ON when either:
- **Motion is detected** in the bathroom
- **Door is opened** (someone entering)
- **Light is manually turned ON** (safeguard assumes occupancy)

When this happens:
1. Sets the occupancy helper to "ON"
2. Turns the bathroom light ON (if not already on)

### Phase 2: Exit Detection & Dual Voice Confirmation (Lights OFF)
The system initiates the exit sequence when:
- **Door closes** (potential exit)
- **Motion clears** after the configured delay (alternative trigger)

When this happens, the system follows a careful two-question process:

#### **First Question (With Warning):**
1. **Waits 5 seconds** for any immediate motion
2. **Checks for ongoing motion** (waits up to 5 minutes if needed)
3. **Asks**: "Is someone still in the bathroom? Please say yes or no. If you cannot answer right now, I will ask again in one minute before turning off the light."
4. **Waits for response** (default 60 seconds, configurable 30-180 seconds)
5. **Actions based on first response:**
   - **"Yes"**: Keeps lights on, confirms "Okay, keeping the light on"
   - **"No"**: Turns off lights, confirms "Turning off the bathroom light"
   - **No response**: Proceeds to second question

#### **Second Question (Final Warning):**
1. **Asks**: "Last chance - is anyone still in the bathroom? Please say yes or no, or I will turn off the light."
2. **Waits for response** (default 90 seconds, configurable 30-180 seconds)
3. **Actions based on second response:**
   - **"Yes"**: Keeps lights on, confirms "Okay, keeping the light on"
   - **"No"**: Turns off lights, confirms "Turning off the bathroom light"
   - **No response**: Assumes empty, turns off lights silently

### Phase 3: Manual Override Protection
The system monitors manual light operations and responds intelligently:

#### Manual Light Turn-ON Safeguards:
When someone manually turns the light ON:
1. **Waits for settling period** (configurable delay, default 10 seconds)
2. **Checks occupancy state**: If automation thinks room is empty
3. **Updates occupancy tracking**: Assumes someone is present
4. **Synchronizes system state**: Prevents future conflicts

#### Manual Light Turn-OFF Safeguards:
When someone manually turns the light OFF:
1. **Waits for settling period** (configurable delay, default 10 seconds)
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
- **Media player/Speaker** for voice announcements (audible from bathroom)
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

| Setting | Description | Default | Range | Notes |
|---------|-------------|---------|-------|-------|
| **Motion Sensor** | Bathroom motion detector | Required | N/A | Must be binary_sensor with motion device_class |
| **Door Sensor** | Bathroom door sensor | Required | N/A | Must be binary_sensor with door device_class |
| **Light** | Light/switch to control | Required | N/A | Can be light or switch entity |
| **Occupancy Helper** | Input boolean for state tracking | Required | N/A | Dedicated helper, not used elsewhere |
| **Media Player** | Speaker for announcements | Required | N/A | Any media_player entity |
| **Motion Clear Delay** | Wait time after motion stops | 30 seconds | 5-300 seconds | Time before considering motion "cleared" |
| **First Question Timeout** | Wait time for first response | 60 seconds | 30-180 seconds | Time to answer first occupancy question |
| **Second Question Timeout** | Wait time for second response | 90 seconds | 30-180 seconds | Time to answer final occupancy question |
| **Manual Override Delay** | Wait after manual operation | 10 seconds | 5-60 seconds | Settling time after manual switch use |
| **Enable Manual Safeguards** | Toggle manual protection | Enabled | On/Off | Recommended to keep enabled |
| **Voice Assistant** | Conversation entity | home_assistant | N/A | Usually default is fine |

## Behavior Examples

### Scenario 1: Normal Exit
1. Person enters → Motion detected → Lights ON
2. Person leaves → Door closes → 5-second delay
3. No motion detected → **First question**: "Is someone still in the bathroom? Please say yes or no. If you cannot answer right now, I will ask again in one minute before turning off the light."
4. No response after 60 seconds → **Second question**: "Last chance - is anyone still in the bathroom? Please say yes or no, or I will turn off the light."
5. No response after 90 seconds → Lights OFF silently

### Scenario 2: Brushing Teeth - Can't Answer Immediately
1. Person in bathroom brushing teeth → Door closes, no motion for 30+ seconds
2. **First question** asked → Person can't respond (mouth full of toothpaste)
3. Wait 60 seconds → **Second question**: "Last chance - is anyone still in the bathroom?"
4. Person finishes brushing → Says "yes" → Lights stay ON
5. System confirms: "Okay, keeping the light on"

### Scenario 3: Washing Face - Water in Eyes
1. Person washing face → Motion stops, door closed
2. **First question** asked → Person can't hear clearly (water running, eyes closed)
3. No response → **Second question** with final warning
4. Person hears second question → Says "still here" → Lights stay ON

### Scenario 4: Quick Response to First Question
1. Door closes, motion stops → **First question** asked
2. Person immediately responds "yes" → Lights stay ON
3. System confirms → **No second question needed**

### Scenario 5: Manual Override - Turn Off While Occupied
1. Person in bathroom → Light manually turned OFF
2. Motion sensor still active → System detects conflict
3. Voice query: "Light was turned off manually. Should I turn it back on?"
4. Person says "no" → System respects choice, updates occupancy state
5. System confirms: "Okay, keeping the light off"

### Scenario 6: Manual Override - Accidental Turn Off
1. Person using bathroom → Accidentally hits light switch OFF
2. Motion still detected → System asks about restoring light
3. Person says "yes" → Light restored immediately
4. System confirms: "Turning the light back on"

### Scenario 7: Using Phone on Toilet - No Motion
1. Person sitting still, using phone → Motion clears after 30 seconds
2. Door still closed → **First question** asked
3. Person distracted by phone → No response to first question
4. **Second question** asked → Person notices, says "occupied"
5. Lights stay ON

## Real-World Scenarios the Dual System Handles

### **Brushing Teeth/Oral Care:**
- First question during active brushing (can't respond)
- Second question after finishing (can respond)

### **Face Washing/Skincare:**
- Water running, eyes closed during first question
- Clearer hearing during second question

### **Shower/Bath Preparation:**
- Undressing, water adjustment during first question
- Ready to respond during second question

### **Using Toilet with Distractions:**
- Phone use, reading during first question
- Attention available for second question

### **Medical/Accessibility Needs:**
- Extra time for those who need longer to respond
- Reduced pressure with advance warning system

## Technical Details

### Automation Mode
- **Mode**: Single
- **Max Exceeded**: Silent
- This prevents multiple instances and ensures clean dual-question sequences

### State Management
The automation uses the input_boolean helper to track occupancy state:
- **ON**: Room is occupied, lights should be on
- **OFF**: Room is empty, lights can be off

Manual operations update this state to maintain synchronization between manual and automatic control.

### Dual Question Timing
- **First Question**: Longer timeout (60s default) with warning about second chance
- **Second Question**: Even longer timeout (90s default) as final opportunity
- **Independent timers**: Each question has its own configurable timeout
- **Progressive warnings**: Language becomes more urgent in second question

### Voice Recognition Handling
- **Unique trigger IDs**: Separate IDs for first and second question responses
- **Comprehensive response matching**: Multiple variations of yes/no accepted
- **Graceful timeout handling**: No errors if responses aren't received

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
- **Dual question conflicts** prevented by unique trigger IDs
- **Manual override conflicts** are resolved through intelligent questioning

## Troubleshooting

### Voice Commands Not Working
- Verify voice assistant is properly configured
- Check media player volume and connectivity
- Test TTS service independently in Developer Tools
- Ensure conversation integration is enabled
- Check automation traces for voice recognition events

### Second Question Not Asked
- Verify First Question Timeout is appropriate
- Check that first question completes without errors
- Ensure media player remains available between questions
- Review automation traces for timeout behaviors

### Questions Asked Too Frequently
- Increase Motion Clear Delay setting
- Check for motion sensor false positives
- Verify door sensor isn't bouncing between states
- Check manual override settings aren't conflicting

### Cannot Respond in Time
- Increase First Question Timeout for more thinking time
- Increase Second Question Timeout for final response
- Check media player volume - ensure questions are audible
- Verify voice commands are being recognized correctly

### Manual Overrides Not Detected
- Verify light entity is correct in configuration
- Check that manual switch actually changes the entity state
- Increase Manual Override Delay if switches are slow
- Disable/re-enable Manual Safeguards setting

### Automation Conflicts
- Ensure only one instance of this automation per bathroom
- Check that no other automations control the same occupancy helper
- Verify voice assistant isn't being used by conflicting automations
- Review automation traces during manual operations

## Advanced Usage

### Multiple Bathrooms
Create separate instances of this blueprint for each bathroom, each with its own:
- Sensors (motion, door)
- Light control
- Input boolean helper (must be unique per bathroom)
- Media player (can be shared if within range)

### Timeout Customization for Different Users
- **Families with children**: Longer timeouts for slower responses
- **Elderly users**: Extended second question timeout
- **Accessibility needs**: Maximum timeout settings
- **Quick households**: Shorter timeouts for faster responses

### Integration with Other Automations
The occupancy helper can be used by other automations:
- Ventilation fan control based on occupancy
- Humidity management coordination
- Security system integration
- Energy monitoring and reporting

**Important**: Don't let other automations modify the occupancy helper directly, as this will interfere with the dual-question system and manual override protection.

### Speaker Placement Considerations
- **Audibility**: Ensure speaker can be heard over running water
- **Multiple speakers**: Use speaker groups for larger bathrooms
- **Volume automation**: Consider time-based volume adjustments
- **Privacy**: Balance audibility with household noise levels

### Customization Options
You can modify the blueprint for:
- **Different languages**: Edit all TTS message content
- **Different voice messages**: Customize the question phrasing
- **Additional voice commands**: Expand conversation command lists
- **Cultural preferences**: Adjust politeness/directness of questions
- **Multiple light entities**: Modify to control light groups

### Smart Switch Integration
Works with:
- **Physical wall switches**: Manual operations detected via entity state changes
- **Smart switches**: Paddle/button press detection
- **Voice assistants**: "Turn off bathroom light" commands trigger manual safeguards
- **Mobile apps**: Manual control through HA interface
- **Motion-sensing switches**: Can be used alongside (disable their auto-off)

## Blueprint File Location
Save this blueprint as `motion_light_with_voice.yaml` in your Home Assistant `blueprints/automation/` directory, or import it through the UI using the blueprint import feature.

## Version History
- **v3.0**: Added dual voice confirmation system with progressive warnings
- **v2.0**: Added manual override protection and safeguards
- **v1.0**: Initial release with basic motion/voice functionality

## Why the Dual Question System?

The two-question approach solves real-world problems that single-question systems can't handle:

1. **Immediate Response Pressure**: People don't feel rushed to drop everything for the first question
2. **Activity Interference**: Allows completion of tasks (brushing teeth, washing face) before responding
3. **Attention Management**: Accounts for distraction, concentration, or multitasking
4. **Accessibility**: Provides more time for those who need it
5. **Mistake Prevention**: Dramatically reduces false light turn-offs
6. **User Confidence**: People trust the system more knowing they get a second chance

The result is a bathroom automation that works harmoniously with real human behavior while maintaining intelligent safety features.
