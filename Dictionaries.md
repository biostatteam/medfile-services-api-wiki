# Usługa: Słowniki

## Baza leków

### Wyszukiwanie leków

```http request
POST /dictionary/drug
Content-Type: application/x-www-form-urlencoded
Authorization: Bearer {{ bearer }}

term=Apap
```

Odpowiedź:

```json opisujący lek
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
    "indicationsJson": [  // dla leków refundowanych - wskazania do refundacji 
      [
        {
          "id": "5229",
          "all": false,
          "isE": null,
          "isD1": null,
          "isD2": null,
          "type": "ChPL",
          "ageTo": null,
          "ageFrom": null,
          "description": "Nadciśnienie tętnicze",
          "paymentLevel": "30%"
        }
      ]
    ],
    "isRpw": false,
    "calculate": false,
    "availability": 0,
    "producer": "US Pharmacia Sp. z o.o.",
    "substance": "Paracetamolum",
    "expired": false,
    "packages": [  // model opisujący opakowanie leku
      {
        "packageType": "",
        "packageCount": "",
        "packageVolume": "1",
        "packageVolumeUnit": "tabl.",
        "packageDescription": ""
      }
    ],
    "alternativeDose": null,  // alternatywne dawkowanie
    "favourite": 0,
    "popularity": 0,
    "model": null,
    "type": "PL",
    "reimbursementPermissions": {   // dla leków refundowanych - określenie uprawnień dodatkowych
      "E": false,
      "D1": true,
      "D2": true
    }
  }
]
```

### Dodawanie ulubionych leków

```http request
POST /dictionary/drug/favourite/{{EAN}}
Authorization: Bearer {{ bearer }}
```

### Usuwanie ulubionych leków

```http request
DELETE /dictionary/drug/favourite/{{EAN}}
Authorization: Bearer {{ bearer }}
```


### Pobieranie listy ulubionych leków

```http request
GET /dictionary/drug/favourite
Authorization: Bearer {{ bearer }}
```

### Szukanie składników receptur

```http request
POST /dictionary/drug-ingredient
Content-Type: application/x-www-form-urlencoded
Authorization: Bearer {{ bearer }}

term=Etanol
```

## Słownik procedur medycznych ICD-9
Międzynarodowa Klasyfikacja Procedur Medycznych, która jest używana do przypisywania kodów różnym procedurom medycznym dostępna w języku polskim oraz angielskim.  
Wyszukiwanie dostępne jest za pomocą parametru: `term`

```http request
GET /dictionary/icd9/{lang}/search{term}
Authorization: Bearer {{ bearer }}
```


## Słownik rozpoznań ICD-10
Międzynarodowa Statystyczna Klasyfikacja Chorób i Problemów Zdrowotnych używana do określania jednostek chorobowych dostępna w języku polskim oraz angielskim. Słownik aktualizowany na bieżąco.  
Wyszukiwanie dostępne jest za pomocą parametru: `term`

```http request
GET /dictionary/icd9/{lang}/search{term}
Authorization: Bearer {{ bearer }}
```

