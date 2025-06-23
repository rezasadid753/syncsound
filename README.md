# 🔊 PulseAudio Bluetooth + Built-in Speaker Sync for Ubuntu

Easily route audio simultaneously to your laptop’s built-in speaker and a Bluetooth speaker with latency adjustment — so they stay perfectly in sync.

---

## 🚀 About

I wanted to play sound on both my laptop’s built-in audio and a Bluetooth speaker at the same time on Ubuntu, but the Bluetooth delay caused annoying echo and lag. After trying multiple solutions, I found a stable and simple setup using PulseAudio’s null sink with manual latency compensation. This guide documents the **exact commands and config** that worked perfectly for me, so you can replicate it hassle-free.

---

## 🛠️ Features

- Send audio to built-in and Bluetooth speakers simultaneously  
- Manual delay on either device to fix lag  
- Simple command-line setup (no heavy config)  
- Works with PipeWire (modern Ubuntu audio stack)  
- Uses common tools: `paprefs`, `pavucontrol`, `pactl`  

---

## 📦 Installation & Setup

### 1. 🖥️ Install GUI tools for easy audio management

```bash
sudo apt update
sudo apt install paprefs pavucontrol
````

* **paprefs** lets you enable simultaneous output if needed
* **pavucontrol** is PulseAudio Volume Control GUI

---

### 2. 🔍 Check available sinks (audio outputs)

```bash
pactl list short sinks
```

Example output (yours will vary):

```
41  combine-sink-37341-29  PipeWire  float32le 2ch 48000Hz  RUNNING
57  alsa_output.pci-0000_00_1f.3.analog-stereo  PipeWire  s32le 2ch 48000Hz  RUNNING
60  alsa_output.platform-snd_aloop.0.analog-stereo  PipeWire  s32le 2ch 48000Hz  RUNNING
148  bluez_output.87_8F_ED_29_C3_F6.1  PipeWire  s16le 2ch 48000Hz  IDLE
```

* Here `alsa_output.pci-0000_00_1f.3.analog-stereo` is the **built-in laptop speaker**
* `bluez_output.87_8F_ED_29_C3_F6.1` is the **Bluetooth speaker**

---

### 3. 🎛️ Set up simultaneous output with latency adjustment

Run these commands to create a virtual sink and loopbacks with delay compensation:

```bash
# Unload any existing loopback modules first
pactl unload-module module-loopback

# Create a null sink named "multiout"
pactl load-module module-null-sink sink_name=multiout sink_properties=device.description=MultiOutput

# Loop audio from "multiout" to built-in speaker with delay (adjust latency_msec here)
pactl load-module module-loopback source=multiout.monitor sink=alsa_output.pci-0000_00_1f.3.analog-stereo latency_msec=250

# Loop audio from "multiout" to Bluetooth speaker with zero delay
pactl load-module module-loopback source=multiout.monitor sink=bluez_output.87_8F_ED_29_C3_F6.1 latency_msec=0
```

> **Note:** Here, `latency_msec=250` on the laptop speaker worked perfectly for me to sync with Bluetooth delay.

---

### 4. 🎧 Use it

* Open **pavucontrol**
* Go to the **Playback** tab
* Change the output device for your audio apps to **MultiOutput** sink

Now audio plays on **both speakers** synced!

---

## 💾 Run After Reboot

PulseAudio loopback modules don’t persist after reboot, so run these commands each time you start your system to restore the setup:

```bash
pactl unload-module module-loopback
pactl load-module module-loopback source=multiout.monitor sink=alsa_output.pci-0000_00_1f.3.analog-stereo latency_msec=250
pactl load-module module-loopback source=multiout.monitor sink=bluez_output.87_8F_ED_29_C3_F6.1 latency_msec=0
```

You can automate this by adding them to a startup script or system service if you like.

---

## 🧪 Testing & Tuning Latency

Try these to check sync:

* Play music with sharp beats or claps and listen carefully
* Use YouTube lip-sync or audio delay test videos
* Record both speakers with a phone to check sync visually

Adjust the `latency_msec` value in the loopback command for your built-in speaker until sync feels perfect.

---

## 🧹 Cleanup

To remove loopbacks and the null sink:

```bash
pactl unload-module module-loopback  # unload loopbacks (repeat if multiple loaded)
pactl unload-module module-null-sink  # unload null sink if needed
```

---

## 💡 Notes

* This setup works on Ubuntu with **PipeWire** replacing PulseAudio (default on recent versions).
* Bluetooth inherently has lag, so you need to delay the other device (usually laptop speaker) to sync.
* Adjust latency as needed depending on your devices and environment.

---

## 🤝 Acknowledgements

I didn’t create these tools; just sharing the steps I used to get this working for my setup. Hope this helps anyone struggling with audio sync!
