# Usługa eZLA

## Konfiguracja usługi

Do komunikacji z ZUS wymagany jest tylko certyfikat wydany przez ZUS dla lekarza lub innego specjalisty.

Dołączamy blok `zus` do sekcji "usługi" w danych użytkownika.

```json
{
    "services": {
        "zus": {
            "certificate": "base64 ...",
            "password": "p4ssw0rd!"
        }
    }
}
```
Gdzie:
- `certificate` - treść pliku zus.pfx w base64
- `password` - hasło do certyfikatu zus.pfx

> __Uwaga__
>
> Na środowisku testowym należy użyć certyfikatu z ZUS należącego do prawdziwego lekarza. Dane które zwróci API i dane które powstaną w API nie będą miały żadnego związku z tym lekarzem i nie spowodują żadnych konsekwencji.


## Nagłówki HTTP

Do komunikacji z ZUS wymagany jest specjalista (*practitioner*).

> __Uwaga__
>
> Dane specjalisty najlepiej pobrać bezpośrednio z ZUS i wysłać je w dokumencie zwolnienia lekarskiego.
> Wielkość liter, znaki interpunkcji mają znaczenie dla ZUS i jeżeli będą nieprawidłowo wprowadzone spowodują błędy w weryfikacji zwolnienia.

- Authorization: Bearer {JWT TOKEN}

## Kody odpowiedzi

1. `200` - Poprawna odpowiedź
2. `403` - Brak uprawnień do wykonania polecenia
3. `404` - Brak danych

## Pobierz dane lekarza z ZUS

```
GET /ezla/doctor/
```

### Prawidłowa odpowiedź

```json
{
  "message": "",
  "body": {
    "Imie": "THEO",
    "Nazwisko": "KANIA",
    "Pesel": "44042102357",
    "NumerPrawaWykonywaniaZawodu": "1063324",
    "NazwaOkregowejIzbyLekarskiej": "Prezydium Wojewódzkiej Rady Narodowej w Częstochowie",
    "StatusLekarza": {
      "DataWydaniaDecyzji": "2000-01-01",
      "Status": "A"
    },
    "PosiadanaSpecjalizacja": [
      {
        "KodSpecjalizacji": "282",
        "NazwaSpecjalizacji": "Ortopedia",
        "StopienSpecjalizacji": "2"
      },
      {
        "KodSpecjalizacji": "281",
        "NazwaSpecjalizacji": "Ortopedia",
        "StopienSpecjalizacji": "1"
      }
    ],
    "Adres": [
      {
        "Ulica": "Maków",
        "NrDomu": "53",
        "KodPocztowy": "41400",
        "Miejscowosc": "Mysłowice"
      }
    ],
    "MiejsceWykonywaniaZawodu": [
      {
        "Nazwa": "PRZYCHODNIA TARCZÓWKOWATYMI",
        "NIP": "3962382940",
        "Ulica": "Zdręby",
        "NrDomu": "112",
        "KodPocztowy": "16423",
        "Miejscowosc": "Zdręby"
      }
    ]
  },
  "error": [],
  "raw": {
    "PobierzDaneLekarzaResponse": {
      "DaneLekarza": {
        "Imie": "THEO",
        "Nazwisko": "KANIA",
        "Pesel": "44042102357",
        "NumerPrawaWykonywaniaZawodu": "1063324",
        "NazwaOkregowejIzbyLekarskiej": "Prezydium Wojewódzkiej Rady Narodowej w Częstochowie",
        "StatusLekarza": {
          "DataWydaniaDecyzji": "2000-01-01",
          "Status": "A"
        },
        "PosiadanaSpecjalizacja": [
          {
            "KodSpecjalizacji": "282",
            "NazwaSpecjalizacji": "Ortopedia",
            "StopienSpecjalizacji": "2"
          },
          {
            "KodSpecjalizacji": "281",
            "NazwaSpecjalizacji": "Ortopedia",
            "StopienSpecjalizacji": "1"
          }
        ],
        "Adres": [
          {
            "Ulica": "Maków",
            "NrDomu": "53",
            "KodPocztowy": "41400",
            "Miejscowosc": "Mysłowice"
          }
        ],
        "MiejsceWykonywaniaZawodu": [
          {
            "Nazwa": "PRZYCHODNIA TARCZÓWKOWATYMI",
            "NIP": "3962382940",
            "Ulica": "Zdręby",
            "NrDomu": "112",
            "KodPocztowy": "16423",
            "Miejscowosc": "Zdręby"
          }
        ]
      },
      "Rezultat": {
        "KodBledu": "0",
        "OpisBledu": "OK"
      }
    }
  }
}
```

