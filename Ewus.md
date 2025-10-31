# Usługa: eWUŚ

Usługa eWUŚ (Elektroniczna Weryfikacja Uprawnień Świadczeniobiorców) została utworzona przez Narodowy Fundusz Zdrowia (NFZ). Umożliwia elektroniczą weryfikację uprawnienia pacjenta do bezpłatnych świadczeń zdrowotnych finansowanych przez NFZ.   
Dzięki niemu można w kilka sekund wiążąco potwierdzić status ubezpieczenia pacjenta.  

## Sprawdzanie statusu pacjenta

Aby sprawdzić status pacjenta, należy wywołać poniższy endpoint.  
Endpoint automatycznie loguje użytkownika do usługi eWUŚ (bez użycia MFA).  
Jeśli użytkownik wymaga dodatkowej autoryzacji, przed sprawdzeniem statusu pacjenta należy wykonać logowanie z użyciem tokenu autoryzacyjnego.  

Po pomyślnym logowaniu endpoint korzysta z aktywnej sesji użytkownika.  
W przypadku wygaśnięcia sesji zostanie zwrócony błąd – należy wówczas ponownie zalogować użytkownika do usługi eWUŚ.  

```http request
GET /ewus/check/{pesel}
```

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

### Błąd logowania użytkownika
W przypadku poniższej odpowiedzi należy ponownie zalogować użytkownika, ponieważ:  
- podane parametry logowania są nieprawidłowe, lub  
- wymagany jest token autoryzacyjny (MFA).  

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

## Zmiana hasła użytkownika w eWUŚ

Usługa eWUŚ umożliwia zmianę hasła użytkownika za pomocą następującego endpointu:  

```http request
POST /ewus/change_password
```

W tokenie należy przekazać specjalistę powiązanego z usługą eWUŚ (practitioner), a w treści żądania dane w formacie JSON:  
```json
{
  "old": "starehasło",
  "new": "nowehasło"
}
```

# eWUŚ MFA
Od 17 listopada 2025 roku logowanie do eWUŚ będzie możliwe wyłącznie z wykorzystaniem uwierzytelnienia wieloskładnikowego (MFA).  
Jest to drugi etap wdrożenia bezpieczeństwa przez NFZ.  
Dotychczas MFA było wymagane jedynie dla serwisów dostępnych przez przeglądarkę internetową (np. SZOI/Portal Świadczeniodawcy).  

Do logowania do eWUŚ będą wykorzystywane te same tokeny, które użytkownicy generują przy logowaniu do serwisów NFZ obsługujących MFA.  

> Uwaga:   
> Usługa testowa NFZ okazjonalnie „gubi” dane sesyjne użytkowników. Wynika to z braku współdzielenia danych autoryzacyjnych między instancjami systemu — w konsekwencji sesje mogą wygasać szybciej niż po 15 minutach.  

## Logowanie użytkownika do usługi

W przypadku aktywnego MFA należy wykonać jawne logowanie, przekazując token autoryzacyjny.  
Po zalogowaniu tworzona jest sesja ważna przez 14 minut. Po jej wygaśnięciu należy ponowić logowanie.  

```http request
GET /ewus/login/{totp}
```

### Odpowiedź pozytywna
```json
{
  "message": "",
  "body": {
    "request": {
      "password": "qwerty!@#",
      "mfa_totp": " 111590",
      "domain": "01",
      "login": "TEST_MFA",
      "operator_type": "SWD",
      "operator_id": "123456789"
    },
    "session_id": "2BB91F34D3DF8193553F800671D128EC",
    "token": "BSjm9A7_8rUuAu0yRfmaoH",
    "login_code": "000",
    "login_message": "Użytkownik został prawidłowo zalogowany."
  },
  "error": [],
  "raw": "<?xml version='1.0' encoding='UTF-8'?><soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\"><soapenv:Header><ns1:session xmlns:ns1=\"http://xml.kamsoft.pl/ws/common\" id=\"2BB91F34D3DF8193553F800671D128EC\" /><ns1:authToken xmlns:ns1=\"http://xml.kamsoft.pl/ws/common\" id=\"BSjm9A7_8rUuAu0yRfmaoH\" /></soapenv:Header><soapenv:Body><ns1:loginReturn xmlns:ns1=\"http://xml.kamsoft.pl/ws/kaas/login_types\">[000] U&amp;#380;ytkownik zosta&amp;#322; prawid&amp;#322;owo zalogowany.</ns1:loginReturn></soapenv:Body></soapenv:Envelope>"
}
```

### Odpowiedź negatywna - błąd logowania użytkownika

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

Endpoint umożliwia wylogowanie użytkownika z usługi eWUŚ.  
Usuwa dane sesyjne i kończy połączenie z NFZ.  
Wylogowanie może być wymagane np. przy zmianie danych logowania (inny użytkownik).  
Endpoint nie wymaga żadnych parametrów.  

```http request
GET /ewus/logout
```

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
