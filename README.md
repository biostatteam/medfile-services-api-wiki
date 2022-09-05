# Medfile Services API

Wersja API: `2.3.0`

Kontakt: <integracja@medfile.pl>

## Zmiany w 2.3.0 z dnia 28.08.2022

1. Reorganizacja dokumentacji oraz spis treści.
2. Poprawienie dokumentacji XDS.b

## Zmiany w 2.2.0 z dnia 28.01.2022

1. Dokumentacja Zdarzeń Medycznych
2. Dokumentacja bazy leków
3. Wprowadzono trasę do tworzenia organizacji
4. Poprawiono przykład z wysyłaniem i weryfikacją erecepty ("erecepta": ...)

## Zmiany w 2.0.3 z dnia 26.11.2020

1. Dla recept i skierowań dodano parametry pacjentów transgranicznych lub nieletnich bez numeru PESEL.

2. Dokumentacja eZLA została przebudowana i zawiera teraz przykładowe odpowiedzi.

3. W dokumentacji eZLA zmodyfikowano punkt 8.1. o instrukcje na temat uruchomienia środowiska testowego.

## Zmiany w 2.0.2 z dnia 12.10.2020

1. W tokenie JWT wymagamy dodatkowego pola organization (uuid) aby obsłużyć np. anulowanie recept oraz pobieranie recept.

2. Nowy kod odpowiedzi: `406` gdy wynik operacji jest błędny np. nie uda się zweryfikować recepty.

3. Anulowanie pakietu recept

4. Drukowanie pakietu recept

5. Pobieranie pakietu recept

6. Określenie kodów odpowiedzi dla wysyłania i weryfikacji recept

## Zmiany w 2.0.1 z dnia 05.10.2020

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

# Spis treści

1. [Wprowadzenie do API](Introduction.md)
2. Usługi P1
    - [e-Recepta](Erecepta.md)
    - [e-Skierowanie](Eskierowania.md)
    - [Zdarzenia Medyczne](ZdarzeniaMedyczne.md)
    - [Index EDM (dokumentacja w trakcie budowy)](XDSRepository.md)
    - [Podpis XaDES](Signature.md)
3. Usługi NFZ
    - [eWUŚ](Ewus.md)
4. Usługi ZUS
    - [eZLA](Ezla.md)
5. [Repozytorium XDS.b](XDSRepository.md)
6. Usługi dodatkowe
    - [Baza leków](Dictionaries.md)
    - Baza ICD 10 (dokumentacja w trakcie budowy)
    - Baza ICD 9 (dokumentacja w trakcie budowy)
    - Snomed GPS (wkrótce)
7. [Informacje dodatkowe](Tools.md)
8. [Asynchroniczna komunikacja](AsynchronousCommunication.md) (dokumentacja w trakcie budowy)