## Pobierz miejsca pracy lekarza z ZUS

```
GET /ezla/doctor/workplace
```

### Prawidłowa odpowiedź

```json
{
  "message": "",
  "body": [
    {
      "Nazwa": "PRZYCHODNIA TARCZÓWKOWATYMI",
      "NIP": "3962382940",
      "Ulica": "Zdręby",
      "NrDomu": "112",
      "KodPocztowy": "16423",
      "Miejscowosc": "Zdręby",
      "IdMiejsca": "231"
    }
  ],
  "error": [],
  "raw": {
    "PobierzMiejsceWykonywaniaZawoduResponse": {
      "Rezultat": {
        "KodBledu": "0",
        "OpisBledu": "OK"
      },
      "MiejsceWykonywaniaZawodu": [
        {
          "Nazwa": "PRZYCHODNIA TARCZÓWKOWATYMI",
          "NIP": "3962382940",
          "Ulica": "Zdręby",
          "NrDomu": "112",
          "KodPocztowy": "16423",
          "Miejscowosc": "Zdręby",
          "IdMiejsca": "231"
        }
      ]
    }
  }
}
```

## Pobierz uprawnienia lekarza na dzień
```
GET /ezla/doctor/permissions/{{today}}
```

### Prawidłowa odpowiedź

```
{
  "message": "",
  "body": true,
  "error": [],
  "raw": {
    "PobierzUprawnieniaNaDzienResponse": {
      "Rezultat": {
        "KodBledu": "0",
        "OpisBledu": "OK"
      },
      "MozliwoscWystawianiaZaswiadczenia": true
    }
  }
}
```

## Pobierz dane pacjenta z ZUS

```
GET /ezla/patient/{pesel}
```

### Odpowiedź

```json
{
  "message": "",
  "body": {
    "Imie": "OLHA",
    "Nazwisko": "GAJEWSKA",
    "DataUrodzenia": "1968-07-18",
    "Adres": [
      {
        "KodPocztowy": "90118",
        "Miejscowosc": "Łódź (Łódź-Śródmieście)",
        "Ulica": "Kilińskiego Jana",
        "NrDomu": "67"
      },
      {
        "KodPocztowy": "28114",
        "Miejscowosc": "Poręba",
        "Ulica": "Poręba",
        "NrDomu": "96D"
      }
    ]
  },
  "error": [],
  "raw": {
    "PobierzDaneUbezpieczonegoResponse": {
      "DaneUbezpieczonego": {
        "Imie": "OLHA",
        "Nazwisko": "GAJEWSKA",
        "DataUrodzenia": "1968-07-18",
        "Adres": [
          {
            "KodPocztowy": "90118",
            "Miejscowosc": "Łódź (Łódź-Śródmieście)",
            "Ulica": "Kilińskiego Jana",
            "NrDomu": "67"
          },
          {
            "KodPocztowy": "28114",
            "Miejscowosc": "Poręba",
            "Ulica": "Poręba",
            "NrDomu": "96D"
          }
        ]
      },
      "Rezultat": {
        "KodBledu": "0",
        "OpisBledu": "OK"
      },
      "PrzekroczonyDziennyLimitDostepow": false
    }
  }
}
```

## Pobierz członków rodziny pacjenta z ZUS

```
GET /ezla/patient/{pesel}/family
```

### Prawidłowa odpowiedź

```json
{
  "message": "",
  "body": [
    {
      "Imie": "TINA",
      "Nazwisko": "SZCZEPAŃSKA",
      "DataUrodzenia": "2000-09-05T00:00:00+00:00"
    }
  ],
  "error": [],
  "raw": {
    "PobierzCzlonkowRodzinyUbezpieczonegoResponse": {
      "Rezultat": {
        "KodBledu": "0",
        "OpisBledu": "OK"
      },
      "CzlonekRodzinyUbezpieczonego": [
        {
          "Imie": "TINA",
          "Nazwisko": "SZCZEPAŃSKA",
          "DataUrodzenia": "2000-09-05T00:00:00+00:00"
        }
      ]
    }
  }
}
```

