# Usługa: e-Recepta – schematy dawkowania i nadzór kuracji (P1)

Zamiast opisowego dawkowania (`dosageInstruction`) dawkowanie można przekazać w postaci ustrukturyzowanych schematów (`dosage`).

Schemat dawkowania jest **wymagany** dla recept objętych nadzorem kuracji P1:
- recepty rocznej (365),
- recepty zwykłej refundowanej.

Schematów można używać również na pozostałych receptach. Na receptach wyłączonych spod nadzoru kuracji schemat nie jest wymagany, a czasu trwania kuracji (`duration`) nie można podawać (patrz niżej).

Podanie czasu trwania kuracji (`duration`) oznacza receptę do nadzoru P1. P1 waliduje taką receptę i wylicza limity ilości leku do wydania – maksymalnie na 120 dni kuracji. Lek na kolejny okres kuracji może zostać wydany nie wcześniej niż po upływie ¾ poprzedniego okresu.

## Recepta roczna (365)

Receptę roczną oznacza się w elemencie `dispenseRequest.validityPeriod`:
- `start` – data wystawienia recepty,
- `end` – data wystawienia + 1 rok.

Podanie `end` oznacza receptę roczną. Recepta bez `end` jest receptą zwykłą.

Recepta roczna wymaga schematu dawkowania z czasem trwania kuracji (`duration`).

## Wyłączenia spod nadzoru kuracji

Czasu trwania kuracji (`duration`) **nie można** podawać na recepcie:
- farmaceutycznej,
- na import docelowy,
- na lek recepturowy,
- na produkt w opakowaniu złożonym (zestaw), dla którego nie określono dawkowania jednostkami alternatywnymi ani dawkowania opakowaniami,
- na produkt, dla którego P1 nie ma danych referencyjnych potrzebnych do nadzoru.

Taka recepta zostanie odrzucona przez P1 (`REG.WER.13314`). Wyjątkiem jest lek OTC – `duration` można podać, ale recepta nie jest objęta nadzorem.

## Elementy schematu dawkowania

1. **Cykle dawkowania** – element `"dosageRepeat"` (na poziomie recepty, obok `dosage`) powtarza kurację opisaną w schematach. Wartość oznacza liczbę **dodatkowych** powtórzeń (domyślnie `0`).
2. **Przerwy w kuracji** – sekwencja z wartością `0` w `"doseQuantity"`/`"quantity"` oznacza przerwę w dawkowaniu.
3. **Podsekwencje** – sekwencję można doprecyzować zagnieżdżoną tablicą `"dosage"`. Sekwencja nadrzędna określa `duration` i `period`, a każda podsekwencja – dawkę i porę przyjęcia (`"when"`, np. `MORN` – rano, `EVE` – wieczorem, `ICV` – pomiędzy kolacją a porą snu).
4. **Zakres dawki** – zamiast stałej dawki (`"doseQuantity"`) można podać zakres od–do (`"doseRange"` z `low` i `high`).
5. **Informacja dla pacjenta** – przy schematach dawkowania przekazywana w elemencie `"infoForPatient"`. Element stosowany jest zamiennie z `"dosageInstruction"`, który przekazuje się na receptach bez schematów dawkowania.

## Kodowanie leku na recepcie (słownik → `medication`)

Dane leku integrator uzupełnia na podstawie słownika leków dla wybranego opakowania (EAN).

| Pole `medication` | Dane ze słownika |
|---|---|
| `name` | nazwa produktu |
| `code` | kod producenta |
| `ean` | kod EAN |
| `kdlek` | kategoria dostępności (Rp, Rpw, Rpz, OTC) |
| `payment` | odpłatność – wybierana przez lekarza spośród poziomów dostępnych dla leku i wskazania (`indicationsJson[].paymentLevel`, `reimbursementPrices`) |
| `package` | pojemność opakowania (patrz niżej) |
| `superContent` | nadopakowanie (patrz niżej) |

### Opakowanie i nadopakowanie (`package`, `superContent`)

- `package` – pojemność opakowania,
- `superContent` – nadopakowanie: liczba i rodzaj opakowań; mnoży pojemność podaną w `package`.

Ilość leku = `package.quantity` × `superContent.quantity`, np. Zibor: 0.2 ml × 10 amp.-strzyk. = 2 ml.

Na tej podstawie P1 sprawdza, czy ilość leku wynikająca z liczby opakowań na recepcie odpowiada ilości wyliczonej ze schematu dawkowania.

Dane pochodzą z elementu `packages` słownika leków:

| Pole JSON | Źródło w słowniku |
|---|---|
| `package.quantity` | `packageVolume` |
| `package.unit` | `packageVolumeUnit` |
| `superContent.quantity` | `packageCount` (brak → `1`) |
| `superContent.unit` | `packageType` (brak → `""`) |

Jeśli słownik nie zawiera pojemności (`packageVolume`), a zawiera liczbę i rodzaj opakowań, to:
- `package` = liczba i rodzaj opakowań (`packageCount` + `packageType`),
- `superContent` = `1` bez jednostki.

**Brak danych w `packages`** – jeśli pola w elemencie `packages` w słowniku są puste, należy skorzystać z opisu słownego z elementu `package` w słowniku i samodzielnie rozdzielić go na `package` i `superContent`, np. „10 amp.-strzyk. 0,2 ml” → `package`: `0.2 ml`, `superContent`: `10 amp.-strzyk.`.

Należy pamiętać, że `"packages": null` wskazuje na lek spoza słownika RPL (e-zdrowie) – w takim przypadku dawkowanie strukturalne jest opcjonalne.

Przykłady:

| Produkt | `packageVolume` | `packageVolumeUnit` | `packageCount` | `packageType` | `package` | `superContent` |
|---|---|---|---|---|---|---|
| Amoksiklav | 14 | tabl. | – | – | `14 tabl.` | `1`, `""` |
| Zyrtec | 75 | ml | 1 | butelka | `75 ml` | `1 butelka` |
| Zibor | 0.2 | ml | 10 | amp.-strzyk. | `0.2 ml` | `10 amp.-strzyk.` |
| Coldrex MaxGrip | – | – | 14 | sasz. | `14 sasz.` | `1`, `""` |

```jsonc
"medication": {
  "name": "Zibor",
  "code": "100006198",
  "ean": "05909990039296",
  "kdlek": "Rp",
  "payment": "100%",
  "package": {         // pojemność opakowania
    "quantity": 0.2,   // packageVolume
    "unit": "ml"       // packageVolumeUnit
  },
  "superContent": {    // nadopakowanie
    "quantity": 10,    // packageCount
    "unit": "amp.-strzyk." // packageType
  }
}
```

### Dawkowanie alternatywne (`alternativeDose`)

Dla części leków słownik zawiera jednostki alternatywne (element `alternativeDose`, typ `JEDNOSTKA_ALTERNATYWNA`). Element jest opcjonalny – dla większości leków nie występuje.

| Pole w słowniku | Znaczenie |
|---|---|
| `packageUnit` | jednostka, w której można podać dawkę |
| `packageQuantity` | liczba tych jednostek w opakowaniu |

Dawkę (`doseQuantity` / `doseRange`) można podać w:
1. jednostce opakowania (`package.unit`, np. `sasz.`),
2. jednostce alternatywnej ze słownika (`alternativeDose[].packageUnit`), jeśli została zdefiniowana.

Przykład – opakowanie 90 saszetek, z których każda zawiera plaster:

```jsonc
// słownik
"alternativeDose": [
  { "type": "JEDNOSTKA_ALTERNATYWNA", "packageUnit": "sasz.",  "packageQuantity": "90.0" },
  { "type": "JEDNOSTKA_ALTERNATYWNA", "packageUnit": "plast.", "packageQuantity": "90.0" }
]

// recepta - obie formy dawki są poprawne
"doseQuantity": { "quantity": 1, "unit": "sasz." }
"doseQuantity": { "quantity": 1, "unit": "plast." }
```

## Jednostki

- **Czas** (`duration`, `period`): `h`, `d`, `wk`, `mo`. Przy wyliczaniu limitów P1 przyjmuje `mo` = 30 dni.
- **Częstotliwość**: `period` + `frequency`, np. „co 8 h” = `period: 8 h, frequency: 1`; „3× dziennie” = `period: 24 h, frequency: 3`.
- **Dawka** (`doseQuantity`, `doseRange`): w jednostce zgodnej z opakowaniem (np. `tabl.`, `kropl.`, `sasz.`) lub w jednostce alternatywnej. Ułamki tabletek zapisuje się dziesiętnie: 1/3 → `0.33333`, 2/3 → `0.66666`, „X i 1/3” → `X.33333`.
- **Ilość do wydania** (`dispenseRequest.quantity`) musi pokrywać ilość leku wyliczoną ze schematu dawkowania.

## Przykłady

### Recepta roczna, *1 sekwencja dawkowania*
- Sekwencja 1: przez 32 dni 3× dziennie po 1 tabletce

