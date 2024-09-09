# Usługa: e-Recepta - obsługa schematów dawkowania (na potrzeby recepty 365)
Podczas wystawiania recept - zamiast zwykłego, opisowego dawkowania możliwe jest przekazanie dawkowania za pomocą schematów.
Schematy dawkowania (wg stanu na sierpień 2024r.) wymagane są w przypadku recepty rocznej na większość leków z kategorii: Rp, Rpz lub leków refundowanych. Sytuacja ta ulega zmianie. Kuracje można użyć również dla recept zwykłych.

Zmiany w schematach po wprowadzeniu funkcjonalności podstawowej:
1. Obsługa cykli dawkowania.
   Użycie elementu ```"dosageRepeat"``` umożliwia powtórzenie kuracji opisanej w schematach.
3. Obsługa przerw w kuracji.
   Sekwencja dawkowania umożliwia określenie przerwy w dawkowaniu - w elemencie ```"doseQuantity"/"quantity"``` należy przekazać wartość ```0```   
4. Obsługa informacji dla pacjenta przekazywanych w schematach dawkowania.  
  W ramach schematów dawkowania można przekazać element ```"infoForPatient"``` zawierający informację dla pacjenta.  
  Element stosowany jest zamiennie z elementem ```"dosageInstruction"```, który przekazywany jest w przypadku recepta niezawierających schematów dawkowania.

## Przykłady
### Dawkowanie *z użyciem 1 sekwencji dawkowania*
- Sekwencja 1: Przez 32dni 3x dziennie po 1 tabletce

```json
[
  "erecepta": [{
    "id": "0000000000000000025326", // unikalny numer dokumentu recepty nadany przez implementatora w ramach oidRoot (przestrzeni organizacji)
    "date": "2024-04-22", // data wystawienia recepty (zawsze data dzisiejsza)
    "type": "prepared", // prepared - gotowy lek, recipe - receptura własna
    "organization": "idabc", // uuid
    "practitioner": "idxyz", // uuid
    "patient": {
      "identifier": [
        {
          "type": "pesel",
          "value": "600322236"
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
          "value": "+48131231230" // opcjonalny
        }
      ],
      "address": [{
          "street": "Wrocławska",
          "houseNumber": "11A",
          "unitId": "3", // mieszkanie
          "city": "Zielona Góra",
          "postalCode": "00-184",
          "country": "Polska"
      }],
      "nfz": "07", // opcjonalne - jak nie ma to X
      "gender": "M",  // płeć pacjenta
      "birthDate": "1940-01-01",  // data urodzenia pacjenta
      "entitlements": [
        {
          "entitlement": "IB", // uprawnienia dodatkowe pacjenta
          "document": "Legitymacja nr. 12312321/23" // numer dokumentu upoważniającego do dodatkowych uprawnień
        }
      ]
    },
    "medication": {
      "name": "Apap Noc",
      "code": "100110151", // kod producenta
      "ean": "05909990960132", // kod EAN leku
      "kdlek": "Rp", // Rp, Rpw, Rpz, OTC
      "payment": "100%", // opcjonalne (domyślnie 100%) R, B, 30%, 50%, 100%
      "package": {
        "quantity": 24, // ilość leku w opakowaniu
        "unit": "tabl." // opcjonalne (domyślnie szt.)
      }
    },
    "dosage": [ // dawkowanie 3x dziennie przez 32 dni po 1 tabletce
      {
        "duration": {
          "quantity": 32,
          "unit": "d"
        },
        "period": {
          "quantity": 24,
          "unit": "h"
        },
        "frequency": 3,
        "doseQuantity": {
          "quantity": 1,
          "unit": "tabl."
        }
      }
    ],
    "kind": "PF", // PA - proauctore, PF - profamiliae, ZW - zwykła (domyślnie)
    "issueMode": "Z", // Z -Zwykła, F - Farmaceutyczna, P - Pielęgniarska, PL - Pielęgniarska na zlecenie lekarza
    "substanceAdminSubstitution": "N", // Nie pozwalaj na zamienniki
    "priorityCode": "UR", // CITO opcjonalne
    "infoForPatient": "popić dużą ilością wody", // dodatkowa informacja dla pacjenta w przypadku użycia schematów dawkowania
    "dispenseRequest": {
        "quantity": 4.0, // ile opakowań/unit wydać
        "unit": "op.", // opcjonalny
        "infoForPerformer": "proszę wymieszać", // opcjonalne informacja dla wydającego leki
        "validityPeriod": {
          "start": "2024-04-22", // opcjonalnie (domyślnie brak) od kiedy można zrealizować receptę, dla recepty rocznej równa dacie wystawienia
          "end": "2025-04-22" // opcjonalnie dla recepty 365 (+1 rok od dnia wystawienia)
        }
    }
  }]
]
```