## Pobierz płatników pacjenta z ZUS

```
GET /ezla/patient/{pesel}/payers/{issuer}
```

Gdzie `{issuer}` to:
- ZUS
- KRUS
- INNE_W_POLSCE
- W_INNYM_PANSTWIE

### Prawidłowa odpowiedź

```json
{
  "message": "",
  "body": [
    {
      "Nazwa": "ALARMOWANO",
      "Imie": "ANDREA",
      "Nazwisko": "KOZŁOWSKA",
      "NIP": "8304421563",
      "Pesel": "68032721861",
      "SeriaNumerPaszportu": "XEV357034",
      "ProfilPUE": false
    }
  ],
  "error": [],
  "raw": {
    "PobierzPlatnikowUbezpieczonegoResponse": {
      "Rezultat": {
        "KodBledu": "0",
        "OpisBledu": "OK"
      },
      "Platnik": [
        {
          "Nazwa": "ALARMOWANO",
          "Imie": "ANDREA",
          "Nazwisko": "KOZŁOWSKA",
          "NIP": "8304421563",
          "Pesel": "68032721861",
          "SeriaNumerPaszportu": "XEV357034",
          "ProfilPUE": false
        }
      ]
    }
  }
}
```

## Pobierz słownik pokrewieństwa

```
GET /ezla/dictionary/kinship
```

### Prawidłowa odpowiedź

```json
{
  "message": "",
  "body": [
    {
      "Kod": "1",
      "Opis": "dziecko"
    },
    {
      "Kod": "2",
      "Opis": "małżonek, rodzice, ojczym, macocha, teściowie, dziadkowie, wnuki, rodzeństwo"
    },
    {
      "Kod": "3",
      "Opis": "inne osoby"
    }
  ],
  "error": [],
  "raw": {
    "PobierzSlownikKodowPokrewienstwaResponse": {
      "KodPokrewienstwa": [
        {
          "Kod": "1",
          "Opis": "dziecko"
        },
        {
          "Kod": "2",
          "Opis": "małżonek, rodzice, ojczym, macocha, teściowie, dziadkowie, wnuki, rodzeństwo"
        },
        {
          "Kod": "3",
          "Opis": "inne osoby"
        }
      ],
      "Rezultat": {
        "KodBledu": "0",
        "OpisBledu": "OK"
      }
    }
  }
}
```


## Pobierz słownik powodów anulowania

```
GET /ezla/dictionary/reasonsOfCancel
```

### Prawidłowa odpowiedź

