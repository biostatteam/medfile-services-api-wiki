# Usługa: Repozytorium XDS.b

## Podstawowe informacje

### Co to jest repozytorium XDS.b

To usługa do długoterminowego przechowywania plików będących dokumentacją medyczną.
Podstawowym przeznaczeniem repozytorium jest udostępnianie dokumentacji medycznej innym podmiotom medycznym.

### Jak uzyskać repozytorium XDS.b?

Obecnie wszystkie repozytoria tworzone są przez support Medfile. 
Identyfikator repozytorium jest generowany i przekazywany zaraz po podpisaniu odpowiedniej umowy.

### Jakie są adresy usługi repozytorium XDS.b

Środowisko testowe: https://dev.xds.rstat.pl

Środowisko produkcyjne: https://xds.rstat.pl (dostępne tylko otrzymaniu specjalnego dostępu)

### Jakie pliki można przechowywać w repozytorium?

Zgodne z ustawą, przede wszystkim pliki w formacie CDA np. wypisy ze szpitali, ale też pliki DICOM np. z rezonansu.

__Ważne:__ W jednym repozytorium mogą znajdować się pliki wielu podmiotów i praktyk. Integrator sam kontroluje ile i jakie pliki pozwala przechowywać w repozytorium swoim podmiotom i praktykom. 

## Integracja z repozytorium

### Autoryzacja w usłudze repozytorium

Repozytorium XDS.b to zupełnie osobna usługa. Integratorzy autoryzują się w repozytorium przy użyciu usługi Medfile API (services.medfile.pl).

Do usługi Medfile API autoryzujemy się tokenem JWT wygenerowanym zgodnie z zasadami opisanymi w sekcji "Autoryzacja w Medfile API".
Aby otrzymać token JWT do autoryzacji w repozytorium, należy pobrać go z Medfile API poniższą trasą. 

```http request
GET /tools/xdstoken/{repositoryId}
```

Otrzymany w odpowiedzi token JWT jest ważny tylko przez kilka minut.
Należy go użyć do każdej operacji na repozytorium w postaci nagłówka;
```
Authorization: Bearer {xdsToken}
```

### Magazyny danych

W ramach jednego repozytorium możliwe jest utrzymywanie wielu magazynów i lokalizacji do przechowywania plików.
Domyślnie Medfile udostępnia zgodnie z cennikiem zawartym w umowie jeden magazyn w chmurze.

#### Pobieranie listy magazynów

Aby pobrać dostępną listę magazynów na pliki, należy wykonać poniższą trasę:

```http request
GET /admin/repository/{repositoryId}/store
```

Odpowiedź:
```json
[
  {
    "id": "storeUUID",
    "name": "storeName",
    "type": "storeType"
  }
]
```

### Wysyłanie plików do magazynu

Pliki wysyła się do magazynu w postaci "multipart" na trasę:

```http request
POST /store/{storeUUID}
```

Przykład żądania multipart:

```http request
POST /store/{storeUUID} HTTP/1.1
Host: dev.xds.rstat.pl
Content-Type: multipart/form-data; boundary=W8idujGj
User-Agent: XDS Api Client v1.0
Authorization: Bearer {xdsToken}
Accept: */*
Accept-Encoding: gzip
Transfer-Encoding: chunked

c
--W8idujGj

7e
Content-Type: text/plain
Content-Transfer-Encoding: 8bit
Content-Disposition: form-data; name="file"; filename="test.txt"

2


a
this is test file content
2


e
--W8idujGj--

0
```

Odpowiedź:

```json
[
  {
    "id": "fileUuid",
    "name": "test.txt"
  }
]
```

### Pobieranie plików bezpośrednio z magazynu

Do pobierania plików z własnego repozytorium można użyć trasy bezpośrednio do magazynu.

```http request
GET /store/{fileId}
```

Gdzie
- fileId - to identyfikator pliku nadany podczasu uploadu

