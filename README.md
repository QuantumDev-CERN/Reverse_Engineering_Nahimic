# Nahimic APO4 → EasyEffects Preset Pack
**For Lenovo LOQ 10 (AMD Ryzen) · Tuned for IEM / Headphone use**

Reverse-engineered from Nahimic APO4 v1.10.11.0 driver files.  
Targets: PipeWire + EasyEffects on Fedora 43 (and any modern Linux).

---

## How to Install

Copy the four `.json` files into your EasyEffects output presets folder:

```bash
mkdir -p ~/.config/easyeffects/output
cp *.json ~/.config/easyeffects/output/
```

Then open EasyEffects → **Presets** tab → select the preset you want.

---

## The 4 Presets

### 🎵 Nahimic-Music-IEM  *(best for music listening)*
**Chain:** `EQ(12 bands) → Limiter`

Nahimic sub-preset: **Dynamic** (the active preset found in Windows registry).  
Signature V-curve: boosted bass and treble with a scooped midrange.

| Stage | What it does | Source |
|---|---|---|
| EQ band 0–9 | +1/+5/+2/−2/−2/0/0/+1/+5/+2 dB at ISO 10-band freqs | `Dynamic.json` + registry |
| band10 | Bass Boost: +6 dB @ 250 Hz, Q=2.020 | `kSet_BassBoostGainDB` |
| band11 | Treble Enhancer: +4 dB @ 8 kHz, Q=2.020 | `kSet_TrebleBoostGainDB` |
| Limiter | Hard ceiling −1 dB, Herm Tr, lookahead 5 ms | `kSet_LimiterThresholdDb` |

---

### 🎮 Nahimic-Gaming-IEM  *(gaming / spatial awareness)*
**Chain:** `EQ(13 bands) → Stereo Tools → Limiter`

Nahimic sub-preset: **Atmospheric** — deep impactful bass with crisp highs.  
Voice Clarity boosted so footsteps and dialogue cut through.  
Stereo widened for improved positional cues (replaces HRTF virtual surround).

| Stage | What it does | Source |
|---|---|---|
| EQ band 0–9 | +3/+8/+6/−1/−1/0/0/+1/+2/+3 dB | `Atmospheric.json` |
| band10 | Bass Boost: +4 dB @ 250 Hz, Q=2.020 | `kSet_BassBoostGainDB` |
| band11 | Treble Enhancer: +3 dB @ 8 kHz, Q=2.020 | `kSet_TrebleBoostGainDB` |
| band12 | Voice Clarity: +6 dB @ 1414 Hz, Q=0.404 | `kSet_VoiceBoostGainDB` + `Generic_Headphones.nsx` |
| Stereo Tools | +3 dB side boost, LR→LR mode | `kSet_StereoWideningState` |
| Limiter | Hard ceiling −1 dB | `kSet_LimiterThresholdDb` |

---

### 🎬 Nahimic-Movie-IEM  *(movies / cinematic content)*
**Chain:** `EQ(12 bands) → Stereo Tools → Compressor → Limiter`

Nahimic sub-preset: **Immersion** — warm low-end with elevated dialogue clarity.  
Compressor tames dynamic range peaks for late-night listening.  
Stereo widened for cinematic soundstage.

| Stage | What it does | Source |
|---|---|---|
| EQ band 0–9 | +1/+4/+5/0/−2/−1/0/+3/+5/+3 dB | `Immersion.json` |
| band10 | Bass Boost: +4 dB @ 250 Hz, Q=2.020 | `kSet_BassBoostGainDB` |
| band11 | Voice Clarity: +6 dB @ 1414 Hz, Q=0.404 | `kSet_VoiceBoostGainDB` |
| Stereo Tools | +3 dB side boost | `kSet_StereoWideningState` |
| Compressor | 2:1 ratio, −36 dB threshold, 150 ms release | `kSet_Compressor1Rate/ThresholdDB/ReleaseSec` |
| Limiter | Hard ceiling −1 dB | `kSet_LimiterThresholdDb` |

