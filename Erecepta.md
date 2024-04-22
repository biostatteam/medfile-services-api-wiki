# Usługa: e-Recepta

## Kody odpowiedzi

1. `200` - Poprawna odpowiedź
2. `403` - Brak uprawnień do wykonania polecenia
3. `404` - Podmiot lub lekarz nie istnieją w wybranym kontekście
4. `406` - Błąd w wysyłaniu recepty (np. nieprawidłowe dane)

## Kontekst wywołania, ważna informacja

Recepty można wystawiać (oraz przeprowadzać na wystawionych operacje) w jednym z trzech kontekstów/trybów:

- `ZW` - kontekst ogólny (zwykły)
- `PA` - kontekst *pro auctore*
- `PF` - kontekst *pro familae*

Konteksty przeprowadzanych operacji muszą być zgodne z kontekstem, w jakim wystawiono receptę.

Przykładowo receptę wystawioną w kontekście osobistym można anulować tylko w takim samym kontekście.
Ponieważ nie będziemy w stanie znaleźć recepty PA, szukając jej w rejestrze ZW. 

## Wystawienie e-recepty

Receptę można wystawić w trzech kombinacjach:
1. __Rp__ - do 5 e-recept w jednym pakiecie (do tablicy `erecepta` można wstawić 5 bloków z receptami).
2. __Rpw__ - tylko 1 e-recepta w jednym pakiecie
3. __receptura własna__ - bez względu na Rp/Rpw tylko jedna receptura może znaleźć się w pakiecie

### Procedura 

Przed wystawieniem e-recepty warto zweryfikować czy jest poprawnie skonstruowana:
```
PUT|POST /erecepta/validate
```

Następnie możemy wysłać e-receptę poleceniem:
```
PUT|POST /erecepta/send
```

Dla obu poleceń treść żądania jest identyczna.

### Treść żądania dla 1 recepty Rp w pakiecie

```json
[
  "erecepta": [{
    "id": "0000000000000000025326", // unikalny numer dokumentu recepty nadany przez implementatora w ramach oidRoot (przestrzeni organizacji)
    "date": "2020-08-31",
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
        "quantity": 60.0, // ilość leku w opakowaniu
        "unit": "tabl." // opcjonalne (domyślnie szt.)
      },

      "activeSubstance": [ // opcjonalne
        {
          "name": "Oxycodon", // opcjonalne gdy podano ilość substancji
          "quantity": 0.0004,
          "unit": "g"
        }
      ]
    },

    "kind": "PF", // PA - proauctore, PF - profamiliae, ZW - zwykła (domyślnie)
    "issueMode": "Z", // Z -Zwykła, F - Farmaceutyczna, P - Pielęgniarska, PL - Pielęgniarska na zlecenie lekarza
    "substanceAdminSubstitution": "N", // Nie pozwalaj na zamienniki
    "priorityCode": "UR", // CITO opcjonalne

    "dosageInstruction": "1x1 przed snem", // dawkowanie, minimum 3 znaki w formacie 1x1, w innych formatach dowolnie
    "dispenseRequest": {
        "quantity": 1.0, // ile opakowań/unit wydać
        "unit": "op.", // opcjonalny
        "infoForPerformer": "proszę wymieszać", // opcjonalne informacja dla wydającego leki
        "validityPeriod": {
          "start": "2020-09-30", // opcjonalnie (domyślnie brak) od kiedy można zrealizować
          "end": "2020-10-30" // opcjonalnie (domyślnie brak) do kiedy można zrealizować (maksymalnie 365 dni od start)
        }
    }
  }]
]
```

### Treść żądania dla 1 recepty Rp recepturowej

```json
{
  "erecepta": [
    {
      "id": "0000000000000000025326",
      "date": "2020-08-31",
      "type": "recipe",
      "organization": "idabc",
      "practitioner": "idxyz",
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
        "address": [
          {
            "street": "Wrocławska",
            "houseNumber": "11A",
            "unitId": "3",
            "city": "Zielona Góra",
            "postalCode": "00-184",
            "country": "Polska"
          }
        ],
        "nfz": "07",
        "gender": "M"
      },
      "medication": {
        "name": "Receptura własna",
        "kdlek": "Rp",
        "payment": "100%",
        "ingredient": [{
          "name": "Sól",
          "quantity": 10,
          "unit": "ml",
          "quantityPer": 100,
          "unitPer": "g"
        },{
          "name": "Sól",
          "annotation": "a"
        },{
          "name": "Calcii carbonas praecipitatus",
          "code": "100009038",
          "annotation": "ad q.s.",
          "quantity": null,
          "unit": "",
          "quantityPer": null,
          "unitPer": null
        },{
          "name": "Mocznik",
          "code": "100020991",
          "annotation": "ad",
          "quantity": null,
          "unit": "",
          "quantityPer": null,
          "unitPer": ""
        }]
      },
      "kind": "ZW",
      "substanceAdminSubstitution": "N",
      "priorityCode": "UR",
      "dosageInstruction": "1x1 przed snem",
      "dispenseRequest": {
        "quantity": 1.0,
        "unit": "op.",
        "infoForPerformer": "proszę wymieszać",
        "recipeSummary": "M.f. Pulv.",
        "validityPeriod": {
          "start": "{{ today }}"
        }
      }
    }
  ]
}
```
#### [Przykłady dawkowania dla recept 365](Erecepta365.md)

## Pobieranie pakietu recept

```
GET /erecepta/{kluczPakietuRecept}[/{kontekst = 'ZW'}]
```

### Poprawna odpowiedź

```json
{
    "erecepta": [
        {

        }
    ]
}
```

## Drukowanie pakietu recept

Drukowanie pakietu recept polega na pobraniu z P1 treści recepty, dodaniu koduPakietuRecept i skonwertowaniu XML w HL7 CDA do formatu HTML.
KluczPakietuRecept potrzebny jest, aby receptę z P1 pobrać.

Parametr kontekst jest opcjonalny, ale należy go ustawić zgodnie z polem kind na dokumencie recepty.
- ZW - dla zwykłych recept (wartość domyślna)
- PF - dla recept pro familiae
- PA - dla recept pro auctorae

```
GET /erecepta/{kluczPakietuRecept}/{kodPakietuRecept}/print/[/{kontekst = 'ZW'}]
```

### Poprawna odpowiedź

```json
{
    "html": "base64 encoded html document..."
}
```

## Anulowanie e-recepty

Anulowanie całego pakietu recept (od 1 do 5 leków).
```
DELETE /erecepta/{kluczPakietuRecept}[/{kontekst = 'ZW'}]
```

Anulowanie wybranej pozycji recepty. ***(Not implemented yet)***
```
DELETE /erecepta/{kluczPakietuRecept}/{kodPozycjiLekuWPakiecieRecept}[/{kontekst = 'ZW'}]
```

### Prawidłowa odpowiedź

```json
{
  "erecepta": [
      {
          //lek 1
      },
      {
          //lek N ...
      }
  ]
}
```

W treści odpowiedzi znajdzie się bezpośrednia odpowiedź od CSIOZ na każdy z leków we wskazanym pakiecie recept w postaci tablicy.