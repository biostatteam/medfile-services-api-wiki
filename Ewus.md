# Usługa: eWUŚ

## Sprawdzanie statusu pacjenta

```
GET /ewus/check/{pesel}
```

### Nagłówki HTTP

Do weryfikacji ubezpieczenia pacjenta wystarczy w tokenie umieścić specjalistę ze skonfigurowaną usługą eWUŚ (*practitioner*).

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

## Zmiana hasła użytkownika w eWUŚ

TODO

## Kody odpowiedzi

1. `200` - Poprawna odpowiedź
2. `403` - Brak uprawnień do wykonania polecenia
3. `404` - Brak danych dla wybranego PESEL
