# Usługa: Baza leków

## Wyszukiwanie leków

```
POST /dictionary/drug
Content-Type: application/x-www-form-urlencoded
Authorization: Bearer {{ bearer }}

term=Apap
```

Odpowiedź:

```json
[
  {
    "name": "APAP dla dzieci Forte smak pomarańczowy",
    "dose": "40 mg\/ml",
    "package": "1 butelka 85 ml + strzykawka 6 ml",
    "code": "100443355",
    "ean": "05909991452063",
    "nameEn": null,
    "isReimbursed": false,
    "percent": "",
    "price": 0,
    "unit": "butelka 85 ml + strzykawka 6 ml",
    "form": "Zawiesina doustna",
    "category": "OTC",
    "amount": "1",
    "indications": "",
    "isRpw": false,
    "calculate": false,
    "availability": 0,
    "producer": "US Pharmacia Sp. z o.o.",
    "substance": "Paracetamolum",
    "expired": false,
    "favourite": 0
  }
]
```

## Dodawanie ulubionych leków

```
POST {{host}}/dictionary/drug/favourite/{{EAN}}
Authorization: Bearer {{ bearer }}
```

## Usuwanie ulubionych leków

```
DELETE /dictionary/drug/favourite/{{EAN}}
Authorization: Bearer {{ bearer }}
```


## Pobieranie listy ulubionych leków

```
GET {{host}}/dictionary/drug/favourite
Authorization: Bearer {{ bearer }}
```

## Szukanie składników receptur

```
POST {{host}}/dictionary/drug-ingredient
Content-Type: application/x-www-form-urlencoded
Authorization: Bearer {{ bearer }}

term=Etanol
```