```jsonc
{
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
      "gender": "M", // płeć pacjenta
      "birthDate": "1940-01-01", // data urodzenia pacjenta
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
        "quantity": 24, // pojemność opakowania (packageVolume)
        "unit": "tabl." // packageVolumeUnit
      },
      "superContent": { // nadopakowanie
        "quantity": 1, // packageCount (brak → 1)
        "unit": "" // packageType (brak → "")
      }
    },
    "dosage": [ // 3× dziennie przez 32 dni po 1 tabletce
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
    "issueMode": "Z", // Z - zwykła, F - farmaceutyczna, P - pielęgniarska, PL - pielęgniarska na zlecenie lekarza
    "substanceAdminSubstitution": "N", // nie pozwalaj na zamienniki
    "priorityCode": "UR", // znacznik CITO, opcjonalny
    "infoForPatient": "popić dużą ilością wody", // informacja dla pacjenta przy schematach dawkowania
    "dispenseRequest": {
        "quantity": 4.0, // ile opakowań/unit wydać
        "unit": "op.", // opcjonalny
        "infoForPerformer": "proszę nie wydawać mniejszych opakowań", // opcjonalna informacja dla wydającego
        "validityPeriod": {
          "start": "2024-04-22", // opcjonalnie (domyślnie brak) od kiedy można zrealizować receptę; dla recepty rocznej równa dacie wystawienia
          "end": "2025-04-22" // opcjonalnie, dla recepty 365 (+1 rok od dnia wystawienia)
        }
    }
  }]
}
```

### Recepta zwykła refundowana, *2 sekwencje dawkowania*
- Sekwencja 1: przez 7 dni po 1 tabletce co 8 h
- Sekwencja 2: następnie przez 13 dni 2× dziennie po 1 tabletce

```jsonc
{
  "erecepta": [{
    "id": "0000000000000000025326", // unikalny numer dokumentu recepty nadany przez implementatora w ramach oidRoot (przestrzeni organizacji)
    "date": "2024-04-22", // data wystawienia recepty (zawsze data dzisiejsza)
    "type": "prepared", // prepared - gotowy lek, recipe - receptura własna
    "organization": "idabc", // uuid
    "practitioner": "idxyz", // uuid
    "patient": { … }, // jak w pierwszym przykładzie
    "medication": {
      "name": "Sulpiryd Hasco",
      "code": "100406970", // kod producenta
      "ean": "05909991380410", // kod EAN leku
      "kdlek": "Rp", // Rp, Rpw, Rpz, OTC
      "payment": "B", // bezpłatny do limitu - recepta refundowana, schemat dawkowania wymagany
      "package": {
        "quantity": 24, // pojemność opakowania (packageVolume)
        "unit": "tabl." // packageVolumeUnit
      },
      "superContent": { // nadopakowanie
        "quantity": 1, // packageCount (brak → 1)
        "unit": "" // packageType (brak → "")
      }
    },
    "dosage": [ // 7 dni po 1 tabletce co 8 h, następnie 13 dni 2× dziennie po 1 tabletce
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
          "quantity": 24,
          "unit": "h"
        },
        "frequency": 2,
        "doseQuantity": {
          "quantity": 1,
          "unit": "tabl."
        }
      }
    ],
    "kind": "PF", // PA - proauctore, PF - profamiliae, ZW - zwykła (domyślnie)
    "issueMode": "Z", // Z - zwykła, F - farmaceutyczna, P - pielęgniarska, PL - pielęgniarska na zlecenie lekarza
    "substanceAdminSubstitution": "N", // nie pozwalaj na zamienniki
    "priorityCode": "UR", // znacznik CITO, opcjonalny
    "infoForPatient": "popić dużą ilością wody", // informacja dla pacjenta przy schematach dawkowania
    "dispenseRequest": {
        "quantity": 2, // ile opakowań/unit wydać
        "unit": "op.", // opcjonalny
        "infoForPerformer": "", // opcjonalna informacja dla wydającego
        "validityPeriod": {
          "start": "2024-05-01" // opcjonalnie (domyślnie brak) od kiedy można zrealizować receptę
        }
    }
  }]
}
```

### Recepta roczna, dawkowanie złożone *z doprecyzowaniem (podsekwencje)*
- Sekwencja 1: przez 7 dni
  - Podsekwencja 1_1: 1 tabletka rano
  - Podsekwencja 1_2: 2 tabletki wieczorem, pomiędzy kolacją a porą snu
- Sekwencja 2: następnie przez 10 dni 2× dziennie od 1 do 2 tabletek

