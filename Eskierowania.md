# Usługa: e-Skierowania

Usługa pozwala na wystawianie e-skierowań oraz przyjmowanie e-skierowań do realizacji.

## Nagłówki
Aby wystawić e-skierowanie, należy w nagłówku przesłać token JWT, w którym zawarty będzie __podmiot__ (`organization`) oraz __specjalista__ (`practitioner`).  
- Authorization: Bearer {token}

## Dostępne trasy

- POST [`/eskierowanie/validate`](Eskierowania.md#weryfikacja-e-skierowania) - walidacja dokumentu e-skierowania
- POST [`/eskierowanie/send`](Eskierowania.md#wystawienie-e-skierowania) - zapisanie dokumentu e-skierowania w systemie P1
- DELETE `/eskierowanie/{id}/` pozwala anulować e-skierowanie
- GET `/eskierowanie/{id}/` odpowiada za pobieranie danych o e-skierowaniu
- GET `/eskierowanie/{id}/print` - wydruk e-skierowania
- GET [`/eskierowanie/readout`](Eskierowania.md#odczyt-dokumentu-e-skierowania-do-realizacji) - odczyt dokumentu skierowania do realizacji
- POST [`/eskierowanie/acceptance`](Eskierowania.md#przyjcie-skierowania-do-realizacji) - przyjęcie dokumentu skierowania do realizacji
- POST [`/eskierowanie/rejection`](Eskierowania.md#odmowa-realizacji-e-skierowania) - odmowa realizacji skierowania
- POST [`/eskierowanie/gaps`](Eskierowania.md#zgoszenie-informacji-o-braku-w-dokumentacji) - zgłoszenie informacji o braku w dokumentacji
- POST [`/eskierowanie/resignation`](Eskierowania.md#rezygnacja-z-realizacji-skierowania) - rezygnacja z realizacji skierowania
- POST [`/eskierowanie/completion`](Eskierowania.md#zakoczenie-realizacji-e-skierowania) - zamkniecie realizacji skierowania

## Kody odpowiedzi
1. `200` - Poprawna odpowiedź
2. `400` - Błąd składni JSON
3. `403` - Brak uprawnień do wykonania polecenia
4. `404` - Zasób nie istnieje
5. `406` - Błąd w wysyłaniu e-skierowania (np. nieprawidłowe dane)

## Pobieranie e-skierowania

Aby odczytać skierowanie, musimy znać [oidRoot](Tools.md) organizacji, która wystawiła skierowanie.

Musimy mieć prawa odczytu skierowania. Możemy odczytywać skierowania wystawione przez naszą organizację lub realizowane przez nią.

```
GET /eskierowanie/{idSkierowania}[/{<a href="API/Dictionaries.md#9.1.">oidRoot</a>}][/(print|preview)]
```

Tylko *idSkierowania* jest obowiązkowym parametrem.

Jeżeli nie podamy parametru *oidRoot* użyty zostanie oidRoot organizacji zawartej w tokenie JWT.
Dzięki temu możemy odczytywać skierowania "naszej" organizacji.

Dodanie znacznika `/print` pozwala na pobranie dokumentu skierowania w formie gotowej do wydruku dla pacjenta.

Dodanie znacznika `/preview` zwróci nam dokument w formie podglądu dla lekarza.

Żądania zwracają dokument HTML.

### Przykłady

- [`GET /eskierowanie/0000000000SK1607436862`](eSkierowanie/examples/GetDocument.json)
- [`GET /eskierowanie/0000000000SK1607436862/print`](eSkierowanie/examples/Print.html)
- [`GET /eskierowanie/0000000000SK1607436862/preview`](eSkierowanie/examples/PreviewDoc.html)
- [`GET /eskierowanie/0000000000SK1607436862/2.16.840.1.113883.3.4424.2.7.67/print`](eSkierowanie/examples/Print.html)
- [`GET /eskierowanie/0000000000SK1607436862/2.16.840.1.113883.3.4424.2.7.67/preview`](eSkierowanie/examples/PreviewDoc.html)

## Weryfikacja e-skierowania

Walidacja dokumentu pozwala poprawić dokument przed finalnym wystawieniem e-skierowania.

```
POST /eskierowanie/validate
```

Przykładowa treść żądania:

```json
{
  "id": "1234567890abcdef789012",
  // dokładnie 22 znakowy identyfikator dokumentu
  "date": "2021-01-23",
  "type": "02.10",
  // typ dokumentu medycznego (patrz słownik)
  "organization": "idabc",
  "practitioner": "idxyz",
  "patient": {
    "identifier": [
      {
        "type": "pesel",
        "value": "40010175826"
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
    // oddział NFZ w którym jest ubezpieczony pacjent
    "gender": "M",
    "birthDate": "1940-01-01"
  },
  "reimbursed": "Y",
  // używać tylko kiedy skierowanie ma być sfinansowane ze środków publicznych
  "nfzDepartment": "01",
  // oddział NFZ z którym organizacja zawarła kontrakt (wymagane jeżeli reimbursed=Y a brak tego w danych organizacji)
  "nfzContract": "01-06-02336-20-03/06",
  // numer umowy z NFZ (wymagane jeżeli reimbursed=Y a brak tego w danych organizacji)
  "title": "Skierowanie do poradni specjalistycznej",
  "description": [
    {
      "title": "Cel badania (uzasadnienie):",
      "content": "Diagnostyka przyczyny niedokrwistości."
    },
    {
      "title": "Badania dotychczas wykonane:",
      "content": "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Quisque sed urna nibh.\nDonec lacus mi, varius sed ex at, elementum mattis mauris. In sit amet quam vel sem pellentesque euismod."
    }
  ],
  "condition": [
    {
      "icd10Code": "H60.0",
      "icd10Display": "Ropień ucha zewnętrznego",
      "bodySite": "R"
      // opcjonalne
    }
  ],
  "procedure": [
    {
      "icd9Code": "18.11",
      "icd9Display": "Wziernikowanie ucha",
      "targetSiteCode": "001.000.000", // opcjonalne (wartość ze słownika https://www.cez.gov.pl/HL7POL-1.3.2/plcda-html-1.3.2/plcda-html/voc-2.16.840.1.113883.3.4424.13.11.86-2020-04-22T000000.html )
      "bodySite": "L" // opcjonalne (wymaga ustawienia targetSiteCode)
    },
    {
      "icd9Code": "96.52",
      "icd9Display": "Płukanie ucha"
    }
  ],
  "observation": "Temperatura ciała: 36,6 <sup>o</sup>C<br/>\nCiśnienie krwi: 140/90",
  // opcjonalne
  "priorityCode": "UR",
  // Cito - opcjonalne
  "performer": "Poradnia Otolaryngologiczna",
  "performerCode": "1610"
  // część viii systemu resortowych kodów identyfikacyjnych
}
```

### Przykłady odpowiedzi:
- [HTTP 200](eSkierowanie/examples/Validate200_1.json)
- [HTTP 400](eSkierowanie/examples/Validate400_1.json)
- [HTTP 406](eSkierowanie/examples/Validate406_1.json)
- [HTTP 406](eSkierowanie/examples/Validate406_2.json)

## Wystawienie e-skierowania

Zapisanie dokumentu skierowania w systemie P1.

```
POST /eskierowanie/send
```

Przed zapisem zalecana jest walidacja dokumentu.

Treść żądania jest taka sama jak treść walidacji dokumentu.

Patrz [`/eskierowanie/validate`](Eskierowania.md#weryfikacja-e-skierowania)

## Odczyt dokumentu e-skierowania do realizacji

Zgodnie z dokumentacją systemu P1:

*Operacja odczytDokumentuSkierowaniaDoRealizacji umożliwia, 
na podstawie otrzymanego od Usługobiorcy klucza lub kodu dostępowego, 
pobranie skierowania wraz z dołączonymi do niego wynikami weryfikacji 
otrzymanymi w ramach zapisu dokumentu do Systemu P1 oraz komentarzami. 
W wyniku wykonania operacji zwrotnie odesłane zostanie: 
treść skierowania wraz z kluczem na podstawie którego można wywołać 
operację odmowaPrzyjeciaDoRealizacjiSkierowania albo przyjecieDoRealizacjiSkierowania, 
przy czym każdy podmiot ma dostęp jedynie do skierowań o statusie WYSTAWIONE 
(operacja nie powoduje zmiany statusu skierowania). 
(...) Uzyskanie danych wymaga posiadania uprawnień twórcy dokumentu lub 
Preautoryzacji uprawnień, Autoryzacji uprawnień.*

Szukanie po kluczu:
```
GET /eskierowanie/readout/key/{eskierowanieKey}[/preview]
```

Szukanie po kodzie dostępowym: 
```
GET /eskierowanie/readout/code/{eskierowanieCode}[/preview]
```

Zazwyczaj parametr `{eskierowanieCode}` przyjmuje postać:

`XXXXYYYYYYYYYYY`

gdzie

- `XXXX` to czterocyfrowy kod otrzymany przez pacjenta (na wydruku lub przez SMS)
- `YYYYYYYYYYY` to identyfikator pacjenta (np. PESEL)

Parametr `{eskierowanieKey}` znajduje się w postaci kodu kreskowego na wydruku skierowania.

Jeżeli użyjemy opcjonalnej flagi `preview` w odpowiedzi otrzymamy HTML zawierający wizualizację skierowania.

### Przykłady

- [`GET /eskierowanie/readout/key/10031894188264384543363841602402324302238120`](eSkierowanie/examples/Readout.json)
- [`GET /eskierowanie/readout/code/0000AC34567890`](eSkierowanie/examples/Readout.json)
- [`GET /eskierowanie/readout/key/10031894188264384543363841602402324302238120/preview`](eSkierowanie/examples/Preview.html)

## Przyjęcie skierowania do realizacji

Zgodnie z dokumentacją systemu P1:

*Operacja przyjecieDoRealizacjiSkierowania umożliwia, na podstawie klucza skierowania, przyjęcie skierowania do realizacji. Przyjęcie wiąże się ze zmianą statusu skierowania na
"U realizatora" - pozytywny wynik wykonania operacji oznacza potwierdzenie zmiany statusu skierowania. (...)
W ramach przyjęcia skierowania wymagane jest określenie czy Usługobiorca zadeklarował realizację świadczenia jako refundowane czy jako pełnopłatne. System P1 weryfikuje możliwość realizacji takiego świadczenia wystawionego jako refundowane (Usługobiorca ma możliwość realizacji skierowania jako pełnopłatne mimo, iż zostało ono wystawione jako skierowanie refundowane). (...) Uzyskanie danych wymaga posiadania uprawnień twórcy dokumentu lub Preautoryzacji uprawnień, Autoryzacji uprawnień.*

```
POST /eskierowanie/acceptance/{eskierowanieKey}[[/R]/{terminWizyty}]
```

Treść (opcjonalnie):
```json
{
  "comment": "Komentarz dotyczący przyjęcia e-skierowania do przyjęcia."
}
```

Parametr `{eskierowanieKey}` znajduje się w postaci kodu kreskowego na wydruku skierowania. Możemy go również odczytać przez [odczyt dokumentu skierowania do realizacji](Eskierowania.md#odczyt-dokumentu-e-skierowania-do-realizacji)

Jeżeli użyjemy opcjonalnej flagi `R`, informujemy system P1, że skierowanie będzie realizowane jako refundowane.

Również opcjonalnie możemy podać termin wizyty w formacie np. YYYY-MM-DD hh:mm (oczywiście parametr należy odpowiednio zakodować)

### Przykłady

- `POST /eskierowanie/acceptance/10031894188264384543363841602402324302238120`
- `POST /eskierowanie/acceptance/10031894188264384543363841602402324302238120/R`
- `POST /eskierowanie/acceptance/10031894188264384543363841602402324302238120/N/2021-03-22 12:30`

## Zakończenie realizacji e-skierowania

Zgodnie z dokumentacją systemu P1:

*Operacja zakonczenieRealizacjiSkierowania umożliwia, na podstawie numeru skierowania, zmianę statusu skierowania na "Zrealizowane" (nie jest wymagane przekazanie żadnego dokumentu realizacji skierowania). Podmiot który dokonał realizacji skierowania może dodać komentarz dot. realizacji, a w przypadku skierowań refundowanych może wskazać identyfikator płatnika z którym podmiot ma zawartą umowę. Nie ma możliwości wykonania niniejszej operacji jeśli status skierowanie jest "Wystawione" albo "Zrealizowane" albo "Anulowane", lub jest w trakcie realizacji przez inny podmiot (status "U Realizatora").*

```
POST /eskierowanie/completion/{idSkierowania}/{<a href="API/Dictionaries.md#9.1.">oidRoot</a>}
```

- idSkierowania - numer skierowania
- [oidRoot](Tools.md) organizacji, która wystawiła skierowanie.

Treść:
```json
{
  "comment": "Komentarz - zakończenie",
  "payer": "01" // opcjonalnie - oddział NFZ płatnika
}
```

### Przykłady

- `POST /eskierowanie/completion/0000000000SK1607436862/2.16.840.1.113883.3.4424.2.7.67`

## Zgłoszenie informacji o braku w dokumentacji

Zgodnie z dokumentacją systemu P1:

*Operacja przekazanieInformacjiOBrakachDokumentacjiSkierowania umożliwia, na podstawie numeru skierowania, dodanie do skierowania komentarza z informacją o brakach w dokumentacji medycznej i niezbędnych uzupełnieniach (operacja dostępna jedynie dla skierowania o statusie „U realizatora„ dla podmiotu który jako ostatni przyjął to skierowanie do realizacji - aktualnie je realizuje). Dodanie komentarza nie wiąże się ze zmianą statusu skierowania.*

```
POST /eskierowanie/gaps/{idSkierowania}/{<a href="API/Dictionaries.md#9.1.">oidRoot</a>}
```

- idSkierowania - numer skierowania
- [oidRoot](Tools.md) organizacji, która wystawiła skierowanie.

Treść:
```json
{
  "comment": "Komentarz o brakach w dokumentacji"
}
```

### Przykłady

- `POST /eskierowanie/gaps/0000000000SK1607436862/2.16.840.1.113883.3.4424.2.7.67`

## Odmowa realizacji e-skierowania

Zgodnie z dokumentacją systemu P1:

*Operacja odmowaPrzyjeciaDoRealizacjiSkierowania umożliwia, na podstawie klucza skierowania, dodanie do skierowania informacji o przyczynach odmowy przyjęcia skierowania do realizacji przed dany podmiot. Odmowa nie wiąże się ze zmianą statusu skierowania (pozostaje status "Wystawione").*

```
POST /eskierowanie/rejection/{eskierowanieKey}
```

Treść (minimum 5 znaków):
```json
{
  "comment": "Komentarz - odrzucenie skierowania do realizacji"
}
```

Parametr `{eskierowanieKey}` znajduje się w postaci kodu kreskowego na wydruku e-skierowania. 
Możemy go również odczytać przez [odczyt dokumentu skierowania do realizacji](Eskierowania.md#odczyt-dokumentu-e-skierowania-do-realizacji)

### Przykłady

- `POST /eskierowanie/rejection/10031894188264384543363841602402324302238120`

## Rezygnacja z realizacji skierowania

Zgodnie z dokumentacją systemu P1:

*Operacja rezygnacjaZRealizacjiSkierowania umożliwia, na podstawie numeru skierowania, wycofanie skierowania do realizacji innym podmiotom leczniczy. Rezygnacja wiąże się ze zmianą statusu skierowania z „U realizatora” na "Wystawione". Rezygnacja może być wykonana tylko przez podmiot, który jako ostatni przyjął to skierowanie do realizacji (aktualnie je realizuje). Po wykonaniu pozytywnym operacji skierowanie może być zrealizowana w każdym podmiocie.*

```
POST /eskierowanie/resignation/{idSkierowania}/{<a href="API/Dictionaries.md#9.1.">oidRoot</a>}
```

- idSkierowania - numer skierowania
- [oidRoot](Tools.md) organizacji, która wystawiła skierowanie.

Treść:
```json
{
  "comment": "Rezygnacja z powodu..."
}
```

### Przykłady

- `POST /eskierowanie/resignation/0000000000SK1607436862/2.16.840.1.113883.3.4424.2.7.67`
