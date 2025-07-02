# Usługa: e-Skierowania

Usługa pozwala na wystawianie e-skierowań oraz przyjmowanie e-skierowań do realizacji.

## Nagłówki
Aby wystawić e-skierowanie, należy w nagłówku przesłać token JWT, w którym zawarty będzie __podmiot__ (`organization`) oraz __specjalista__ (`practitioner`).  
- Authorization: Bearer {token}

## Dostępne trasy

#### Tworzenie e-skierowania:
- POST [`/eskierowanie/validate`](Eskierowania.md#weryfikacja-e-skierowania) - walidacja dokumentu e-skierowania
- POST [`/eskierowanie/send`](Eskierowania.md#wystawienie-e-skierowania) - zapisanie dokumentu e-skierowania w systemie P1
- DELETE `/eskierowanie/{id}/` - anulowanie dokumentu e-skierowania
- GET `/eskierowanie/{id}/` - pobieranie danych o e-skierowaniu
- GET `/eskierowanie/{id}/print` - wydruk e-skierowania
  
#### Wyszukiwanie e-skierowań:
- GET `/eskierowanie/searchown` - wyszukiwanie własnych e-skierowań sprawozdanych do systemu P1
- GET `/eskierowanie/search`  - wyszukiwanie e-skierowań usługobiorcy w systemie P1

#### Pobieranie e-skierowania do realizacji:
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

## e-skierowanie do uzdrowiska

Poniżej przykładowa treść żądania skierowania do uzdrowiska:

```json
{
  "id": "0000000000SKR00UZ00008",  // dokładnie 22 znakowy identyfikator dokumentu
  "date": "2025-07-01",
  "type": "02.10LU",  // typ dokumentu medycznego (patrz słownik)
  "serviceMode": "TS",  // tryb realizacji świadczenia (patrz słownik)
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
    "telecom": [   // dane kontaktowe pacjenta
      {
        "value": "+48131231230"  // telefon do pacjenta
      },
      {
        "value": "sms:+48131231230"  // możliwość kontaktu sms  (patrz słownik)
      },
      {
        "value": "email:ssenior@mojemail.co"  // adres email
      }
    ],
    "address": [
      {
        "use": "PST",  // określenie adresu do korespondencji (wymagane dla skierowania uzdrowiskowegom w przypadku korespondencji papierowej z pacjentem)
        "street": "Wrocławska",
        "houseNumber": "11A",
        "unitId": "3",
        "city": "Zielona Góra",
        "postalCode": "00-184",
        "country": "Polska"
      }
    ],
    "nfz": "07",  // oddział NFZ w którym jest ubezpieczony pacjent
    "gender": "M",
    "birthDate": "1940-01-01"
  },
  "reimbursed": "Y",  // dla skierowania do uzdrowiska - wymagane zawsze
  "nfzDepartment": "01",  // dla skierowania do uzdrowiska - wymagane zawsze (oddział NFZ z którym organizacja zawarła kontrakt)
  "nfzContract": "01-06-02336-20-03/06",   // dla skierowania do uzdrowiska - wymagane zawsze (numer umowy z NFZ)
  "title": "Skierowanie na leczenie uzdrowiskowe",
  "physicalFindings": [
     {
       "type": "p1_skieruzdro_bp_cts",
       "value": "105"
     },
     {
       "type": "p1_skieruzdro_bp_ctr",
       "value": "85"
     },
     {
       "type": "p1_skieruzdro_bp_mc",
       "value": "88.5"
     },
     {
       "type": "p1_skieruzdro_bp_wz",
       "value": "190.5"
     },
     {
       "type": "p1_skieruzdro_bp_te",
       "value": "90"
     },
     {
       "type": "p1_skieruzdro_bp_skora_wezly_chlonne",
       "value": "Bez wykwitów patologicznych. Węzły chłonne obwodowe bez zmian"
     },
     {
       "type": "p1_skieruzdro_bp_uklad_oddechowy_ocw",
       "value": "Osłuchowo szmer pęcherzykowy. Wydolny"
     },
     {
       "type": "p1_skieruzdro_bp_uklad_krazenia_ocw_nyha",
       "value": "Nadciśnienie tętnicze z nadkomorowymi zab. rytmu w sys. Farmakoterapii. Wydolny - NYHA 0/I"
     },
     {
       "type": "p1_skieruzdro_bp_uklad_trawienny",
       "value": "2002 rok - Cholecystectomia laparoskopowa. W normie wieku."
     },
     {
       "type": "p1_skieruzdro_bp_uklad_moczoplciowy_ocw_nerek",
       "value": "2000 rok - Histerectomia (mięśniaki macicy). Nerki wydolne. W normie wieku."
     },
     {
       "type": "p1_skieruzdro_bp_uklad_ruchu",
       "value": "Bólowe ograniczenie ruchomości kr. L-S, CC oraz st. kolanowych"
     },
     {
       "type": "p1_skieruzdro_bp_zdolnosc_samoobslugi",
       "value": "TAK"
     },
     {
       "type": "p1_skieruzdro_bp_ocena_sprawnosc_ruchowa",
       "value": "Poruszający się przy pomocy (kule, balkonik, wózek)"
     },
     {
       "type": "p1_skieruzdro_bp_uklad_nerwowy_narzady_zmyslow",
       "value": "W normie wieku"
     },
     {
       "type": "p1_skieruzdro_bp_przeciwskazania_nat_surowce_lecznicze",
       "value": "nie"
     }
  ],
  "reason": [
  	 {
       "code": "ULA"
     },
	   {
       "code": "PJZ"
     },
	   {
       "code": "INN",
       "display": "Edukacja zdrowotna"	
     }
  ],
  "condition": [
     {
       "icd10Code": "H60.0",
       "icd10Display": "Ropień ucha zewnętrznego",
       "bodySite": "R"
     }
  ],
  "referralHistory": {
     "oncologyTreatment": true,
     "sanatoriumIn3y": "Tak, Ciechocinek 2020",
     "immunizationHistory": [
         {
           "icd9Code": "99.395",
           "icd9Display": "Szczepienie przeciw błonicy/tężcowi/krztuścowi/polio/Hib/WZW typu B"
         },
         {
           "icd9Code": "99.33",
           "icd9Display": "Szczepienie przeciw gruźlicy"
         }
     ],
     "narrative": "- Pacjent z bólami kręgosłupa.<br/> - Inny problem<br/>- Dotychczas stosowano leczenie objawowe.<br/>- Stosowano głównie lek...<br/>"
  },
  "observation": [
     {
       "icd9Code": "A01",
       "display": "Mocz badanie ogólne",
       "effectiveTime": "20220608",
       "narrative": "<table><thead><tr><th>Nazwa badania</th><th>Wynik badania</th><th>Uwaga</th><th>Zakres referencyjny</th></tr></thead><tbody><tr><td colspan=\"4\" styleCode=\"Bold\">Mocz - badanie ogólne</td></tr><tr><td>Barwa</td><td>jasnożółty</td><td/><td/></tr><tr><td>Przejrzystość</td><td>zupełna</td><td/><td/></tr><tr><td>PH (mocz)</td><td>7,0</td><td/><td>4,6 - 8,0</td></tr><tr><td>Ciężar właściwy</td><td>1,015</td><td/><td>1,015 - 1,030</td></tr><tr><td>Białko</td><td>198,0 mg/dl</td><td>0,00</td><td/></tr></tbody></table>"
     },
     {
       "icd9Code": "C59",
       "display": "Odczyn opadania krwinek czerwonych",
       "effectiveTime": "20220608",
       "narrative": "<table><thead><tr><th>Nazwa badania</th><th>Wynik badania</th><th>Uwaga</th><th>Zakres referencyjny</th></tr></thead><tbody><tr><td colspan=\"4\" styleCode=\"Bold\">OB. -odczyn opadania krwinek czerwonych</td></tr><tr><td>OB</td><td>2 mm/h</td><td/><td>2-15</td></tr></tbody></table>"
     },
     {
       "icd9Code": "C55",
       "display": "Morfologia krwi, z pełnym różnicowaniem granulocytów",
       "effectiveTime": "20220608",
       "narrative": "<table><thead><tr><th>Nazwa badania</th><th>Wynik badania</th><th>Zakres referencyjny</th></tr></thead><tbody><tr><td colspan=\"3\" styleCode=\"Bold\">Morfologia krwi</td></tr><tr><td>WBC Krwinki białe</td><td>8,3 K/µl</td><td>4,0 - 10,0</td></tr><tr><td>RBC Krwinki czerwone</td><td>3,35 M/µl</td><td>4,0 - 5,0</td></tr><tr><td>PLT Płytki krwi</td><td>331,0 K/µl</td><td>140,0 - 400,0</td></tr><tr><td>HGB Hemoglobina</td><td>7,8 g/dl</td><td>12,0 - 16,0</td></tr><tr><td>HCT Hematokryt</td><td>27,1 %</td><td>37,0 - 47,0</td></tr><tr><td>MCHC</td><td>28,8 g/dl</td><td>31,0 - 36,0</td></tr><tr><td>MCV</td><td>80,9 fl</td><td>80,0 - 96,0</td></tr><tr><td>MCH</td><td>23 pg</td><td>26,0 - 32,0</td></tr></tbody></table>"
     },
     {
       "icd9Code": "89.522",
       "display": "Elektrokardiografia z 12 lub więcej odprowadzeniami (z opisem)",
       "effectiveTime": "20220608",
       "narrative": "W załączniku"
     },
     {
       "icd9Code": "44.16",
       "display": "Gastroskopia",
       "effectiveTime": "20240608",
       "narrative": "W załączniku"
     }
  ],
  "attachment": [
     {
       "identifierRoot": "2.16.840.1.113883.3.4424.2.7.1321.7.1",
       "identifier": "2345678",
       "code": "77599-9",
       "codeP1": "06.10",
       "description": "Badanie z dnia 15 maja"
     },
     {
       "identifierRoot": "2.16.840.1.113883.3.4424.2.7.1321.7.1",
       "identifier": "2345678",
       "code": "77599-9",
       "codeP1": "06.10",
       "description": "Badanie z dnia 15 maja"
     }
  ],
  "socialHistory": "Informacje o szkole..."
}
```

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


## Wyszukiwanie e-skierowań:
#### GET `/eskierowanie/searchown` - wyszukiwanie własnych e-skierowań sprawozdanych do systemu P1
Parametry określające kryteria wyszukiwania:
- date_from
- date_to
- status (WYSTAWIONE, U_REALIZATORA, ZREALIZOWANE, ANULOWANE)

W wyniku wykonania operacji zwracane są informacje o znalezionych skierowaniach danego pracownika medycznego. Maksymalna liczba zwracanych wyników jest ograniczona przez P1. Po przekroczeniu limitu system P1 zwraca błąd wykonania operacji (nie zwraca wyników), a w celu poprawnego wyszukania należy zawęzić kryteria wyszukiwania.

*Przykład odpowiedzi:*
```
    "wynikiWyszukiwaniaSkierowan": [
      {
        "dataWystawieniaSkierowania": "2024-07-30T00:00:00+02:00",
        "numerSkierowania": {
          "extension": "7d208e90000000000540RT",
          "root": "2.16.840.1.113883.3.4424.2.7.1294.4.1"
        },
        "statusSkierowania": "WYSTAWIONE"
      }
    ]
```

#### GET `/eskierowanie/search` - wyszukiwanie e-skierowań usługobiorcy w systemie P1
Parametry określające kryteria wyszukiwania:
- identifier (wymagany)
- identifier_type (domyślnie numer PESEL)
- date_from
- date_to
- status (WYSTAWIONE, U_REALIZATORA, ZREALIZOWANE, ANULOWANE)

Operacja umożliwia uzyskanie listy skierowań danego Usługobiorcy na podstawie zadanych parametrów wyszukiwania na potrzeby usługodawcy. Jako wynik wyszukiwania zwracane są jedynie informacje o skierowaniach wystawionych przez pracownika medycznego odpytującego oraz skierowań do których ma dostęp. 

*Przykład odpowiedzi:*
```
    "wynikiWyszukiwaniaSkierowanUslugobiorcy": [
      {
        "numerSkierowania": {
          "numerSkierowania": {
            "extension": "7d208e90000000000540RT",
            "root": "2.16.840.1.113883.3.4424.2.7.1294.4.1"
          }
        },
        "statusSkierowania": "WYSTAWIONE",
        "dataWystawieniaSkierowania": "2024-07-30T00:00:00+02:00",
        "identyfikatorPodmiotuWystawcy": {
          "extension": "000000927560",
          "root": "2.16.840.1.113883.3.4424.2.4.68"
        },
        "identyfikatorPracownikaWystawcy": {
          "extension": "7540444",
          "root": "2.16.840.1.113883.3.4424.1.6.2"
        },
        "podmiotNazwa": "Artur260 Praktyczny",
        "wystawcaNazwa": "Praktyczny Artur260"
      }
    ]
```

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
