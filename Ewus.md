# Usługa: eWUŚ

Usługa eWUŚ (Elektroniczna Weryfikacja Uprawnień Świadczeniobiorców) została utworzona przez Narodowy Fundusz Zdrowia (NFZ). Umożliwia elektroniczną weryfikację uprawnienia pacjenta do bezpłatnych świadczeń zdrowotnych finansowanych przez NFZ.  
Dzięki niej można w kilka sekund wiążąco potwierdzić status ubezpieczenia pacjenta.  

## Sprawdzanie statusu pacjenta

```http request
GET /ewus/check/{pesel}
```

Aby sprawdzić status pacjenta, należy wcześniej wywołać endpoint `/ewus/login/{totp}` służący do zalogowania (patrz: [Logowanie użytkownika do usługi](#logowanie)).  
Po pomyślnym zalogowaniu endpoint korzysta z aktywnej sesji użytkownika.  
W przypadku wygaśnięcia sesji zostanie zwrócony [błąd braku sesji](#brak-sesji) – należy wówczas ponownie zalogować użytkownika do usługi eWUŚ.  

### Nagłówki HTTP

Do weryfikacji ubezpieczenia pacjenta należy przesłać token dostępu zawierający specjalistę powiązanego z usługą eWUŚ (*practitioner*).  

- Authorization: Bearer {JWT TOKEN}

### Odpowiedź pozytywna

```json
{
  "message": "",
  "body": {
    "operation_date": "2020-10-20T20:26:05+0200",
    "operation_id": "L1712M01200000001",
    "system_name": "eWUS",
    "system_version": "test",
    "status": 1,
    "operator_id": "3",
    "operator_domain": "01",
    "operator_external_id": "TEST3",
    "expiration_date": "2020-10-20T23:59:59+0200",
    "insurance_status": 1,
    "prescription_symbol": "DN",
    "patient_pesel": "19323197988",
    "patient_first_name": "ImięTAK",
    "patient_last_name": "NazwiskoTAK",
    "patient_notes": [
      "Pacjent objęty kwarantanną do dnia 03-11-2020"
    ]
  },
  "error": [],
  "raw": "<?xml version='1.0' encoding='UTF-8'?><soapenv:Envelope xmlns:soapenv=\"http:\/\/schemas.xmlsoap.org\/soap\/envelope\/\"><soapenv:Body><ns3:executeServiceReturn xmlns:ns3=\"http:\/\/xml.kamsoft.pl\/ws\/broker\" xmlns:xsi=\"http:\/\/www.w3.org\/2001\/XMLSchema-instance\" xsi:type=\"ns3:ServiceResponse\"><location xmlns=\"http:\/\/xml.kamsoft.pl\/ws\/common\"><namespace>nfz.gov.pl\/ws\/broker\/cwu<\/namespace><localname>checkCWU<\/localname><version>5.0<\/version><\/location><ns3:date>2020-10-20T20:26:05.363+02:00<\/ns3:date><ns3:payload><ns3:textload><ns2:status_cwu_odp xmlns:ns2=\"https:\/\/ewus.nfz.gov.pl\/ws\/broker\/ewus\/status_cwu\/v5\" data_czas_operacji=\"2020-10-20T20:26:05.357+02:00\" id_operacji=\"L1712M01200000001\"><ns2:status_cwu>1<\/ns2:status_cwu><ns2:numer_pesel>19323197988<\/ns2:numer_pesel><ns2:system_nfz nazwa=\"eWUS\" wersja=\"test\" \/><ns2:swiad><ns2:id_swiad>TEST3<\/ns2:id_swiad><ns2:id_ow>01<\/ns2:id_ow><ns2:id_operatora>3<\/ns2:id_operatora><\/ns2:swiad><ns2:pacjent><ns2:data_waznosci_potwierdzenia>2020-10-20+02:00<\/ns2:data_waznosci_potwierdzenia><ns2:status_ubezp ozn_rec=\"DN\">1<\/ns2:status_ubezp><ns2:imie>ImięTAK<\/ns2:imie><ns2:nazwisko>NazwiskoTAK<\/ns2:nazwisko><ns2:informacje_dodatkowe><ns2:informacja kod=\"KWARANTANNA-COVID19\" poziom=\"O\" wartosc=\"Pacjent objęty kwarantanną do dnia 03-11-2020\" \/><\/ns2:informacje_dodatkowe><\/ns2:pacjent><Signature xmlns=\"http:\/\/www.w3.org\/2000\/09\/xmldsig#\"><SignedInfo><CanonicalizationMethod Algorithm=\"http:\/\/www.w3.org\/TR\/2001\/REC-xml-c14n-20010315\" \/><SignatureMethod Algorithm=\"http:\/\/www.w3.org\/2000\/09\/xmldsig#rsa-sha1\" \/><Reference URI=\"\"><Transforms><Transform Algorithm=\"http:\/\/www.w3.org\/2000\/09\/xmldsig#enveloped-signature\" \/><\/Transforms><DigestMethod Algorithm=\"http:\/\/www.w3.org\/2000\/09\/xmldsig#sha1\" \/><DigestValue>\/LRG3hEBgCjz\/VKJUD42STW4Ppc=<\/DigestValue><\/Reference><\/SignedInfo><SignatureValue>LoXGFPtinfK6KfA9eDzN3\/qXv3KxKKJoXRTRkFtuwIiC5Z154CdrT3R3IusE\/ZoVuXbXuyvVZ5qI\nXs5G94bB0lZE+a9Fgec5bplWhnOsVg\/qnJaCG2EQfuahP8vxmtuKes2O1o8cYC1oBpyMTq\/qI1Wa\nWCtDarXPapLulyJ7N+c=<\/SignatureValue><\/Signature><\/ns2:status_cwu_odp><\/ns3:textload><\/ns3:payload><\/ns3:executeServiceReturn><\/soapenv:Body><\/soapenv:Envelope>"
}
```

<a id="brak-sesji"></a>

### Brak aktywnej sesji użytkownika
Poniższa odpowiedź jest zwracana przez endpointy eWUŚ (poza logowaniem), gdy użytkownik nie jest zalogowany do usługi lub jego sesja wygasła.  
Należy wówczas ponownie zalogować użytkownika z użyciem nowego kodu TOTP (patrz: [Logowanie użytkownika do usługi](#logowanie)).

```json
{
  "message": "",
  "body": {},
  "error": [
    {
      "text": "Brak sesji operatora. Wymagane ponowne zalogowanie.",
      "code": 401
    }
  ],
  "raw": ""
}
```

## Zmiana hasła użytkownika w eWUŚ

Usługa eWUŚ umożliwia zmianę hasła użytkownika za pomocą następującego endpointu:  

```http request
POST /ewus/change_password
```

Zmiana hasła wymaga aktywnej sesji użytkownika - przed wywołaniem endpointu należy zalogować użytkownika do usługi (patrz: [Logowanie użytkownika do usługi](#logowanie)).  
W przypadku braku sesji zwracany jest [błąd braku sesji](#brak-sesji).  

W tokenie należy przekazać specjalistę powiązanego z usługą eWUŚ (*practitioner*), a w treści żądania dane w formacie JSON:  
```json
{
  "old": "starehasło",
  "new": "nowehasło"
}
```

<a id="ewus-mfa"></a>

# eWUŚ MFA
Od 17 listopada 2025 roku logowanie do eWUŚ jest możliwe wyłącznie z wykorzystaniem uwierzytelnienia wieloskładnikowego (MFA).  
Jest to drugi etap wdrożenia zabezpieczeń przez NFZ.  
Wcześniej MFA było wymagane jedynie dla serwisów dostępnych przez przeglądarkę internetową (np. SZOI/Portal Świadczeniodawcy).  

Do logowania do eWUŚ wykorzystywane są te same tokeny, które użytkownicy generują przy logowaniu do serwisów NFZ obsługujących MFA.  

eWUŚ i eZWM to w NFZ odrębne systemy, dlatego dla każdej z usług tworzona jest osobna sesja i wymagane jest osobne logowanie.  
Sesje są przechowywane po stronie API - przez cały okres ważności sesji (14 minut) użytkownik nie musi ponawiać logowania.  

> Uwaga:  
> Usługa testowa NFZ okazjonalnie „gubi” dane sesyjne użytkowników. Wynika to z braku współdzielenia danych autoryzacyjnych między instancjami systemu — w konsekwencji sesje mogą wygasać przed upływem 14 minut.  

## Zalecany przebieg pracy

1. Zalogowanie użytkownika do usługi z użyciem kodu TOTP: `GET /ewus/login/{totp}`.
2. Wywoływanie operacji eWUŚ (sprawdzenie statusu pacjenta, zmiana hasła) w ramach aktywnej sesji.
3. Po otrzymaniu [błędu braku sesji](#brak-sesji) (sesja wygasła lub została utracona) - ponowne zalogowanie z nowym kodem TOTP i powtórzenie operacji.
4. Opcjonalnie - wylogowanie: `GET /ewus/logout` (nie jest wymagane, sesja wygasa automatycznie).

Kod TOTP może zostać podany przez użytkownika albo wygenerowany automatycznie przez system dziedzinowy (własny generator kodów TOTP). Automatyczne generowanie pozwala na ponowne logowanie bez udziału użytkownika.

<a id="logowanie"></a>

## Logowanie użytkownika do usługi

```http request
GET /ewus/login/{totp}
```

W przypadku aktywnego MFA należy wykonać jawne logowanie, przekazując token autoryzacyjny (kod TOTP).  
Po zalogowaniu tworzona jest sesja ważna przez 14 minut. Po jej wygaśnięciu należy ponowić logowanie.  

### Odpowiedź pozytywna
```json
{
  "message": "",
  "body": {
    "session_id": "2BB91F34D3DF8193553F800671D128EC",
    "token": "BSjm9A7_8rUuAu0yRfmaoH",
    "login_code": "000",
    "login_message": "Użytkownik został prawidłowo zalogowany."
  },
  "error": [],
  "raw": ""
}
```

### Odpowiedź negatywna - błąd logowania użytkownika
Zwracana, gdy podane parametry logowania (w tym kod TOTP) są nieprawidłowe.

```json
{
  "message": "",
  "body": {},
  "error": [
    {
      "text": "Brak identyfikacji operatora. Podane parametry logowania są nieprawidłowe.",
      "code": 401
    }
  ],
  "raw": ""
}
```

## Wylogowanie z usługi

```http request
GET /ewus/logout
```

Endpoint umożliwia wylogowanie użytkownika z usługi eWUŚ.  
Usuwa dane sesyjne i kończy połączenie z NFZ.  
Wylogowanie nie jest wymagane (sesja wygasa automatycznie po 14 minutach), ale może być potrzebne np. przy zmianie danych logowania (inny użytkownik).  
Endpoint nie wymaga żadnych parametrów.  

### Odpowiedź pozytywna
```json
{
  "message": "",
  "body": {
    "request": {
      "session_id": "D47547163779209D4F168EFE5F3F4CF9",
      "token": "BS_pYhL6yuvsTUs8U2gkQG"
    },
    "logout_message": "Wylogowany"
  },
  "error": [],
  "raw": "<?xml version='1.0' encoding='UTF-8'?><soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\"><soapenv:Body><ns1:logoutReturn xmlns:ns1=\"http://xml.kamsoft.pl/ws/kaas/login_types\">Wylogowany</ns1:logoutReturn></soapenv:Body></soapenv:Envelope>"
}
```