```json
{
  "message": "",
  "body": [
    {
      "Kod": "U",
      "Nazwa": "błąd w danych identyfikacyjnych ubezpieczonego",
      "Opis": "Przyczyna o kodzie \"U\" ustawiana w przypadku błędnych danych identyfikacyjnych ubezpieczonego"
    },
    {
      "Kod": "P",
      "Nazwa": "błąd w danych płatnika składek",
      "Opis": "Przyczyna o kodzie \"P\" ustawiana w przypadku błędnych danych płatnika składek"
    },
    {
      "Kod": "N",
      "Nazwa": "błąd w danych dotyczących niezdolności do pracy",
      "Opis": "Przyczyna o kodzie \"N\" ustawiana w przypadku błędnych danych dotyczących niezdolności do pracy"
    },
    {
      "Kod": "M",
      "Nazwa": "błąd w danych miejsca udzielania świadczeń zdrowotnych",
      "Opis": "Przyczyna o kodzie \"M\" ustawiana w przypadku błędnych danych miejsca udzielania świadczeń zdrowotnych"
    },
    {
      "Kod": "I",
      "Nazwa": "inny powód",
      "Opis": "Przyczyna o kodzie \"I\" ustawiana w przypadku jeśli powód anulowania jest inny niż dostępnie wymienionych"
    },
    {
      "Kod": "B",
      "Nazwa": "błąd systemu",
      "Opis": "Przyczyna o kodzie \"B\" ustawiana w przypadku pojawienia się błędu systemu"
    },
    {
      "Kod": "A",
      "Nazwa": "błąd w danych adresowych ubezpieczonego",
      "Opis": "Przyczyna o kodzie \"A\" ustawiana w przypadku błędnych danych adresowych ubezpieczonego"
    },
    {
      "Kod": "E",
      "Nazwa": "błędne dane",
      "Opis": "Przyczyna o kodzie \"E\" ustawiana w przypadku błędnych danych - przypadek elektronizacji błędnie wypełnionego ZLA"
    },
    {
      "Kod": "X",
      "Nazwa": "błędna data wystawienia",
      "Opis": "Przyczyna o kodzie \"X\" ustawiana w przypadku błędnej dacie wystawienia ZLA - przypadek elektronizacji błędnie wypełnionego ZLA"
    }
  ],
  "error": [],
  "raw": {
    "PobierzSlownikPrzyczynAnulowaniaResponse": {
      "Przyczyna": [
        {
          "Kod": "U",
          "Nazwa": "błąd w danych identyfikacyjnych ubezpieczonego",
          "Opis": "Przyczyna o kodzie \"U\" ustawiana w przypadku błędnych danych identyfikacyjnych ubezpieczonego"
        },
        {
          "Kod": "P",
          "Nazwa": "błąd w danych płatnika składek",
          "Opis": "Przyczyna o kodzie \"P\" ustawiana w przypadku błędnych danych płatnika składek"
        },
        {
          "Kod": "N",
          "Nazwa": "błąd w danych dotyczących niezdolności do pracy",
          "Opis": "Przyczyna o kodzie \"N\" ustawiana w przypadku błędnych danych dotyczących niezdolności do pracy"
        },
        {
          "Kod": "M",
          "Nazwa": "błąd w danych miejsca udzielania świadczeń zdrowotnych",
          "Opis": "Przyczyna o kodzie \"M\" ustawiana w przypadku błędnych danych miejsca udzielania świadczeń zdrowotnych"
        },
        {
          "Kod": "I",
          "Nazwa": "inny powód",
          "Opis": "Przyczyna o kodzie \"I\" ustawiana w przypadku jeśli powód anulowania jest inny niż dostępnie wymienionych"
        },
        {
          "Kod": "B",
          "Nazwa": "błąd systemu",
          "Opis": "Przyczyna o kodzie \"B\" ustawiana w przypadku pojawienia się błędu systemu"
        },
        {
          "Kod": "A",
          "Nazwa": "błąd w danych adresowych ubezpieczonego",
          "Opis": "Przyczyna o kodzie \"A\" ustawiana w przypadku błędnych danych adresowych ubezpieczonego"
        },
        {
          "Kod": "E",
          "Nazwa": "błędne dane",
          "Opis": "Przyczyna o kodzie \"E\" ustawiana w przypadku błędnych danych - przypadek elektronizacji błędnie wypełnionego ZLA"
        },
        {
          "Kod": "X",
          "Nazwa": "błędna data wystawienia",
          "Opis": "Przyczyna o kodzie \"X\" ustawiana w przypadku błędnej dacie wystawienia ZLA - przypadek elektronizacji błędnie wypełnionego ZLA"
        }
      ],
      "Rezultat": {
        "KodBledu": "0",
        "OpisBledu": "OK"
      }
    }
  }
}
```


## Pobierz słownik powodów unieważnienia

```
GET /ezla/dictionary/reasonsOfInvalidation
```

### Poprawna odpowiedź

```json
{
  "message": "",
  "body": [
    {
      "Kod": "A",
      "Nazwa": "anulowanie",
      "Opis": ""
    },
    {
      "Kod": "S",
      "Nazwa": "zniszczenie",
      "Opis": ""
    },
    {
      "Kod": "Z",
      "Nazwa": "zaginięcie",
      "Opis": ""
    },
    {
      "Kod": "K",
      "Nazwa": "kradzież",
      "Opis": ""
    }
  ],
  "error": [],
  "raw": {
    "PobierzSlownikPrzyczynUniewaznieniaResponse": {
      "Przyczyna": [
        {
          "Kod": "A",
          "Nazwa": "anulowanie",
          "Opis": ""
        },
        {
          "Kod": "S",
          "Nazwa": "zniszczenie",
          "Opis": ""
        },
        {
          "Kod": "Z",
          "Nazwa": "zaginięcie",
          "Opis": ""
        },
        {
          "Kod": "K",
          "Nazwa": "kradzież",
          "Opis": ""
        }
      ],
      "Rezultat": {
        "KodBledu": "0",
        "OpisBledu": "OK"
      }
    }
  }
}
```


## Pobierz profil rehabilitacji dla kodu ICD 10

```
GET /ezla/dictionary/rehabilitationProfile/{icd10Code}
```

