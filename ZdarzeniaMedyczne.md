# Usługa: Zdarzenia Medyczne

## Wstęp

Raportowanie Zdarzeń Medycznych to jedna z funkcji udostępnianych przez P1. Raporty są podzielone na tzw. zasoby (Resources). Zasoby są przesyłane w formacie HL7 Fhir - https://hl7.org/fhir/

Pełna dokumentacja opisująca struktury zasobów znajduje się w dokumentacji integracyjnej od P1 na isus.ezdrowie.gov.pl w pliku o nazwie P1-DS-Dokumentacja-integracyjna-Systemu-P1-w-zakresie-obsługi-ZM v15.1 - gdzie v15.1 to numer obowiązującej wersji, która zmienia się raz na kilka tygodni (dość często).

Wersjonowanie w P1 polega na tym, że każda zmiana w zasobie wymaga wysłania nowego dokumentu XML zawierającego wszystkie informacje. Dokument po wysłaniu do P1 staje się wersją aktualną z numerem równym `poprzedni numer + 1`.

### Zasoby dostępne w P1

- Encounter - podstawowe dane Zdarzenia Medycznego
- Patient - dane osobowe Pacjenta
- Procedure - dane dotyczące wykonanych procedur medycznych (ICD-9)
- Condition - dane dotyczące rozpoznań (ICD-10)
- Provenance - zasób do potwierdzania autentyczności danych (używany w wybranych przypadkach)
- DocumentReference - powstaje po wysłaniu plików do Indeksu EDM (inna usługa)
- Organization - dane organizacji/podmiotu bazuje na rejestrze rpwdl.csioz.gov.pl
- Observation - dane dotyczące cech Pacjenta
- Device - dane urządzeń i implantów Pacjenta
- Coverage - opisuje uprawnienia Pacjenta (NFZ)
- Claim - dane dotyczące pozycji rozliczeniowych (NFZ)
- Immunization - dane nt. szczepień Pacjenta
- AllergyIntolerance - alergie Pacjenta

### Pobieranie zasobu istniejącego w P1

```
GET /zm/{zasób}/{id<\d+>}/{_history([/]\d+)?}
```

- {zasób} - to np. Patient
- {id} - to identyfikator nadany przez P1 - możemy o otrzymać tworząc nowy zasób lub przez przeszukiwanie P1 (listowanie)
- {_history} - to parametr opcjonalny do pobieranie starszej wersji zasobu

### Konwertowanie zasobu do HTML (np. do wydruku)

```
GET /zm/{zasób}/{id<\d+>}/{_history([/]\d+)?}
Accept: text/html
```

- aby skonwertować zasób do HTML (z XML źródłowego) należy w nagłówku Accept wskazać format `text/html`

### Szukanie zasobów w P1 (listowanie)

```
GET /zm/{zasób}/?{parametry}
```

- {zasób} - to np. Patient
- {parametry} - to lista parametrów typu GET (query params) do przeszukiwania P1, każdy zasób ma inne parametry i są one opisane dokładnie w dokumentacji P1, nie umieszczamy ich tutaj ponieważ zmieniają się bardzo często i lepszym źródłem informacji jest dokument źródłowy.

#### Przykład wyszukiwania pacjenta po numerze PESEL, imieniu i nazwisku:

```
GET /zm/Patient/?plpatient=urn:oid:2.16.840.1.113883.3.4424.1.1.616|40010151673&plgiven=Sylwester&plfamily=Senior
```

- 2.16.840.1.113883.3.4424.1.1.616 - to OID reprezentujący numer PESEL (krajowy identyfikator dla Polski). Każdy identyfikator taki jak paszport, dowód osobisty, karta żeglarska ma inny OID w zależności od kraju.

### Tworzenie nowych zasobów w P1

```
POST /zm/{zasób}

{XML}
```

- {zasób} - to np. Patient
- {XML} W body żądania należy przesłać plik XML z zasobem w formacie FHIR. Alternatywnie planujemy udostępnić opcję wysyłania danych w formacie Json zgodnym z FHIR - przykłady są w trakcie budowy.

### Aktualizacja istniejącego zasobu w P1

```
PUT /zm/{zasób}/{id}

{XML}
```

- {zasób} - to np. Patient
- {id} - to identyfikator wewnętrzny w P1 nadany temu zasobowi przez P1
- {XML} W body żądania należy przesłać plik XML z zasobem w formacie FHIR.

### Pobieranie rejestru OID dla identyfikatorów pacjentów

```
GET /zm/dictionary/personalid/{kraj}/{typ}
```

- {kraj} - nazwa kraju - opcjonalna (tekst np. Polska)
- {typ} - typ identyfikator - opcjonalny (wartość 0-5)

### Usuwanie zasobów w P1

```
DELETE /zm/{zasób}/{id<\d+>}
```

- {zasób} - to np. Patient
- {id} - to identyfikator wewnętrzny w P1 nadany temu zasobowi przez P1

### Pobieranie innych słowników potrzebnych w ZM

```
GET /zm/dictionary/p1/{typ}
```

- {typ} - typ słownika - opcjonalny (tekst np. PLMEDICALEVENTADMITSOURCE)

### Podpisywanie zasobów

Aby wybrane zdarzenie medyczne było kompletne musi zostać podpisane.
Podpisuje się zestaw dokumentów wchodzących w skład całego Encounter.

> __Uwaga__
> 
> Tylko podpisane dokumenty widoczne będą dla innych lekarzy oraz w portalu pacjenta.

Przykład żądania podpisania jednego zdarzenia medycznego:

```http request
POST /zm/sign
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}

{
  "Patient": [{"id": "540", "ver":  "4051"}],
  "Encounter": [{"id": "2683447", "ver":  "1"}],
  "Condition": [{"id": "2683453", "ver":  "1"}],
  "Procedure": [{"id": "2683450", "ver":  "1"}]
}
```

Gdzie "id" przy zasobach to identyfikatory nadane przez P1 dla dokumentów wysłanych do ich rejestru.
Najnowszy numer wersji również uzyskamy, pobierając zasób z P1.

> __Uwaga__
> 
> Zasoby mają ograniczoną ilość modyfikacji w P1.
> Oznacza to, że w pewnym momencie nie będzie można zaktualizować zasobu Patient i np. będziemy musieli podpisać zasób Patient, który zawiera inne dane niż nasze aktualne informacje (np. adres pacjenta).

> __Uwaga__
> 
> Każdy typ zdarzenia medycznego np. Porada, Pobyt ma inne wymagania co do występowania Condition i Procedure. Niektóre wymagają aby zaraportować przynajmniej jedną procedurę a inne aby zaraportować procedurę oraz diagnozę. Szczegóły znajdują się w najnowszej wersji dokumentacji P1.