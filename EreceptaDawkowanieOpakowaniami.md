

### Dawkowanie w opakowaniach (dawkomat uproszczony)

#### Kiedy można stosować

| Rodzaj recepty (`type`) | Dawkowanie `{op.}` |
|---|---|
| `prepared` - lek gotowy | ✅ dozwolone |
| `supplement` - środek spożywczy specjalnego przeznaczenia żywieniowego (S) | ✅ dozwolone |
| `recipe` - lek recepturowy | ❌ błąd 400 |
| lek importowany (import docelowy) | ❌ błąd 400 |
| `product` - wyrób medyczny | nie dotyczy (zob. [Opis budowy recepty na Wyroby Medyczne](EreceptaWM.md)) |

Lista leków, które można dawkować opakowaniami jest ograniczona. Decyduje o tym system e-zdrowie. W bazie leków w API - są to leki oznaczone parametrem `"packageDose": true`.  
Przykład leku poniżej:

```json
[
  {
    "name": "Nutramigen 3 LGG Complete",
    "dose": "",
    "package": "400 g (puszka)",
    "code": "3692621",
    "ean": "08712045038833",
    "nameEn": "",
    "isReimbursed": true,
    "percent": "30%",
    "price": 0,
    ...
   "packageDose": true          // określenie możliwości dawkowania opakowaniami
  }
]
```

#### Jak przekazać

Dawkowanie w opakowaniach włącza się, przekazując w `doseQuantity` jednostkę **`"unit": "{op.}"`**.

| Element | Znaczenie | Wymagany |
|---|---|---|
| `doseQuantity.quantity` | liczba opakowań | tak |
| `doseQuantity.unit` | zawsze `{op.}` | tak |
| `period` | na jaki okres starcza dawka (`quantity` + `unit`) | tak |
| `duration` | łączny czas trwania dawkowania (`quantity` + `unit`) | nie |
| `frequency` | musi wynosić `1` (lub zostać pominięty) | nie |

Dopuszczalne jednostki `period` i `duration`: `d` (dni), `wk` (tygodnie), `mo` (miesiące).

Ograniczenia:
- częstotliwość `frequency` o ile występije - musi być równa 1,
- dawkowanie `{op.}` nie może mieć podsekwencji (zagnieżdżonego `dosage`),
- jednostka `{op.}` nie może być mieszana z inną jednostką dawki w tej samej recepcie (odrzuca to P1 - reguła `REG.070`).

Jeśli `doseQuantity.unit` jest inna niż `{op.}`, dawkowanie budowane jest jak dotychczas.

#### Przykład 1 - bez czasu trwania: „1 op. na 2 tygodnie”

```json
"dosage": [
    {
        "period": {             // okres dawkowania: na 2 tygodnie
            "quantity": 2.0,
            "unit": "wk"
        },
        "doseQuantity": {       // dawka: 1 opakowanie
            "quantity": 1.0,
            "unit": "{op.}"
        }
    }
]
```

Treść na recepcie:

> **D.S. 1 op. na 2 tygodnie**

#### Przykład 2 - z czasem trwania: „2 op. na 1 miesiąc przez 2 miesiące”

```json
"dosage": [
    {
        "duration": {           // czas trwania: przez 2 miesiące
            "quantity": 2.0,
            "unit": "mo"
        },
        "period": {             // okres dawkowania: na 1 miesiąc
            "quantity": 1.0,
            "unit": "mo"
        },
        "frequency": 1,         // opcjonalne, tylko wartość 1
        "doseQuantity": {       // dawka: 2 opakowania
            "quantity": 2.0,
            "unit": "{op.}"
        }
    }
]
```

Treść na recepcie:

> **D.S. 2 miesiące**  
> **2 op. na 1 miesiąc**

#### Przykład 3 - kilka sekwencji

W jednej recepcie można przekazać kilka sekwencji dawkowania w opakowaniach:

```json
"dosage": [
    {
        "duration":     { "quantity": 4.0, "unit": "mo" },
        "period":       { "quantity": 2.0, "unit": "mo" },
        "doseQuantity": { "quantity": 2.0, "unit": "{op.}" }
    },
    {
        "period":       { "quantity": 2.0, "unit": "mo" },
        "doseQuantity": { "quantity": 1.0, "unit": "{op.}" }
    }
]
```
Treść na recepcie:

> **D.S. 4 miesiące**  
> **2 op. na 2 miesiące,**  
> **następnie**  
> **1 op. na 2 miesiące**  

Łączny czas leczenia na recepcie liczony jest w pełnych dniach (zaokrąglenie w górę). Przy braku `duration` przyjmowany jest jeden okres, np. `period` = 2 wk daje 14 dni.

### Walidacja dawkowania (dawkomat)

Wraz z dawkowaniem w opakowaniach zaostrzono walidację wszystkich danych dawkomatu (`dosage`). Błędne dane zwracają teraz kod **`400`**.

Zasady:
- `period.unit` i `duration.unit` są wymagane i muszą mieć jedną z wartości: `h`, `d`, `wk`, `mo`. Inne jednostki nie są obsługiwane.
- `period.quantity` i `duration.quantity` muszą być większe od 0, są liczbami całkowitymi

Komunikaty błędów:

| Sytuacja | Komunikat (`400 Bad request`) |
|---|---|
| `duration` bez jednostki lub z nieobsługiwaną jednostką | `Czas trwania dawkowania (duration) wymaga jednostki (durationUnit). Dozwolone: h, d, wk, mo` |
| `period` bez jednostki lub z nieobsługiwaną jednostką | `Nieobsługiwana jednostka okresu dawkowania (periodUnit). Dozwolone: h, d, wk, mo` |
| `duration.quantity` ≤ 0 | `Czas trwania dawkowania (duration) musi być większy od 0` |
| `period.quantity` ≤ 0 | `Okres dawkowania (period) musi być większy od 0` |
| `{op.}` dla leku recepturowego lub importowanego | `Dawkowanie w opakowaniach ({op.}) nie jest dozwolone dla leku recepturowego, importowanego ani wyrobu medycznego` |
| `{op.}` z `frequency` > 1 | `Dawkowanie w opakowaniach ({op.}) dopuszcza jedną dawkę na okres (frequency = 1)` |
| `{op.}` z podsekwencjami | `Dawkowanie w opakowaniach ({op.}) nie może mieć podsekwencji dawkowania` |

**Uwaga** przekazanie jednostki w `dosage` jest wymagane.  
Nie dotyczy to elementu `dispenseRequest.duration` (recepta na wyrób medyczny). Tam `duration` nadal przekazuje się jako liczbę dni (integer) bez jednostki:  
> ```json
> "dispenseRequest": {
>     "quantity": 1.0,
>     "unit": "op.",
>     "validityPeriod": {
>         "start": "2026-11-06"
>     },
>     "duration": 12
> }
> ```