### Poprawna odpowiedź

```json
{
  "message": "",
  "body": "",
  "error": [],
  "raw": {
    "SprawdzProfilRehabilitacjiResponse": {
      "Rezultat": {
        "KodBledu": "0",
        "OpisBledu": "OK"
      }
    }
  }
}
```

## Wystawienie eZLA

Zanim wyślemy eZLA zalecamy weryfikację dokumentu i jego zwartości.

```
POST /ezla/validate
```

Gdy nie będzie już błędów (ostrzeżenia można pominąć), to można przystąpić bezpiecznie do wystawienie eZLA.

```
POST /ezla/send
```

### Treść żądania (identyczna dla weryfikacji i wystawienia)

```json
{
  "institution": 1, // 1 = ZUS, 2 = KRUS, 3 = INNE_W_POLSCE, 4 = W_INNYM_PANSTWIE
  "patient": {
    "identifier": [
      {
        "type": "pesel",
        "value": "40010151673"
      }
    ],
    "name": [
      {
        "family": "Senior",
        "given": [
          "Sylwester"
        ]
      }
    ],
    "telecom": [
      {
        "value": "+48131231230"
      }
    ],
    "gender": "M",
    "birthDate": "1940-01-01",
    "address": [
      {
        "street": "Wroc\u0142awska",
        "houseNumber": "11A",
        "unitId": "",
        "city": "Zielona G\u00f3ra",
        "postalCode": "65-001",
        "country": "Polska"
      }
    ],
    "language": "pl"
  },
  "from": "", // data zwolnienia od YYYY-MM-DD
  "to": "", // data zwolnienia do YYYY-MM-DD
  "hospitalization": {
    "from": null, // null or YYYY-MM-DD
    "to": null // null or YYYY-MM-DD
  },
  "suggestion": "", // 1 – chory powinien leżeć, 2 – chory może chodzić
  "disease1": "", // A, B, C, D, E
  "disease2": "", // A, B, C, D, E
  "disease3": "", // A, B, C, D, E
  "disease4": "", // A, B, C, D, E
  "familyMember": {
    "code": null, 
        // 1 - dziecko
        // 2 - małżonek, rodzice, ojczym, macocha, teściowie, dziadkowie, wnuki, rodzeństwo
        // 3 - inne osoby
    "date": null // null lub YYYY-MM-DD
  },
  "payers": [{
    "type": null, // 1 = nip, 2 = pesel, 3 = paszport
    "id": null // nip, pesel, paszport
  }],
  "organization": {
    "identifier": [
      {
        "type": "nip",
        "value": "6391001234"
      }
    ],
    "name": "BioStat Sp. z o.o.",
    "address": [{
      "street": "ul. Dubois",
      "houseNumber": "5A",
      "unitId": "13",
      "city": "Warszawa",
      "postalCode": "00-184"
    }]
  },
  "practitioner": {
    "identifier": [
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
    ]
  },
  "dateOfDigitalization": "", // data wystawienia YYYY-MM-DD
  "reasonInPast": "", // uzasadnienie zwolnienia wstecznego, pominąć dla diseaseStatisticalNumber z psychiatrii
  "zoz": false, // bool, true - gdy pacjent musi zostać w ośrodku ZOZ
  "doNotSend": false, // bool, true - gdy nie chcemy wysyłać informacji płatnikowi automatycznie
  "diseaseStatisticalNumber": "A10" // ICD 10 dictionary - only root codes, without .1 etc.
}
```


### Odpowiedź

eZLA składa się z wielu różnych dokumentów.

- 2 dokumenty dla ezla w trybie bieżącym dla jednego płatnika
- 4 dokumenty dla ezla w trybie bieżącym dla 2 płatników
- 8 dokumentów dla ezla w trybie bieżącym i wstecznym dla 2 płatników

Gdzie połowa wszystkich dokumentów to kopie a druga połowa to oryginały.
Każdy dokument jest drukowany osobno i wysyłany osobno do płatników/zus.

