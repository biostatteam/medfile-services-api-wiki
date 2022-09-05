# Interaktywna dokumentacja (swagger)

Udostępniamy dokumentację interaktywną do szybkiego przetestowania wybranych usług.

> [Dokumentacja Interaktywna Swagger Open API 3.0.0](https://app.swaggerhub.com/apis/BioStatDevelopment/Medfile_API_server/1) 

Do testowania potrzebujemy:
- nadanego dostępu do środowiska testowego
- wygenerowanego tokenu JWT dla konkretnego przypadku (np. dla recepty wymagany jest organization i practitioner w tokenie, ale do szukania leków tylko practitioner)

Procedura:
1. Z prawej strony klikamy na listę wyboru `servers` i wybieramy https://dev.services.medfile.pl.
2. Następnie klikamy `Authorize` i wklejamy utworzony lokalnie token JWT.

Od teraz możliwe jest testowanie interaktywnej dokumentacji.


# Słowniki

## `identifier` dla organizacji i praktyki

1. `type: rpwdl` pierwsza część kodu resortowego którą można odczytać na stronie https://rpwdl.csioz.gov.pl/ dla wybranego podmiotu. Tzw. numer księgi resortowej.
    - Przykład: 000000089955
    - Wymagany dla praktyk oraz podmiotów

1. `type: rpwdlUnit` piąta część kodu resortowego. Tzw. numer jednotki organizacyjnej pod którą wystawiamy receptę. Odczytamy ją z https://rpwdl.csioz.gov.pl/.
    - Przykłady: 01, 02, 99
    - Wymagany tylko dla podmiotów

1. `type: regon` numer REGON podmiotu lub praktyki. REGON 9 cyfrowy to podstawowy numer danego podmiotu lub praktyki. REGON 14 cyfrowy to połączony numer podstawowy podmiotu i dodatkowy numer jednostki np. 00004 (nie jest to tożsame z rpwdlUnit). Możemy to wyczytać z https://rpwdl.csioz.gov.pl/.
    - Przykład: 24154444300004, 241544443
    - Wymagany 9 cyfrowy dla praktyk i 14 cyfrowy dla podmiotów (jednostek).

1. `type: oidRoot` numer OID nadany przez P1 dla każdego certyfikatu który będzie użyty do komunikacji z CSIOZ. Możemy go otrzymać wyłącznie w wiadomości e-mail zaraz po poprawnym wygenerowaniu certyfikatów TLS i WSS.
    - Przykład: 2.16.840.1.113883.3.4424.2.7.67
    - Wymagany dla podmiotów i praktyk

1. `type: chamber` numer organu wydającego pozwolenie działalności dla praktyk i podmiotów. Można go wyczytać z https://rpwdl.csioz.gov.pl/.
    - Przykład: L-63 gdzie L = Okręgowa Rada Lekarska a 63 = Wielkopolska Izba Lekarska
    - Wymagany tylko dla praktyk lekarskich

## `telecom` - numer telefonu

Dopuszczalny format numeru telefonicznego to:
```
+48600600600
```
Gdzie +48 to numer kierunkowy kraju (Polski).

## `practitioner.role` rola lekarza

Znaczenie: kontekst opisujący stanowisko osoby wysyłającej żądanie do P1.

Przeznaczenie: wystawienie e-recepty i e-skierowania.

Dopuszczalne wartości:
- `LEK` - lekarz / felczer
- `LEKD` - lekarz dentysta

## `practitioner.qualifications` specjalizacje lekarza

Znaczenie: kody i nazwy specjalizacji, które posiada wybrany lekarz.

Przeznaczenie: wystawianie e-skierowań.

Dopuszczalne wartości: reguluje to rozporządzenie, źródło https://isap.sejm.gov.pl/isap.nsf/download.xsp/WDU20190000602/O/D20190602.pdf

Przykład:
```json
{
  "code": "0713",
  "display": "medycyna rodzinna"
}
```

## `erecepta.type` typ recepty

Tym parametrem decydujemy czy wystawiamy receptę na lek gotowy czy recepturę własną.

Dopuszczalne wartości:
- `prepared` lek gotowy np. Apap
- `recipe` receptura np. lista składników

## `patient.nfz`

Oddział NFZ, do którego należy pacjent.

- 01 - Dolnośląski
- 02 - Kujawsko-Pomorski
- 03 - Lubelski
- 04 - Lubuski
- 05 - Łódzki
- 06 - Małopolski
- 07 - Mazowiecki
- 08 - Opolski
- 09 - Podkarpacki
- 10 - Podlaski
- 11 - Pomorski
- 12 - Śląski
- 13 - Świętokrzyski
- 14 - Warmińsko-Mazurski
- 15 - Wielkopolski
- 16 - Zachodniopomorski

## `patient.permissions`

Uprawnienia dodatkowe pacjenta do zakupu bezpłatnych lub tańszych leków.

Np.: 

AZ, IB, IW, PO, WP, ZK, S, C, BW, DN, CN, IN.



Szczegółowe informacje można odszukać tutaj (zakładka "Uprawnienia dodatkowe"):

https://www.nfz.gov.pl/dla-pacjenta/recepty-i-leki/

## `patient.permissionsDocument`

Dokument potwierdzający uprawnienia dodatkowe pacjenta.

Dopuszczalne wartości:
- tekst np. "Legitymacja morska nr. 123123/123213"

## `eskierowanie.type`

Typ skierowania wg klasyfikacji dokumentów P1

- `02.10` - Skierowanie na badanie lub leczenie
- `02.11` - Prośba o objęcie opieką
- `02.12` - Skierowanie na konsultację
