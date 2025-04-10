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


### Obsługa wskazań refundacyjnych

W przypadku recept na lek gotowy, refundowany - istnieje możliwość przekazania do systemu e-zdrowie wskazania refundacyjnego. Informację tą należy przekazać w JSON w nowym elemencie **attributes**. Element ten jest opcjonalny, nie należy go przekazywać bez parametrów *name* oraz *value*.  
Atrybut *value* zawiera kod wskazania refundacyjnego publikowanego na liście refundacyjnej.  
Kod wskazania należy umieścić w JSON jako parametr elementu erecepta - w następujący sposób: 
```
    "attributes": [
        {
          "name": "WSKAZANIE_REFUNDACYJNE",
          "value": "24799"
        }
    ]
```
Element "WSKAZANIE_REFUNDACYJNE" jest przekazywany do platformy e-zdrowie wraz z receptą - w obiekcie receptaDaneDodatkoweMT.    


### Sposób badania pacjenta

Wersja 27.0.2 e-recepty wprowadziła obsługę dodatkowych atrybutów "RODZAJ_SWIADCZENIA" oraz "TRYB_BADANIA".  
Atrybuty te wymagane są w przypadku recept na leki psychotropowe oraz środki odurzające (wskazane w Rozporządzeniu). 

Oba atrybuty należy przekazać (zgodnie z wymaganiami) w JSON w elemencie **attributes**. Element ten jest opcjonalny, nie należy go przekazywać bez parametrów *name* oraz *value*.  
Przykład poniżej:
```
    "attributes": [
        {
          "name": "TRYB_BADANIA",
          "value": "BADANIE_OSOBISTE"
        }
    ]
```

Poniżej przedstawiony jest przykład użycia obu elementów w jednej recepcie.


### Treść żądania dla 1 recepty Rp w pakiecie

