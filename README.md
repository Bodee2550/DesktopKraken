Standalone NZXT Kraken Elite LCD Dashboard (No NZXT CAM)
A lightweight, standalone Windows desktop application that drives the 640×640 circular LCD on NZXT Kraken Elite coolers directly over USB—completely replacing NZXT CAM.
Built for users who want live CPU/GPU telemetry over custom images or animated GIFs with rock-solid sleep/wake recovery, USB disconnect/replug resilience, thermal warning gradients, and near-zero idle CPU usage.
---
Why This Exists
NZXT CAM's web integrations frequently freeze or fail to reconnect after Windows sleeps and wakes, especially when the LCD is detached and mounted externally. This app talks directly to the Kraken's USB Bulk (`WinUSB`) and HID (`HidUsb`) interfaces using strict double-buffering, listens for Windows power (`WM_POWERBROADCAST`) and USB (`WM_DEVICECHANGE`) events, and automatically re-initializes the display every time your PC wakes up or the cable is replugged.
---
Features
Direct USB LCD Control (No NZXT CAM): Talks directly to `hidapi` (Interface 1) and `pyusb`/`libusb` (Interface 0) with double-buffered 4 MB memory slots and verified HID acknowledgments.
Sleep/Wake & USB Fault Watchdog: Hooks Windows `PBT_APMSUSPEND`, `PBT_APMRESUMEAUTOMATIC`, `PBT_APMRESUMESUSPEND`, and `WM_DEVICECHANGE`, optionally blanks the screen on sleep, and auto-recovers with exponential backoff and zero crashes.
High-FPS Animated GIF & Static Image Support: Hardware-accelerated GIF playback on the cooler with circular edge masking, shared global color palette optimization, phase-locked stat swaps, and support for up to 60+ frames.
Automatic Hardware Detection: Automatically reads your CPU (Intel & AMD via Registry + LibreHardwareMonitor/PawnIO) and GPU (NVIDIA via NVML, AMD/Intel via LibreHardwareMonitor) and formats names cleanly to fit the circular layout.
Thermal Gradient Gauges: Smooth 3-stop rounded arcs (`Cool → Warm → Red`) with customizable thresholds and warning-tinted icons when temperatures climb.
Extensible Sensor Hub: Built-in support for CPU/GPU temp, load, wattage, clock speed, VRAM %, RAM usage, and AIO liquid temperature.
System Tray + 1:1 Live Preview UI: PySide6 settings app with real-time circular preview, orientation (`0°/90°/180°/270°`), brightness, custom `.ttf`/`.otf` fonts, background zoom/fit/dimming, and multi-profile JSON persistence.
---
Supported Hardware & Requirements
Component	Requirement / Supported Models
Coolers	NZXT Kraken 2023 Elite (`VID 1E71`, `PID 300C`, tested on fw `2.1.0`) & Kraken Elite V2 2024 (`PID 3012`) — 640×640 LCD
OS	Windows 11 / Windows 10 (64-bit)
Python	Python 3.12 – 3.14 (64-bit, added to `PATH`)
CPUs	Intel Core (reads `CPU Package`) & AMD Ryzen (reads `Core (Tctl/Tdie)`)
GPUs	NVIDIA RTX/GTX (via NVML) & AMD/Intel fallback (via LibreHardwareMonitor)
Privileges	Administrator rights (required once for reading CPU temperatures via PawnIO)
> **Important Cooling Note:** If your Kraken AIO pump and radiator fans are still actively cooling your CPU, disabling NZXT CAM leaves the pump and fans on their last hardware curve (or motherboard fan headers). This app drives the **LCD only** and does not control pump or fan speeds.
---
Step-by-Step Installation
1. Download the Repository
Clone the repo or click Code → Download ZIP and extract it to a clean folder (for example, `C:\KrakenDashboard`):
```powershell
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>
```
2. Install Python Dependencies
Open PowerShell as Administrator inside the project folder and run:
```powershell
pip install -r requirements.txt
```
3. Install LibreHardwareMonitor & PawnIO (for CPU Temps)
Windows Defender blocks the older `WinRing0` kernel driver, so CPU temperatures use LibreHardwareMonitor v0.9.6 together with the signed PawnIO driver. Run the included setup script in your Administrator PowerShell:
```powershell
powershell -ExecutionPolicy Bypass -File .\setup_lhm.ps1
```
This script automatically:
Downloads, extracts, and unblocks `LibreHardwareMonitorLib.dll` into `.\lhm\`.
Installs PawnIO via `winget` (if not already installed — you can also download it manually from pawnio.eu).
Prints your Kraken's current Windows USB driver bindings.
4. Verify USB Drivers (WinUSB + HID Split)
The Kraken Elite exposes two separate USB interfaces under `VID_1E71`:
Interface	Hardware Path	Purpose	Required Windows Driver
Interface 0	`MI_00` (`NZXT Kraken Elite`)	Bulk OUT pixel & GIF transfers	`WINUSB`
Interface 1	`MI_01` (`USB Input Device`)	HID control, brightness & buckets	`HidUsb` (default Windows driver)
Parent	Composite Device	USB composite hub	`usbccgp` (default Windows driver)
Check the table printed at the end of `setup_lhm.ps1`:
If `MI_00` already shows `WINUSB` and `MI_01` shows `HidUsb`, you are ready to go—skip Zadig.
If `MI_00` is not bound to `WINUSB`:
Download and open Zadig.
Click Options → List All Devices.
Select ONLY `NZXT Kraken Elite (Interface 0)` (confirm USB ID `1E71 300C` or `1E71 3012` and `MI_00`).
Select WinUSB as the target driver and click Replace Driver.
Never replace the parent `USB Composite Device` or `Interface 1 (MI_01)`.
5. Permanently Disable NZXT CAM
Close NZXT CAM completely (check the system tray), then disable its background service in an Administrator PowerShell so it never hijacks the LCD after waking from sleep:
```powershell
Stop-Service -Name "CAMService" -Force
Set-Service -Name "CAMService" -StartupType Disabled
Get-Service -Name "CAMService" | Select-Object Name, Status, StartType
```
> **Note:** Do **not** disable `camsvc` (*Capability Access Manager Service*) — that is a built-in Windows service unrelated to NZXT CAM.
Also open Task Manager → Startup apps and set NZXT CAM to Disabled.
6. Run the Dashboard
From Administrator PowerShell (shows live console output):
```powershell
  python .\main.py
  ```
One-click elevated launch (no console window):
Double-click `run.bat`.
On first launch, the app auto-detects your CPU and GPU, creates `config.json`, opens the settings window with a live 1:1 circular preview, and immediately starts driving the Kraken LCD.
---
Optional: Auto-Start with Windows (Zero UAC Prompt)
Because reading CPU temperatures requires Administrator privileges, standard Startup folder shortcuts will either lack CPU temps or pop up a UAC prompt. Use the included Task Scheduler installer instead—it starts the app elevated and minimized in the system tray at logon with no UAC prompt:
```powershell
powershell -ExecutionPolicy Bypass -File .\install_autostart.ps1
```
To remove auto-start later:
```powershell
powershell -ExecutionPolicy Bypass -File .\uninstall_autostart.ps1
```
---
Optional: Build a Standalone `.exe`
If you want to package the app into a standalone Windows executable:
```powershell
pip install pyinstaller
powershell -ExecutionPolicy Bypass -File .\build_exe.ps1
```
The output executable will be built at `dist\KrakenEliteDashboard\KrakenEliteDashboard.exe`.
---
Customization Guide
Backgrounds: Drop your favorite `.gif`, `.png`, or `.jpg` files into the `Gifs\` folder and select them under Background. Adjust dimming (`0–85%`), zoom, X/Y offset, and GIF Max Frames (`12–90`) to balance clip length against stat swap speed.
Thermal Gradient Bars: Under CPU & GPU Sides, toggle Thermal gradient bar and customize your Cool, Warm, and Hot colors along with the Min/Max temperature range. Icons gently tint red as temps approach the hot threshold.
Custom Fonts: Pick from built-in Windows fonts under Style & Fonts or click Browse .ttf... to load any `.ttf` or `.otf` font file.
Orientation: Choose `0°`, `90°`, `180°`, or `270°` under Profiles & Display (applied losslessly in software so custom images and GIFs always rotate reliably).
Profiles: Create, duplicate, delete, and switch saved profiles from either the settings window or the system tray menu. All settings are stored in `config.json`.
---
Verifying Sleep/Wake Recovery (10+ Cycle Test)
Open the app, confirm live stats are updating on the LCD, and enable Blank / dim LCD to 0% when Windows sleeps under Profiles & Display.
Click Test Sleep/Wake in the left pane to confirm software suspend/resume recovery completes cleanly in ~4 seconds.
Put Windows into real Sleep (Start → Power → Sleep), wait 15–30 seconds, and wake the PC. Repeat across 10+ cycles.
Check `logs\kraken_dashboard.log` (or the Sensors & Logs tab). Each cycle should log:
`System SUSPEND event received — preparing LCD for sleep.`
`System RESUME event received — scheduling USB re-init in 2.5s.`
`Connected: NZXT Kraken ...`
---
Troubleshooting
Symptom	Likely Cause & Fix
Device not detected	Ensure `CAMService` is Stopped & Disabled, and verify `MI_00` is bound to `WINUSB` (`setup_lhm.ps1` prints current drivers). Click Force Reconnect in the tray menu.
CPU temperature stuck at `0°C`	Run as Administrator (`run.bat` or elevated PowerShell), verify `lhm\LibreHardwareMonitorLib.dll` exists and is unblocked, and confirm PawnIO is installed. (Note: some aggressive kernel anti-cheats can temporarily block PawnIO.)
Black or frozen LCD after hard power glitch	Shut down the PC and switch the PSU off (or unplug USB + SATA power to the Kraken) for 30 seconds to reset the cooler's microcontroller, then boot back up.
Animated GIF looks choppy	Increase GIF Max Frames under Background (default `60`). Very large 640×640 clips that exceed the 4 MB hardware bucket automatically step down colors first before reducing frames—check Sensors & Logs for exact frame count, FPS, and KB size.
Screen wakes up blank	Increase the wake delay slightly if your motherboard USB controller enumerates slowly, or check `logs\kraken_dashboard.log` for USB errors.
---
Project Structure
```text
├── main.py                  # PySide6 tray app, 1:1 preview UI, power/USB event hooks & worker
├── kraken_driver.py         # Direct double-buffered WinUSB + HID driver (1E71:300C / 3012)
├── renderer.py              # 2x supersampled Dual Info UI, thermal gradients & GIF encoder
├── sensors.py               # Extensible Intel/AMD CPU + NVIDIA/AMD GPU sensor hub
├── config.py                # Dataclasses & atomic JSON profile loader/saver
├── requirements.txt         # Python dependencies
├── setup_lhm.ps1            # Downloads LibreHardwareMonitor, checks PawnIO & USB drivers
├── run.bat                  # One-click elevated launcher
├── install_autostart.ps1    # Installs elevated Task Scheduler logon startup
├── uninstall_autostart.ps1  # Removes Task Scheduler auto-start entry
├── build_exe.ps1            # Optional PyInstaller standalone EXE builder
├── lhm/                     # Holds LibreHardwareMonitorLib.dll (populated by setup_lhm.ps1)
├── Gifs/                    # Optional folder for your background GIFs/images
├── .gitignore               # Keeps logs, config.json, and third-party binaries out of git
├── LICENSE                  # MIT License
└── README.md                # Full setup & troubleshooting documentation
```
---
Acknowledgments
Layout inspired by Dual Info (`brunoandradebr/nzxt`) by Bruno Andrade.
Kraken Elite USB HID/Bulk protocol insights informed by the open-source `liquidctl` community.
Hardware telemetry powered by LibreHardwareMonitor, PawnIO, and NVIDIA NVML.
