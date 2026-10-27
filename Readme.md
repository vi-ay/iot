<a id="top"></a>

# 🍓 IoT Practicals – Raspberry Pi (Important Questions)

Quick-revision README: **wiring → setup → code → run** for every important question.
All code below is taken as-is from `IOT.zip` and `confirm.zip`. Pin numbers are **physical (BOARD) pin numbers** unless stated otherwise.

## 📑 Index

**Reference**
- [Pin reference & common setup](#common)

**Module 1**
1. [Q1 – LED blinking patterns](#q1)
2. [Q2 – Telegram control of GPIO](#q2)
3. [Q3 – TM1637 digital clock](#q3)
4. [Q5 – ADS1115 as oscilloscope](#q5)
5. [Q6 – 16x2 LCD display text](#q6)
6. [Q7 – RFID read / write](#q7)
7. [Q8 – Fingerprint sensor](#q8)
8. [Q9 – Pi Camera image & video](#q9)

**Module 2**
9. [Q11 – Fingerprint + LCD + RFID UID matched / unmatched](#q11)

10. [Q12 – Door lock (fingerprint + relay)](#q12)

11. [Q13 – MQTT publisher / subscriber controlling LED](#q13)
12. [Q14 – Flask web dashboard controlling relay](#q14)
13. [Q15 – Two-factor authentication (RFID + fingerprint + relay)](#q15)
14. [Q16 – Motion detector (camera + LCD)](#q16)

---

<a id="common"></a>
## 🔧 Pin reference & common setup

| Type | Physical pins |
|---|---|
| 3.3V | 1, 17 |
| 5V | 2, 4 |
| GND | 6, 9, 14, 20, 25, 30, 34, 39 |
| UART TXD / RXD | 8 / 10 |
| I2C SDA / SCL | 3 / 5 |
| SPI MOSI / MISO / SCLK / CE0 | 19 / 21 / 23 / 24 |

**Serial setup (Fingerprint – Q8, Q11, Q12, Q15):**
```bash
sudo apt update
sudo apt install -y python3-pip python3-serial
sudo pip3 install --break-system-packages adafruit-circuitpython-fingerprint
sudo usermod -a -G dialout $USER
sudo raspi-config
```
`raspi-config` → **Interface Options → Serial Port**
- "Login shell accessible over serial?" → **No**
- "Serial port hardware enabled?" → **Yes**

```bash
sudo nano /boot/firmware/config.txt      # if it opens empty, Ctrl+X and use /boot/config.txt
```
Add at the bottom: `dtoverlay=disable-bt` → save (`Ctrl+O`, `Enter`, `Ctrl+X`) →
```bash
sudo reboot
```

**SPI setup (RFID – Q7, Q11, Q15):** `sudo raspi-config` → Interface Options → **SPI → Yes** → reboot → `lsmod | grep spi`.

<p align="right"><a href="#top">⬆ Back to top</a></p>

---

<a id="q1"></a>
## Q1. Create different blinking patterns with LEDs using GPIO pins

**Wiring**

| LED | Pi pin |
|---|---|
| LED1 | 7 |
| LED2 | 11 |
| LED3 | 15 |
| LED4 | 40 |
| Common GND | 39 |

**Code** – `Prac1.py`
```python
import RPi.GPIO as GPIO

import time

GPIO.setmode(GPIO.BOARD)
GPIO.setup(7,GPIO.OUT)
GPIO.setup(11,GPIO.OUT)
GPIO.setup(15,GPIO.OUT)
GPIO.setup(40,GPIO.OUT)

for i in range(5):
    
    GPIO.output(7,True)
    time.sleep(1)

    GPIO.output(11,False)
    #time.sleep(1)

    GPIO.output(15,True)
    #    time.sleep(1)

    GPIO.output(40,False)
        #time.sleep(1)

    print("ON")
        #time.sleep(1)
    
    GPIO.output(7,False)
        #time.sleep(1)
    GPIO.output(11,True)
        #time.sleep(1)
    GPIO.output(15,False)
        #time.sleep(1)
    GPIO.output(40,True)
        #time.sleep(1)

print("Done")
GPIO.cleanup()
```

**Run:** `python3 Prac1.py`

<p align="right"><a href="#top">⬆ Back to top</a></p>

---

<a id="q2"></a>
## Q2. Use Telegram to control Raspberry Pi GPIO

**Wiring:** LED → **Pin 11** (with resistor to GND).

**Create the bot:** Telegram → **@BotFather** → `/newbot` → copy the **token**.

**Setup**
```bash
sudo apt update
sudo pip3 install telepot --break-system-packages
```

**Code** – `Prac2.py`  (bot tokens in the zip file are replaced with `YOUR_BOT_TOKEN` here – paste your own)
```python
import sys
import time
import random
import datetime
import telepot
import RPi.GPIO as GPIO
from telepot.loop import MessageLoop

red=11

now=datetime.datetime.now()
GPIO.setmode(GPIO.BOARD)
GPIO.setwarnings(False)

GPIO.setup(red,GPIO.OUT)
GPIO.output(red,0)


def action(msg):
    chat_id = msg['chat']['id']
    command = msg['text']

    print ('Got command: %s' %command)
    if 'On' in command:
        message="Turn On"
        message=message+" red"
        GPIO.output(red,1)
        bot.sendMessage(chat_id,message)

    if 'Off' in command:
        message="Turn Off"
        message=message+" red"
        GPIO.output(red,0)
        bot.sendMessage(chat_id, message)

##bot = telepot.Bot('YOUR_BOT_TOKEN')
##bot = telepot.Bot('YOUR_BOT_TOKEN')
##bot=telepot.Bot('YOUR_BOT_TOKEN')
bot = telepot.Bot('YOUR_BOT_TOKEN')
print(bot.getMe())
MessageLoop(bot,action).run_as_thread()
print ('I am listening...')

while 1:
     time.sleep(10)
```

**Run:** `python3 Prac2.py`, then send `On` / `Off` to your bot in Telegram.

<p align="right"><a href="#top">⬆ Back to top</a></p>

---

<a id="q3"></a>
## Q3. Interface TM1637 with Raspberry Pi and display current time

**Wiring**

| TM1637 | Pi pin |
|---|---|
| GND | 39 |
| VCC | 4 |
| DIO | 18 |
| CLK | 16 |

**Setup** – library file must be in the same folder (also present in `IOT.zip` as `tm1637.py`):
```bash
wget https://raspberrytips.nl/files/tm1637.py
```

**Code** – `Prac3.py`
```python
import tm1637
import sys
import time
import datetime
import RPi.GPIO as GPIO

Display= tm1637.TM1637(23,24,tm1637.BRIGHT_HIGHEST)
Display.Clear()
Display.SetBrightnes(1)
while(True):
    now = datetime.datetime.now()
    hour = now.hour
    minute=now.minute
    second=now.second
    currenttime=[int(hour/10),hour%10,int(minute/10),minute%10 ]
    Display.Show(currenttime)
    Display.ShowDoublepoint(second%2)
    time.sleep(1)
  
```

**Run:** `python3 Prac3.py`

<p align="right"><a href="#top">⬆ Back to top</a></p>

---

<a id="q5"></a>
## Q5. Interface ADS1115 with Raspberry Pi to act as oscilloscope

**Wiring** *(not in your notes – standard ADS1115 connections, verify with your module)*

| ADS1115 | Pi pin |
|---|---|
| VDD | 1 (3.3V) |
| GND | 6 |
| SCL | 5 |
| SDA | 3 |
| A0 | Signal input |

**Setup:** enable I2C in `raspi-config`, reboot, then
```bash
sudo pip3 install Adafruit_ADS1x15 matplotlib --break-system-packages
```

**Code** – `Prac6.py`
```python
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation
import Adafruit_ADS1x15
# Create an ADS1115 ADC (16-bit) instance.
adc = Adafruit_ADS1x15.ADS1115()
GAIN = 1
val = [ ]

# Start continuous ADC conversions on channel 0 using the previous gain value.
adc.start_adc(0, gain=GAIN)
print('Reading ADS1x15 channel 0')
fig, ax = plt.subplots()
ax.set_ylim(-5000,5000)
ax.set_title('Oscilloscope')
ax.grid(True)
ax.set_ylabel('ADC outputs')
line, = ax.plot([], 'ro-', label='Channel 0')
ax.legend(loc='lower right')
def update(cnt):
    value = adc.get_last_result()
    print('Channel 0: {0}'.format(value))
    line.set_data(list(range(len(val))), val)
    ax.relim()
    ax.autoscale_view()
    val.append(int(value))
if(cnt>50):
    val.pop(0)
    ani = FuncAnimation(fig, update, interval=500)
    plt.show()
```

**Run:** `python3 Prac6.py`

> ⚠️ As it appears in the zip, the lines from `if(cnt>50):` onwards are not indented inside `update()`. If you get a `NameError: name 'cnt' is not defined`, move `if(cnt>50): val.pop(0)` inside `update()` and put the last two lines (`ani = ...`, `plt.show()`) at the left margin.

<p align="right"><a href="#top">⬆ Back to top</a></p>

---

<a id="q6"></a>
## Q6. Interface 16x2 LCD with Raspberry Pi to display text

**Wiring for the code below (uses BCM numbers → physical pins)**

| LCD | Pi pin |
|---|---|
| VSS | GND |
| VDD | 5V |
| V0 | Resistor → GND |
| RS | Pin 32 (BCM 12) |
| RW | GND |
| E | Pin 26 (BCM 7) |
| D4 | Pin 33 (BCM 13) |
| D5 | Pin 35 (BCM 19) |
| D6 | Pin 37 (BCM 26) |
| D7 | Pin 40 (BCM 21) |
| LED+ (BLA) | 5V |
| LED− (BLK) | GND |

**I2C module wiring (from your notes)**

| Module | Pi pin |
|---|---|
| VSS | GND |
| VCC | 5V |
| V0 | Potentiometer data |
| Potentiometer (+) / (−) | 5V / GND |
| SDA | Pin 3 |
| SCL | Pin 5 |

**Setup**
```bash
cd /home/pi/Adafruit_Python_CharLCD-master
sudo python3 setup.py install
```

**Code** – `Prac7.py`
```python
import Adafruit_CharLCD as LCD
import RPi.GPIO as GPIO

GPIO.setmode(GPIO.BCM)

lcd1 = 12
lcd2 = 7
lcd3 = 13
lcd4 = 19
lcd5 = 26
lcd6 = 21

lcd = LCD.Adafruit_CharLCD(lcd1, lcd2, lcd3, lcd4, lcd5, lcd6, 0, 16, 2)

lcd.message("Soujas mane")
print("printed")
```

**Run:** `python3 Prac7.py`

> The zip's `Prac7.py` is the parallel-wiring (Adafruit_CharLCD) version; there is no I2C LCD program in either zip.

<p align="right"><a href="#top">⬆ Back to top</a></p>

---

<a id="q7"></a>
## Q7. Interface RFID module with Raspberry Pi to read and write tags/cards

**Wiring (RC522)**

| RC522 | Pi pin |
|---|---|
| SDA | 24 |
| SCK | 23 |
| MOSI | 19 |
| MISO | 21 |
| IRQ | Unused |
| GND | 6 |
| RST | 22 |
| 3.3V | 1 |

**Setup**
```bash
sudo apt-get update
sudo raspi-config                  # Interfacing Options → SPI → Enable, then reboot
lsmod | grep spi
sudo apt-get install python3-dev python3-pip
sudo python3 -m pip install spidev --break-system-packages
sudo python3 -m pip install mfrc522 --break-system-packages
```

**Write** – `Prac4.1.py`
```python
import RPi.GPIO as GPIO
from mfrc522 import SimpleMFRC522

reader = SimpleMFRC522()

try:
    text = input('New data:')
    print("Now place your tag to write")
    reader.write(text)
    print("Written")
finally:
    GPIO.cleanup()
```

**Read** – `Prac4.2.py`
```python
import RPi.GPIO as GPIO
from mfrc522 import SimpleMFRC522

reader = SimpleMFRC522()

try:
    id, text = reader.read()
    print(id)
    print(text)
finally:
    GPIO.cleanup()
      
```

**Run:** `python3 Prac4.1.py` (write) then `python3 Prac4.2.py` (read).

<p align="right"><a href="#top">⬆ Back to top</a></p>

---

<a id="q8"></a>
## Q8. Interface Fingerprint Sensor with Raspberry Pi

**Wiring**

| Sensor wire | Pi pin |
|---|---|
| Red | 3.3V |
| Black | GND |
| Green | Pin 8 (TXD) |
| Yellow | Pin 10 (RXD) |

**Setup:** follow the [serial setup](#common) and reboot.

**Code** – `Prac8.py`
```python
import serial
import adafruit_fingerprint

uart = serial.Serial('/dev/serial0', 57600, timeout=1)
f = adafruit_fingerprint.Adafruit_Fingerprint(uart)


def get_image():
    while f.get_image() != adafruit_fingerprint.OK:
        pass


def enroll(slot):
    print("Place finger...")
    get_image()

    if f.image_2_tz(1) != adafruit_fingerprint.OK:
        print("Failed to convert fingerprint image.")
        return

    print("Remove finger")
    while f.get_image() != adafruit_fingerprint.NOFINGER:
        pass

    print("Place same finger again...")
    get_image()

    if f.image_2_tz(2) != adafruit_fingerprint.OK:
        print("Failed to convert fingerprint image.")
        return

    if f.create_model() != adafruit_fingerprint.OK:
        print("Fingers did not match")
        return

    if f.store_model(slot) != adafruit_fingerprint.OK:
        print("Failed to store fingerprint")
        return

    print("Enrolled at slot", slot)


def verify():
    print("Place finger...")
    get_image()

    if f.image_2_tz(1) != adafruit_fingerprint.OK:
        print("Failed to convert fingerprint image.")
        return

    if f.finger_search() == adafruit_fingerprint.OK:
        print(
            "RIGHT FINGER - id",
            f.finger_id,
            "confidence",
            f.confidence
        )
    else:
        print("WRONG FINGER")


while True:
    c = input("\n1 enroll 2 verify 3 quit: ")

    if c == "1":
        slot = int(input("slot number: "))
        enroll(slot)

    elif c == "2":
        verify()

    elif c == "3":
        break

    else:
        print("Invalid choice")
```

**Run:** `python3 Prac8.py` → **1** enroll (slot number) → **2** verify.

<p align="right"><a href="#top">⬆ Back to top</a></p>

---

<a id="q9"></a>
## Q9. Interface Pi Camera to capture image and video

**Setup**
```bash
sudo apt update
sudo reboot
sudo apt update
sudo apt install -y python3-picamera2
sudo apt install -y python3-libcamera python3-kms++
```

**Capture image** – `Prac5.1.py`
```python
from picamera2 import Picamera2
from time import sleep
picam2 = Picamera2() # Initialize camera
config = picam2.create_still_configuration() # Configure camera for still capture
picam2.configure(config)
picam2.start()
sleep(2) # Let camera adjust exposure
picam2.capture_file("image2.jpg")
picam2.stop()
print("Image captured and saved as image.jpg")
```

**Record video** – `Prac5.2.py`
```python
from picamera2 import Picamera2
from picamera2.encoders import H264Encoder
from picamera2.outputs import FileOutput
from time import sleep

picam2 = Picamera2()

video_config = picam2.create_video_configuration()  # Configure camera for video
picam2.configure(video_config)

encoder = H264Encoder()
output = FileOutput("video.h264")

picam2.start()

print("Starting recording...")
picam2.start_recording(encoder, output)

sleep(20)

picam2.stop_recording()
picam2.stop()

print("Recording completed.")
```

**Run:** `python3 Prac5.1.py` / `python3 Prac5.2.py`

<p align="right"><a href="#top">⬆ Back to top</a></p>

---

<a id="q11"></a>
## Q11. Fingerprint Sensor + 16x2 LCD: display "RFID UID matched / unmatched"

**Wiring**

*Fingerprint*

| Wire | Pi pin |
|---|---|
| Red | 3.3V |
| GND | GND |
| Yellow | Pin 10 |
| Green | Pin 8 |

*RFID (RC522)*

| RC522 | Pi pin |
|---|---|
| SDA | 24 |
| SCK | 23 |
| MOSI | 19 |
| MISO | 21 |
| IRQ | Unused |
| GND | GND |
| RST | 22 |
| 3.3V | 17 |

*LCD*

| LCD pin | Raspberry Pi |
|---|---|
| VSS | GND |
| VDD | 5V |
| V0 | Resistor → GND (contrast) |
| RS | Pin 29 |
| RW | GND |
| E | Pin 31 |
| D4 | Pin 33 |
| D5 | Pin 35 |
| D6 | Pin 37 |
| D7 | Pin 40 |
| LED+ (BLA) | 5V |
| LED− (BLK) | GND |

**Setup**
```bash
sudo apt update
sudo apt install -y python3-pip python3-serial
sudo pip3 install --break-system-packages adafruit-circuitpython-fingerprint mfrc522 RPLCD
sudo usermod -a -G dialout $USER
```
Then the [serial setup + SPI enable](#common) and reboot.

**Code** – `p11.py` (shows **UID MATCHED / UID UNMATCHED** using `ALLOWED_TAGS`)
```python
import serial
import adafruit_fingerprint
import RPi.GPIO as GPIO
from RPLCD.gpio import CharLCD
from mfrc522 import SimpleMFRC522
from time import sleep

ALLOWED_TAGS = [695924236005]

GPIO.setwarnings(False)

lcd = CharLCD(pin_rs=29, pin_e=31, pins_data=[33, 35, 37, 40],
              numbering_mode=GPIO.BOARD, cols=16, rows=2, dotsize=8)

uart = serial.Serial('/dev/serial0', 57600, timeout=1)
f = adafruit_fingerprint.Adafruit_Fingerprint(uart)

reader = SimpleMFRC522()


def show(line1, line2=""):
    lcd.clear()
    lcd.write_string(line1[:16])
    if line2:
        lcd.cursor_pos = (1, 0)
        lcd.write_string(line2[:16])


def get_image():
    while f.get_image() != adafruit_fingerprint.OK:
        pass


def enroll(slot):
    show("Place finger")
    print("Place finger...")
    get_image()
    f.image_2_tz(1)
    show("Remove finger")
    print("Remove finger")
    while f.get_image() != adafruit_fingerprint.NOFINGER:
        pass
    show("Place again")
    print("Place same finger again...")
    get_image()
    f.image_2_tz(2)
    if f.create_model() != adafruit_fingerprint.OK:
        show("Did not match", "Try again")
        print("Fingers did not match")
        sleep(2)
        return
    f.store_model(slot)
    show("Enrolled", "Slot " + str(slot))
    print("Enrolled at slot", slot)
    sleep(2)


def scan_finger():
    show("Place finger")
    print("Place finger...")
    get_image()
    f.image_2_tz(1)
    if f.finger_search() == adafruit_fingerprint.OK:
        show("FINGER MATCHED", "ID " + str(f.finger_id))
        print("FINGER MATCHED - id", f.finger_id)
    else:
        show("FINGER", "UNMATCHED")
        print("FINGER UNMATCHED")
    sleep(3)


def scan_rfid():
    show("Scan RFID card")
    print("Scan RFID card...")
    uid, text = reader.read()
    print("UID:", uid)
    if uid in ALLOWED_TAGS:
        show("UID MATCHED", str(uid))
        print("UID MATCHED")
    else:
        show("UID UNMATCHED", str(uid))
        print("UID UNMATCHED")
    sleep(3)


try:
    show("System ready")
    sleep(2)
    while True:
        c = input("\n1 enroll finger  2 scan finger  3 scan RFID  4 quit : ")
        if c == "1":
            enroll(int(input("slot number: ")))
        elif c == "2":
            scan_finger()
        elif c == "3":
            scan_rfid()
        elif c == "4":
            break
        show("System ready")
finally:
    lcd.clear()
    lcd.close(clear=True)
    GPIO.cleanup()
```

**Run:** `python3 p11.py` – menu: 1 enroll finger, 2 scan finger, 3 scan RFID, 4 quit.

> `ALLOWED_TAGS` holds the UID of your card. Replace it with your own card's UID if you use a different card.

<details>
<summary>Variant: <code>Prac11.py</code> (accepts any RFID tag, always shows "RFID MATCHED")</summary>

```python
import serial
import adafruit_fingerprint
import RPi.GPIO as GPIO
from RPLCD.gpio import CharLCD
from mfrc522 import SimpleMFRC522
from time import sleep

GPIO.setwarnings(False)

# LCD setup
lcd = CharLCD(
    pin_rs=29,
    pin_e=31,
    pins_data=[33, 35, 37, 40],
    numbering_mode=GPIO.BOARD,
    cols=16,
    rows=2,
    dotsize=8
)

# Fingerprint sensor setup
uart = serial.Serial('/dev/serial0', 57600, timeout=1)
f = adafruit_fingerprint.Adafruit_Fingerprint(uart)

# RFID reader setup
reader = SimpleMFRC522()


def show(line1, line2=""):
    lcd.clear()
    lcd.write_string(line1[:16])

    if line2:
        lcd.cursor_pos = (1, 0)
        lcd.write_string(line2[:16])


def get_image():
    while f.get_image() != adafruit_fingerprint.OK:
        pass


def enroll(slot):
    show("Place finger")
    print("Place finger...")

    get_image()
    f.image_2_tz(1)

    show("Remove finger")
    print("Remove finger")

    while f.get_image() != adafruit_fingerprint.NOFINGER:
        pass

    show("Place again")
    print("Place same finger again...")

    get_image()
    f.image_2_tz(2)

    if f.create_model() != adafruit_fingerprint.OK:
        show("Did not match", "Try again")
        print("Fingers did not match")
        
        return

    f.store_model(slot)

    show("Enrolled", "Slot " + str(slot))
    print("Enrolled at slot", slot)
    


def scan_finger():
    show("Place finger")
    print("Place finger...")

    get_image()
    f.image_2_tz(1)

    if f.finger_search() == adafruit_fingerprint.OK:
        show("FINGER MATCHED", "ID " + str(f.finger_id))
        print("FINGER MATCHED - ID", f.finger_id)
    else:
        show("FINGER", "UNMATCHED")
        print("FINGER UNMATCHED")

    


def scan_rfid():
    show("Scan RFID card")
    print("Scan RFID card...")

    uid, text = reader.read()

    print("UID:", uid)

    # Accept ANY RFID tag
    show("RFID MATCHED", str(uid))
    
    print("RFID ACCEPTED - UID:", uid)

    


try:
    show("System ready")

    while True:
        c = input(
            "\n1 enroll finger\n"
            "2 scan finger\n"
            "3 scan RFID\n"
            "4 quit\n"
            "Enter choice: "
        )

        if c == "1":
            slot = int(input("Slot number: "))
            enroll(slot)

        elif c == "2":
            scan_finger()

        elif c == "3":
            scan_rfid()

        elif c == "4":
            break

        else:
            print("Invalid choice")

        show("System ready")

finally:
    lcd.clear()
    lcd.close(clear=True)
    GPIO.cleanup()
```

</details>

<p align="right"><a href="#top">⬆ Back to top</a></p>

---

<a id="q12"></a>
## Q12. Simulate door lock with fingerprint sensor and relay module

**Wiring**

*Relay*

| Relay | Pi pin |
|---|---|
| IN | 11 |
| VCC | 17 |
| GND | GND |

*Fingerprint*

| Wire | Pi pin |
|---|---|
| Red | Pin 1 |
| GND | GND |
| Yellow | Pin 10 |
| Green | Pin 8 |

**Setup**
```bash
sudo apt update
sudo apt install -y python3-pip python3-serial
sudo pip3 install --break-system-packages adafruit-circuitpython-fingerprint
sudo usermod -a -G dialout $USER
```
Then the [serial setup](#common) and reboot.

**Code** – `m2p12.py`
```python
import serial
import adafruit_fingerprint
import RPi.GPIO as GPIO
from time import sleep

PIN = 11
UNLOCK_TIME = 5

GPIO.setmode(GPIO.BOARD)
GPIO.setup(PIN, GPIO.OUT)
GPIO.output(PIN, GPIO.HIGH)

uart = serial.Serial('/dev/serial0', 57600, timeout=1)
f = adafruit_fingerprint.Adafruit_Fingerprint(uart)


def get_image():
    while f.get_image() != adafruit_fingerprint.OK:
        pass


def enroll(slot):
    print("Place finger...")
    get_image()
    f.image_2_tz(1)
    print("Remove finger")
    while f.get_image() != adafruit_fingerprint.NOFINGER:
        pass
    print("Place same finger again...")
    get_image()
    f.image_2_tz(2)
    if f.create_model() != adafruit_fingerprint.OK:
        print("Fingers did not match")
        return
    f.store_model(slot)
    print("Enrolled at slot", slot)


def unlock():
    print("ACCESS GRANTED - door unlocked")
    GPIO.output(PIN, GPIO.LOW)
    sleep(UNLOCK_TIME)
    GPIO.output(PIN, GPIO.HIGH)
    print("Door locked again")


def scan():
    print("Place finger on sensor...")
    get_image()
    f.image_2_tz(1)
    if f.finger_search() == adafruit_fingerprint.OK:
        print("Matched id", f.finger_id, "confidence", f.confidence)
        unlock()
    else:
        print("ACCESS DENIED - unknown finger")


try:
    while True:
        c = input("\n1 enroll  2 scan  3 quit : ")
        if c == "1":
            enroll(int(input("slot number: ")))
        elif c == "2":
            scan()
        elif c == "3":
            break
finally:
    GPIO.output(PIN, GPIO.HIGH)
    GPIO.cleanup()
```

**Run:** `python3 m2p12.py` → **1** enroll → **2** scan. Matching finger energises the relay for 5 s, otherwise *ACCESS DENIED*.

<p align="right"><a href="#top">⬆ Back to top</a></p>

---

<a id="q13"></a>
## Q13. Basic Publisher–Subscriber using MQTT – subscribe and control LED

**Wiring:** LED long leg → **Pin 11**, short leg → resistor → GND.

**Setup**
```bash
sudo apt-get update
sudo apt-get install -y mosquitto mosquitto-clients
sudo pip3 install --break-system-packages paho-mqtt
sudo systemctl enable mosquitto
sudo systemctl start mosquitto
systemctl status mosquitto
```

**Subscriber** – `subscriber(p13-1).py`
```python
import paho.mqtt.client as mqtt
import RPi.GPIO as GPIO

BROKER = "localhost"
TOPIC = "home/led"
PIN = 11

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BOARD)
GPIO.setup(PIN, GPIO.OUT)
GPIO.output(PIN, GPIO.LOW)


def on_connect(client, userdata, flags, rc):
    print("Connected to broker, subscribing to", TOPIC)
    client.subscribe(TOPIC)


def on_message(client, userdata, msg):
    cmd = msg.payload.decode().strip().lower()
    print("Received:", cmd)

    if cmd == "on":
        GPIO.output(PIN, GPIO.HIGH)
        print("LED ON")
    elif cmd == "off":
        GPIO.output(PIN, GPIO.LOW)
        print("LED OFF")
    else:
        print("Unknown command")


try:
    client = mqtt.Client(mqtt.CallbackAPIVersion.VERSION1)
except AttributeError:
    client = mqtt.Client()

client.on_connect = on_connect
client.on_message = on_message

client.connect(BROKER, 1883, 60)

print("Subscriber running. Press Ctrl+C to stop.")

try:
    client.loop_forever()
except KeyboardInterrupt:
    print("\nStopped")
finally:
    GPIO.output(PIN, GPIO.LOW)
    GPIO.cleanup()
```

**Publisher** – `publisher(p13-2).py`
```python
import paho.mqtt.client as mqtt

BROKER = "localhost"
TOPIC = "home/led"

try:
    client = mqtt.Client(mqtt.CallbackAPIVersion.VERSION1)
except AttributeError:
    client = mqtt.Client()

client.connect(BROKER, 1883, 60)
client.loop_start()

print("Publisher ready. Type on / off / quit")

try:
    while True:
        cmd = input("> ").strip().lower()
        if cmd == "quit":
            break
        client.publish(TOPIC, cmd)
        print("Published:", cmd, "to", TOPIC)
finally:
    client.loop_stop()
    client.disconnect()
    print("Publisher closed")
```

**Run:** Terminal 1: `python3 "subscriber(p13-1).py"` → Terminal 2: `python3 "publisher(p13-2).py"` → type `on` / `off` / `quit`.

<p align="right"><a href="#top">⬆ Back to top</a></p>

---

<a id="q14"></a>
## Q14. Web dashboard using Flask – control relay module via web

**Wiring**

| Relay | Pi pin |
|---|---|
| VCC | 1 |
| GND | 6 |
| IN | 11 |

**Setup**
```bash
sudo apt install -y python3-flask
sudo systemctl enable mosquitto
```

**Folder structure**
```
relayweb/
|- app.py
|- templates/
	|- index.html
```

**`app.py`** (`Prac14.py`)
```python
from flask import Flask, render_template, jsonify
import RPi.GPIO as GPIO

PIN = 11

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BOARD)
GPIO.setup(PIN, GPIO.OUT)
GPIO.output(PIN, GPIO.HIGH)

app = Flask(__name__)

state = False

@app.route('/')
def home():
    return render_template('index.html')

@app.route('/on')
def on():
    global state
    GPIO.output(PIN, GPIO.LOW)
    state = True
    return jsonify(state=state)

@app.route('/off')
def off():
    global state
    GPIO.output(PIN, GPIO.HIGH)
    state = False
    return jsonify(state=state)

app.run(host='192.168.40.94', port=5000)
```

**`templates/index.html`**
```html
<!DOCTYPE html>
<html>
<head>
    <title>GPIO Control</title>
    <style>
        body {
            font-family: Arial;
            text-align: center;
            margin-top: 100px;
        }

        button {
            padding: 15px 30px;
            margin: 10px;
            font-size: 20px;
            cursor: pointer;
        }

        #status {
            font-size: 24px;
            margin: 20px;
        }
    </style>
</head>

<body>

    <h1>Raspberry Pi GPIO Control</h1>

    <div id="status">LED is OFF</div>

    <button onclick="turnOn()">ON</button>
    <button onclick="turnOff()">OFF</button>

    <script>
        function turnOn() {
            fetch('/on')
                .then(response => response.json())
                .then(data => {
                    document.getElementById("status").innerText =
                        data.state ? "LED is ON" : "LED is OFF";
                });
        }

        function turnOff() {
            fetch('/off')
                .then(response => response.json())
                .then(data => {
                    document.getElementById("status").innerText =
                        data.state ? "LED is ON" : "LED is OFF";
                });
        }
    </script>

</body>
</html>
```

**Run:** `python3 app.py`, then open `http://<pi-ip>:5000` in a browser and press **ON / OFF**.

> `app.run(host='192.168.40.94', ...)` is the Pi's IP on your lab network. Change it to your Pi's IP (`hostname -I`) or use `'0.0.0.0'` if you're on a different network.

<details>
<summary>Simpler HTML variant (<code>Prac14_index.html</code>)</summary>

```html
<html>
<body>
<button onclick="send('on')">ON</button>
<button onclick="send('off')">OFF</button>
<p id="s">OFF</p>
<script>
function send(a)
{
fetch('/'+a).then(r=>r.json()).then(d=>
{
document.getElementById('s').textContent = d.state ? 'ON':'OFF';
});
}
</script>
</body>
</html>
```

</details>

<p align="right"><a href="#top">⬆ Back to top</a></p>

---

<a id="q15"></a>
## Q15. Two-factor authentication: fingerprint + RFID + relay

**Wiring**

*Relay*

| Relay | Pi pin |
|---|---|
| IN | 11 |
| VCC | 3.3V |
| GND | GND |

*Fingerprint*

| Wire | Pi pin |
|---|---|
| Red | 3.3V |
| GND | GND |
| Yellow | Pin 10 |
| Green | Pin 8 |

*RFID (RC522)*

| RC522 | Pi pin |
|---|---|
| SDA | 24 |
| SCK | 23 |
| MOSI | 19 |
| MISO | 21 |
| IRQ | Unused |
| GND | 6 |
| RST | 22 |
| 3.3V | 17 |

**Setup**
```bash
sudo apt update
sudo apt install -y python3-pip python3-serial
sudo pip3 install --break-system-packages adafruit-circuitpython-fingerprint mfrc522
sudo usermod -a -G dialout $USER
```
Then the [serial setup + SPI enable](#common) and reboot.

**Code** – `p15 -1.py`
```python
import serial
import adafruit_fingerprint
import RPi.GPIO as GPIO
from mfrc522 import SimpleMFRC522
from time import sleep

PIN = 11
UNLOCK_TIME = 5

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BOARD)
GPIO.setup(PIN, GPIO.OUT)
GPIO.output(PIN, GPIO.HIGH)

uart = serial.Serial('/dev/serial0', 57600, timeout=1)
f = adafruit_fingerprint.Adafruit_Fingerprint(uart)

reader = SimpleMFRC522()


def get_image():
    while f.get_image() != adafruit_fingerprint.OK:
        pass


def enroll(slot):
    print("Place finger...")
    get_image()
    f.image_2_tz(1)
    print("Remove finger")
    while f.get_image() != adafruit_fingerprint.NOFINGER:
        pass
    print("Place same finger again...")
    get_image()
    f.image_2_tz(2)
    if f.create_model() != adafruit_fingerprint.OK:
        print("Fingers did not match")
        return
    f.store_model(slot)
    print("Enrolled at slot", slot)


def unlock():
    print("ACCESS GRANTED - unlocked")
    GPIO.output(PIN, GPIO.LOW)
    sleep(UNLOCK_TIME)
    GPIO.output(PIN, GPIO.HIGH)
    print("Locked again")


def authenticate():
    print("\n--- FACTOR 1: RFID ---")
    print("Scan your card...")
    uid, text = reader.read()
    print("UID:", uid)
    print("Card accepted")
    sleep(1)

    print("\n--- FACTOR 2: FINGERPRINT ---")
    print("Place finger...")
    get_image()
    f.image_2_tz(1)

    if f.finger_search() != adafruit_fingerprint.OK:
        print("ACCESS DENIED - fingerprint not recognised")
        return

    print("Fingerprint accepted - id", f.finger_id)
    unlock()


try:
    while True:
        c = input("\n1 enroll finger  2 authenticate  3 quit : ")
        if c == "1":
            enroll(int(input("slot number: ")))
        elif c == "2":
            authenticate()
        elif c == "3":
            break
finally:
    GPIO.output(PIN, GPIO.HIGH)
    GPIO.cleanup()
```

**Run:** `python3 "p15 -1.py"` → **1** enroll → **2** authenticate (Factor 1: RFID card, Factor 2: fingerprint). Relay unlocks for 5 s only if both pass.

<p align="right"><a href="#top">⬆ Back to top</a></p>

---

<a id="q16"></a>
## Q16. Motion detector using camera and 16x2 display

**Wiring:** camera ribbon in the CSI port + LCD:

| LCD pin | Raspberry Pi |
|---|---|
| VSS | GND |
| VDD | 5V |
| V0 | Resistor → GND (contrast) |
| RS | Pin 29 |
| RW | GND |
| E | Pin 31 |
| D4 | Pin 33 |
| D5 | Pin 35 |
| D6 | Pin 37 |
| D7 | Pin 40 |
| LED+ (BLA) | 5V |
| LED− (BLK) | GND |

**Setup**
```bash
sudo apt update
sudo apt install -y python3-picamera2 python3-opencv
sudo pip3 install --break-system-packages RPLCD
```

**Code** – `p16.py`
```python
import cv2
import numpy as np
import RPi.GPIO as GPIO
from RPLCD.gpio import CharLCD
from picamera2 import Picamera2
from time import sleep

THRESHOLD = 500000

GPIO.setwarnings(False)

lcd = CharLCD(pin_rs=29, pin_e=31, pins_data=[33, 35, 37, 40],
              numbering_mode=GPIO.BOARD, cols=16, rows=2, dotsize=8)


def show(line1, line2=""):
    lcd.clear()
    lcd.write_string(line1[:16])
    if line2:
        lcd.cursor_pos = (1, 0)
        lcd.write_string(line2[:16])


show("Starting camera")

cam = Picamera2()
cam.configure(cam.create_preview_configuration(main={"size": (320, 240)}))
cam.start()
sleep(2)

show("Monitoring...", "No motion")
print("Motion detector running. Press Ctrl+C to stop.")

previous = cv2.cvtColor(cam.capture_array(), cv2.COLOR_RGB2GRAY)
previous = cv2.GaussianBlur(previous, (21, 21), 0)

state = False

try:
    while True:
        frame = cv2.cvtColor(cam.capture_array(), cv2.COLOR_RGB2GRAY)
        frame = cv2.GaussianBlur(frame, (21, 21), 0)

        diff = cv2.absdiff(previous, frame)
        thresh = cv2.threshold(diff, 25, 255, cv2.THRESH_BINARY)[1]
        score = int(np.sum(thresh))

        if score > THRESHOLD:
            if not state:
                show("MOTION", "DETECTED")
                print("MOTION DETECTED  score:", score)
                state = True
        else:
            if state:
                show("Monitoring...", "No motion")
                print("No motion")
                state = False

        previous = frame
        sleep(0.3)

except KeyboardInterrupt:
    print("\nStopped")

finally:
    cam.stop()
    lcd.clear()
    lcd.close(clear=True)
    GPIO.cleanup()
```

**Run:** `python3 p16.py` – LCD shows **Monitoring... / No motion**, and **MOTION / DETECTED** when movement is seen. Stop with `Ctrl+C`.

<p align="right"><a href="#top">⬆ Back to top</a></p>

