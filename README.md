# Medfile Services API

Wersja API: `2.3.6`

Kontakt: <integracja@medfile.pl>

## Spis treści

1. [Wprowadzenie do API](Introduction.md)
   - [Jak uzyskać dostęp do API?](IntegrationProcess.md)
   - [Najczęściej zadawane pytania](FAQ.md)
2. Usługi P1
   - [e-Recepta](Erecepta.md)
     - [Dawkowanie sekwencyjne \- Przykłady dawkowania dla recept 365](Erecepta365.md)
     - [Recepta na Wyroby Medyczne](EreceptaWM.md)
     - [Recepta na import docelowy](EreceptaImportDocelowy.md)
   - [e-Skierowanie](Eskierowania.md)
   - [e-Zgody](Ezgody.md)
   - [Zdarzenia Medyczne](ZdarzeniaMedyczne.md)
   - [Index EDM](XDSRepository.md)
   - [Podpis XaDES](Signature.md)
3. Usługi NFZ
   - [eWUŚ](Ewus.md)
   - [eZWM](Ezwm.md) (dokumentacja w trakcie budowy)
4. Usługi ZUS
   - [eZLA](Ezla.md)
5. [Repozytorium XDS.b](XDSRepository.md)
6. Usługi dodatkowe
   - [Baza leków](Dictionaries.md)
   - [Słowniki dla eZWM](EzwmDictionaries.md)
   - [Słownik ICD 10](Dictionaries.md#słownik-rozpoznań-icd-10)
   - [Słownik ICD 9](Dictionaries.md#słownik-procedur-medycznych-icd-9)
   - Snomed GPS (wkrótce)
7. [Informacje dodatkowe](Tools.md)
   - [Interaktywna dokumentacja Swagger Open API 3.0.0](Tools.md#interaktywna-dokumentacja-swagger)
   - [Dodatkowe słowniki](Tools.md#słowniki)
8. [Asynchroniczna komunikacja](AsynchronousCommunication.md)

## Changelog

### Zmiany w 2.3.7 z dnia 15.09.2025

1. Opracowano receptę na import docelowy, dodano nową sekcję: [Recepta na import docelowy](EreceptaImportDocelowy.md)

### Zmiany w 2.3.6 z dnia 20.01.2025

1. W ramach obsługi e-recepty dodano przekazywanie danych dodatkowych (receptaDaneDodatkoweMT): 
     - dodano element attributes w JSON


### Zmiany w 2.3.5 z dnia 06.09.2024

1. Rozszerzono szablony dawkowania (opis w sekcji [Schematy dawkowania recept](Erecepta365.md)):
     - dodano obsługę przerw
     - dodano obsługę cykli
     - umożliwiono przekazanie "Informacji dla pacjenta"
     - rozszerzono obsługę "Kategorii dostępności leku"
2. Nowa sekcja: [Recepta na Wyroby Medyczne](EreceptaWM.md)
   
### Zmiany w 2.3.4 z dnia 22.04.2024

1. Nowa sekcja: [Dawkowanie sekwencyjne \- Przykłady dawkowania dla recept 365](Erecepta365.md)

### Zmiany w 2.3.3 z dnia 04.10.2022

1. Nowa sekcja: [e-Zgody](Ezgody.md)

### Zmiany w 2.3.2 z dnia 20.09.2022

1. [Repozytorium XDS.b ITI-43 - pobieranie dokumentów z innego repozytorium](XDSRepository.md#iti-43---pobieranie-dokumentów-z-innego-repozytorium)
2. [Repozytorium XDS.b - pobieranie plików bezpośrednio z magazynu](XDSRepository.md#pobieranie-plików-bezpośrednio-z-magazynu)

### Zmiany w 2.3.1 z dnia 05.09.2022

1. Nowa sekcja: [Jak uzyskać dostęp do API?](IntegrationProcess.md)
2. Nowa sekcja: [Najczęściej zadawane pytania](FAQ.md)
3. Nowa sekcja: [Materiały dodatkowe: Narzędzie do testowania integracji (Swagger Open API 3.0.0)](Tools.md#interaktywna-dokumentacja-swagger)

### Zmiany w 2.3.0 z dnia 28.08.2022

1. Reorganizacja dokumentacji oraz spis treści.
2. Poprawienie dokumentacji XDS.b

### Zmiany w 2.2.0 z dnia 28.01.2022

1. Dokumentacja Zdarzeń Medycznych
2. Dokumentacja bazy leków
3. Wprowadzono trasę do tworzenia organizacji
4. Poprawiono przykład z wysyłaniem i weryfikacją erecepty ("erecepta": ...)

### Zmiany w 2.0.3 z dnia 26.11.2020

1. Dla recept i skierowań dodano parametry pacjentów transgranicznych lub nieletnich bez numeru PESEL.
2. Dokumentacja eZLA została przebudowana i zawiera teraz przykładowe odpowiedzi.
3. W dokumentacji eZLA zmodyfikowano punkt 8.1. o instrukcje na temat uruchomienia środowiska testowego.

### Zmiany w 2.0.2 z dnia 12.10.2020

1. W tokenie JWT wymagamy dodatkowego pola organization (uuid) aby obsłużyć np. anulowanie recept oraz pobieranie recept.
2. Nowy kod odpowiedzi: `406` gdy wynik operacji jest błędny np. nie uda się zweryfikować recepty.
3. Anulowanie pakietu recept
4. Drukowanie pakietu recept
5. Pobieranie pakietu recept
6. Określenie kodów odpowiedzi dla wysyłania i weryfikacji recept

### Zmiany w 2.0.1 z dnia 05.10.2020

1. Numeracja sekcji w dokumentacji.
2. Poprawka w obiekcie `telecom` - wymagana składnia to
   ```json
   [{"value":"+48600600600"}]
   ```
3. W obiektach practitioner oraz organization zmiana z liczby mnogiej na pojedynczą listy `identifiers`. Po zmianie składnia wygląda następująco:
   ```json
   "identifier": [
      {
        "type": "rpwdl",
        "value": "000000792087"
      }
   ]
   ```