### Dawkowanie *z użyciem 2 sekwencji*, recepta zwykła
- Sekwencja 1: Przez 7 dni po 1 tabletce co 8h
- Sekwencja 2: następnie 13 dni 2x dziennie po 1 tabletce

```json
[
  "erecepta": [{
    "id": "0000000000000000025326", // unikalny numer dokumentu recepty nadany przez implementatora w ramach oidRoot (przestrzeni organizacji)
    "date": "2024-04-22", // data wystawienia recepty (zawsze data dzisiejsza)
    "type": "prepared", // prepared - gotowy lek, recipe - receptura własna
    "organization": "idabc", // uuid
    "practitioner": "idxyz", // uuid
    "patient": {
      "identifier": [
        {
          "type": "pesel",
          "value": "600322236"
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
          "value": "+48131231230" // opcjonalny
        }
      ],
      "address": [{
          "street": "Wrocławska",
          "houseNumber": "11A",
          "unitId": "3", // mieszkanie
          "city": "Zielona Góra",
          "postalCode": "00-184",
          "country": "Polska"
      }],
      "nfz": "07", // opcjonalne - jak nie ma to X
      "gender": "M",  // płeć pacjenta
      "birthDate": "1940-01-01",  // data urodzenia pacjenta
      "entitlements": [
        {
          "entitlement": "IB", // uprawnienia dodatkowe pacjenta
          "document": "Legitymacja nr. 12312321/23" // numer dokumentu upoważniającego do dodatkowych uprawnień
        }
      ]
    },
    "medication": {
      "name": "Apap Noc",
      "code": "100110151", // kod producenta
      "ean": "05909990960132", // kod EAN leku
      "kdlek": "Rp", // Rp, Rpw, Rpz, OTC
      "payment": "100%", // opcjonalne (domyślnie 100%) R, B, 30%, 50%, 100%
      "package": {
        "quantity": 24, // ilość leku w opakowaniu
        "unit": "tabl." // opcjonalne (domyślnie szt.)
      }
    },
    "dosage": [ // Sekwencje. Przez 7 dni po 1 tabletce co 8h + Przez 13 dni 2x dziennie po 1 tabletce
      {
        "duration": {
          "quantity": 7,
          "unit": "d"
        },
        "period": {
          "quantity": 8,
          "unit": "h"
        },
        "frequency": 1,
        "doseQuantity": {
          "quantity": 1,
          "unit": "tabl."
        }
      },
      {
        "duration": {
          "quantity": 13,
          "unit": "d"
        },
        "period": {
          "quantity": 1,
          "unit": "d"
        },
        "frequency": 2,
        "doseQuantity": {
          "quantity": 1,
          "unit": "tabl."
        }
      }
    ],
    "kind": "PF", // PA - proauctore, PF - profamiliae, ZW - zwykła (domyślnie)
    "issueMode": "Z", // Z -Zwykła, F - Farmaceutyczna, P - Pielęgniarska, PL - Pielęgniarska na zlecenie lekarza
    "substanceAdminSubstitution": "N", // Nie pozwalaj na zamienniki
    "priorityCode": "UR", // CITO opcjonalne
    "infoForPatient": "popić dużą ilością wody", // dodatkowa informacja dla pacjenta w przypadku użycia schematów dawkowania
    "dispenseRequest": {
        "quantity": 2, // ile opakowań/unit wydać
        "unit": "op.", // opcjonalny
        "infoForPerformer": "proszę wymieszać", // opcjonalne informacja dla wydającego leki
        "validityPeriod": {
          "start": "2024-05-01" // opcjonalnie (domyślnie brak) data, od kiedy można zrealizować receptę
        }
    }
  }]
]
```

