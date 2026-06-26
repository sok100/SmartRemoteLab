Da. Eu aș aborda proiectul metodic, astfel încât fiecare etapă să producă un rezultat reutilizabil și documentabil în GitHub. Mai jos este planul pe care l-aș urma eu.

---

# Phase 1 - Verify the BLE Connection

## 1. Install the required tools

Ubuntu / Linux Mint:

```bash
sudo apt update

sudo apt install \
bluez \
bluez-tools \
blueman \
btmon \
libinput-tools \
evtest \
python3-pip \
git
```

Install the Python packages:

```bash
pip install bleak
pip install pydbus
pip install construct
```

---

## 2. Check the Bluetooth adapter

```bash
hciconfig
```

or

```bash
bluetoothctl list
```

Expected output:

```text
Controller XX:XX:XX:XX:XX:XX
```

---

## 3. Enable Bluetooth

```bash
bluetoothctl
```

Inside the shell:

```text
power on

agent on

default-agent

scan on
```

---

# Phase 2 - Pair the Samsung Remote

Start pairing mode on the remote.

Usually:

```
Back + Play/Pause
```

Now in bluetoothctl:

```text
scan on
```

Find:

```
Smart Control 2016
```

Then

```text
pair MAC

trust MAC

connect MAC
```

Example

```text
pair 2C:99:75:94:7A:4C

trust 2C:99:75:94:7A:4C

connect 2C:99:75:94:7A:4C
```

---

# Phase 3 - Dump GATT Services

Install

```bash
sudo apt install bluez
```

Then

```bash
bluetoothctl
```

Inside:

```
menu gatt
```

Then

```
list-attributes
```

Save everything.

Example

```
docs/gatt/TM2360E.txt
```

---

# Phase 4 - Record Bluetooth Traffic

This is probably the most important step.

Open Terminal 1

```bash
sudo btmon
```

Save the output

```bash
sudo btmon | tee capture.txt
```

Leave it running.

---

Open Terminal 2

Reconnect

```bash
bluetoothctl
```

```
connect 2C:99:75:94:7A:4C
```

Now press

```
Volume+

Volume-

Mute

Home

Back

Settings

123

Voice

Channel+

Channel-

```

Stop btmon.

You now have

```
capture.txt
```

---

# Phase 5 - See if Linux Already Receives Events

Sometimes Samsung already exposes Linux events.

Run

```bash
cat /proc/bus/input/devices
```

Look for

```
Samsung
```

or

```
Bluetooth HID
```

---

Run

```bash
sudo evtest
```

If you see

```
Samsung Smart Remote
```

select it.

Now press buttons.

Example

```
KEY_VOLUMEUP

KEY_VOLUMEDOWN

KEY_BACK
```

If events appear:

Great.

No decoder needed.

---

Also try

```bash
sudo libinput debug-events
```

---

# Phase 6 - Inspect HID Reports

Install

```bash
sudo apt install hid-tools
```

Then

```bash
hid-recorder
```

or

```bash
hid-decode
```

If supported.

---

# Phase 7 - Read BLE Characteristics

Install

```bash
pip install bleak
```

Create

```
tools/read_characteristics.py
```

The program should

* connect
* enumerate services
* enumerate characteristics
* print UUIDs
* print properties
* read values

Save output as

```
docs/services.txt
```

---

# Phase 8 - Identify Notification Handles

We need to know

Which characteristic sends notifications.

Usually

```
Notify

Indicate
```

Example

```
Handle 0x0027

Notify

UUID ...

```

---

# Phase 9 - Subscribe to Notifications

Using Bleak

```python
await client.start_notify(...)
```

Then

```
Volume+

↓

03 01 55

Volume-

↓

03 01 56

Home

↓

02 09 14
```

Now the mapping starts.

---

# Phase 10 - Build the Mapping Table

Example

| Button  | Hex Report |
| ------- | ---------- |
| Volume+ | 03 01 55   |
| Volume- | 03 01 56   |
| Home    | 02 09 14   |
| Back    | 02 09 15   |
| OK      | 01 07 22   |

---

# Phase 11 - Export JSON

```json
{
  "remote":"TM2360E",
  "buttons":[
    {
      "name":"VolumeUp",
      "report":"030155"
    },
    {
      "name":"VolumeDown",
      "report":"030156"
    }
  ]
}
```

---

# Phase 12 - Create a Virtual Keyboard

Using

```
python-uinput
```

or

```
evdev
```

Example

```
Volume+

↓

BLE Report

↓

Python

↓

KEY_VOLUMEUP

↓

Linux
```

No kernel module needed.

---

# Phase 13 - Android

Once Linux decoding works, Android becomes much easier.

Instead of

```
BLE

↓

Raw packets
```

you will already have

```
BLE

↓

Decoded Button

↓

Android Accessibility Service

↓

Perform Action
```

---

# Suggested GitHub Milestones

## Milestone 1

* [ ] Scan BLE devices
* [ ] Pair Samsung Remote
* [ ] Dump GATT
* [ ] Read Characteristics

---

## Milestone 2

* [ ] Capture btmon logs
* [ ] Capture notifications
* [ ] Decode packets

---

## Milestone 3

* [ ] Learn Mode
* [ ] JSON export
* [ ] Button database

---

## Milestone 4

* [ ] Linux daemon
* [ ] Virtual keyboard
* [ ] Virtual mouse

---

## Milestone 5

* [ ] Android companion
* [ ] Windows support
* [ ] Cross-platform SDK

## Recomandarea mea

Înainte să scriem orice linie de cod, aș face un **"baseline" complet** al telecomenzii. Adică să colectăm și să salvăm în repository toate informațiile brute:

1. `bluetoothctl info <MAC>`
2. `list-attributes` (GATT complet)
3. ieșirea `btmon` în timpul împerecherii
4. ieșirea `btmon` în timp ce apeși fiecare buton
5. descriptorul HID (foarte important)
6. verificarea dacă apare vreun dispozitiv `/dev/input/event*`

Acest set de date va deveni referința pentru modelul TM2360E și va face mult mai ușoară dezvoltarea și compararea cu alte modele Samsung în viitor.
