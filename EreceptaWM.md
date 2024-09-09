# Usługa: e-Recepta na wyroby medyczne

Recepta na wyroby medyczne bazuje na formacie recepty leku gotowego, ale zawiera uproszczoną strukturę schematu dawkowania.
W ramach struktury - dla wyrobów refundowanych przesyłana jest jedynie informacja o długości czasu trwania kuracji.
Receptę na wyrób medyczny charakteryzuje: 
  - element ```"type": "product"```
  - element ```"kdlek": "Rp"```
  - brak elementu opisującego pełne schematy dawkowania ```"dosage"```

Receptę na WM można wystawić na 2 sposoby - wyznacznikiem jest odpłatność ("payment"):
1. **Recepta refundowana** - cechy:
    - ```"payment"``` *(opdłatność)* - wszystkie poza 100% (R, B, 30%, 50%)
    - ```"duration"``` *(okres dawkowania)* - wymagane dla recepty rocznej
    - informacje dla pacjenta przekazywane są za pomocą elementu:
        - ```"infoForPatient"``` gdy podany jest ```duration``` *(recepta roczna)*
        - ```"dosageInstruction"``` gdy recepta nie zawiera okresu dawkowania
2. **Recepta nierefundowana** - cechy:
    - ```"payment"``` - odpłatność równa 100%
    - nie może zawierać elementów ```"duration"``` oraz ```"infoForPatient"```
    - informacje dla pacjenta przekazywane za pomocą ```"dosageInstruction"```

Recepta roczna (365) obsługiwana jest analogicznie jak w przypadku recepty na lek gotowy - należy dodać element ```"end"``` w ramach ```"validityPeriod"```.
 

## Przykłady
### Recepta na wyroby medyczne ***refundowana***

```json
{
  "erecepta": [{
    "id": "0000000000000000025326", // unikalny numer dokumentu recepty nadany przez implementatora w ramach oidRoot (przestrzeni organizacji)
    "date": "2024-04-22",
    "type": "product", // prepared - gotowy lek, recipe - receptura własna, product - wyrób medyczny
    "organization": "idabc", // uuid
    "practitioner": "idxyz", // uuid
    "patient": {
      "identifier": [
        {
          "type": "pesel",
          "value": "60032223611"
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
	  "name": "ACCU-CHEK ACTIVE",
	  "ean": "4015630056316", // kod EAN leku
	  "kdlek": "Rp",
	  "payment": "R",  // odpłatność refundowana (R, B, 30%, 50%)
	  "package": {
	  	"quantity": 50.0, // ilość produktów w opakowaniu
	  	"unit": "szt."    // opcjonalne (domyślnie szt.)
	  }	  
	},	
    "kind": "ZW", // PA - proauctore, PF - profamiliae, ZW - zwykła (domyślnie)
    "issueMode": "Z", // Z -Zwykła, F - Farmaceutyczna, P - Pielęgniarska, PL - Pielęgniarska na zlecenie lekarza
    "priorityCode": "UR", // CITO opcjonalne
    "infoForPatient": "po każdym posiłku", // opcjonalnie - informacja dla pacjenta w przypadku przekazywania duration
    "dispenseRequest": {
        "quantity": 4.0, // ile opakowań wydać
        "unit": "op.", // opcjonalny
        "duration": 200,  // wymagane dla recepty refundowanej rocznej (okres dawkowania)
        "validityPeriod": {
          "start": "2024-04-22", // opcjonalnie (domyślnie brak) od kiedy można zrealizować
          "end": "2025-04-22" // opcjonalnie (domyślnie brak) do kiedy można zrealizować (recepta roczna)
        }
    }
  }]
}
```

### Recepta na wyroby medyczne ***nierefundowana***

```json
{
  "erecepta": [{
    "id": "0000000000000000025326", // unikalny numer dokumentu recepty nadany przez implementatora w ramach oidRoot (przestrzeni organizacji)
    "date": "2024-04-22",
    "type": "product", // prepared - gotowy lek, recipe - receptura własna, product - wyrób medyczny
    "organization": "idabc", // uuid
    "practitioner": "idxyz", // uuid
    "patient": {
      "identifier": [
        {
          "type": "pesel",
          "value": "60032223611"
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
	  "name": "ACCU-CHEK ACTIVE",
	  "ean": "4015630056316", // kod EAN leku
	  "kdlek": "Rp",
	  "payment": "100%",  // brak refundacji
	  "package": {
	  	"quantity": 50.0, // ilość produktów w opakowaniu
	  	"unit": "szt."    // opcjonalne (domyślnie szt.)
	  }	  
	},	
    "kind": "ZW", // PA - proauctore, PF - profamiliae, ZW - zwykła (domyślnie)
    "issueMode": "Z", // Z -Zwykła, F - Farmaceutyczna, P - Pielęgniarska, PL - Pielęgniarska na zlecenie lekarza
    "priorityCode": "UR", // CITO opcjonalne
    "dosageInstruction": "po każdym posiłku", // opcjonalnie  - informacja dla pacjenta w przypadku przesyłania recepty bez czasu trwania (duration)
    "dispenseRequest": {
        "quantity": 4.0, // ile opakowań wydać
        "unit": "op.", // opcjonalny
        "validityPeriod": {
          "start": "2024-04-22", // opcjonalnie (domyślnie brak) od kiedy można zrealizować
          "end": "2025-04-22" // opcjonalnie (domyślnie brak) do kiedy można zrealizować (recepta roczna)
        }
    }
  }]
}
```

