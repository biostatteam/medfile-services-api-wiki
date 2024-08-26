# Usługa: e-Recepta - przykłady dawkowania dla recept 365

### Recepta 365 z dawkowaniem płaskim 3x dziennie przez 32 dni po 1 tabletce

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
      "kdlek": "Rp", // Rp lub Rpw
      "payment": "100%", // opcjonalne (domyślnie 100%) R, B, 30%, 50%, 100%

      "package": {
        "quantity": 24, // ilość leku w opakowaniu
        "unit": "tabl." // opcjonalne (domyślnie szt.)
      }
    },
    "dosage": [ // dawkowanie 3x dziennie przez 32 dni po 1 tabletce
      {
        "dosageInstruction": "nie z innymi lekami",
        "duration": {
          "quantity": 30,
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

    "dosageInstruction": "1x1 przed snem", // dawkowanie, minimum 3 znaki w formacie 1x1, w innych formatach dowolnie
    "dispenseRequest": {
        "quantity": 4.0, // ile opakowań/unit wydać
        "unit": "op.", // opcjonalny
        "infoForPerformer": "proszę wymieszać", // opcjonalne informacja dla wydającego leki
        "validityPeriod": {
          "start": "2024-04-22", // opcjonalnie (domyślnie brak) od kiedy można zrealizować
          "end": "2025-04-22" // opcjonalnie (domyślnie brak) do kiedy można zrealizować (maksymalnie 365 dni od start)
        }
    }
  }]
]
```

### Recepta 365 z dawkowaniem sekwencyjnym. Przez 7 dni po 1 tabletce co 8h + Przez 13 dni 2x dziennie po 1 tabletce

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
      "kdlek": "Rp", // Rp lub Rpw
      "payment": "100%", // opcjonalne (domyślnie 100%) R, B, 30%, 50%, 100%

      "package": {
        "quantity": 24, // ilość leku w opakowaniu
        "unit": "tabl." // opcjonalne (domyślnie szt.)
      }
    },
    "dosage": [ // Sekwencje. Przez 7 dni po 1 tabletce co 8h + Przez 13 dni 2x dziennie po 1 tabletce
      {
        "dosageInstruction": "Sekwencja 1 - uwagi",
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
        "dosageInstruction": "Sekwencja 2 - uwagi",
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

    "dosageInstruction": "1x1 przed snem", // dawkowanie, minimum 3 znaki w formacie 1x1, w innych formatach dowolnie
    "dispenseRequest": {
        "quantity": 2, // ile opakowań/unit wydać
        "unit": "op.", // opcjonalny
        "infoForPerformer": "proszę wymieszać", // opcjonalne informacja dla wydającego leki
        "validityPeriod": {
          "start": "2024-04-22", // opcjonalnie (domyślnie brak) od kiedy można zrealizować
          "end": "2025-04-22" // opcjonalnie (domyślnie brak) do kiedy można zrealizować (maksymalnie 365 dni od start)
        }
    }
  }]
]
```

### Recepta 365 z dawkowaniem sekwencyjnym i doprecyzowaniem (subsekwencje).
- Sekwencje z doprecyzowaniem (podsekwencje).
- Sekwencja 1: Przez 7 dni
- Podsekwencja 1_1: 1 tabletka rano
- Podsekwencja 1_2: 2 tabletki wieczorem, pomiędzy kolacją a porą snu
- Sekwencja 2: Przez kolejnych 10 dni 2x dziennie od 1 do 2 tabletek,

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
      "kdlek": "Rp", // Rp lub Rpw
      "payment": "100%", // opcjonalne (domyślnie 100%) R, B, 30%, 50%, 100%

      "package": {
        "quantity": 24, // ilość leku w opakowaniu
        "unit": "tabl." // opcjonalne (domyślnie szt.)
      }
    },
    // Sekwencje z doprecyzowaniem (podsekwencje).
    // Sekwencja 1: Przez 7 dni
    // Podsekwencja 1_1: 1 tabletka rano
    // Podsekwencja 1_2: 2 tabletki wieczorem, pomiędzy kolacją a porą snu
    // Sekwencja 2: Przez kolejnych 10 dni 2x dziennie od 1 do 2 tabletek,
    "dosage": [
      {
        "dosageInstruction": "Sekwencja 1 - uwagi",
        "duration": {
          "quantity": 7,
          "unit": "d"
        },
        "period": {
          "quantity": 1,
          "unit": "d"
        },
        "dosage": [
          {
            "dosageInstruction": "Podsekwencja 1_1 - uwagi",
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
          {
            "dosageInstruction": "Podsekwencja 1_2 - uwagi",
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
      {
        "dosageInstruction": "Sekwencja 2 - uwagi",
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

    "dosageInstruction": "1x1 przed snem", // dawkowanie, minimum 3 znaki w formacie 1x1, w innych formatach dowolnie
    "dispenseRequest": {
        "quantity": 3, // ile opakowań/unit wydać
        "unit": "op.", // opcjonalny
        "infoForPerformer": "proszę wymieszać", // opcjonalne informacja dla wydającego leki
        "validityPeriod": {
          "start": "2024-04-22", // opcjonalnie (domyślnie brak) od kiedy można zrealizować
          "end": "2025-04-22" // opcjonalnie (domyślnie brak) do kiedy można zrealizować (maksymalnie 365 dni od start)
        }
    }
  }]
]
```