W odpowiedzi otrzymają Państwo zawartość pliku:

```text
eg. this is test file content
```

# Indeks P1

Pliki, które znajdują się w repozytorium XDS.b, nie są jeszcze dostępne dla żadnych podmiotów medycznych.
P1 prowadzi indeks plików udostępnianych przez podmioty. W tym indeksie znajdują się informacje o położeniu plików i ich metadane np. relacja do Zdarzeń Medycznych oraz informacje o podmiocie, który je w indeksie umieścił.

## Repozytorium P1

Zanim podmiot zacznie rejestrować pliki w indeksie P1, musi utworzyć repozytorium w P1.
Każdy podmiot może mieć wiele repozytoriów w P1. Np. z różnych programów gabinetowych.

Rejestrowanie nowego repozytorium w P1 wymaga wskazania jedynie certyfikatów TLS oraz WSS podmiotu lub praktyki. Będą one służyły do autoryzacji pomiędzy usługami P1.

## Konfiguracja certyfikatów TLS oraz WSS organizacji:

```http request
POST /admin/repository/{repositoryId}/certs
```

Treść żądania:
```json
{
  "tls": {
    "certificate": "tls_certificate_base64_encoded",
    "password": "tls_certificate_password"
  },
  "wss": {
    "certificate": "wss_certificate_base64_encoded",
    "password": "wss_certificate_password"
  }
}
```
## Rejestrowanie repozytorium w indeksie EDM P1

Wykonanie tej operacji utworzy nowe repozytorium w indeksie EDM P1.
Każdy podmiot może posiadać wiele różnych indeksów w P1. Oznacza to, że jedno repozytorium XDS.b (w Medfile API) możne być podpięte do wielu indeksów EDM w P1.

```http request
POST /repository/{id}/register
```

Gdzie:
- `id` - Unikalny identyfikator repozytorium uzyskany od Medfile

## ITI-42 - Rejestrowanie pliku w indeksie EDM P1

```
POST /file/{id}/index
```
Gdzie:
- `id` - Identyfikator pliku w repozytorium XDS.b (Medfile API)

Treść żądania:

```json
{
  "organization": {
    "name": "Szpital Biela\u0144ski",
    "identifier": [
      {
        "type": "2.16.840.1.113883.3.4424.2.3.1",
        "value": "000000927257"
      },
      {
        "type": "oidRoot",
        "value": "2.16.840.1.113883.3.4424.2.7.952"
      },
      {
        "type": "speciality",
        "value": "Szpital uzdrowiskowy dla dzieci"
      },
      {
        "type": "specialityCode",
        "value": "1008"
      },
      {
        "type": "medicalField",
        "value": "Chirurgia og\u00f3lna"
      },
      {
        "type": "medicalFieldCode",
        "value": "05"
      }
    ]
  },
  "practitioner": {
    "identifier": [
      {
        "type": "2.16.840.1.113883.3.4424.1.6.2",
        "value": "8310830"
      }
    ],
    "name": [
      {
        "family": "Brz\u0119czek",
        "given": [
          "Szymon"
        ]
      }
    ],
    "role": "LEK"
  },
  "patient": {
    "identifier": [
      {
        "type": "2.16.840.1.113883.3.4424.1.1.616",
        "value": "40010151673"
      }
    ],
    "name": [
      {
        "family": "SENIOR",
        "given": [
          "SYLWESTER"
        ]
      }
    ],
    "address": [
      {
        "city": "Warszawa",
        "street": "Ulica",
        "state": "Mazowieckie",
        "postalCode": "00-001",
        "country": "POL",
        "houseNumber": "1"
      }
    ],
    "gender": "M",
    "birthDate": "1940-01-01"
  },
  "submissionUniqueId": "3a991c33-3637-42c2-81f1-78bf93194859",
  "uniqueId": "3a991c33-3637-42c2-81f1-78bf93194860",
  "languageCode": "pl-PL",
  "storageCategory": "2022",
  "medicalEventId": "0000000000ZM165848903900000000123456",
  "typeCode": "10006-5",
  "classCode": "06.10",
  "confidentialityCode": "N",
  "format": "urn:extPL:pl-cda"
}
```

