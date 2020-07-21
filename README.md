# RFID card reader z szyfrowaną komunikacją i autoryzacją

## Instrukcja uruchomienia

Aby urochomić ten program, należy:
  - Zainstalować `mosquitto` z [tej strony](https://mosquitto.org/download/).
  - Utworzyć nowy folder `certs` w folderze mosquitto. (e.g. `C:\Program Files\mosquitto\certs`).
  - Zainstalować OpenSSL na swój system.
  - Utworzyć sertyfikaty po kolei uruchamiając:

    + `openssl genrsa -des3 -out ca.key 2048` i wprowadzić hasło.
    + `openssl req -new -x509 -days 1826 -key ca.key -out ca.crt` pomijając wszystkie opcji.
    + `openssl genrsa -out server.key 2048`
    + Sprawdź hostname urządzenia przez `hostname` w cmd.
    `openssl req -new -out server.csr -key server.key` używając `hostname` z poprzedniego kroku w opcji `Common Name`.
    + `openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 360`
    + W rezultacie musze być 6 plików.

  - Do utworzonego folderu `certs` skopiować pliki `server.crt`, `server.key` i `ca.crt`.
  - Skopiować `mosquitto.conf` i zamienić opcję `cafile, certfile, keyfile`, jeżeli mosquitto był zainstalowany w innym miejscu.
  - Dodać plik `ca.crt` w folder projektu.
  - Wstaw plik `aclfile.conf` do folderu mosquitto.
  - Uruchomić `pip install -r requirements.txt`.
  - Zresetować poprzednią bazę danych używając komendy `python create_database.py`.
  - Urochomić brokera używając `mosquitto -c "C:\Program Files\mosquitto\mosquitto.conf" -v`.
  - Uruchomić scrypt `client.py` na urządzeniu Raspberry Pi lub Virtual Machine (PC).
  - Uruchomić scrypt `server.py` na serwerze (PC lub to samo urządzenie Raspberry Pi)
  
## Struktura

```text
.
├── certs
│   ├── ca.crt
│   ├── server.crt
│   └── server.key
├── .env.example
├── mosquitto.conf
├── aclfile.conf
├── client.py
├── server.py
├── create_database.py
└── seeder.py
```

## Opis składowych i działania aplikacji

**`.env`**

Plik z konfiguracją brokera i inną. Zmień hasło i nazwę użytkownika, żeby zalogować się do systemu.

```text
USER=admin
PASS=admin
```

**`create_database.py`**

Tworzy nową bazę danych i usuwa poprzednią.

**`seeder.py`** 

Generuje 1000 fejków w tablicah `workers` i `cards` dla sprawdzenia działalności programu.

**`client.py`**

Podłączamy się do Brokera i odsyłujemy Card ID do brokera (topic=`worker/test`).
Sprawdzamy poprawność Card ID (ID może być tylko liczbą). Ten plik będzie używany na
urządzeniu Raspberry Pi (lub na Virtual Machine).

**`server.py`**

Podłączamy się do Brokera, otrzymujemy message w potrzebnym topiku. Używamy ten plik na serwerze, czyli na 
PC lub innym urządzeniu (w tym i Raspberry Pi). Przy uruchomieniu plika mamy utworzony GUI z 
różnymi przyciskami (symulowany system urządzający - admin panel).

**`aclfile.conf`**

Zapisz poniższy kod do pliku:
```text
user server
topic read card/id
topic read auth/login
topic write auth/res

user client
topic write card/id
topic write auth/login
topic read auth/res
```

## Baza danych projektu

Dla tego projektu jest używana biblioteka `sqlite3`. SQLite jest bardzo wygodna i prosta w 
tej sytuacji.

##### Tablica `workers`

Przechowujemy tu wszystkich pracowników.
```text
workers
+- id           INTEGER, AUTOINCREMENT
+- name         TEXT, NOT NULL
+- card_id      INTEGER, NULLABLE
+- hired_at     DATETIME CURRENT_TIMESTAMP
```

##### Tablica `cards`
Przechowujemy tu wszystkie karty elektroniczne.
```text
cards
+- id           INTEGER, AUTOINCREMENT
+- name         TEXT, NOT NULL
+- worker_id    INTEGER, NULLABLE
+- created_at   DATETIME CURRENT_TIMESTAMP
```

##### Tablica `records`
Przechowujemy tu zapisy wszystkich wyjść/wejść pracowników.
```text
records
+- id           INTEGER, AUTOINCREMENT
+- card_id      INTEGER, NOT NULL
+- worker_id    INTEGER, NULLABLE
+- timestamp    DATETIME CURRENT_TIMESTAMP
```

## Screenshots

![alt zero](https://imgur.com/H6sRVQY.png)

![alt first](https://imgur.com/vngDjtB.png)

![alt second](https://imgur.com/IY9fQ2z.png)

![alt third](https://imgur.com/9vnHAbh.png)