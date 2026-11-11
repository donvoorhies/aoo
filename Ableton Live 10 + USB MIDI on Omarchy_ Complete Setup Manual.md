On this page

[System overview](#overview) [1. Install WINE & dependencies](#install) [2. Configure the WINE prefix](#prefix) [3. Install Ableton Live 10](#ableton) [4. Connect & verify MIDI](#verify) [5. Configure MIDI in Live](#configure) [6. MP3 & audio tips](#tips)

# Ableton Live 10 + USB MIDI on Omarchy

End-to-end guide: WINE installation, prefix setup, Ableton Live 10, ALSA MIDI bridge and MP3 codecs.

[Download as PDF](#)

## System overview

Operating system

Omarchy (Arch Linux-based, PipeWire audio backend)

Compatibility layer

WINE (64-bit `WINEPREFIX`, wow64 mode) and Winetricks

DAW

Ableton Live 10 (64-bit)

Hardware

Any class-compliant plug-and-play USB MIDI controller (e.g. Korg nanoKONTROL, keyboards, pad controllers)

## 1. Install WINE, Winetricks and Linux audio/codec dependencies

Before installing Ableton, install WINE, Winetricks, the 32-bit and 64-bit ALSA MIDI libraries, and the 64-bit GStreamer codecs (needed by `winegstreamer` for MP3 decoding and multimedia initialization).

### Enable the multilib repository (if not already active)

Required on Arch/Omarchy for 32-bit compatibility libraries. Edit `/etc/pacman.conf`, make sure `[multilib]` and its `Include` line are uncommented, then sync:

```
sudo pacman -Syu
```

### Install WINE, Winetricks and base dependencies

```
sudo pacman -S wine winetricks wine-mono wine-gecko cabextract
```

### Install ALSA audio and MIDI bridge libraries

```
sudo pacman -S alsa-utils alsa-lib alsa-plugins lib32-alsa-lib lib32-alsa-plugins pipewire-alsa
```

### Install 64-bit GStreamer plugins (for MP3 decoding)

```
sudo pacman -S gst-plugins-base gst-plugins-good gst-plugins-bad gst-plugins-ugly gst-libav
```

### Grant hardware audio and MIDI permissions

```
sudo usermod -aG audio $USER
```

Log out and back in if your user was not previously in the `audio` group.

## 2. Initialize and configure the WINE prefix

Create a clean 64-bit WINE environment, install essential Windows fonts and runtimes, and explicitly enable the ALSA driver in the WINE registry so ALSA MIDI ports appear in Ableton.

### Initialize a 64-bit WINE prefix

```
WINEARCH=win64 WINEPREFIX=~/.wine wineboot -u
```

If prompts appear to install Mono or Gecko, click **Install**. To use a custom folder instead of `~/.wine`, replace the path and pass that same `WINEPREFIX` to all subsequent commands.

### Install core fonts and Windows components via Winetricks

```
winetricks corefonts vcrun2015 l3codecx
```

`corefonts` prevents missing UI text in Ableton menus, `vcrun2015` supplies the Visual C++ runtimes, and `l3codecx` registers Windows MP3 codec support alongside GStreamer.

### Enable ALSA MIDI alongside PulseAudio/PipeWire in the registry

1. Launch the Registry Editor: `wine regedit`
2. Navigate to `HKEY_CURRENT_USER\Software\Wine\Drivers`
3. Edit or create the String Value named `Audio` and set its value to `pulse,alsa`
4. Restart the WINE background server to apply the changes:

   ```
   wineserver -k
   ```

## 3. Install Ableton Live 10

Extract your Ableton Live 10 (64-bit) Windows installer archive in Linux so `Setup.msi` or `Ableton Live 10 Installer.exe` is accessible on your drive.

### Run the installer inside WINE

```
wine ~/Downloads/Ableton\ Live\ 10\ Suite\ Installer.exe
```

If your download contains an `.msi` file instead of an `.exe`, run it with:

```
wine msiexec /i Setup.msi
```

Follow the standard Windows setup wizard using the default `C:\ProgramData\Ableton\...` path.

### Launch and authorize Ableton Live 10

Start Ableton from your application launcher or terminal:

```
wine "C:\ProgramData\Ableton\Live 10 Suite\Program\Ableton Live 10 Suite.exe"
```

Authorize via the browser or an offline authorization file (`Authorize.auz`). Under **Options > Preferences > Licenses Maintenance**, set Automatic Updates to **Never** to prevent Windows updater conflicts.

## 4. Connect and verify any USB MIDI controller in Linux

WINE enumerates ALSA MIDI hardware during startup, so always connect your controller before launching Ableton.

1. Plug in your USB MIDI controller(s) before starting Ableton Live 10.
2. Verify that Linux and WINE detect the MIDI clients:

   ```
   aconnect -l
   ```

   Confirm that both your controller (e.g. client 24: 'nanoKONTROL') and client 128: 'WINE midi driver' appear. Client numbers vary dynamically.
3. Optional manual port routing: if WINE does not link to a controller automatically, connect the ports using the client IDs shown by `aconnect -l`:

   ```
   aconnect <controller_client>:0 <wine_client>:0
   aconnect <wine_client>:1 <controller_client>:0
   ```

## 5. Configure MIDI and mapping in Ableton Live 10

Open **Options > Preferences** (Ctrl + ,) and select the **Link/MIDI** tab.

1. **Control Surface (optional).** If your device has an official Ableton script (Akai APC, Novation Launchpad, Ableton Push), select it from the Control Surface dropdown and match its Input and Output ports. Otherwise leave it set to None.
2. **Configure MIDI ports (bottom section).** Enable the switches that fit your controller's Input port:

   | Port | Enable (On / yellow) for |
   | --- | --- |
   | **Track** | Keyboards, drum pads and sequencers, so they can play notes into MIDI instrument tracks. |
   | **Remote** | Knobs, faders and buttons (like the Korg nanoKONTROL) so they work with MIDI Learn mode (Ctrl + M). |
   | **Sync** | External hardware synths or drum machines that need Ableton's MIDI clock tempo signal. |
3. **Manual MIDI mapping.** Press Ctrl + M (or click **MIDI** in the top-right corner), click any control in Ableton, move the physical fader, knob or button on your controller, then press Ctrl + M again to exit.
4. **Save as default template.** Go to **Options > Preferences > File Folder** and click **Save Current Set as Default** so your controller setup loads in every new project.

## 6. Performance tips for MP3s and audio

- **Speed up MP3 imports.** Ableton decodes MP3s through WINE's GStreamer bridge into a temporary WAV file on first load. Once decoded, the file stays in your Decoding Cache folder (set under Preferences > File Folder) and loads instantly afterwards.
- **Batch-convert large files natively on Linux.** For long DJ mixes or large sample packs, convert MP3s to WAV in Linux before importing to bypass WINE translation overhead:

  ```
  for f in *.mp3; do ffmpeg -i "$f" "${f%.mp3}.wav"; done
  ```
- **Disable Auto-Warp on long tracks.** In **Options > Preferences > Record Warp Launch**, set Auto-Warp Long Samples to Off to prevent CPU spikes when dragging in full-length audio files.