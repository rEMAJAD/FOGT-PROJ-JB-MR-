# Bezprzewodowy pomiar temperatury i wilgotności – Raspberry Pi Pico + Raspberry Pi 4 + HC-12

Projekt składa się z nadajnika (Raspberry Pi Pico) oraz odbiornika (Raspberry Pi 4) komunikujących się poprzez moduły HC-12 pracujące w paśmie 433 MHz. Nadajnik wykonuje pomiary temperatury i wilgotności czujnikiem DHT11 i wysyła dane w formacie tekstowym. Odbiornik odbiera dane, interpretuje je, wyświetla na OLED i reaguje alarmem, jeśli przekroczone zostaną progi.

---

# 1. Zawartość repozytorium

- `nadawczaPICO.py` – kod nadajnika (MicroPython)
- `odbiorczaRP4.py` – kod odbiornika (Python na Raspberry Pi 4)
- Dokumentacja PDF
- Informacje o autorach

---

# 2. Opisy funkcji – nadajnik (nadawczaPICO.py)

## `blink()`
Sygnalizuje wykonanie pomiaru krótkim zapaleniem diody LED.  
Umożliwia wizualne monitorowanie pracy nadajnika.

## `send_measurement(temp, hum)`
Przyjmuje odczytaną temperaturę i wilgotność, formatuje je do postaci:

`T:<temp>;H:<hum>`

i wysyła przez interfejs UART do modułu HC-12.  
Dodatkowo wypisuje wysłaną ramkę w konsoli USB.

## Inicjalizacja czujnika DHT11
Wykonuje testowy pomiar startowy, aby upewnić się, że czujnik jest gotowy i nie wystąpi błąd w pierwszym cyklu pracy.

## Pętla główna (`while True:`)
W każdym cyklu:
1. Wywołuje `blink()` – sygnał pomiaru.
2. Odczytuje temperaturę i wilgotność z DHT11.
3. Wyświetla dane w konsoli.
4. Przekazuje wartości do funkcji `send_measurement()`.
5. Czeka określony czas (`MEASURE_INTERVAL`).

W przypadku błędów DHT11 wypisuje komunikat, ale kontynuuje pracę.

---

# 3. Opisy funkcji – odbiornik (odbiorczaRP4.py)

## `blink_led()`
Zapala LED na krótką chwilę po poprawnym odebraniu i sparsowaniu danych z modułu HC-12.  
Jest to potwierdzenie prawidłowej komunikacji.

## `set_buzzer(on: bool)`
Włącza lub wyłącza buzzer w zależności od wartości logicznej argumentu.  
Buzzer jest aktywowany, gdy:
- temperatura przekroczy zadany próg,
- wilgotność przekroczy zadany próg.

## `update_oled(temperature, humidity, alarm, ok_read=True)`
Odpowiada za pełny rendering ekranu OLED.  
Wyświetla:
- temperaturę,
- wilgotność,
- stan alarmowy,
- komunikat o błędzie danych, jeśli `ok_read=False`.

Funkcja czyści cały ekran i rysuje nowy zestaw informacji.

## `parse_line(line: str)`
Odpowiada za analizę surowej ramki tekstowej z HC-12.  
Działanie:
1. Usuwa białe znaki.
2. Sprawdza obecność separatora `;`.
3. Weryfikuje poprawność formatu `T:xx.x` oraz `H:yy.y`.
4. Konwertuje dane na wartości liczbowe.

Zwraca:
- `(True, temperatura, wilgotność)` przy poprawnym formacie,
- `(False, None, None)` przy błędzie.

## `read_remote_with_retries(retries=5)`
Próbuje odebrać poprawną ramkę danych wykonując kilka odczytów UART.  
Po każdej próbie przekazuje odebraną linię do `parse_line()`.  
Zwraca pierwszą poprawną ramkę, jeśli taka wystąpi.

## Pętla główna (`while True:`)
W każdym cyklu:
1. Próbuje odczytać dane z HC-12 funkcją `read_remote_with_retries()`.
2. Jeśli dane poprawne:
   - wywołuje `blink_led()`,
   - wyświetla dane na konsoli,
   - porównuje temperaturę i wilgotność z progami alarmowymi,
   - włącza buzzer w przypadku alarmu,
   - aktualizuje OLED (`update_oled()`).
3. Jeśli dane niepoprawne:
   - wyświetla komunikat błędu,
   - wyłącza buzzer,
   - rysuje ekran „Brak danych”.

4. Czeka 1 sekundę i powtarza cykl.

## Blok `except KeyboardInterrupt` oraz `finally`
- Umożliwia bezpieczne zakończenie programu.
- Wyłącza buzzer.
- Czyści i resetuje GPIO.
- Czyści ekran OLED.
- Zamyka port szeregowy.

---

# 4. Format transmisji danych

Nadajnik wysyła dane w postaci:

