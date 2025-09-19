# Autoryzacja w Medfile Services API

## JWT Token

Do autoryzacji używamy tokentów JWT o krótkim terminie ważności z podpisem.
Po podpisaniu umowy firma BioStat udostępnia klucze do szyfrowania i weryfikacji tokentów JWT.

Token JWT wymagana następujących pól:
- *iss* - nadany identyfikator integratora
- *aud* - stała wartość "medfile"
- *iat* - czas (unix timestamp) wygenerowania tokena
- *exp* - czas (unix timestamp) wygaśnięcia tokena (zalecany maks. iat + 60)
- *practitioner* - opcjonalnie (kiedy wymagany) identyfikator lekarza w którego kontekście wywoływana jest akcja
- *organization* - opcjonalnie (kiedy wymagany) identyfikator podmiotu w którego kontekście wywoływana jest akcja

Ponadto:

- pole *sub* nie powinno być używane (zarezerwowane do przyszłych zastosowań)

## Przekazywanie tokenu do API

Token przekazujemy w każdym request wysyłanym do API jako nagłówek HTTP.

Przykład:

```
Authorization: Bearer JWT_TOKEN
```

## Nieprawidłowa autoryzacja

Nieprawidłowo podpisany token będzie skutkował odpowiedzią HTTP: `401` - Brak autoryzacji - token JWT jest nie tak.

# Kody odpowiedzi

- `200` - Poprawna odpowiedź
- `400` - Nieprawidłowe żądanie - np. metoda GET gdy wymagany POST
- `401` - Brak autoryzacji - token JWT jest nie tak
- `403` - Brak dostępu do wybranych zasobów
- `404` - Nie znaleziono zasobów np. recepty czy danych lekarza
- `406` - Not Acceptable/Wynik inny od sukcesu w przypadku np. weryfikacji recepty.
- `500` - Gdy wystąpi nieobsłużony problem
- `501` - Gdy funkcjonalność jeszcze nie jest uruchomiona - możemy to zwracać dla `eskierowanie`

# Organizacja - podmiot lub praktyka

Organizacją nazywamy dowolny podmiot medyczny lub praktykę lekarską.

Do pracy z Medfile API wymagane jest utworzenie przynajmniej jednej organizacji, która będzie używana we właściwych usługach w tym np. erecepta.

## Tworzenie nowej organizacji

PUT /organization/

```json
{
   "type": "LEK", // typ organizacji, domyślnie jednostka lub praktyka lekarska (LEK). Dostępne: ["LEK", "FIZJO", "PIEL", "APTEKA", "ZAOP_MED"]
   "name": "BioStat Sp. z o.o.",
   "identifier": [
      {
         "type": "rpwdl",
         "value": "000000792087"
      },
      {
         // Tylko dla jednostki podmiotu
         "type": "rpwdlUnit",
         "value": "01"
      },
      {
         // Tylko dla jednostki podmiotu
         "type": "unitName",
         "value": "Szpital nr 1"
      },
      {
         // Tylko dla komórki podmiotu
         "type": "rpwdlCell",
         "value": "001"
      },
      {
         // Tylko dla komórki podmiotu
         "type": "cellName",
         "value": "Ogólna izba przyjęć"
      },
      {
         // Tylko dla praktyki lekarskiej
         "type": "medicalChamber",
         "value": "68"
      },
      {
         // Tylko dla praktyki lekarskiej - miejsce wykonywania zawodu
         "type": "praxisCode",
         "value": "001"
      },
      {
         "type": "regon",
         "value": "24154444300004"
      },
      {
         "type": "nip",
         "value": "6391001234"
      },
      {
         "type": "oidRoot",
         "value": "2.16.840.1.113883.3.4424.2.7.67"
      },
      {
         // Specjalizacja organizacji
         "type": "speciality",
         "value": "Poradnia neurologiczna"
      },
      {
         // Specjalizacja organizacji (VIII cz. kodu resortowego)
         "type": "specialityCode",
         "value": "1220"
      }
   ],
   "telecom": [
      {
         "value": "+48131231230"
      }
   ],
   "address": [
      {
         "street": "ul. Dubois",
         "city": "Warszawa",
         "postalCode": "00-184",
         "country": "pl",
         "houseNumber": "5A",
         "unitId": "13"
      }
   ],
   "nfzDepartment": "07",
   // opcjonalny oddział NFZ z którym organizacja zawarła umowę
   "nfzContract": "01-06-02336-20-03/06",
   // opcjonalny numer umowy z NFZ
   "services": {
      "p1": {
         "tls": [],
         "wss": []
      }
   }
}
```

# Specjalista - użytkownik korzystający z usług w Medfile API

Specjalistą jest każdy użytkownik, który będzie korzystał z API.

Większość usług wymaga do działania specjalisty/lekarza.

Identyfikator lekarza przekazujemy do API w tokenie JWT pod kluczem `practitioner`.

Uwaga: Należy zwrócić uwagę, aby identyfikatory były poprawne i nienadmiarowe.
Np. nie należy wysyłać dwóch numerów NPWZ.
Błędy popełnione na tym etapie często są trudne do wyłapania ze względu na różną logikę biznesową różnych usług.

Identyfikator specjalisty w formacie UUIDv4 otrzymają Państwo przy utworzeniu użytkownika (zapisie).


## Tworzenie nowego użytkownika

```
PUT /practitioner/
```

### Treść żądania

```json
{
  "identifier": [
    {
      "type": "pesel",
      "value": "600322236"
    },
    {
      "type": "npwz",
      "value": "1044400"
    }
  ],
  "name": [
    {
      "family": "Kowalski",
      "given": [
        "Jan",
        "Henryk"
      ]
    }
  ],
  "telecom": [ // wymagane m.in. dla recept PA, PF
    {
      "value": "+48131231230"
    }
  ],
  "address": [ // wymagane m.in. dla recept PA, PF
    {
      "street": "ul. Dubois",
      "houseNumber": "5A",
      "unitId": "13",
      "city": "Warszawa",
      "postalCode": "00-184"
    }
  ],
  "qualifications": [ // Wymagane dla e-skierowań
    {
      "code": "0713",
      "display": "medycyna rodzinna"
    },
    {
      "code": "0731",
      "display": "alergologia"
    }
  ],
  "role": "LEK" // rola jaką pełni użytkownik np. LEK, LEKD, FIZJO, PIEL etc.
}
```

### Pozytywna odpowiedź

```json
{
  "uuid": "cd986fde-066d-4f3d-8144-67d44f955656",
  "identifier": [
    {
      "type": "pesel",
      "value": "98030801601"
    },
    {
      "type": "npwz",
      "value": "7689753"
    }
  ],
  "name": [
    {
      "family": "Praktyczny",
      "given": [
        "Artur09"
      ]
    }
  ],
  "telecom": [
    {
      "value": "+48131231230"
    }
  ],
  "address": [
    {
      "street": "ul. Dubois",
      "city": "Warszawa",
      "postalCode": "00-184",
      "country": "pl",
      "houseNumber": "5A",
      "unitId": "13"
    }
  ],
  "qualifications": [
    {
      "code": "0713",
      "display": "medycyna rodzinna"
    },
    {
      "code": "0731",
      "display": "alergologia"
    }
  ],
  "role": "LEK"
}
```

### Negatywna odpowiedź

```json
{
  "error": [{
    "code": "",
    "text": "Proszę podać nazwisko."
  }]
}
```

## Aktualizacja specjalisty

```
POST /practitioner/{practitioner.uuid}
```

### Treść żądania

Treść jest identyczna z trasą `PUT /practitioner/`.
Aktualizowane są tylko przekazane informacje.

W celu usunięcia wybranych informacji należy przesłać wartość `null` dla wybranych pól.

Przykład usunięcia numeru PESEL ze specjalisty.
Oraz jednoczesna zmiana jego numeru PWZ.

```json
{
  "identifier": [
    {
      "type": "pesel",
      "value": null
    },
    {
      "type": "npwz",
      "value": "99999999"
    }
  ]
}
```

## Usuwanie specjalisty

```
DELETE /practitioner/{practitioner.uuid}
```

### Treść żądania

Brak.

### Pozytywna odpowiedź

```json
{
  "uuid": "f0a5d245-2f7d-4fa5-ad76-04ede898987d",
  "identifier": [
    {
      "type": "pesel",
      "value": "98030801601"
    },
    {
      "type": "npwz",
      "value": "7689753"
    }
  ],
  "name": [
    {
      "family": "Praktyczny",
      "given": [
        "Artur09"
      ]
    }
  ],
  "telecom": [
    {
      "value": "+48131231230"
    }
  ],
  "address": [
    {
      "street": "ul. Dubois",
      "city": "Warszawa",
      "postalCode": "00-184",
      "country": "pl",
      "houseNumber": "5A",
      "unitId": "13"
    }
  ],
  "qualifications": [
    {
      "code": "0713",
      "display": "medycyna rodzinna"
    },
    {
      "code": "0731",
      "display": "alergologia"
    }
  ],
  "role": "LEK"
}
```

### Negatywna odpowiedź

```json
{
  "error": [
    {
      "code": "",
      "text": "Specjalista nie istnieje."
    }
  ]
}
```

# Konfiguracja usług

## Certyfikaty WSS i TLS do komunikacji z P1

```
POST /services/p1/{organizationUuid}
```

```json
{
  "tls": {
    "certificate": "__base64 encoded p12 file__",
    "password": "p4ssw0rd!"
  },
  "wss": {
    "certificate": "__base64 encoded p12 file__",
    "password": "p4ssw0rd!"
  }
}
```

## Ustawienia eWUŚ dla użytkownika

```
POST /services/ewus/{practitionerUuid}
```

Przykładowa konfiguracja dla lekarza:
```json
{
  "domain": "01",
  "operatorType": "LEK",
  "identifier": "abc123",
  "username": "john",
  "password": "p4ssw0rd!"
}
```
Przykładowa konfiguracja dla świadczeniodawcy:
```json
{
  "domain": "01",
  "operatorType": "SWD",
  "identifier": "T/11/040",
  "username": "john",
  "password": "p4ssw0rd!"
}
```


1. `domain` - oddział NFZ
   - 01 - Dolnośląski
   - 02 - Kujawsko-Pomorski
   - 03 - Lubelski
   - 04 - Lubuski
   - 05 - Łódzki
   - 06 - Małopolski
   - 07 - Mazowiecki
   - 08 - Opolski
   - 09 - Podkarpacki
   - 10 - Podlaski
   - 11 - Pomorski
   - 12 - Śląski
   - 13 - Świętokrzyski
   - 14 - Warmińsko-Mazurski
   - 15 - Wielkopolski
   - 16 - Zachodniopomorski
   - 17 - Centrala
1. `operatorType` - typ operatora
   - LEK - Lekarz
   - SWD - Świadczeniodawca
1. `identifier` - identyfikator świadczeniodawcy (nadany przez NFZ) lub identyfikator lekarza (SNRL)
1. `username` - nazwa użytkownika (konto użytkownika założone w NFZ)
1. `password` - hasło dostępowe dla użytkownika `username`

Konfiguracja jest zależna od rodzaju operatora oraz od województwa. Oddziały wojewódzkie: 02,03,07,10,13,14,15,16 nie wymagają przekazywania identyfikatora świadczeniodawcy/lekarza.


## Ustawienie konta do wystawienia eZWM

```
POST /services/ezwm/{practitionerUuid}
```

```json
{
  "domain": "01",
  "operatorType": "SWD",
  "identifier": "abc123",
  "username": "john",
  "password": "p4ssw0rd!"
}
```

1. `operatorType` - typ operatora
   - LEK - Lekarz
   - SWD - Świadczeniodawca   
1. `identifier` - identyfikator świadczeniodawcy (nadany przez NFZ) lub identyfikator lekarza (SNRL)
1. `username` - nazwa użytkownika (konto użytkownika założone w NFZ)
1. `password` - hasło dostępowe dla użytkownika `username`

Konfiguracja (podobnie jak eWUŚ) jest zależna od rodzaju operatora oraz od województwa. Oddziały wojewódzkie: 02,03,07,10,13,14,15,16 nie wymagają przekazywania identyfikatora świadczeniodawcy/lekarza.


## Ustawienie certyfikatu ZUS lekarza

```
POST /services/zus/{practitionerUuid}
```

```json
{
  "certificate": "__base64 encoded pfx file__",
  "password": "p4ssw0rd!"
}
```