### Dawkowanie złożone *z użyciem doprecyzowania (podsekwencje)*
- Sekwencja 1: Przez 7 dni
- Podsekwencja 1_1: 1 tabletka rano
- Podsekwencja 1_2: 2 tabletki wieczorem, pomiędzy kolacją a porą snu
- Sekwencja 2: następnie 10 dni 2x dziennie od 1 do 2 tabletek,

```json
[
  "erecepta": [{
    "id": "0000000000000000025326", // unikalny numer dokumentu recepty nadany przez implementatora w ramach oidRoot (przestrzeni organizacji)
    "date": "2024-04-22",
    "type": "prepared", // prepared - gotowy lek, recipe - receptura własna
    "organization": "idabc", // uuid
    "practitioner": "idxyz", // uuid
    "patient": {
      "identifier": [
        {
          "type": "pesel",
          "value": "600322236"
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
          "value": "+48131231230" // opcjonalny
        }
      ],
      "address": [{
          "street": "Wrocławska",
          "houseNumber": "11A",
          "unitId": "3", // mieszkanie
          "city": "Zielona Góra",
          "postalCode": "00-184",
          "country": "Polska"
      }],
      "nfz": "07", // opcjonalne - jak nie ma to X
      "gender": "M",  // płeć pacjenta
      "birthDate": "1940-01-01",  // data urodzenia pacjenta
      "entitlements": [
        {
          "entitlement": "IB", // uprawnienia dodatkowe pacjenta
          "document": "Legitymacja nr. 12312321/23" // numer dokumentu upoważniającego do dodatkowych uprawnień
        }
      ]
    },
    "medication": {
      "name": "Apap Noc",
      "code": "100110151", // kod producenta
      "ean": "05909990960132", // kod EAN leku
      "kdlek": "Rp", // Rp, Rpw, Rpz, OTC
      "payment": "100%", // opcjonalne (domyślnie 100%) R, B, 30%, 50%, 100%
      "package": {
        "quantity": 24, // ilość leku w opakowaniu
        "unit": "tabl." // opcjonalne (domyślnie szt.)
      }
    },
    "dosage": [
      {  // Sekwencja 1: Przez 7 dni
        "duration": {
          "quantity": 7,
          "unit": "d"
        },
        "period": {
          "quantity": 1,
          "unit": "d"
        },
        "dosage": [
          { // Podsekwencja 1_1: 1 tabletka rano
            "when": [
              {
                "code": "MORN",
                "codeSystem": "2.16.840.1.113883.4.642.1.76",
                "display": "rano"
              }
            ],
            "doseQuantity": {
              "quantity": 1,
              "unit": "tabl."
            }
          },
          {  // Podsekwencja 1_2: 2 tabletki wieczorem, pomiędzy kolacją a porą snu
            "when": [
              {
                "code": "EVE",
                "codeSystem": "2.16.840.1.113883.4.642.1.76",
                "display": "wieczorem"
              },
              {
                "code": "ICV",
                "codeSystem": "2.16.840.1.113883.5.139",
                "display": "pomiędzy kolacją a porą snu"
              }
            ],
            "doseQuantity": {
              "quantity": 2,
              "unit": "tabl."
            }
          }
        ]
      },
      { // Sekwencja 2: Przez kolejnych 10 dni 2x dziennie od 1 do 2 tabletek,
        "duration": {
          "quantity": 10,
          "unit": "d"
        },
        "period": {
          "quantity": 24,
          "unit": "h"
        },
        "frequency": 2,
        "doseRange": { // liczba tabletek od - do
          "low": {
            "quantity": 1,
            "unit": "tabl."
          },
          "high": {
            "quantity": 2,
            "unit": "tabl."
          }
        }
      }
    ],
    "kind": "PF", // PA - proauctore, PF - profamiliae, ZW - zwykła (domyślnie)
    "issueMode": "Z", // Z -Zwykła, F - Farmaceutyczna, P - Pielęgniarska, PL - Pielęgniarska na zlecenie lekarza
    "substanceAdminSubstitution": "N", // Nie pozwalaj na zamienniki
    "priorityCode": "UR", // CITO opcjonalne
    "dispenseRequest": {
        "quantity": 3, // ile opakowań/unit wydać
        "unit": "op.", // opcjonalny
        "infoForPerformer": "proszę wymieszać", // opcjonalne informacja dla wydającego leki
        "validityPeriod": {
          "start": "2024-04-22", // opcjonalnie (domyślnie brak) od kiedy można zrealizować receptę, dla recepty rocznej równa dacie wystawienia
          "end": "2025-04-22" // opcjonalnie dla recepty 365 (+1 rok od dnia wystawienia)        }
    }
  }]
]
```

