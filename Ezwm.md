# Usługa eZLA

## Konfiguracja usługi

> __Uwaga__
>
> Na środowisku testowym należy użyć konta utworzonego na koncie produkcyjnym.


### Sprawdź, czy ustawienia usługi są prawidłowe

```http request
GET /ezwm/checklogin
Authorization: Bearer {{ token }}
```

### Wyślij dokument eZWM

```http request
PUT /ezwm/document
Authorization: Bearer {{ token }}
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<zlecenie xmlns="https://ezwm.nfz.gov.pl/xml/e-zpo/dok-zlecenia/v2.1" nr-zlecenia-nfz="T5-PC00013R-00000052">
     <miejsce-wystawienia-zlecenia nazwa="Test" regon="241544443">
         <adres kod-poczt="12-345" miejscowosc="Abc" ulica="ul ewfdsf sd" nr-domu="234" nr-lokalu="1" kod-kraju="PL"/>
     </miejsce-wystawienia-zlecenia>
    <pacjent typ-id-osoby="P" id-osoby="88072198883" nazwisko="Senior" imie="Sylwester"/>
    <okreslenie-wyrob-med>
        <okulary>
            <soczewka rodzaj="B" oko="L" sfera="0.5" cylinder="1.0" pryzma="0.0"/>
        </okulary>
    </okreslenie-wyrob-med>
    <dane-dt-wystawienia data-wystawienia="{{ today }}">
        <osoba-uprawniona typ-pers-zlec="11" id-zlec="2999999" nazwisko="MEDFILE" imie="KATARZYNA"/>
    </dane-dt-wystawienia>
</zlecenie>
```

### Aktualizuj dokument eZWM - nowa wersja

```http request
PUT /ezwm/document/{{ documentUuid }}/{{ versionNumber }}
Authorization: Bearer {{ token }}
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<zlecenie xmlns="https://ezwm.nfz.gov.pl/xml/e-zpo/dok-zlecenia/v2.1" nr-zlecenia-nfz="T5-PC00013R-00000052">
     <miejsce-wystawienia-zlecenia nazwa="Test" regon="241544443">
         <adres kod-poczt="12-345" miejscowosc="Abc" ulica="ul ewfdsf sd" nr-domu="234" nr-lokalu="1" kod-kraju="PL"/>
     </miejsce-wystawienia-zlecenia>
    <pacjent typ-id-osoby="P" id-osoby="88072198883" nazwisko="Senior" imie="Sylwester"/>
    <okreslenie-wyrob-med>
        <okulary>
            <soczewka rodzaj="B" oko="L" sfera="0.5" cylinder="1.0" pryzma="0.0"/>
        </okulary>
    </okreslenie-wyrob-med>
    <dane-dt-wystawienia data-wystawienia="{{ today }}">
        <osoba-uprawniona typ-pers-zlec="11" id-zlec="2999999" nazwisko="MEDFILE" imie="KATARZYNA"/>
    </dane-dt-wystawienia>
</zlecenie>
```

### Pobierz status dokumentu

Używając numeru PESEL:
```http request
GET /ezwm/documentstatus?zlec=T5-PC00013R-00000014&pesel=88072198883
Authorization: Bearer {{ token }}
```

Używając numeru technicznego dokumentu:
```http request
GET /ezwm/documentstatus?zlec=T5-PC00013R-00000015&idTech=34519800000000000100007839
Authorization: Bearer {{ token }}
```

### Pobierz dokument zlecenia

Używając kodu zlecenia
```http request
GET /ezwm/document/zlecenia?zlec=T5-PC00013R-00000015&kod=88072198883
Authorization: Bearer {{ token }}
```

Używając numeru technicznego dokumentu:
```http request
GET /ezwm/document/zlecenia?zlec=T5-PC00013R-00000015&idTech=34519800000000000100007839
Authorization: Bearer {{ token }}
```


### Pobierz wynik weryfikacji

Używając kodu zlecenia
```http request
GET /ezwm/document/wynik-weryfikacji?zlec=T5-PC00013R-00000015&kod=88072198883
Authorization: Bearer {{ token }}
```

Używając numeru technicznego dokumentu:
```http request
GET /ezwm/document/wynik-weryfikacji?zlec=T5-PC00013R-00000015&idTech=34519800000000000100007839
Authorization: Bearer {{ token }}
```


### Pobierz podsumowanie w pliku PDF

Używając kodu zlecenia
```http request
GET /ezwm/document/info-zlecenia-pdf?zlec=T5-PC00013R-00000015&kod=88072198883
Authorization: Bearer {{ token }}
```

Używając numeru technicznego dokumentu:
```http request
GET /ezwm/document/info-zlecenia-pdf?zlec=T5-PC00013R-00000015&idTech=34519800000000000100007839
Authorization: Bearer {{ token }}
```

### Pobierz wydruk PDF

Używając kodu zlecenia
```http request
GET /ezwm/document/zlecenia-pdf?zlec=T5-PC00013R-00000015&kod=88072198883
Authorization: Bearer {{ token }}
```

Używając numeru technicznego dokumentu:
```http request
GET /ezwm/document/zlecenia-pdf?zlec=T5-PC00013R-00000015&idTech=34519800000000000100007839
Authorization: Bearer {{ token }}
```

### Pobierz wydruk PDF bez weryfikacji

Używając kodu zlecenia
```http request
GET /ezwm/document/zlecenia-bez-weryf-pdf?zlec=T5-PC00013R-00000015&kod=88072198883
Authorization: Bearer {{ token }}
```

Używając numeru technicznego dokumentu:
```http request
GET /ezwm/document/zlecenia-bez-weryf-pdf?zlec=T5-PC00013R-00000015&idTech=34519800000000000100007839
Authorization: Bearer {{ token }}
```

### Anuluj dokument zlecenia

```http request
PUT /ezwm/document
Authorization: Bearer {{ token }}
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<zlecenie xmlns="https://ezwm.nfz.gov.pl/xml/e-zpo/dok-anulowania-zlec/v2.1" nr-zlecenia-nfz="T5-PC00013R-00000052">
    <dane-anulowania data-anulowania="{{ today }}" tryb-anulowania="K" przyczyna="Bo tak.">
        <podmiot-anulujacy-zlecenia nazwa="Ogólna izba przyjęć" regon="06140990200000">
            <osoba-uprawniona typ-pers-zlec="11" id-zlec="9905698" nazwisko="Artur09" imie="Praktyczny"/>
        </podmiot-anulujacy-zlecenia>
    </dane-anulowania>
</zlecenie>
```