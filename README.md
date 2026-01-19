# Bezprzewodowy pomiar temperatury i wilgotłości – Raspberry Pi Pico + Raspberry Pi 4 + HC-12

Projekt składa się z dwóch modułów: nadajnika (Pico) i odbiornika (RPi4). Nadajnik wykonuje pomiar czujnikiem DHT11 i wysyła dane radiowo przez HC-12. Odbiornik odbiera dane, analizuje je, wyświetla na OLED oraz uruchamia alarm po przekroczeniu progów.

## 1. Opis funkcji – nadajnik (nadawczaPICO.py)

**blink()** – Krótkie zapalenie LED sygnalizujące wykonanie pomiaru.  
**send_measurement(temp, hum)** – Formatuje dane do `T:<temp>;H:<hum>` i wysyła je UART-em przez HC-12.  
**Inicjalizacja DHT11** – Jednorazowy testowy odczyt, by upewnić się, że czujnik działa.  
**Pętla główna** – Co kilka sekund: miga LED, odczytuje temperaturę i wilgotność, wypisuje je i wysyła ramkę danych.

## 2. Opis funkcji – odbiornik (odbiorczaRP4.py)

**blink_led()** – Krótkie zapalenie LED po poprawnym odebraniu ramki radiowej.  
**set_buzzer(on)** – Włącza lub wyłącza buzzer w zależności od tego, czy wystąpił alarm.  
**update_oled(temp, hum, alarm, ok_read)** – Czyści i aktualizuje ekran OLED (dane lub komunikat błędu).  
**parse_line(line)** – Sprawdza format `T:xx.x;H:yy.y`, rozdziela wartości i konwertuje je na liczby.  
**read_remote_with_retries(retries)** – Próbuje wielokrotnie odebrać poprawną ramkę z HC-12.  
**Pętla główna** – Odbiera dane, sygnalizuje LED, porównuje wartości z progami, obsługuje alarm i aktualizuje ekran; przy błędach wyświetla komunikat i wycisza buzzer.

## 3. Format przesyłanych danych

T:<temperatura>;H:<wilgotność>

Przykład:
T:23.5;H:46.0

## 4. Uruchomienie

**Pico:** wgrać MicroPython, skopiować `nadawczaPICO.py`, podłączyć DHT11, LED i HC-12, uruchomić.  
**RPi4:** podłączyć HC-12, OLED, LED, buzzer; zainstalować biblioteki OLED; uruchomić `odbiorczaRP4.py`.

## 5. Informacje

Projekt edukacyjny – swobodny do modyfikacji.
