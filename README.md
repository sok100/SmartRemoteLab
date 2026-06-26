# SmartRemoteLab

Open Source toolkit for exploring, decoding and integrating Samsung Smart Remote Bluetooth Low Energy (BLE) remotes.

The project focuses on understanding Samsung Smart Remote communication over Bluetooth LE and making it usable from Linux, Android, Windows and embedded platforms.

The toolkit provides utilities for:

- discovering BLE services
- decoding GATT notifications
- parsing HID reports
- mapping buttons
- recording traffic
- replaying commands
- emulating keyboard and mouse
- documenting Samsung proprietary services

The project does NOT modify or replace the firmware inside the remote control.
SmartRemoteLab/

# Goals

✔ Scan Samsung Smart Remotes

✔ Dump GATT services

✔ Decode HID reports

✔ Record button presses

✔ Learn unknown buttons

✔ Build a protocol database

✔ Emulate keyboard

✔ Emulate mouse

✔ Android companion application

✔ Linux daemon

✔ Wireshark integration

# Non Goals

This project does not:

- modify Samsung firmware
- bypass pairing security
- attack Samsung TVs
- crack encrypted firmware

The purpose of the project is interoperability and protocol documentation.


│
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── CHANGELOG.md
├── ROADMAP.md
├── SECURITY.md
│
├── docs/
│   ├── architecture.md
│   ├── protocol.md
│   ├── pairing.md
│   ├── hid-report.md
│   ├── android.md
│   ├── linux.md
│   ├── windows.md
│   ├── sniffing.md
│   ├── gatt.md
│   └── reverse-notes.md
│
├── captures/
│   ├── btmon/
│   ├── wireshark/
│   ├── hid/
│   └── pairing/
│
├── examples/
│   ├── bleak/
│   ├── python/
│   ├── android/
│   └── linux/
│
├── tools/
│   ├── scanner.py
│   ├── logger.py
│   ├── recorder.py
│   ├── decoder.py
│   ├── replay.py
│   ├── monitor.py
│   └── hiddump.py
│
├── daemon/
│   ├── linux/
│   ├── android/
│   └── windows/
│
├── protocol/
│   ├── reports.py
│   ├── samsung.py
│   ├── parser.py
│   ├── hid.py
│   └── constants.py
│
├── tests/
│
└── images/



# Learn Mode

One of the core features planned for SmartRemoteLab is Learn Mode.

Instead of manually analyzing Bluetooth packets, users will simply run:

srl learn

The application will then guide the user through an interactive button discovery process.

Example:

Press Volume+

✔ Report received:
03 14 2A

Mapped to: Volume Up

--------------------------------

Press Volume-

✔ Report received:
03 14 2B

Mapped to: Volume Down

--------------------------------

Press Home

✔ Report received:
02 05 8F

Mapped to: Home

After all buttons have been recorded, SmartRemoteLab automatically generates a device profile such as:

database/TM2360E.json

The generated profile contains the Bluetooth reports associated with every button and can be reused without repeating the learning process.

The goal is to build a community-driven database covering as many Samsung Smart Remote models as possible.

