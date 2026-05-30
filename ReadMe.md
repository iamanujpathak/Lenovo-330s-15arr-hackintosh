# Lenovo IdeaPad 330S-15ARR macOS Sequoia Hackintosh Guide

An open-source guide and EFI configuration baseline for running macOS Sequoia on the AMD Ryzen-powered Lenovo IdeaPad 330S-15ARR. 

> ⚠️ **DISCLAIMER:** This repository is shared for educational and helper purposes. Hackintoshing involves deep firmware-level modifications. Proceed at your own risk. Always back up your existing files and operating systems before starting.

---

## 💻 Hardware Specifications

| Component | Specification | Status | Driver/Fix Used |
| :--- | :--- | :--- | :--- |
| **CPU** | AMD Ryzen (Raven Ridge Mobile APU) | 🟢 Working | AMD Vanilla Patches |
| **GPU** | AMD Radeon Vega Graphics | 🟢 Working | `NootedRed.kext` (Metal Enabled) |
| **Audio** | Realtek ALC230 (`DEV_0230`) | 🟢 Working | `AppleALC.kext` (layout-id: 13, 18, or 20) |
| **Trackpad** | ELAN061E I2C Touchpad | 🟢 Working | `VoodooI2C.kext` + `VoodooI2CELAN.kext` |
| **Keyboard** | Lenovo PS2 Keyboard | 🟢 Working | `VoodooPS2Controller.kext` |
| **Wi-Fi/BT** | Factory M.2 Card | 🟡 Card Dependent | Swap to Intel/Broadcom recommended |

---

## 🛠️ Step 1: BIOS Modifications (Crucial First Step)

AMD APUs require specific memory configurations that the factory Lenovo BIOS hides by default. To prevent instant boot panics, you must access advanced settings or use a modified/custom BIOS layout to adjust these values:

1. **Increase VRAM Dedicated Memory:** * Locate the **UMAF (Unified Memory Architecture Frame Buffer)** size configuration.
   * Change the setting from the default (typically 1GB or auto) to explicitly **2GB** (or greater). This is mandatory for `NootedRed` to drive macOS metal acceleration cleanly without system crashes.
   If Bios is locked use Smokeless_UMAF and:
    - Navigate through the menus: Device Manager → AMD CBS → NBIO Common Options → GFX Configuration.
    - Change IGPU Configuration from Auto or Specified to UMA_SPECIFIED.
    - A new option called UMA Frame Buffer Size will appear. Change it from 256M to 1G or 2G.
2. **Standard Hackintosh Toggles:**
   * **Disable:** Secure Boot, Fast Boot, AMD SVM (Virtualization - unless required, keep disabled during install).
   * **Enable:** AHCI SATA Mode, UEFI Boot Mode.

---

## 💾 Step 2: Creating the Bootable USB

1. Download a clean macOS Sequoia raw image file.
2. Download and install **[BalenaEtcher](https://balenaetcher.balena.io/)**.
3. Flash the raw image to a 16GB+ USB flash drive using BalenaEtcher.
4. Once flashed, your OS will create an unmapped partition block. Use a partition utility (like MiniTool Partition Wizard on Windows or Disk Utility) to mount the hidden **EFI partition** of the USB.
5. Replace the blank folder with the `EFI` repository files supplied here.

> 🔗 **macOS Sequoia Installer Link:** https://www.youtube.com/redirect?event=comments&redir_token=QUFFLUhqbF95dVNTTzBROHdmOC1jSUhNUDg1ZUY5MWtpQXxBQ3Jtc0tuWV84MVZUUVV6VlQwdHAwTnJOTjl4ZlpjZzBrNHEySmQxZC1MWFRxTEZuZWlROEhMSHRuanZFMWVwLUJXbU5DUGZpUEgySUNob3djQTQ4OVcwb3hjQVpMMW04Tzg3bmhSM1EzenNGbFk1TWczMUF4Zw&q=https%3A%2F%2Fwww.mediafire.com%2Ffile%2Fanhkso77omulhr4%2FOlarila%2BSequoia%2B15.7.5.raw%2Ffile

---

## 🧩 Step 3: Mandatory Post-Boot Quirks & Fixes

Once you boot into macOS Sequoia, you must execute these configurations to ensure overall stability:

### 1. UI Render Freeze (NootedRed Issue #235)
Due to a known shader rendering discrepancy inside `NootedRed` (At the time of writting this, the hackintosh community is working on it) with certain rendering pipes, visiting complex websites (like the GitHub Settings panel or high-fidelity wallpaper render contexts) will crash the browser or window server.
* 💡 **The Fix:** follow the threads on https://github.com/ChefKissInc/NootedRed/issues/235, and apply workarround depending on preinstall or post install stages

### 2. Trackpad Preferences Pane Force Hook
If your multi-touch gestures trace perfectly but your system preference applet only displays a basic "Mouse" menu interface layout:
1. Open your terminal emulator window and enter:
   ```bash
   defaults write com.apple.AppleMultitouchTrackpad TrackpadThreeFingerDrag -bool true
   killall lssave; defaults delete com.apple.systempreferences
   ```

> 💡 **Tip:** Ensure you always run an OpenCore NVRAM Reset on system restarts whenever tweaking interface input states.

---

## 🚫 Key Caveats & Troubleshooting

* ⚠️ **Random Boot Time Hangs:** If your system locks up instantly at the `mbinit` or `apfs_module_start` verbose screen lines, it is likely a driver execution race condition. To solve it, check the `boot-args` sequence inside `config.plist` and ensure `-vi2cpad` is mapped alongside an interleaved driver delay flag (`v2i-delay=500`) to let the core files unpack ahead of peripheral arrays.
* ⚠️ **Complete System UI Lockups:** If you encounter unexpected system-wide freezing during intensive operations, you can temporarily disable graphics acceleration by passing `-radvesa` to your `boot-args` array to safely debug files in a raw software-rendered safe environment.
* ⚠️ **Native Wireless Integration:** The stock internal PCIe Wi-Fi chip will not support AirDrop, Continuity, or Handoff. For full ecosystem interoperability under Sequoia, it is highly recommended to physically swap the internal M.2 card out for an Intel AX210 (using `AirportItlwm`) or a Fenvi BCM94360NG (patched via OpenCore Legacy Patcher).

---

## 🤝 Credits

* **Acidanthera** for OpenCore, Lilu, AppleALC, and VoodooPS2.
* **ChefKissInc** for the game-changing `NootedRed` AMD graphics driver suite.
* **VoodooI2C Team** for bringing precise gesture parameters to laptop tracking pads.