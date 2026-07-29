PS5 DUALSENSE R2 Trigger RAPID-FIRE MOD ESP32 C3 Super Mini— Made by Vegueta1


⚠️FULLY WORKING STATUS — READ BEFORE INSTALLING.

Introducing the PS5 Dual Sense R2 Trigger Rapid-Fire Mod modular rapid-fire system targeting the R2 trigger on a Dual Sense controller revision (BDM-010)

This project is open-source and designed to be accessible for hobbyists, with a noob-friendly installation guide. A huge shoutout to RDC for His legendary Dual Sense PCB scans and hardware insights, which made this project possible.

Status: Beta – Currently Working to expand support to other Board This Is a Beta to confirm that work on other boards.



⚠️ Important Notes.
Online Play at your own risk! This release is provided as is, with no warranty or guarantee of fitness for any purpose. In online environments may result in bans, account suspension, or other penalties. You are solely responsible for how you use this project.

Source Code Updated Version v7.4.4 Beta is attached download it and paste it on Arduino IDE

FEATURES:



How to Connect

Power on the mod.

Connect your phone/PC to the WiFi network:
- SSID: RapidFireMod_v7.4.4
- Password: Default Password Is 12345678

Open your browser and go to: http://192.168.4.1

Modes

1. OFF Mode
Rapid fire completely disabled.
Normal controller operation.
Automatically activates after ~5 minutes of inactivity.

2. Continuous Mode (Custom SPS)
Holds rapid fire as long as you keep R2 pressed.
Custom SPS (Shots Per Second): Adjustable from 1 to 40 (recommended 4–30 for best stability and game compatibility).
Ideal for automatic weapons, sustained fire, etc.

3. Burst Mode (Custom Shots)
Fires a precise number of shots with each trigger pull.
Custom Burst Count: Adjustable from 1 to 50 shots.
Perfect for semi-auto weapons or controlled bursts (e.g. 3–5 shots).

All Features Explained

Calibration (Very Important!)
Manual Calibration:
Cal Released (trigger not pressed)
Cal Pressed (trigger fully pressed)
Apply Manual Cal
Reset Calibration → Restores safe default values.
Good calibration = reliable trigger detection.

Advanced Settings

Jitter (0-100%)
Adds small random variations to firing timing. Makes it harder to detect and feels more natural. Recommended: 10-20%. Use this on online play for less risk of getting banned.

Hysteresis (0-100%)
Creates a "dead zone" around the trigger threshold to prevent rapid on/off chatter. Recommended: 10-15%.

Debounce (0-500ms)
Ignores very quick or noisy trigger changes. Recommended: 40-60ms.

Press Percent (0-100%)
How far you need to press the trigger to activate firing. Recommended: 90-95%.

Pulse Width (5-200ms)
Duration of each individual "press" signal.
Lower values = better for high SPS.
Recommended: 25-40ms.

Polarity
Decreasing (Default)
Increasing


Global Buttons

Turn OFF

Reset Trigger State – Fixes stuck or weird behavior

Auto Fix – Quickly applies safer debounce + hysteresis values


Best Practices & Tips

Always start with Smart Auto-Calibrate.

30 SPS is an excellent sweet spot — very stable and effective.

For higher SPS (35+), reduce Pulse Width to 5-10ms.

Use moderate jitter for a more human-like feel.
If the mod feels

inconsistent, recalibrate and use Reset Trigger State.

Troubleshooting

No firing / wrong detection → Recalibrate
Choppy or stuck firing → Reset Trigger State or power cycle

SPS not stable → Lower SPS slightly or reduce Pulse Width

Too sensitive → Increase Debounce and Hysteresis

Enjoy the mod!

This version is highly optimized for reliability and ease of use. Feel free to share your results, settings, or suggestions.



Visual Diagram for ESP32 C3 Super Mini Installation.

Esp32 C3 Super Mini Wiring Diagram.JPG


PCB PS5 Controller triggers R2 Assembly: You Don't need to remove R2 trigger you can solder the point taking the flex out only, fast and easy.


BWL-010, TRIGGER ASSEMBLY:
BWL-010_SIDE_A.JPG



BWL-020, TRIGGER ASSEMBLY:
BWL-020_SIDE_A.JPG


What You Need:

- ESP32 C3 Super Mini

- Download Source code paste it on Arduino IDE and Flash it

- Data‑capable USB cable

- Windows, macOS, or Linux PC

Connect to the ESP32's Wi‑Fi

- SSID: RapidFireMod_v7.4.4

- Password: Default Password Is 12345678

- IP:192.168.4.1

On your PC/phone:

1. Open Wi‑Fi settings.

2. Connect to RapidFireMod_v7.4.4

3. Ignore No Internet Warning — that's normal.