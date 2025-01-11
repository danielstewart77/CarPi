![Kivy](https://img.shields.io/badge/Built%20in-Kivy-purple) ![Python](https://img.shields.io/badge/Powered%20by-Python-yellow) ![Arch](https://img.shields.io/badge/Arch-ARM64-blue) ![OS](https://img.shields.io/badge/OS-Raspbian-red)

# obiPiDash
A Python project to pull vehicle information from a Bluetooth OBD-II adapter and output it to a nice GUI on any touchscreen display.

## Features
- Coolant Temp, Intake Temp, Battery Voltage, STFT, LTFT, Throttle Pos, Engine Load, Spark Advance, Gear Indicator, RPM (w/ MAX), Speed (w/ MAX)
- Adjustable Warning thresholds for Coolant Temp, Intake Temp, STFT, LTFT, RPM and Speed
- DTC Read and Clear Function
- Selectable C/F MPH/KPH
- Brightness Control

## Screenshots & Photos
Coming soon...

## Hardware and Setup Used
- Raspberry Pi 5 (8GB RAM)
- Raspberry Pi OS (64-bit) - Bookworm
- Python 3.11.2
- Display (TBD)

## Installation

### 1. Install Required Packages
```bash
sudo apt update
sudo apt install libfreetype6-dev libgl1-mesa-dev libgles2-mesa-dev libdrm-dev libgbm-dev libudev-dev libasound2-dev liblzma-dev libjpeg-dev libtiff-dev libwebp-dev git build-essential -y
sudo apt install gir1.2-ibus-1.0 libdbus-1-dev libegl1-mesa-dev libibus-1.0-5 libibus-1.0-dev libice-dev libsm-dev libsndio-dev libwayland-bin libwayland-dev libxi-dev libxinerama-dev libxkbcommon-dev libxrandr-dev libxss-dev libxt-dev libxv-dev x11proto-randr-dev x11proto-scrnsaver-dev x11proto-video-dev x11proto-xinerama-dev -y
sudo apt install gstreamer1.0-tools gstreamer1.0-plugins-bad gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-ugly gstreamer1.0-vaapi pkg-config python3-setuptools libgstreamer1.0-dev libmtdev-dev xclip xsel libjpeg-dev -y
sudo apt install python3-pip
```

### 2. Install SDL3 and Related Libraries
#### SDL
```bash
git clone https://github.com/libsdl-org/SDL.git
sudo mkdir -p /usr/local/lib/pkgconfig
sudo chown $USER:$USER /usr/local/lib/pkgconfig
cd SDL
cmake -S . -B build && cmake --build build --parallel $(nproc)
sudo cmake --install build
cd ..
```

#### SDL_image
```bash
git clone https://github.com/libsdl-org/SDL_image.git
cd SDL_image
./external/download.sh
cmake -S . -B build -DSDLIMAGE_VENDORED=ON
cmake --build build --parallel $(nproc)
sudo cmake --install build
cd ..
```

#### SDL_mixer
```bash
git clone https://github.com/libsdl-org/SDL_mixer.git
cd SDL_mixer
./external/download.sh
cmake -S . -B build -DSDLMIXER_VENDORED=ON
cmake --build build --parallel $(nproc)
sudo cmake --install build
cd ..
```

#### SDL_ttf
```bash
git clone https://github.com/libsdl-org/SDL_ttf.git
cd SDL_ttf
./external/download.sh
cmake -S . -B build -DSDLTTF_VENDORED=ON
cmake --build build --parallel $(nproc)
sudo cmake --install build
cd ..
```

### 3. Update System and Install Python OBD
```bash
sudo apt update
sudo apt upgrade -y
pip3 install obd
```

### 4. Set Up Virtual Environment
```bash
git clone https://github.com/charliehoward/obdPiDash.git
cd obdPiDash
python3 -m venv venv
source ./venv/bin/activate
pip install --upgrade pip setuptools Cython kivy
python main.py
```

### 5. Configure Touchscreen
Edit the Kivy config file:
```bash
sudo nano ~/.kivy/config.ini
```
Modify the following lines:
- Remove mouse cursor:
  ```ini
  show_cursor = 0
  ```
- Adjust input settings:
  ```ini
  [input]
  mouse = mouse
  mtdev_%(name)s = probesysfs,provider=mtdev
  hid_%(name)s = probesysfs,provider=hidinput
  ```

### 6. Configure Bluetooth
```bash
sudo bluetoothctl
agent on
default-agent
scan on
pair xx:xx:xx:xx:xx:xx  # Replace with your BT MAC
connect xx:xx:xx:xx:xx:xx
trust xx:xx:xx:xx:xx:xx
```

### 7. Set Up Startup Service
Create a service file:
```bash
sudo nano /etc/systemd/system/obdPiDash.service
```
Add the following content:
```ini
[Unit]
Description=Start OBDPi

[Service]
ExecStart=/bin/sh /home/pi/obdPiLauncher.sh
ExecStop=/usr/bin/pkill -9 -f main.py
WorkingDirectory=/home/pi/obdPiDash/
StandardOutput=inherit
StandardError=inherit
User=pi

[Install]
WantedBy=multi-user.target
```
Enable the service and make the launcher executable:
```bash
sudo systemctl enable obdPiDash.service
sudo chmod a+x ./obdPiLauncher.sh
```

### 8. Optional: Clean Up Boot
To remove boot text and logos, edit `/boot/config.txt`:
```bash
sudo nano /boot/config.txt
```
Add:
```ini
disable_splash=1
boot_delay=0
```
Modify `/boot/cmdline.txt`:
```bash
sudo nano /boot/cmdline.txt
```
Change:
```txt
console=tty1
```
To:
```txt
console=tty3 splash quiet logo.nologo vt.global_cursor_default=0 consoleblank=0
```
Comment out the following lines in `/etc/pam.d/login`:
```txt
# session    optional   pam_lastlog.so
# session    optional   pam_motd.so motd=/run/motd.dynamic
# session    optional   pam_motd.so noupdate
```
Suppress login messages:
```bash
run touch ~/.hushlogin
```

### 9. Reboot and Test
Restart your system to finalize setup:
```bash
sudo reboot