*(TrebleBoost is OFF in Nahimic's Movie profile — omitted intentionally)*

---

### 📞 Nahimic-Communication-IEM  *(voice calls / meetings)*
**Chain:** `EQ(13 bands) → Compressor → Limiter`

Nahimic sub-preset: **Balanced** — aggressive sub-bass cut for call clarity,  
strong Voice Clarity lift for intelligibility, tight compressor for levelling.

| Stage | What it does | Source |
|---|---|---|
| EQ band 0–9 | −12/0/0/−3/−2/0/+2/+3/+1/0 dB | `Balanced.json` |
| band10 | Bass Boost: +2 dB @ 250 Hz, Q=2.020 | `kSet_BassBoostGainDB` |
| band11 | Treble Enhancer: +4 dB @ 8 kHz, Q=2.020 | `kSet_TrebleBoostGainDB` |
| band12 | Voice Clarity: +8 dB @ 1414 Hz, Q=0.404 | `kSet_VoiceBoostGainDB` |
| Compressor | 2:1 ratio, −36 dB threshold, 50 ms release | `kSet_Compressor2Rate/ThresholdDB/ReleaseSec` |
| Limiter | Hard ceiling −1 dB | `kSet_LimiterThresholdDb` |

---

## Extraction Methodology

All values come directly from binary-decoded Nahimic driver files — nothing estimated.

| File | What was extracted |
|---|---|
| `NH3ProductSettings0.cab` → NSX XML | Profile enable/disable flags, BassBoost gain, VoiceBoost gain, TrebleBoost gain, compressor state, stereo widening state |
| `Settings/v1/EQPresets/**/*.json` | All 17 named 10-band EQ sub-presets (exact dB values per band) |
| `nhamic2.reg` (8.5 MB) | Live registry: IEEE 754 floats — all kSet_ values per profile GUID, Compressor Band1/2 Rate/Release/Threshold, DRC, Limiter threshold |
| `Generic_Headphones.nsx` | BassBoost BandWidthOctave=0.707, VoiceBoost Low=500Hz/High=4000Hz |
| `NahimicAPO4API.dll` (strings) | Complete kSet_ parameter namespace, confirmed no hidden headphone DSP parameters |
| `Pages/AudioPage.xml` | Confirmed UI names: Bass Boost, Treble Enhancer, Voice Clarity, Smart Loudness, Virtual Surround |

### Computed Q Values (from Nahimic BandWidthOctave parameters)
```
Formula:  Q = sqrt(2^N) / (2^N - 1)   where N = bandwidth in octaves

10-band GEQ (N=1 oct):           Q = sqrt(2) / (2-1)         = 1.414214
Bass Boost / Treble (N=0.707 oct): Q = sqrt(1.631) / 0.631   = 2.020310
Voice Clarity (N=3 oct):          Q = sqrt(8) / (8-1)        = 0.404061
Voice Clarity centre freq:         fc = sqrt(500 × 4000)      = 1414.214 Hz
```

### Compressor Ratio (from Nahimic Rate parameter)
```
Nahimic Rate = slope of gain curve above threshold
Rate = 0.5  →  output rises 0.5 dB per 1 dB input above threshold
Compression ratio = 1 / (1 - Rate) = 1 / 0.5 = 2.0 : 1
```

---

## What is NOT Implemented

| Nahimic Feature | Why Not Implemented |
|---|---|
| **Virtual Surround (HRTF)** | Uses 5 proprietary A-Volute HRTF binaries (`Binaural_medium_LISTEN*.hex`). Format has 56-byte header + UTF-16LE metadata + custom binary HRIR data. Cannot decode without A-Volute SDK. EasyEffects Convolver could host it if format were known. |
| **Smart Loudness (DRC)** | `kSet_DRC*` — absent from `Generic_Headphones.nsx`, confirming it is a speaker-only feature. Correctly omitted. |
| **Device Optimization FIR** | 2048-tap FIR speaker correction (`kSet_DeviceOptimizationFilterFL/FR`). Speaker-only, absent from headphone NSX. |
| **Reverb** | References internal filter bank by ID with no extractable IR. |

---

## Unimplemented Sub-Presets

These are additional named EQ curves in the driver that were not built as standalone presets.  
You can manually add them by editing the EQ band gains:

**Music:** Brilliant (0/0/−1/−2/−2/0/+1/+4/+6/+2), Heavy Bass (+2/+7/+5/0/−1/0/0/0/+2/+1), Vocals (−3/−3/−2/+1/+3/+4/+5/+3/+3/+1)  
**Gaming:** FPS Competition (−10/−2/+6/+4/0/−1/+2/+3/+2/0), Fast-paced (−6/−1/+3/+1/0/0/+1/+5/+6/+5), Racing (+1/+6/+7/+1/0/0/0/0/+3/+6)  
**Movie:** Dialogues (−3/−3/−1/+1/+3/+4/+5/+3/+3/+1)  
**Communication:** Clear Chat (−12/−4/−2/+2/0/0/+4/+4/+5/0), Livestream (−12/+3/+5/+2/+3/−4/0/+4/+5/+6)

---

## License
MIT — use freely, give attribution if you share.  
Nahimic® is a trademark of A-Volute. This project is not affiliated with or endorsed by A-Volute or Lenovo.