- organization
    - name - name of organization
    - identifier - array of organization identifiers
        - type - type of identifier
        - value - identifier value


- practitioner
    - identifier - array of practitioner identifiers
        - type - type of identifier
        - value - identifier value
    - name - array of practitioner names, only first element will be sent
        - family - practitioner family name
        - given - array of practitioner given names
    - role - practitioner role, one of the following values:
      ```
      LEK - medical doctor
      LEKD - dentist
      FIZJO - physiotherapist
      FARM - pharmacist
      PIEL - nurse
      POL - midwife
      DIAG - laboratory diagnostician
      FEL - feldsher
      RAT - paramedic
      PROF - medical professional
      PADM - administrative employee
      ASYS - medical assistant
      HIGSZKOL - school hygienist
      ```

- patient
    - identifier - array of patient identifiers
        - type - type of identifier
        - value - identifier value
    - name - array of patient names
        - family - patient family name
        - given - array of patient given names, only first one will be sent
    - address - array of patient addresses
        - city - name of city
        - street - name of street
        - state - name of state or province
        - postalCode - postal code
        - country - ISO 3166 3-character country code
        - houseNumber - house number
    - gender - patient gender, one of the following values:
      ```
      O - other
      M - male
      F - female
      U - unknown
      ```
    - birthDate - patient birth date

- submissionUniqueId - submission id in your system, must be unique
- uniqueId - document id in your system, must be unique
- languageCode -  document language (IETF language tag)
- storageCategory - year of document destruction
- medicalEventId - id of medical event
- typeCode - document type code from LOINC dictionary
- classCode - document type code from P1 dictionary
- confidentialityCode - document confidentiality code, one of the following values:
  ```
  N - normal
  R - restricted
  V - very restricted
  ```
- format - document format from "Kody formatów P1" dictionary

W odpowiedzi otrzymają Państwo identyfikator z Indeksu EDM P1 dla danego pliku:

```
urn:uuid:da3907a8-3457-4361-8e8a-b20a8fb0247b
```

## ITI-42 - Modyfikacja informacji w indeksie EDM P1

```
POST /file/{id}/transform/{transformed_id}
```

Gdzie:
- `id` - Identyfikator pliku w repozytorium XDS.b (Medfile API)
- `transformed_id` - Identyfikator pliku z indeksu EDM P1 - tj. urn:uuid:.... z powyższego przykładu.

Treść żądania jest identyczna z rejestracją pliku w indeksie EDM P1.
Dodatkowo mogą Państwo ustawić dodatkowe informacje:

```json
{
  ...
  "transformationType": "Translation into LanguageExample",
  "transformationTypeCode": "LanguageExample",
  "transformationTypeCodeSystemName": "Example translation types"
}
```

- transformationType - transformation display name
- transformationTypeCode - transformation code value
- transformationTypeCodeSystemName - name of transformation code system

W odpowiedzi otrzymają Państwo identyfikator z Indeksu EDM P1 dla danego pliku:

```
urn:uuid:da3907a8-3457-4361-8e8a-b20a8fb0247b
```

## ITI-42 - Rejestrowanie załączników do dokumentów

```
POST /file/{id}/append/{appended_to_id}
```

Gdzie:
- `id` - Identyfikator pliku w repozytorium XDS.b (Medfile API)
- `appended_to_id` - Identyfikator pliku z indeksu EDM P1 - tj. urn:uuid:.... z powyższego przykładu.

Treść żądania jest identyczna z rejestracją pliku w indeksie EDM P1.
Dodatkowo mogą Państwo ustawić dodatkowe informacje:

```json
{
  ...
  "appendType": "Translation into LanguageExample",
  "appendTypeCode": "LanguageExample",
  "appendTypeCodeSystemName": "Example translation types"
}
```
Gdzie:
- `appendType` - type of appended document display name
- `appendTypeCode` - type of appended document code value
- `appendTypeCodeSystemName` - name of code system

W odpowiedzi otrzymają Państwo identyfikator z Indeksu EDM P1 dla danego pliku:

```
urn:uuid:da3907a8-3457-4361-8e8a-b20a8fb0247b
```

## ITI-42 - Zamień zawartość indeksu EDM P1

```
POST /file/{id}/replace/{replaced_id}
```

Gdzie:
`id` - Identyfikator pliku w repozytorium XDS.b (Medfile API)
`transformed_id` - Identyfikator pliku z indeksu EDM P1 - tj. urn:uuid:.... z powyższego przykładu.

W odpowiedzi otrzymają Państwo identyfikator z Indeksu EDM P1 dla danego pliku:

```
urn:uuid:da3907a8-3457-4361-8e8a-b20a8fb0247b
```

Zastąpiony w ten sposób dokument zostanie oznaczony jako "DEPRECATED" w wyniku powyższej operacji.

## ITI-57 - Aktualizacja dokumentu w indeksie EDM P1

```
POST /file/{id}/update
```

Gdzie:
- `id` - Identyfikator pliku w repozytorium XDS.b (Medfile API)

Treść żądania jest identyczna z rejestracją pliku w indeksie EDM P1.

W wyniku otrzymają Państwo identyfikator dokumentu.

```
urn:uuid:da3907a8-3457-4361-8e8a-b20a8fb0247b
```

## ITI-57 - Anulowanie dokumentu z indeksu EDM P1

```
POST /file/{id}/cancel

id - your file id
```

Treść żądania zgodna z rejestracją pliku w indeksie EDM P1.
W treści muszą znaleźć się dane organizacji, lekarza, pacjenta oraz dane dotyczące `submissionUniqueId`.

## ITI-18 - search P1 registry

```
POST /repository/{id}/search
```

Gdzie:
- `id` - Identyfikator repozytorium XDS.b w Medfile API (potrzebne, aby określić "Kto szuka")

Treść żądania:

```json
{
  "organization": {},
  "practitioner": {},
  "patient": {},
  "queryType": "findDocuments",
  "uuidReturn": true,
  ...query parameters
}
```

Gdzie:
- `organization`, practitioner oraz patient zgodnie z rejestracją plików w indeksie EDM P1.
- `queryType` - typ kwerendy np.: findDocuments, getDocuments, getAll
- `uuidReturn` - opcjonalne, jeżeli `true` to zwrócone zostaną tylko identyfikatory znalezionych dokumentów
- `query parameters` - treść kwerend, opisano poniżej
-
### Kwerenda do szukania dokumentu

Znajdź dokumenty (XDSDocumentEntry objects) w indeksie EDM P1.

Parametry:
```json
{
  "classCode": "06.10",
  "typeCode": "10006-5",
  "medicalFieldCode": "05",
  "creationTimeFrom": "2022-01-01 10:00:00",
  "creationTimeTo": "2022-08-02 10:00:00",
  "serviceStartTimeFrom": "2022-01-01 10:00:00",
  "serviceStartTimeTo": "2022-08-02 10:00:00",
  "serviceStopTimeFrom": "2022-01-01 10:00:00",
  "serviceStopTimeTo": "2022-08-02 10:00:00",
  "specialityCode": "1008",
  "eventCode": "00.03",
  "confidentialityCode": "N",
  "authorPerson": "Szymon",
  "format": "urn:extPL:pl-cda",
  "documentEntryStatus": "approved",
  "documentEntryType": "stable"
}
```