```jsonc
{
  "erecepta": [{
    "id": "0000000000000000025326", // unikalny numer dokumentu recepty nadany przez implementatora w ramach oidRoot (przestrzeni organizacji)
    "date": "2024-04-22",
    "type": "prepared", // prepared - gotowy lek, recipe - receptura własna
    "organization": "idabc", // uuid
    "practitioner": "idxyz", // uuid
    "patient": { … }, // jak w pierwszym przykładzie
    "medication": {
      "name": "Apap Noc",
      "code": "100110151", // kod producenta
      "ean": "05909990960132", // kod EAN leku
      "kdlek": "Rp", // Rp, Rpw, Rpz, OTC
      "payment": "100%", // opcjonalne (domyślnie 100%) R, B, 30%, 50%, 100%
      "package": {
        "quantity": 24, // pojemność opakowania (packageVolume)
        "unit": "tabl." // packageVolumeUnit
      },
      "superContent": { // nadopakowanie
        "quantity": 1, // packageCount (brak → 1)
        "unit": "" // packageType (brak → "")
      }
    },
    "dosage": [
      { // Sekwencja 1: przez 7 dni
        "duration": {
          "quantity": 7,
          "unit": "d"
        },
        "period": {
          "quantity": 24,
          "unit": "h"
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
          { // Podsekwencja 1_2: 2 tabletki wieczorem, pomiędzy kolacją a porą snu
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
      { // Sekwencja 2: następnie przez 10 dni 2× dziennie od 1 do 2 tabletek
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
    "issueMode": "Z", // Z - zwykła, F - farmaceutyczna, P - pielęgniarska, PL - pielęgniarska na zlecenie lekarza
    "substanceAdminSubstitution": "N", // nie pozwalaj na zamienniki
    "priorityCode": "UR", // znacznik CITO, opcjonalny
    "dispenseRequest": {
        "quantity": 3, // ile opakowań/unit wydać
        "unit": "op.", // opcjonalny
        "infoForPerformer": "", // opcjonalna informacja dla wydającego
        "validityPeriod": {
          "start": "2024-04-22", // opcjonalnie (domyślnie brak) od kiedy można zrealizować receptę; dla recepty rocznej równa dacie wystawienia
          "end": "2025-04-22" // opcjonalnie, dla recepty 365 (+1 rok od dnia wystawienia)
        }
    }
  }]
}
```

### Recepta roczna, dawkowanie złożone *z przerwą i cyklami*
- Sekwencja 1: przez 5 dni 3× dziennie po 1 tabl.
- Sekwencja 2: następnie 2 dni przerwy
- Sekwencja 3: następnie przez 7 dni 1× dziennie po 1 tabl.
- Cykl wykonać łącznie 5 razy (`"dosageRepeat": 4` – 4 dodatkowe powtórzenia)

```jsonc
{
  "erecepta": [{
    "id": "0000000000000000025326",
    "date": "2024-09-10",
    "type": "prepared", // prepared - gotowy lek, recipe - receptura własna
    "organization": "idabc",
    "practitioner": "idxyz",
    "patient": { … }, // jak w pierwszym przykładzie
    "medication": {
      "name": "Apap Noc",
      "code": "100110151",
      "ean": "05909990960132",
      "kdlek": "Rp", // Rp, Rpw, Rpz, OTC
      "payment": "100%",
      "package": {
        "quantity": 24,
        "unit": "tabl."
      },
      "superContent": {
        "quantity": 1,
        "unit": ""
      }
    },
    "dosage": [
      { // Sekwencja 1: przez 5 dni 3× dziennie po 1 tabl.
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
      { // Sekwencja 2: następnie 2 dni przerwy
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
      { // Sekwencja 3: następnie przez 7 dni 1× dziennie po 1 tabl.
        "duration": {
          "quantity": 7,
          "unit": "d"
        },
        "period": {
          "quantity": 24,
          "unit": "h"
        },
        "frequency": 1,
        "doseQuantity": {
          "quantity": 1,
          "unit": "tabl."
        }
      }
    ],
    "dosageRepeat": 4, // opcjonalnie (domyślnie 0) - liczba dodatkowych powtórzeń cyklu
    "kind": "ZW",
    "issueMode": "Z",
    "substanceAdminSubstitution": "N",
    "priorityCode": "UR", // znacznik CITO, opcjonalny
    "infoForPatient": "w czasie przerwy dużo odpoczywać", // opcjonalnie, przy schematach dawkowania
    "dispenseRequest": {
        "quantity": 5, // 5 cykli × 22 tabl. = 110 tabl. → 5 op. × 24 tabl. = 120 tabl.
        "unit": "op.",
        "infoForPerformer": "",
        "validityPeriod": {
          "start": "2024-09-10", // opcjonalnie (domyślnie brak) - recepta roczna
          "end": "2025-09-10" // opcjonalnie (domyślnie brak) - recepta roczna
        }
    }
  }]
}
```
