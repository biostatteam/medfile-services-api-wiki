# Usługa: eZWM

## Konfiguracja usługi

> __Uwaga__
>
> Na środowisku testowym można użyć konta utworzonego na koncie produkcyjnym.
> Możliwe jest też założenie w NFZ pełnego konta testowego.

### Informacje ogólne

Do wystawienia zlecenia na środki pomocnicze potrzebny jest dostęp do słowników:
 - wyrobów medycznych i środków pomocniczych, zawierającego oprócz kodów wyrobów - również słowników:
   - kryterium przyznania wyrobu
   - kryterium skrócenia okresu użytkowania
 - podstawy ubezpieczenia (dane pacjenta) - w przypadku potwierdzenia ubezpieczenia w inny sposób niż eWUŚ
 - uprawnień dodatkowych pacjenta
 - instytucji właściwych (dla pacjentów z Unii Europejskiej)
 - słownika ICD-10

Trasy obsługujące w/w słowniki zostały opisane na podstronach:
 - [Słowniki](Dictionaries.md)
 - [Słownik EZWM](EzwmDictionaries.md)

Utworzenie XML-a ze zleceniem EZWM leży po stronie systemu dziedzinowego. API udostępnia potrzebne słowniki oraz endpointy służące do obsługi wniosku w NFZ.

Sposób tworzenia wniosku został opisany w dokumentacji znajdującej się na stronie NFZ pod adresem:
https://www.nfz.gov.pl/dla-swiadczeniodawcy/sprawozdawczosc-elektroniczna/interfejsy-integracyjne/ezwm/

### Nagłówki HTTP
Do komunikacji z NFZ w zakresie zleceń na wyroby medyczne wystarczy w tokenie umieścić specjalistę (analogicznie jak w usłudze eWUŚ - *practitioner*).
- Authorization: Bearer {JWT TOKEN}

