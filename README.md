# Logitech G27 with Ridge Racer 7 on RPCS3

How to get a Logitech G27 or G29 working in Ridge Racer 7 with force feedback and with the wheel buttons
usable in menus. 

Tested with RPCS3 0.0.43-20663, a Logitech G29 over G HUB, Ridge Racer 7 BLUS30001 and PS3 firmware 4.92.

## 1. Use the US version

The game has to be BLUS30001. The European BCES00009 and the Platinum release do not support USB wheels.

## 2. Enable the emulated wheel

`Configuration` > `Emulated Devices` > `Logitech G27 Wheel`

- check `Enabled (requires game restart)`
- `Compatibility limit`: `G27 (Default)`
- under "Use the device with the following mapping for force feedback" pick your wheel
- `Force feedback direction encoding`: `Steering Axis (Default)`

## 3. Map the wheel as pad so menus work

Save dialogs and menus read `cellPad`, not the USB wheel. With Player 1 set to `Null` you get stuck on the
first save dialog.

`Configuration` > `Pads` > `Player 1`

- `Handler`: `MMJoystick`
- `Device`: `Joystick #1`

| PS3 | Binding | PS3 | Binding |
|---|---|---|---|
| Cross | `Button 1` | Select | `Button 9` |
| Square | `Button 2` | Start | `Button 10` |
| Circle | `Button 3` | R3 | `Button 11` |
| Triangle | `Button 4` | L3 | `Button 12` |
| R1 | `Button 5` | D-Pad Up | `POV Up` |
| L1 | `Button 6` | D-Pad Down | `POV Down` |
| R2 | `Button 7` | D-Pad Left | `POV Left` |
| L2 | `Button 8` | D-Pad Right | `POV Right` |

The USB wheel keeps working next to the pad, so steering and force feedback come from the emulated wheel and
the buttons come from the pad.

## 4. Wheel software settings

In G HUB:

- operating range: 200 degrees
- sensitivity: leave at default
- centering spring: off
- force feedback strength: 100, adjust in game if needed

RPCS3 reads the axis linearly, without deadzone or curve, so any curve from the wheel software lands on top
of the game's own steering. The game never sends a rotation range command, so the physical range in G HUB is
the only way to change how direct the steering feels. 


## Troubleshooting

| Symptom | Cause |
|---|---|
| Wheel completely ignored | Wrong game version, use BLUS30001 |
| Wheel detected, no force feedback | Emulated wheel not enabled, or game not restarted after enabling |
| Force feedback pulls the wrong way | Wheel dialog: `Reverse force feedback effects`, or direction encoding `Cartesian` |
| Steering works, pedals dead | Compatibility limit set to Driving Force Pro or below |
| Pad mapping has no effect | Device is not `Joystick #<n>` |