```json
{
  "message": "",
  "body": "",
  "error": [],
  "raw": {
    "CustomWalidujDokumentyResponse": {
      "Rezultat": {
        "KodBledu": "0",
        "OpisBledu": "OK"
      },
      "RezultatWalidacji": [
        {
          "document": 0,
          "content": "<?xml version=\"1.0\" encoding=\"UTF-8\" standalone=\"no\"?>\n<KEDU xmlns=\"http:\/\/www.zus.pl\/2015\/KED_ZLA_1\" xmlns:xsi=\"http:\/\/www.w3.org\/2001\/XMLSchema-instance\" wersja_schematu=\"1\">\n  <naglowek.KEDU>\n    <program>\n      <producent>Medfile<\/producent>\n      <symbol>eZLA<\/symbol>\n      <wersja>1.0<\/wersja>\n    <\/program>\n  <\/naglowek.KEDU>\n  <ZUSZLA id_dokumentu=\"155565\" kolejnosc=\"0\">\n    <I>\n      <p1\/>\n      <p2>ORYGINAŁ<\/p2>\n    <\/I>\n    <II>\n      <p1>68071892605<\/p1>\n      <p2>Filip<\/p2>\n      <p3>Jakubowski<\/p3>\n      <p4>1<\/p4>\n    <\/II>\n    <III>\n      <p1>25624<\/p1>\n      <p2>Kielce<\/p2>\n      <p3>Mieszka I<\/p3>\n      <p4>1<\/p4>\n    <\/III>\n    <IV>\n      <p1>\n        <p1>2020-11-27<\/p1>\n        <p2>2020-11-27<\/p2>\n      <\/p1>\n      <p3>2<\/p3>\n      <p8>K02<\/p8>\n    <\/IV>\n    <V>\n      <p1>1<\/p1>\n      <p2>6707035254<\/p2>\n    <\/V>\n    <VI>\n      <p1>PRZYCHODNIA TARCZÓWKOWATYMI<\/p1>\n      <p2>16423<\/p2>\n      <p3>Zdręby<\/p3>\n      <p4>Zdręby<\/p4>\n      <p5>16423<\/p5>\n    <\/VI>\n    <VII>\n      <p1>1063324<\/p1>\n      <p2>THEO<\/p2>\n      <p3>KANIA<\/p3>\n    <\/VII>\n    <VIII>\n      <p1>2020-11-26<\/p1>\n      <p2>2020-11-26<\/p2>\n      <p6>false<\/p6>\n      <p7>false<\/p7>\n      <p8>3962382940<\/p8>\n    <\/VIII>\n  <\/ZUSZLA>\n<\/KEDU>\n",
          "type": "ORYGINAŁ",
          "error": [
            {
              "NrRef": [
                "0",
                "1"
              ],
              "Rodzaj": "OSTRZEZENIE",
              "KodBledu": "W145",
              "OpisBledu": "Wymagane pole: Seria i numer zaświadczenia.",
              "Lokalizacja": "KEDU\/ZUSZLA\/I\/p1\/p1 KEDU\/ZUSZLA\/I\/p1\/p2"
            },
            {
              "NrRef": [
                "0"
              ],
              "Rodzaj": "OSTRZEZENIE",
              "KodBledu": "185",
              "OpisBledu": "Identyfikator KEDU nie został zarezerwowany dla podanej serii i numeru ZLA",
              "Lokalizacja": "\/KEDU\/ZUSZLA[@id_dokumentu] "
            }
          ]
        },
        {
          "document": 1,
          "content": "<?xml version=\"1.0\" encoding=\"UTF-8\" standalone=\"no\"?>\n<KEDU xmlns=\"http:\/\/www.zus.pl\/2015\/KED_ZLA_1\" xmlns:xsi=\"http:\/\/www.w3.org\/2001\/XMLSchema-instance\" wersja_schematu=\"1\">\n  <naglowek.KEDU>\n    <program>\n      <producent>Medfile<\/producent>\n      <symbol>eZLA<\/symbol>\n      <wersja>1.0<\/wersja>\n    <\/program>\n  <\/naglowek.KEDU>\n  <ZUSZLA id_dokumentu=\"155566\" kolejnosc=\"1\">\n    <I>\n      <p1\/>\n      <p2>KOPIA<\/p2>\n    <\/I>\n    <II>\n      <p1>68071892605<\/p1>\n      <p2>Filip<\/p2>\n      <p3>Jakubowski<\/p3>\n      <p4>1<\/p4>\n    <\/II>\n    <III>\n      <p1>25624<\/p1>\n      <p2>Kielce<\/p2>\n      <p3>Mieszka I<\/p3>\n      <p4>1<\/p4>\n    <\/III>\n    <IV>\n      <p1>\n        <p1>2020-11-27<\/p1>\n        <p2>2020-11-27<\/p2>\n      <\/p1>\n      <p3>2<\/p3>\n    <\/IV>\n    <V>\n      <p1>1<\/p1>\n      <p2>6707035254<\/p2>\n    <\/V>\n    <VI>\n      <p1>PRZYCHODNIA TARCZÓWKOWATYMI<\/p1>\n      <p2>16423<\/p2>\n      <p3>Zdręby<\/p3>\n      <p4>Zdręby<\/p4>\n      <p5>16423<\/p5>\n    <\/VI>\n    <VII>\n      <p1>1063324<\/p1>\n      <p2>THEO<\/p2>\n      <p3>KANIA<\/p3>\n    <\/VII>\n    <VIII>\n      <p1>2020-11-26<\/p1>\n      <p2>2020-11-26<\/p2>\n      <p6>false<\/p6>\n      <p7>false<\/p7>\n      <p8>3962382940<\/p8>\n    <\/VIII>\n  <\/ZUSZLA>\n<\/KEDU>\n",
          "type": "KOPIA",
          "error": [
            {
              "NrRef": [
                "0",
                "1"
              ],
              "Rodzaj": "OSTRZEZENIE",
              "KodBledu": "W145",
              "OpisBledu": "Wymagane pole: Seria i numer zaświadczenia.",
              "Lokalizacja": "KEDU\/ZUSZLA\/I\/p1\/p1 KEDU\/ZUSZLA\/I\/p1\/p2"
            },
            {
              "NrRef": [
                "1"
              ],
              "Rodzaj": "OSTRZEZENIE",
              "KodBledu": "185",
              "OpisBledu": "Identyfikator KEDU nie został zarezerwowany dla podanej serii i numeru ZLA",
              "Lokalizacja": "\/KEDU\/ZUSZLA[@id_dokumentu] "
            }
          ]
        }
      ]
    }
  }
}
```

