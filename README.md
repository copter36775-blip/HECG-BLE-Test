# HECG01 Clinical Console

Browser console for the **HECG01** ambulatory ECG recorder — Seeed XIAO nRF52840 Sense +
ADS1292R analog front end + microSD, over Bluetooth Low Energy.

### ▶ Open it: **https://copter36775-blip.github.io/HECG-BLE-Test/**

One self-contained HTML file. No build step, no dependencies, no server, no network
requests of any kind once the page has loaded — everything runs in the browser and talks
straight to the board over BLE.

---

## Browser support

Web Bluetooth is required, and it is not everywhere.

| Platform | Works | Notes |
|---|---|---|
| **Android — Chrome / Edge** | ✅ | The intended way to use this on a phone |
| **Windows / macOS / Linux — Chrome, Edge** | ✅ | |
| **iPhone / iPad — Safari** | ❌ | Apple does not implement Web Bluetooth in *any* iOS browser engine |
| **iPhone / iPad — Bluefy** | ✅ | A separate app from the App Store, with its own BLE-capable engine |
| Firefox (any platform) | ❌ | Web Bluetooth not implemented |

Web Bluetooth also requires a **secure context**, which is the reason this page is hosted
here rather than opened as a local file.

## Connecting

1. Power the recorder and wait for it to start advertising.
2. Open the page and press **CONNECT**.
3. Pick **`HECG01 Holter`** from the chooser.

On Android 12 and later, Chrome asks for **Nearby devices** permission the first time; on
older Android it asks for **Location**, which is what BLE scanning was gated behind then.
Either has to be allowed or the chooser stays empty.

**Do not pair the recorder in the operating system's Bluetooth settings.** The board
accepts one connection at a time, so an OS that has paired with it and auto-reconnects
will hold that connection and the browser will not be able to take it. Web Bluetooth does
not need pairing.

## เชื่อมจากมือถือ (สรุปสั้น)

1. **Android** — เปิดด้วย **Chrome** ได้เลย · **iPhone** ต้องใช้แอป **Bluefy** (Safari ใช้ไม่ได้)
2. เปิดลิงก์ด้านบน → กด **CONNECT** → เลือก **`HECG01 Holter`**
3. อนุญาต **Nearby devices** (Android 12+) หรือ **Location** (เก่ากว่านั้น)
4. **อย่า pair ใน Settings ของเครื่อง** — บอร์ดรับได้ทีละ 1 connection ถ้า OS ถือลิงก์ไว้
   เบราว์เซอร์จะแย่งไม่ได้

## What it does

- Live four-trace scope — raw and filtered ECG, raw and filtered respiration —
  with the raw and filtered traces time-aligned rather than merely overlaid
- Heart rate by Pan–Tompkins, R-wave amplitude, respiration rate, RR tachogram
- Rhythm card with the same thresholds as the offline analysis, checked strip by
  strip against it (`web02/tools/test_rhythm_rt.py`, 10 / 10)
- ADS1292R register inspector and lead-off status
- microSD browser: file list, preview, download, and the write-cadence card
  (per-write duration trace, buffer fill, data-at-risk)
- Recorder control — continuous, timed, and **Holter** (32 KiB bursts, 20 s flush)
- Power tab — sleep duty, discharge fit with its assumptions listed individually,
  and **pack characterisation**: derive this pack's own voltage → percent curve from
  one full discharge instead of reading percent off a generic LiPo table

## Firmware

This is **Console V5** (the design record is `web03/docs/UI_V5.md` in the main
repository): a status band in the header answering *is it recording, and is the
signal good?* with the alarm hierarchy lead-off > rhythm > signal quality >
information, every wave labelled with its scale (25 mm/s · 10 mm/mV), vitals that
say when they are stale, and dialogs you can drive from the keyboard.

This console speaks to firmware **V2.1.6**. Older firmware still connects; features
added later degrade to blank tiles rather than wrong numbers.

V2.1.6 writes the battery voltage (start, end, lowest) and the board temperature of
each recording into its footer, and the file inspector shows them together with the
stop reason and the measured sample rate - so a recording that ended on LOW BATTERY
says at what voltage without taking the card out.

Raw traces show input-referred millivolts on the left axis and the ADC code on the
right, and the readout gives both (`540,300 = 25.9782 mV`); the chart header states
the LSB, 48.08 nV at the 2.42 V reference and gain 6. Respiration has its own window: a quarter of the ECG speed by
default, the 6.25 mm/s bedside monitors use, because a breath is 20-70 µV and takes
3-10 s - a 5-second ECG window never showed more than one of them.

V2.1.4 stamps each block at the moment the samples were acquired rather than when
they reached the card, measures the real sample rate on the device and reports it
in the `SYS` message. The console shows it next to the nominal rate, and works out
its own decoded rate over a 60-second regression instead of counting packets per
second - the arrival count follows the radio's connection interval, not the ADC.
The ADS1292R runs from its internal oscillator, measured at -180 to -1600 ppm
against the MCU crystal and inside the datasheet's tolerance, so the difference is
real and worth showing.

## Editing

`index.html` here is a published copy. It is generated from `web02/hecg_console.html`
in the HECG01 project, which is where changes belong — the test suite
(smoke, UI contract, accessibility, write cadence, battery trend, pack curve) runs
against that file, not this one.
