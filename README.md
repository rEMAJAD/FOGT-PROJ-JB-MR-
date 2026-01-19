# Bezprzewodowy pomiar temperatury i wilgotności – Raspberry Pi Pico + Raspberry Pi 4 + HC-12

Projekt realizuje bezprzewodowy system pomiaru temperatury i wilgotności. Nadajnik oparty o Raspberry Pi Pico wykonuje pomiary czujnikiem DHT11 i przesyła je drogą radiową przez moduł HC-12. Odbiornik zbudowany na Raspberry Pi 4 odbiera, interpretuje dane i prezentuje je na wyświetlaczu OLED z dodatkową sygnalizacją LED i alarmem akustycznym.

## 1. Zawartość repozytorium

- `nadawczaPICO.py` – kod mikronadajnika (Raspberry Pi Pico)
- `odbiorczaRP4.py` – kod mikroodbiornika (Raspberry Pi 4)
- `Projekt_FOGT_JB_MR.pdf` – dokumentacja techniczna projektu
- `zespol.txt` – informacje o autorach

## 2. Wymagany sprzęt

### Nadajnik (Pico)
- Raspberry Pi Pico
- Czujnik DHT11
- Moduł radiowy HC-12
- Dioda LED + rezystor
- Przewody połączeniowe

### Odbiornik (RPi 4)
- Raspberry Pi 4
- Moduł HC-12
- Wyświetlacz OLED SSD1306 (SPI)
- LED + buzzer
- Przewody połączeniowe

## 3. Opis działania nadajnika (Raspberry Pi Pico)

Nadajnik inicjalizuje czujnik DHT11, wykonuje cykliczny pomiar temperatury i wilgotności, sygnalizuje jego wykonanie krótkim impulsem LED oraz wysyła dane w formacie tekstowym `T:<temp>;H:<hum>` przez UART do modułu HC-12.

### Opis funkcji nadajnika

- **blink()** – krótki błysk LED informujący o wykonaniu pomiaru.
- **send_measurement(temp, hum)** – przygotowanie ramki tekstowej oraz jej wysłanie przez UART (HC-12) oraz wypisanie treści dla konsoli USB.
- **pętla główna** – odpowiada za wykonywanie cyklicznych pomiarów i obsługę błędów czujnika.

### Kod źródłowy nadajnika
```python
import machine
import utime
import dht

PIN_DHT = 16        
PIN_LED = 14        
UART_PORT = 0
UART_TX = 0         
UART_RX = 1         
UART_BAUD = 9600    
MEASURE_INTERVAL = 5  

sensor = dht.DHT11(machine.Pin(PIN_DHT))
led = machine.Pin(PIN_LED, machine.Pin.OUT)

uart = machine.UART(
    UART_PORT,
    baudrate=UART_BAUD,
    tx=machine.Pin(UART_TX),
    rx=machine.Pin(UART_RX)
)

def blink():
    led.value(1)
    utime.sleep(0.1)
    led.value(0)

def send_measurement(temp, hum):
    msg = "T:{:.1f};H:{:.1f}\n".format(temp, hum)
    uart.write(msg)
    print("Wyslano:", msg.strip())

try:
    sensor.measure()
    _ = sensor.temperature()
    _ = sensor.humidity()
    utime.sleep(1)
except OSError:
    pass

while True:
    try:
        blink()
        sensor.measure()
        temp = sensor.temperature()
        hum = sensor.humidity()
        print("Temp:", temp, "C  Wilg:", hum, "%")
        send_measurement(temp, hum)
    except OSError as e:
        print("Blad odczytu DHT11:", e)
    utime.sleep(MEASURE_INTERVAL)
