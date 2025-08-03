# Advanced-Bathroom-Occupancy-with-Voice-Query
"Manages a bathroom light using motion and door sensors. When the room appears to be empty (door closes and motion stops), it asks a question on a media player to confirm occupancy before turning off the light. Requires a configured Voice Assistant and an input_boolean helper."

#The blueprint controls the input_boolean in two places:
1. Turns it ON when occupancy is detected:
yaml# When motion is detected OR door opens
- service: input_boolean.turn_on
  target:
    entity_id: !input occupancy_helper
2. Turns it OFF when room is confirmed empty:
yaml# When user says "no" or doesn't respond to voice query
- service: input_boolean.turn_off
  target:
    entity_id: !input occupancy_helper
The logic flow is:

Motion detected OR door opens → Sets input_boolean to ON + turns light ON
Door closes → Checks if input_boolean is ON, then starts the voice confirmation process
Voice response "no" OR timeout → Sets input_boolean to OFF + turns light OFF
Voice response "yes" → Keeps input_boolean ON + keeps light ON

Important notes:

The input_boolean is used as both a state tracker (to know if the room was previously occupied) and a condition checker (to prevent unnecessary voice queries)
You don't need any other automations - this blueprint is completely self-contained
The input_boolean you select in the blueprint configuration should be dedicated to this automation and not modified by other automations, or it could cause conflicts