### Dawkowanie złożone *z użyciem przerwy oraz cykli*
- Sekwencja 1: Przez 5 dni co 1 dzień 3 tabl. dziennie
- Sekwencja 2: następnie 2 dni przerwy
- Sekwencja 3: następnie 7 dni 1 x dziennie po 1 tabl.
- Cykl powtórzyć 3 razy

```json
[
  "erecepta": [{
    "id": "0000000000000000025326", 
    "date": "2024-09-10",
    "type": "prepared", // prepared - gotowy lek, recipe - receptura własna
    "organization": "idabc", 
    "practitioner": "idxyz", 
    "patient": {
      "identifier": [
        {
          "type": "pesel",
          "value": "600322236"
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
      "address": [{
          "street": "Wrocławska",
          "houseNumber": "11A",
          "unitId": "3", 
          "city": "Zielona Góra",
          "postalCode": "00-184",
          "country": "Polska"
      }],
      "nfz": "07",
      "gender": "M",  // płeć pacjenta
      "birthDate": "1940-01-01",  // data urodzenia pacjenta
      "entitlements": [
        {
          "entitlement": "IB", 
          "document": "Legitymacja nr. 12312321/23"
        }
      ]
    },
    "medication": {
      "name": "Apap Noc",
      "code": "100110151", 
      "ean": "05909990960132", 
      "kdlek": "Rp", // Rp, Rpw, Rpz, OTC
      "payment": "100%", 
      "package": {
        "quantity": 24, 
        "unit": "tabl." 
      }
    },
    "dosage": [
      {  // Sekwencja 1: 5 dni 3 razy dziennie po 1 tabl.
        "duration": {
          "quantity": 5,
          "unit": "d"
        },
        "period": {
          "quantity": 24,
          "unit": "h"
        },
        "frequency": 3,
        "doseQuantity": {
          "quantity": 1,
          "unit": "tabl."
        }
      },
      {  // Sekwencja 2: następnie 2 dni przerwy
        "duration": {
          "quantity": 2,
          "unit": "d"
        },
        "period": {
          "quantity": 24,
          "unit": "h"
        },
        "frequency": 1,
        "doseQuantity": {
          "quantity": 0,
          "unit": "tabl."
        }
      },
      {  // Sekwencja 3: następnie 7 dni co 1 dzień 1 tabl.
        "duration": {
          "quantity": 7,
          "unit": "d"
        },
        "period": {
          "quantity": 1,
          "unit": "d"
        },
        "frequency": 1,
        "doseQuantity": {
          "quantity": 1,
          "unit": "tabl."
        }
      }
    ],
    "dosageRepeat": 3, // opcjonalnie (domyślnie 0) - powtórzenie cyklu kuracji
    "kind": "ZW",
    "issueMode": "Z",
    "substanceAdminSubstitution": "N", 
    "priorityCode": "UR", 
    "infoForPatient" : "w casie przerwy dużo odpoczywać", // opcjonalny dla schentmaów dawkowania 
    "dispenseRequest": {
        "quantity": 3, 
        "unit": "op.", 
        "infoForPerformer": "",
        "validityPeriod": {
          "start": "2024-09-10", // opcjonalnie (domyślnie brak) oznaczenie recepty rocznej
          "end": "2025-09-10" // opcjonalnie (domyślnie brak) oznaczenie recepty rocznej
        }
    }
  }]
]
```


