# Logitech G27 with Ridge Racer 7 on RPCS3

How to get a Logitech G27 or G29 working in Ridge Racer 7 with force feedback and with the wheel buttons
usable in menus. No patch, no custom build, only settings.

Tested with RPCS3 0.0.43-20663, a Logitech G29 over G HUB, Ridge Racer 7 BLUS30001 and PS3 firmware 4.92.

## 1. Use the US version

The game has to be BLUS30001. The European BCES00009 and the Platinum release do not support USB wheels.
The RPCS3 log prints the serial at boot as `SYS: Serial: ...`.

## 2. Enable the emulated wheel

`Configuration` > `Emulated Devices` > `Logitech G27 Wheel`

- check `Enabled (requires game restart)`
- `Compatibility limit`: `G27 (Default)`
- under "Use the device with the following mapping for force feedback" pick your wheel
- `Force feedback direction encoding`: `Steering Axis (Default)`

Restart the game afterwards. The setting is only read at startup.

RPCS3 reports the wheel as `046d:c294` at boot and switches to another Logitech personality when the game
asks for it. Ridge Racer 7 asks for the G25:

```
logitech_g27: Change device mode : buf=[F8 01 ... F8 10 ...], cmd=0x10, arg=0x0, lim=4 -> pers=2(G25)
```

Steering, pedals and force feedback all run through this mode. Do not force a different product ID.

Keep the compatibility limit at G27 or G25. On `Driving Force Pro` and below RPCS3 emulates the old Driving
Force protocol, which has one combined pedal axis, and that axis is hardcoded to `0x7f` (both pedals
released). The pedals then do nothing.

## 3. Map the wheel as pad so menus work

Save dialogs and menus read `cellPad`, not the USB wheel. With Player 1 set to `Null` you get stuck on the
first save dialog.

`Configuration` > `Pads` > `Player 1`

- `Handler`: `MMJoystick`
- `Device`: `Joystick #1`

The device name is the part people get wrong. This handler uses the old winmm API (`joyGetPosEx`), not
DirectInput, so it generates names as `Joystick #<n>`, starting at 1. The product name from device manager is
accepted without complaint but reads no input at all.

Suggested mapping. `Button N` is bit `N-1` of the button bitmask.

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
the only way to change how direct the steering feels. 900 degrees also works, it just feels very indirect in
this game.

## 5. Frame limit and VSync

`Configuration` > `GPU`: set `Frame limit` to 60 and enable `VSync`.

Ridge Racer 7 running above 60 fps desyncs and can die with

```
SIG: Thread terminated due to fatal error: Dead FIFO commands queue state has been detected!
```

The picture then freezes on the last frame and the window stops responding, because the RSX thread is gone
while the rest keeps running. You can only kill `rpcs3.exe`.

## 6. Intro video can hang

The intro video sometimes freezes at the transition between two video streams. The log then shows

```
cellVdecGetPicItem failed with 0x80610104
cellVdec: Video au decode has been waiting for a consumer for 5 seconds
```

Press Cross or Start while the video runs so the transition is skipped.

## Troubleshooting

| Symptom | Cause |
|---|---|
| Wheel completely ignored | Wrong game version, use BLUS30001 |
| Wheel detected, no force feedback | Emulated wheel not enabled, or game not restarted after enabling |
| Force feedback pulls the wrong way | Wheel dialog: `Reverse force feedback effects`, or direction encoding `Cartesian` |
| Steering works, pedals dead | Compatibility limit set to Driving Force Pro or below |
| Pad mapping has no effect | Device is not `Joystick #<n>` |
| Every player falls back to Null | Broken `Default.yml`. The log says `Input: Loaded empty pad config` |
| Freeze on the intro video | Section 6 |
| Freeze during a race, dead window | RSX dead FIFO, see section 5 |

## Optional: check your own button numbers

Save as `joyprobe.py`, run it, press a button. `buttons=...001` means `Button 1`.

```python
import ctypes, time
from ctypes import wintypes

winmm = ctypes.WinDLL("winmm")

class JOYINFOEX(ctypes.Structure):
    _fields_ = [("dwSize", wintypes.DWORD), ("dwFlags", wintypes.DWORD),
        ("dwXpos", wintypes.DWORD), ("dwYpos", wintypes.DWORD), ("dwZpos", wintypes.DWORD),
        ("dwRpos", wintypes.DWORD), ("dwUpos", wintypes.DWORD), ("dwVpos", wintypes.DWORD),
        ("dwButtons", wintypes.DWORD), ("dwButtonNumber", wintypes.DWORD),
        ("dwPOV", wintypes.DWORD), ("dwReserved1", wintypes.DWORD), ("dwReserved2", wintypes.DWORD)]

ji = JOYINFOEX()
ji.dwSize = ctypes.sizeof(ji)
ji.dwFlags = 0xFF
last = None

while True:
    if winmm.joyGetPosEx(0, ctypes.byref(ji)) == 0:
        state = (ji.dwButtons, ji.dwXpos, ji.dwYpos, ji.dwPOV)
        if state != last:
            print(f"buttons={ji.dwButtons:025b} x={ji.dwXpos} y={ji.dwYpos} pov={ji.dwPOV}")
            last = state
    time.sleep(0.05)
```

Turning the wheel to each stop should move `x` towards 0 and 65535. If it stops earlier, the wheel software
is limiting the axis, not RPCS3.