```json
{
  "erecepta": [{
    "id": "0000000000000000025326", // unikalny numer dokumentu recepty nadany przez implementatora w ramach oidRoot (przestrzeni organizacji)
    "date": "2020-08-31",
    "type": "prepared", // prepared - gotowy lek, recipe - receptura własna, product - wyrób medyczny
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
      "gender": "M", // płeć pacjenta
      "birthDate": "1940-01-01",  //data urodzenia pacjenta
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
      "kdlek": "Rp", // Rp, Rpw, Rpz, OTC  (kategoria dostępności leku)
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
    },
    "attributes": [ // opcjonalnie (domyślnie brak) - dodatkowe atrybuty recepty przekazywane w obiekcie receptaDaneDodatkoweMT
        {
          "name": "WSKAZANIE_REFUNDACYJNE",  // atrybut określający wskazanie do refundacji leku
          "value": "24799"
        },
        {
          "name": "TRYB_BADANIA", // atrybut dot. sposobu badania pacjenta przed wystawieniem recepty Rpw
          "value": "BADANIE_OSOBISTE"
        }
    ]
  }]
}
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
        "gender": "M",
        "birthDate": "1940-01-01"  //data urodzenia pacjenta
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
#### [Przykłady dawkowania dla recept zawierających szablony dawkowania](Erecepta365.md)
#### [Opis budowy recepty na Wyroby Medyczne](EreceptaWM.md)

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


## Rozszerzone wyszukiwanie e-recept wystawionych pacjentowi

Endpoint umożliwia uzyskanie z systemu P1 listy recept wystawionych określonemu pacjentowi. Wraz z danymi znajdującymi sie na recepcie otrzymujemy jej klucz umożliwiający pobranie pełnego XMLa - np. celem wydruku.  

W ramach wywołania operacji w P1 sprawdzane są uprawnienia dostępu do recept.  
Domyślnie system P1 zwraca wynik obejmujący recepty wystawione przez użytkownika lub recept widocznych dla użytkownika. W określonych przypadkach można jednak rozszerzyć listę (używając zmienionego trybu dostępu do danych).  
Przykładem może być dostęp SPECJALNY - dla lekarzy specjalistów (dostęp do wszystkich recept Usługobiorcy, którego wiek pozwala na zastosowanie uprawnień seniora „S”, lub dziecka „DZ”) lub BTG – tryb ratowania życia, zgodnie z uprawnieniem wynikającym z art. Art. 35. 1 pkt. 4 Ustawy o SIOZ. 

*UWAGA: Każdy przypadek wyszukiwania recept z zastosowaniem ustawienia trybu ratującego życie jest odnotowywany w systemie P1, w celu monitorowania ew. nadużyć.*


```
GET /erecepta/searchext
```
Operacja pozwala na wyszukiwanie recept według następujących parametrów:
- identifier (wymagany) - identyfikator usługobiorcy 
- identifier_type (wymagany)  - rodzaj identyfikatora odbiorcy (domyślnie pesel)
- date_from  - data wystawienia recepty (początkowa data, od której zwracane będą dane)
- date_to  - ustawienie okresu zwracanych danych (data do)
- status - ograniczenie listy recept do wybranego statusu (WYSTAWIONA, ZREALIZOWANA, CZESCIOWO_ZREALIZOWANA, ANULOWANA lub ZABLOKOWANA).
- drug_name - umożliwia ograniczenie listy do wybranego leku
- prescription_number - parametr pozwalający wyszukać dane konkretnej recepty oraz dokumentów z nią powiązanych (wyszukiwanie tylko własnych recept)
- access_mode - tryb dostępu do danych (domyślnie pusty)
- page - określa numer strony z wynikami (w przypadku ilości większej niż ustawiony *page_size*)
- page_size - określa ilość zwracanych pozycji na stronę (domyślnie 20)
- order - parametr określający czy lista zwracanych recept ma być sortowana po dacie wystawienia w kolejności rosnącej czy malejącej

### Przykładowa odpowiedź

```json
{
  "message": "",
  "body": {
    "liczbaDokumentow": XX,
    "wynikiRozszerzonegoWyszukiwaniaReceptUslugobiorcy": [
      {
        "dataWystawieniaRecepty": "2024-11-04T00:00:00+01:00",
        "dataRealizacjiOd": "2024-11-04T00:00:00+01:00",
        "dataRealizacjiDo": "2025-11-04T00:00:00+01:00",
        "identyfikatorOpakowaniaLeku": "05909990664559",
        "iloscLeku": 1,
        "kluczRecepty": "10012350010000000008026420811716549443000000",
        "nazwaPrzepisanegoLeku": "Apap 500 mg Tabletki powlekane",
        "numerRecepty": {
          "extension": "MEDFILE000000000000000",
          "root": "2.16.840.1.113883.3.4424.2.7.0000.2.1"
        },
        "rodzajLeku": "GOTOWY",
        "statusMozliwosciRealizacjiRecepty": true,
        "statusRecepty": "WYSTAWIONA",
        "wielkoscOpakowania": "100 tabl.",
        "poziomOdplatnosciRecepty": "100%"
      },
      {
       ...
      }
}
```

Operacja zwraca następujące dane:
- liczba wszystkich wyszukanych dokumentów
- datę wystawienia recepty
- datę realizacji od (jeśli została wskazana na recepcie)
- datę realizacji do (dla recept elektronicznych z terminem ważności 365 dni)
- dane dot. leku:
  - identyfikator leku (EAN)
  - ilość przepisanego leku
  - nazwa leku
  - wielkość opakowania leku
  - poziom odpłatności za lek
- uprawnienia dodatkowe
- status recepty
- status możliwości realizacji recepty
- klucz recepty
- numer recepty

Jeśli recepta została zrealizowana bądź częściowo zrealizowana informacje są rozszerzone o:
- datę pierwszego wydania leku
- dane dokumentu realizacji recepty (sprzedaży leku)
  - datę wystawienia (sprzedaży leku)
  - identyfikator wydanego opakowania (EAN)
  - nazwę wydanego leku
  - ilość sprzedanego leu
  - poziom odpłatności
