# Usługa: eZWM

## Konfiguracja usługi

> __Uwaga__
>
> Na środowisku testowym należy użyć konta utworzonego na koncie produkcyjnym. Nie ma możliwości utworzenia kont pracujących wyłącznie na środowisku produkcyjnym

### Informacje ogólne

Do wystawienia zlecenia na środki pomocnicze potrzebny jest dostęp do słowników:
 - wyrobów medycznych i środków pomocniczych, a wraz z nim - dodatkowych słowników:
   - kryterium przyznania wyrobu
   - kryterium skrócenia okresu użytkownania
 - podstawy ubezpieczenia (dane pacjenta) - w przypadku potwierdzenia ubezpieczenia w inny sposób niż eWUŚ
 - uprawnień dodatkowych pacjenta
 - instytucji właściwych (dla pacjentów z unii europejskiej)
 - słownika ICD-10
Utworzenie XMLa ze zleceniem lezy po stronie systemu dziedzinowego. API udostępnia potrzebne słowniki oraz endpointy służące do obsługi wniosku w NFZ.

Sposób tworzenia wniosku został opisany w dokumentacji znajdującej się na stronie NFZ pod adresem:
https://www.nfz.gov.pl/dla-swiadczeniodawcy/sprawozdawczosc-elektroniczna/interfejsy-integracyjne/ezwm/

### Nagłówki HTTP
Do komunikacji z NFZ w zakresie zleceń na wyroby medyczne wystarczy w tokenie umieścić specjalistę (analogicznie jak w usłudze eWUŚ) - practitioner).
- Authorization: Bearer {JWT TOKEN}

### Sprawdź, czy ustawienia usługi są prawidłowe

Wywołanie endpoint bez dodatkowych parametrów. Zwracana jest informacja o poprawnym przekazaniu parametrów do obsługi ezwm.
```http request
GET /ezwm/checklogin
```

### Wyślij dokument eZWM
Endpoint do przekazania NFZ dokumentu zlecenia na wyroby medyczne:
```http request
PUT /ezwm/document
```
Należy przekazać dokument zgodny ze specyfikacją:  ```https://ezwm.nfz.gov.pl/xml/e-zpo/dok-zlecenia/v2.1```

Przykładowy plik zlecenia na wyroby medyczne *(soczewki)*:
```
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
Endpoint umożliwiający aktualizację zlecenia w systemie NFZ:
```http request
PUT /ezwm/document/{{ documentUuid }}/{{ versionNumber }}
```
Przy poprawie należy przekazać nowy - poprawiony XML zlecenia.
```
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

```http request
GET /ezwm/documentstatus?zlec=T5-PC00013R-00000014&pesel=88072198883
```

NFZ umożliwia pobranie aktualnego statusu dokumentu. Można tego dokonać przekazujac numer dokumentu wraz z kodem dostępu (numerem PESEL lub datą urodzenia) lub numerem technicznym dokumentu.

Pobranie statusu używając numeru PESEL:
```http request
GET /ezwm/documentstatus?zlec=T5-PC00013R-00000014&pesel=88072198883
Authorization: Bearer {{ token }}
```

Pobranie statusu za pomocą numeru technicznego dokumentu:
```http request
GET /ezwm/documentstatus?zlec=T5-PC00013R-00000015&idTech=34519800000000000100007839
Authorization: Bearer {{ token }}
```
Dostępne statusy (dla wystawiającego zlecenie):  
**R – Zarejestrowane** – Dokument zlecenia został poprawnie przetworzony przez system  
**W – W trakcie weryfikacji** – Zlecona została weryfikacja zlecenia (zlecenie oczekuje na weryfikację, bądź jest w trakcie weryfikacji)  
**P – Zweryfikowane pozytywnie** - Zlecenie uzyskało pozytywną decyzję systemu NFZ - możliwa jest realizacja zlecenia   
**N – Zweryfikowane negatywnie** - Zlecenie uzyskało negatywną decyzję systemu NFZ - nie jest możliwa realizacja zlecenia  
**A – Anulowane** – dla zlecenia zarejestrowano dokument anulowania (w przypadku anulowania zlecenia na zaopatrzenie comiesięczne, nie może być zrealizowany żaden okres, którego zlecenie dotyczy)  
**Z – Zlecenia ma zarejestrowaną i potwierdzoną realizację** (w przypadku zaopatrzenia comiesięcznego wszystkie okresy zlecenia są albo zrealizowane albo anulowano ich realizację, przy czym przy anulowaniu okresów realizacji, chociaż jeden okres zlecenia musi być zrealizowany)  