## Pobieranie dokumentu eZLA

```
GET /ezla/document/{ezlaSeries}/{ezlaNumber}
```

Zwraca dokument XML z dokumentem eZLA i jego statusem.

## Wydruk dokumentu eZLA

```
GET /ezla/document/{ezlaSeries}/{ezlaNumber}/print
```

Zwraca dokument HTML z naniesionym dokumentem eZLA.

## Lista wystawionych ZLA przez lekarza

Umożliwia pobranie listy ZLA wystawionych przez lekarza.
Możliwe jest użycie dodatkowych parametrów:

```
GET /ezla/listprocessedzla
```

- [stronnicowania](Pagination.md)
- [sortowania](Sort.md)
- [filtrowania](Filter.md)

### Przykłady

- `GET /ezla/listprocessedzla`
- `GET /ezla/listprocessedzla?start=0&count=20&sort_col[1]=DataWystawieniaZla&sort_order[1]=DESC&sort_col[2]=NumerZla&sort_order[2]=DESC`
- `GET /ezla/listprocessedzla?start=0&count=20&sort_col=DataWystawieniaZla&sort_order=DESC&cond_col[0]=SeriaZla&cond_val[0]=ZY&cond_col[1]=NumerZla&cond_val[1]=9952547`
- `GET /ezla/listprocessedzla?cond_col[0]=DataWystawieniaZla&cond_op[0]=largerThanOrEqualTo&cond_val[0]=2021-01-01&cond_col[1]=DataWystawieniaZla&cond_op[1]=lessThan&cond_val[1]=2021-02-01`

### Przykładowa odpowiedź