W przypadku aktywnego MFA przed wywołaniem pozostałych endpointów należy zalogować użytkownika do usługi eZWM (patrz: [eZWM MFA](#ezwm-mfa)). Kolejne wywołania korzystają z aktywnej sesji użytkownika.

### Sprawdź, czy ustawienia usługi są prawidłowe
```http request
GET /ezwm/checklogin
```
> __Uwaga__
>
> Endpoint jest przestarzały (*deprecated*) i zostanie usunięty w kolejnych wersjach API.
> Do logowania użytkownika należy używać endpointu `GET /ezwm/login/{totp}` (patrz: [eZWM MFA](#ezwm-mfa)).

Wywołanie endpointu bez dodatkowych parametrów. Zwracana jest informacja o poprawnym przekazaniu parametrów do obsługi eZWM oraz o stanie sesji użytkownika (czy użytkownik jest zalogowany do usługi, a sesja jest aktywna).  
W przypadku braku aktywnej sesji zwracany jest [błąd braku sesji](#brak-aktywnej-sesji-użytkownika) - należy wówczas zalogować użytkownika do usługi eZWM.


### Brak aktywnej sesji użytkownika
Poniższa odpowiedź jest zwracana przez endpointy eZWM (poza logowaniem), gdy użytkownik nie jest zalogowany do usługi lub jego sesja wygasła.  
Należy wówczas ponownie zalogować użytkownika z użyciem nowego kodu TOTP (patrz: [Logowanie użytkownika do usługi](#logowanie-użytkownika-do-usługi)).

```json
{
  "message": "",
  "error": [
    {
      "code": 401,
      "text": "Brak sesji operatora. Wymagane ponowne zalogowanie."
    }
  ],
  "raw": "",
  "body": ""
}
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

### Aktualizuj dokument eZWM - przekazanie nowej wersji zlecenia
Endpoint umożliwiający aktualizację zlecenia w systemie NFZ:
```http request
PUT /ezwm/document/{{ documentUuid }}/{{ versionNumber }}
```
Przy poprawie należy przekazać:
- nowy - poprawiony XML zlecenia
- uuid (będący wygenerowanym: *id-tech-dokumentu*)
- version - kolejny numer wersji przekazywanego zlecenia

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

NFZ umożliwia pobranie aktualnego statusu dokumentu. Można tego dokonać, przekazując numer dokumentu wraz z kodem dostępu (numerem PESEL lub datą urodzenia) lub numerem technicznym dokumentu.

Pobranie statusu używając numeru PESEL:
```http request
GET /ezwm/documentstatus?zlec=T5-PC00013R-00000014&pesel=88072198883
Authorization: Bearer {{ token }}
```

Pobranie statusu za pomocą numeru technicznego dokumentu (*id-tech-dokumentu-nfz*):
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
**Z – Zlecenie ma zarejestrowaną i potwierdzoną realizację** (w przypadku zaopatrzenia comiesięcznego wszystkie okresy zlecenia są albo zrealizowane, albo anulowano ich realizację, przy czym przy anulowaniu okresów realizacji chociaż jeden okres zlecenia musi być zrealizowany)  

### Pobierz dokument dot. zlecenia

NFZ umożliwia pobranie różnych dokumentów dla osoby uprawnionej do wystawienia zlecenia.  
Do pobrania każdego z dokumentów należy przekazać:
- numer zlecenia (*nr-zlecenia-nfz*)
- idTech (*id-tech-dokumentu-nfz*)

Dokumenty możliwe do pobrania (type):  
**zlecenia** - Dokument XML przesłanego zlecenia  
**wynik-weryfikacji** - Dokument z wynikiem weryfikacji zlecenia (wynik przetwarzania dokumentu zlecenia)  
**info-zlecenia-pdf** - Druk informacyjny zlecenia elektronicznego w formacie PDF  
**zlecenia-pdf** - Wydruk zlecenia w formacie PDF  
**zlecenia-bez-weryf-pdf** - Wydruk zlecenia bez weryfikacji w formacie PDF  

Każdy z w/w elementów można pobrać używając endpointu:
```http request
GET /ezwm/document/zlecenia?zlec=T5-PC00013R-00000015&idTech=34519800000000000100007839
```
Do endpointu należy przekazać odpowiedni typ dokumentu:
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


### Anuluj dokument zlecenia

Anulowanie dokumentu odbywa się za pomocą endpointu, za pomocą którego przekazywane są dokumenty zlecenia. W tym przypadku należy przekazać dokument anulujący zlecenie, zgodny ze specyfikacją: ```https://ezwm.nfz.gov.pl/xml/e-zpo/dok-anulowania-zlec/v2.1```

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

# eZWM MFA
Od 17 listopada 2025 roku logowanie do usług NFZ (w tym eZWM) będzie możliwe wyłącznie z wykorzystaniem uwierzytelnienia wieloskładnikowego (MFA).  
Jest to drugi etap wdrożenia bezpieczeństwa przez NFZ.  
Dotychczas MFA było wymagane jedynie dla serwisów dostępnych przez przeglądarkę internetową (np. SZOI/Portal Świadczeniodawcy).  

Do logowania do eZWM wykorzystywane są te same tokeny, które użytkownicy generują przy logowaniu do serwisów NFZ obsługujących MFA.  

eZWM i eWUŚ to w NFZ odrębne systemy, dlatego dla każdej z usług tworzona jest osobna sesja i wymagane jest osobne logowanie.  
Sesje są przechowywane po stronie API - przez cały okres ważności sesji (14 minut) użytkownik nie musi ponawiać logowania.  

> Uwaga:  
> Usługa testowa NFZ okazjonalnie „gubi” dane sesyjne użytkowników. Wynika to z braku współdzielenia danych autoryzacyjnych między instancjami systemu — w konsekwencji sesje mogą wygasać przed upływem 14 minut.  

## Zalecany przebieg pracy

1. Zalogowanie użytkownika do usługi z użyciem kodu TOTP: `GET /ezwm/login/{totp}`.
2. Wywoływanie operacji eZWM (wysyłka, aktualizacja, status, pobieranie i anulowanie dokumentów) w ramach aktywnej sesji.
3. Po otrzymaniu [błędu braku sesji](#brak-aktywnej-sesji-użytkownika) (sesja wygasła lub została utracona) - ponowne zalogowanie z nowym kodem TOTP i powtórzenie operacji.
4. Opcjonalnie - wylogowanie: `GET /ezwm/logout` (nie jest wymagane, sesja wygasa automatycznie).

Kod TOTP może zostać podany przez użytkownika albo wygenerowany automatycznie przez system dziedzinowy (własny generator kodów TOTP). Automatyczne generowanie pozwala na ponowne logowanie bez udziału użytkownika.

## Logowanie użytkownika do usługi

W przypadku aktywnego MFA należy wykonać jawne logowanie, przekazując token autoryzacyjny (kod TOTP).  
Po zalogowaniu tworzona jest sesja ważna przez 14 minut - moment jej wygaśnięcia zwracany jest w polu `expires_at`. Po jej wygaśnięciu należy ponowić logowanie.  
W tokenie JWT należy przekazać specjalistę powiązanego z usługą eZWM (*practitioner*).  

```http request
GET /ezwm/login/{totp}
Authorization: Bearer {{ token }}
```

### Odpowiedź pozytywna
```json
{
  "message": "Biostat\\EzwmHelper\\Mfa\\LoginTotpResult",
  "body": {
    "login_code": "000",
    "login_message": "Użytkownik został prawidłowo zalogowany",
    "expires_at": "2026-09-25T09:32:32+00:00"
  },
  "error": [],
  "raw": ""
}
```

### Odpowiedź negatywna - błąd logowania użytkownika
Zwracana, gdy podane parametry logowania (w tym kod TOTP) są nieprawidłowe.
Pole `code` jest puste - taką odpowiedź zwraca NFZ.

```json
{
  "message": "",
  "error": [
    {
      "code": "",
      "text": "Brak identyfikacji operatora. Podane parametry logowania są nieprawidłowe."
    }
  ],
  "raw": "",
  "body": ""
}
```

## Wylogowanie z usługi

Endpoint umożliwia wylogowanie użytkownika z usługi eZWM.  
Usuwa dane sesyjne i kończy połączenie z NFZ.  
Wylogowanie nie jest wymagane (sesja wygasa automatycznie po 14 minutach), ale może być potrzebne np. przy zmianie danych logowania (inny użytkownik).  
Endpoint nie wymaga żadnych parametrów.  

```http request
GET /ezwm/logout
```

### Odpowiedź pozytywna
```json
{
  "message": "Biostat\\EzwmClient\\Auth\\SoapEnvelope\\Messages\\LogoutOutput",
  "body": {
    "logout_message": "Wylogowany"
  },
  "error": [],
  "raw": ""
}
```