### Pobierz dokument zlecenia

NFZ umożliwia pobranie różnych dokumentów dla osoby uprawnionej do wystawienia zlecenia. Do pobrania każdego z dokumnetów należy przekazać id-tech-dokumentu-nfz. Dokumenty możliwe do pobrania:  
**dok-wynikweryfikacji** - Dokument z wynikiem weryfikacji zlecenia dla etapu Z (wynik przetwarzania dokumentu zlecenia)  
**dok-zlecenia-pdf** - Dokument zlecenia w formacie pdf – część I i II zlecenia (jeśli zlecenie zostało zweryfikowane przez system NFZ i jest dostępny wynik weryfikacji)  
**dok-zlecenia** - Dokument zlecenia  
**dok-info-zlecenia-pdf** - Druk informacyjny zlecenia elektronicznego w formacie pdf   

Każdy z w/w elementów można pobrać używając endpoint:
```http request
GET /ezwm/document/zlecenia?zlec=T5-PC00013R-00000015&idTech=34519800000000000100007839
```
Do endpoint należy przekazać odpowiedni typ dokumentu: 
- zlecenia,
- wynik-weryfikacji,
- info-zlecenia-pdf,
- zlecenia-pdf,
- zlecenia-bez-weryf-pdf

#### Pobierz wynik weryfikacji

```http request
GET /ezwm/document/wynik-weryfikacji?zlec=T5-PC00013R-00000015&idTech=34519800000000000100007839
```

#### Pobierz podsumowanie w pliku PDF

```http request
GET /ezwm/document/info-zlecenia-pdf?zlec=T5-PC00013R-00000015&idTech=34519800000000000100007839
```

#### Pobierz wydruk PDF

```http request
GET /ezwm/document/zlecenia-pdf?zlec=T5-PC00013R-00000015&idTech=34519800000000000100007839
```

#### Pobierz wydruk PDF bez weryfikacji

```http request
GET /ezwm/document/zlecenia-bez-weryf-pdf?zlec=T5-PC00013R-00000015&idTech=34519800000000000100007839
```

### Anuluj dokument zlecenia

Anulowanie dokumentu odbywa się za pomocą  endpoint, za pomocą któego przekazywane są dokumenty zlecenia. W tym przypadku należy przekazać dokument anulujący zlecenie, zgodny ze specyfikacją: ```https://ezwm.nfz.gov.pl/xml/e-zpo/dok-anulowania-zlec/v2.1```

```http request
PUT /ezwm/document
```

Przykład dokumentu:
```
<?xml version="1.0" encoding="UTF-8"?>
<zlecenie xmlns="https://ezwm.nfz.gov.pl/xml/e-zpo/dok-anulowania-zlec/v2.1" nr-zlecenia-nfz="T5-PC00013R-00000052">
    <dane-anulowania data-anulowania="{{ today }}" tryb-anulowania="K" przyczyna="Bo tak.">
        <podmiot-anulujacy-zlecenia nazwa="Ogólna izba przyjęć" regon="06140990200000">
            <osoba-uprawniona typ-pers-zlec="11" id-zlec="9905698" nazwisko="Artur09" imie="Praktyczny"/>
        </podmiot-anulujacy-zlecenia>
    </dane-anulowania>
</zlecenie>
```
