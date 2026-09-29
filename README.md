<div align="center">

<img src="assets/header.svg" width="100%" alt="Mohammed Zeeshan. I find out why systems fail quietly.">

<img src="assets/typing.svg" width="100%" alt="reading the logs before blaming the cloud">

<a href="https://www.linkedin.com/in/mohammed-zeeshan-75602632a"><img src="https://img.shields.io/badge/LinkedIn-Mohammed%20Zeeshan-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn"></a>
<a href="mailto:mohammedzeeshan.cse2024@citchennai.net"><img src="https://img.shields.io/badge/Email-CIT%20mail-EA4335?style=for-the-badge&logo=gmail&logoColor=white" alt="Email"></a>
<img src="https://img.shields.io/badge/Chennai-India-FF9933?style=for-the-badge&logo=googlemaps&logoColor=white" alt="Chennai, India">

</div>

<br>

Third-year CSE student at Chennai Institute of Technology, India.

Most of what I've built turned out to be about one problem: a system that looks fine while it's wrong. A seafarer's health band that said "all safe" because two of its three alarms could never fire. Drone telemetry that was encrypted and could still be replayed. Those are the bugs I like hunting.

<br>

## Case file 01 · the sensor that read zero

An ESP32 band for seafarers (ECG, motion, water contact, RFID crew ID) showed nothing useful for months, and the WiFi got the blame. The logs told a different story.

<img src="assets/case-oceanshield.svg" width="100%" alt="Investigation log: uploads were arriving every 30 seconds, but 991 records showed the sensors reading 0 V, two sensors correlated at +0.9992, the firmware divided a 12-bit ADC by 4, and two of three alarm rules could never fire">

The rewrite measures a real heart rate instead of sending one random ECG sample every 30 seconds:

<img src="assets/ecg-rpeak.svg" width="100%" alt="Animated ECG trace with an adaptive threshold, R-peaks marked, 800 ms RR intervals and a 75 BPM readout">

<br>

## Case file 02 · the packet that came back

AirLock, runner-up at HackSymmetric, protects drone-to-base telemetry. Encryption stops people from reading a packet. It doesn't stop them from recording one and sending it again. So every packet carries a unique ID, and the receiver remembers what it has already seen.

<img src="assets/airlock-flow.svg" width="100%" alt="Encrypted packets travel from sender to receiver to dashboard; an eavesdropper replays packet 41 and the receiver rejects it because the ID was already seen">

<details>
<summary><b>The same flow as a sequence diagram</b></summary>
<br>

```mermaid
sequenceDiagram
    autonumber
    participant A as Sender (Laptop A)
    participant E as Eavesdropper
    participant B as Receiver (Laptop B)
    participant C as Dashboard (Laptop C)
    A->>B: ECC key exchange, then an AES session key that rotates every 2 s
    A->>B: packet 41 with AES-GCM ciphertext, SHA-256 hash and msg_id
    E-->>E: quietly records packet 41
    B->>B: check structure, freshness and that msg_id is new
    B->>B: decrypt, verify, log to SQLite
    B->>C: REST API feeds the live map and packet stats
    E->>B: replays packet 41
    B->>B: msg_id 41 is already in the recent-packet cache
    B-->>C: alert, replay rejected
```

</details>

<br>

## Field notes

Things that each cost me an evening, so they don't have to cost you one.

| What bit me | What I do now |
|---|---|
| `analogRead(pin) / 4` on the ESP32's 12-bit ADC floored small signals to exactly 0 | Keep the raw 0 to 4095 counts and scale later |
| ADC2 pins (GPIO 0, 2, 4, 12-15, 25-27) stop reading analog values while WiFi is on | Analog sensors go on ADC1 (GPIO 32-39) |
| The RFID reader shared UART0 with the USB console, so crew IDs came through as 0 | Give it its own UART (Serial2 on GPIO 16/17) |
| The EM-18's TX line idles at 5 V and GPIO16 isn't 5 V tolerant | A 1k / 2k divider on that line |
| Calling `WiFi.begin()` inside the retry loop kept aborting the connection it had started | Call it once per attempt, then wait |
| An alarm rule that can never fire looks exactly like "everything is safe" | Replay every rule over the whole recorded history |
| Encryption alone doesn't stop someone resending a valid packet | Unique message IDs and a recent-packet cache |
| The free ThingSpeak tier drops writes that come less than 15 s apart | Upload every 30 s |

<br>

## Toolbox

<p align="center">
  <img src="assets/stack.svg" alt="Python, C++, Arduino, MATLAB, Flask, FastAPI, PostgreSQL, SQLite, Redis, MongoDB, Docker, Azure, Kotlin, Node.js, Express, React, TypeScript, Git">
</p>

<br>

## The contribution graph, eaten

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/z33shxn/z33shxn/output/snake-dark.svg">
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/z33shxn/z33shxn/output/snake.svg">
  <img alt="A snake eating the contribution graph" src="https://raw.githubusercontent.com/z33shxn/z33shxn/output/snake.svg">
</picture>

<img src="assets/footer.svg" width="100%" alt="If something fails quietly, I want to know why.">