```json
{
  "message": "",
  "body": {
    "totalCount": 418,
    "zla": [
      {
        "PaszportUbezpieczonego": "BM1576116",
        "ImieUbezpieczonego": "Sylwester",
        "NazwiskoUbezpieczonego": "Senior",
        "NiezdolnoscDoPracyOd": "2021-02-09",
        "NiezdolnoscDoPracyDo": "2021-02-20",
        "LiczbaDniPobytuWSzpitalu": 0,
        "IdentyfikatorPlatnika": "3962382940",
        "TypIdentyfikatoraPlatnika": "1",
        "StatystycznyNumerChoroby": "K02",
        "NpwzLekarza": "1063324",
        "ImieLekarza": "Theo",
        "NazwiskoLekarza": "Kania",
        "StatusZla": "Wystawione",
        "DataWystawieniaZla": "2021-02-09",
        "SeriaZla": "ZY",
        "NumerZla": "9952547",
        "ImieAsystenta": "",
        "NazwiskoAsystenta": ""
      }
    ]
  },
  "error": [],
  "raw": {
    "PobierzListePrzetworzonychZlaLekarzaResponse": {
      "Rezultat": {
        "KodBledu": "0",
        "OpisBledu": "OK"
      },
      "LiczbaWszystkichRekordow": 418,
      "ZaswiadczenieLekarskie": [
        {
          "PaszportUbezpieczonego": "BM1576116",
          "ImieUbezpieczonego": "Sylwester",
          "NazwiskoUbezpieczonego": "Senior",
          "NiezdolnoscDoPracyOd": "2021-02-09",
          "NiezdolnoscDoPracyDo": "2021-02-20",
          "LiczbaDniPobytuWSzpitalu": 0,
          "IdentyfikatorPlatnika": "3962382940",
          "TypIdentyfikatoraPlatnika": "1",
          "StatystycznyNumerChoroby": "K02",
          "NpwzLekarza": "1063324",
          "ImieLekarza": "Theo",
          "NazwiskoLekarza": "Kania",
          "StatusZla": "Wystawione",
          "DataWystawieniaZla": "2021-02-09",
          "SeriaZla": "ZY",
          "NumerZla": "9952547",
          "ImieAsystenta": "",
          "NazwiskoAsystenta": ""
        }
      ]
    }
  }
}
```

### Filtrowanie wyników

Parametr:
- `cond_col` - jedno lub wiele pól słownikowych (nazwa pola), po którym ma być filtrowanie.

- `cond_op` - pole słownikowe; operator logiczny filtrowania.
    - Dostępne wartości:
        - `equalTo` - *domyślny operator*
        - `isNotEmpty`
        - `lessThan`
        - `lessThanOrEqualTo`
        - `largerThan`
        - `largerThanOrEqualTo`
        - `contains`
        - `startsWith`
        - `isEmpty`

- `cond_val` - warunek filtrowania lub warunki (odpowiednio do `cond_col`)
- `cond_concat` - w jaki sposób mają być złączone warunki
    - Dostępne wartości:
        - `AND` - *domyślny operator*
        - `OR`
    - Odpowiedź usługi będzie sumą wyników warunku filtrowania w przypadku logicznego złączenia warunków `OR` oraz iloczynem warunków w przypadku `AND`

#### Przykład
- `GET /ezla/listprocessedzla?cond_col=NumerZla&cond_val=9952547`
- `GET /ezla/listprocessedzla?cond_col[0]=NazwiskoUbezpieczonego&cond_val[0]=Senior`
- `GET /ezla/listprocessedzla?cond_col[0]=DataWystawieniaZla&cond_op[0]=largerThanOrEqualTo&cond_val[0]=2021-01-01&cond_col[1]=DataWystawieniaZla&cond_op[1]=lessThan&cond_val[1]=2021-02-01&cond_col=AND`

### Stronicowanie wyników

Parametr:
- `start` - numer rekordu, od którego usługa ma zwracać wyniki; `0` oznacza pierwszy rekord spełniający kryteria wyszukiwania i sortowania.
- `count` - oczekiwana maksymalna liczba rekordów do zwrócenia przez usługę.

#### Przykład
`GET /ezla/listprocessedzla?start=0&count=20`

### Sortowanie wyników

Parametr:
- `sort_col` - jedno lub wiele pól słownikowych (nazwa pola), po którym ma być sortowanie.
  W przypadku wielu pól sortowanie odbywa się w kolejności ich wystąpienia.
- `sort_order` - kierunek sortowania. Można zdefiniować dla każdego pola oddzielnie lub dla wszystkich oddzielnie.
    - Dostępne wartości:
        - `ASC`
        - `DESC`

#### Przykład
- `GET /ezla/listprocessedzla?sort_col=DataWystawieniaZla`
- `GET /ezla/listprocessedzla?sort_col=DataWystawieniaZla&sort_order=DESC`
- `GET /ezla/listprocessedzla?sort_col=DataWystawieniaZla&sort_order=DESC`
- `GET /ezla/listprocessedzla?sort_col[0]=DataWystawieniaZla&sort_order[0]=DESC&sort_col[1]=NumerZla&sort_order[1]=ASC`