Gdzie:
- classCode - document type code from 'KLAS_DOK_P1' dictionary
- typeCode - document type code from 'LOINC' dictionary
- medicalFieldCode - medical field code from 'Dziedzina medyczna' dictionary
- creationTimeFrom - filter documents by creation time
- creationTimeTo - filter documents by creation time
- serviceStartTimeFrom - filter documents by medical procedure start time
- serviceStartTimeTo - filter documents by medical procedure start time
- serviceStopTimeFrom - filter documents by medical procedure stop time
- serviceStopTimeTo - filter documents by medical procedure stop time
- specialityCode - organization speciality from 'Specjalność komórki organizacyjnej' dictionary
- eventCode - medical procedure code from 'ICD-9 PL' dictionary
- confidentialityCode - document confidentiality code, one of the following values:
  ```
  N - normal
  R - restricted
  V - very restricted
  ```
- authorPerson -
- format - document format from "Kody formatów P1" dictionary
- documentEntryStatus - document entry status, one of the following values:
  ```
  approved - default value
  deprecated
  ```
- documentEntryType - document entry status, one of the following values:
  ```
  stable
  onDemand
  ```

Wszystkie parametry są opcjonalne.

### Kwerenda do pobierania dokumentów

Pobieranie dokumentów (kolekcji XDSDocumentEntry).
Dzięki tej metodzie możemy pobrać dokumenty których UUID lub uniqueId znamy.

Treść żądania:

```json
{
  "documentEntryEntryUUID": [
    "urn:uuid:802916a6-7adc-43cc-b33e-d1d1fe13696b"
  ],
  "documentEntryUniqueId": [
    {
      "type": "2.16.840.1.113883.3.4424.2.7.952",
      "value": "bd914a4a-36a5-457e-bc12-fcbbed3f9cae"
    }
  ]
}
```

Gdzie:
- documentEntryEntryUUID - array containig uuids of document entries to retrieve
- documentEntryUniqueId - array containig document unique identifiers of documents to retrieve
    - type - oid
    - value - document id

Przynajmniej jeden z parametrów jest wymagany: documentEntryEntryUUID lub documentEntryUniqueId.

### Kwerenda do pobierania wszystkich informacji o pacjencie
Get all registry content for a patient

parameters:
```json
{
  "documentEntryStatus": "approved",
  "submissionSetStatus": "approved",
  "folderStatus": "approved",
  "format": "urn:extPL:pl-cda",
  "confidentialityCode": "N",
  "documentEntryType": "stable"
}
```

Gdzie:
- documentEntryStatus, format, confidentialityCode, documentEntryType - description in [find documents query](#findDocumentsQuery) section,
- submissionSetStatus - submission set status, values as in documentEntryStatus
- folderStatus - folder status, values as in documentEntryStatus

## ITI-43 - pobieranie dokumentów z innego repozytorium

Transakcja ITI-43 realizowana jest za pośrednictwem Medfile API (services.medfile.pl).

Do usługi Medfile API autoryzujemy się tokenem JWT wygenerowanym zgodnie z zasadami opisanymi w sekcji "Autoryzacja w Medfile API".

```http request
GET /xds/{repositoryId}/{documentIdRoot}/{documentIdExtension}/{patientIdRoot}/{patientIdExtension}/{purpose}
```

Gdzie:
- repositoryId - id repozytorium, w którym przechowywany jest dokument
- documentIdRoot - część root identyfikatora dokumentu
- documentIdExtension - część extension identyfikatora dokumentu
- patientIdRoot - część root identyfikatora pacjenta, którego dotyczy dokument
- patientIdExtension - część extension identyfikatora pacjenta, którego dotyczy dokument
- purpose - powód wykorzystania danych, kod zgodny ze słownikiem https://www.hl7.org/fhir/v3/PurposeOfUse/vs.html lub kod CONTT (dostęp w trybie kontynuacji leczenia)

W odpowiedzi otrzymają Państwo zawartość żądanego dokumentu lub komunikat błędu jeżeli dostęp nie zostanie udzielony przez repozytorium.