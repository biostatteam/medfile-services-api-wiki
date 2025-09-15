# Usługa: e-Recepta na import docelowy

Recepta na import docelowy bazuje na formacie recepty leku gotowego, bez opisu szczegółowego schametu dawkowania.
Podlega wielu obostrzeniom nałożonym przez ustawodawcę:
  - wystawić może tylko lekarz (```"issueMode": "Z"```)
  - recepta nie może być na lek recepturowy lub wyrób medyczny (element ```"type": "prepared"```)
  - opdłatność (```"payment"```) - wyłącznie 100% lub R - inne nie są obsługiwane
  - brak elementu opisującego pełne schematy dawkowania ```"dosage"``` - nie można użyc dawkomatu
  - należy wskazać numer zapotrzebowania (element ```"nzap": "xxx"```)

Recepta na import docelowy może być receptą roczną, jednak wtedy stosowane jest dawkowanie proste - bez schematów.
 

## Przykład
### Recepta na import docelowy

```json
{
    "erecepta": [
        {
            "id": "MEDFILE0000000008195PR",  // unikalny numer dokumentu recepty nadany przez implementatora w ramach oidRoot (przestrzeni organizacji)
            "date": "2025-09-15",
            "type": "prepared",  // prepared - gotowy lek
            "organization": "idabc", // uuid
            "practitioner": "idxyz", // uuid
            "patient": {
                "identifier": [
                    {
                        "type": "p1oid",
                        "value": "2.16.840.1.113883.3.4424.1.1.616|40010151673"
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
                ],
                "address": [
                    {
                        "street": "jasna",
                        "city": "katowice",
                        "postalCode": "12-312",
                        "country": "pl",
                        "houseNumber": "32",
                        "unitId": ""
                    }
                ],
                "nfz": "09",
                "gender": "M",
                "birthDate": "1940-01-01"
            },
            "medication": {
                "name": "Elvanse",
                "code": "100494445",
                "ean": "07038319127401",
                "kdlek": "Rpw",
                "package": {
                    "quantity": 30.0,
                    "unit": "kaps."
                },
                "activeSubstance": [
                    {
                        "quantity": 900.0,
                        "unit": "mg",
                        "name": "Lisdexamfetamini dimesylas"
                    }
                ],
                "payment": "100%",  // odpłatność 100% lub R
                "form": "Kapsułki twarde",
                "strength": "30 mg"
            },
            "kind": "ZW",
            "issueMode": "Z",
            "dosageInstruction": "3x1",  // opis dawkowania leku
            "dispenseRequest": {
                "quantity": 1.0,
                "unit": "op.",
                "validityPeriod": {
                    "start": "2025-09-15" // opcjonalnie (domyślnie brak) od kiedy można zrealizować
                }
            },
            "attributes": [  // dodatkowe atrybuty dodawane do recepty (tryb badania / wskazanie)
                {
                    "name": "TRYB_BADANIA",
                    "value": "BADANIE_OSOBISTE"
                }
            ],
            "nzap": "IMP/2025/12/0023"  // numer zaopatrzenia - element przełączający receptę na "Receptę na import docelowy"
        }
    ]
}
```
