# Usługa e-Zgody

Usługa pozwala na:
- sprawdzenie zgody wydanej przez pacjenta dla danego podmiotu lub pracownika medycznego.
- uzyskanie czasowej zgody pacjenta na dostęp do dokumentacji medycznej (Zdarzeń Medycznych)

## Nagłówki
Aby wystawić e-skierowanie, należy w nagłówku przesłać token JWT, w którym zawarty będzie __podmiot__ (`organization`) oraz __specjalista__ (`practitioner`).
- Authorization: Bearer {token}

## Dostępne trasy

- GET `/permissions/channels/{patientIdentifier}` - pobieranie kanałów komunikacji pacjenta, np. portal pacjenta, komunikacja e-mail, komunikacja SMS.
- POST `/permissions/authorizationrequest/{patientIdentifier}/{authorizerIdentifier}` - wysyłanie do pacjenta, lub opiekuna pacjenta prośby o nadanie uprawnień do dokumentacji medycznej.
- POST `/permissions/authorize/{requestId}/{authorizeCode}` - po otrzymaniu od pacjenta kodu autoryzacyjnego używamy tej trasy, aby otrzymać uprawnienia do dokumentacji pacjenta.
- GET `/permissions/medicalrecords/{id}/{patientIdentifier}/{authorizerIdentifier}(?/{time})` - trasa sprawdza, czy posiadamy dostęp do dokumentacji pacjenta, opcjonalnie możemy wskazać konkretną datę i godzinę, w której uprawnienia chcemy sprawdzić.

## Pobieranie kanałów komunikacji pacjenta

```http request
GET {{host}}/permissions/channels/2.16.840.1.113883.3.4424.1.1.616%7c70032816894
Content-Type: application/json
Authorization: Bearer {{ jwtToken }}
```

Przykładowa odpowiedź:
```json
{
  "message": "",
  "body": {
    "czyPowiadomieniaEmail": false,
    "czyPowiadomieniaSMS": false,
    "czyKontoIKP": true
  },
  "error": [],
  "raw": {
    "kanalyKomunikacjiUslugobiorcy": {
      "czyPowiadomieniaEmail": false,
      "czyPowiadomieniaSMS": false,
      "czyKontoIKP": true
    },
    "wynik": {
      "major": "urn:csioz:p1:kod:major:Sukces",
      "minor": "",
      "komunikat": "Sukces"
    }
  }
}
```

## Wysyłanie wniosku o nadanie uprawnień

```http request
POST {{host}}/permissions/authorizationrequest/2.16.840.1.113883.3.4424.1.1.616%7c73051914886
Content-Type: application/json
Authorization: Bearer {{ jwtToken }}
```

Przykładowa odpowiedź:
```json
{
  "message": "",
  "body": {
    "idWniosku": "2c5bc59a-c09c-4066-8353-21b4b92977c4"
  },
  "error": [],
  "raw": {
    "wynik": {
      "idWniosku": "2c5bc59a-c09c-4066-8353-21b4b92977c4"
    }
  }
}
```

>Gdzie __idWniosku__ jest potrzebne do kolejnej trasy, do potwierdzenia zgody pacjenta.

Drugi przykład, gdy prosimy opiekuna o udzielenie dostępu do dokumentacji pacjenta:

```http request
POST {{host}}/permissions/authorizationrequest/2.16.840.1.113883.3.4424.1.1.616%7c70032816894/2.16.840.1.113883.3.4424.1.1.616%7c73051914886
Content-Type: application/json
Authorization: Bearer {{ jwtToken }}
```

Przykładowa odpowiedź:
```json
{
  "message": "",
  "body": {
    "idWniosku": "d5efbf82-2d94-4007-a939-887210361cd1"
  },
  "error": [],
  "raw": {
    "wynik": {
      "idWniosku": "d5efbf82-2d94-4007-a939-887210361cd1"
    }
  }
}
```

## Potwierdzenie zgody pacjenta

```http request
POST {{host}}/permissions/authorize/d5efbf82-2d94-4007-a939-887210361cd1/0000
Content-Type: application/json
Authorization: Bearer {{ jwtToken }}
```

Poprawna odpowiedź zawiera ponownie idWniosku:

```json
{
  "message": "",
  "body": {
    "idWniosku": "d5efbf82-2d94-4007-a939-887210361cd1"
  },
  "error": [],
  "raw": {
    "wynik": {
      "idWniosku": "d5efbf82-2d94-4007-a939-887210361cd1"
    }
  }
}
```

Gdzie:
- __0000__ to testowy kod autoryzacyjny, na środowisku produkcyjnym pacjent musi ten kod przekazać lekarzowi.

Błędna odpowiedź może na środowisku testowym zawierać np. informację o "Worker error".

## Sprawdzenie uprawnień do dokumentacji pacjenta

```http request
GET {{host}}/permissions/medicalrecords/2.16.840.1.113883.3.4424.1.1.616%7c73051914886/2.16.840.1.113883.3.4424.1.1.616%7c51021985937
Content-Type: application/json
Authorization: Bearer {{ jwtToken }}
```

Przykładowa odpowiedź:
```json
{
  "message": "",
  "body": {
    "wynik": "ZGODA_UDZIELONA",
    "waznoscNaDataCzas": "2022-10-04T18:45:05+02:00"
  },
  "error": [],
  "raw": {
    "wynik": "PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiIHN0YW5kYWxvbmU9Im5vIj8+PHBvZHBpc2FuYUluZm9ybWFjamFPVWR6aWVsb25lalpnb2R6aWVNVCB4bWxucz0iaHR0cDovL2NzaW96Lmdvdi5wbC9wMS96Z29keS9tdC92MjAxOTA1MzAiIHhtbG5zOm5zMj0iaHR0cDovL2NzaW96Lmdvdi5wbC9wMS93c3BvbG5lL210L3YyMDE4MDUwOSIgeG1sbnM6bnMzPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwLzA5L3htbGRzaWcjIiB4bWxuczpuczQ9Imh0dHA6Ly9jc2lvei5nb3YucGwvcDEvemdvZGFOYVN3aWFkY3plbmllL210L3YyMDE5MDUyOCI+PGluZm9ybWFjamFPVWR6aWVsb25lalpnb2R6aWVNVD48d3luaWsgeG1sbnM6eHM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDEvWE1MU2NoZW1hIiB4bWxuczp4c2k9Imh0dHA6Ly93d3cudzMub3JnLzIwMDEvWE1MU2NoZW1hLWluc3RhbmNlIiB4c2k6dHlwZT0ieHM6c3RyaW5nIj5aR09EQV9VRFpJRUxPTkE8L3d5bmlrPjxpZGVudHlmaWthdG9yUGFjamVudGE+PG5zMjpleHRlbnNpb24+NzMwNTE5MTQ4ODY8L25zMjpleHRlbnNpb24+PG5zMjpyb290PjIuMTYuODQwLjEuMTEzODgzLjMuNDQyNC4xLjEuNjE2PC9uczI6cm9vdD48L2lkZW50eWZpa2F0b3JQYWNqZW50YT48dXBvd2F6bmlvbnk+PGlkZW50eWZpa2F0b3JCaXpuZXNvd3k+PG5zMjpleHRlbnNpb24+NTEwMjE5ODU5Mzc8L25zMjpleHRlbnNpb24+PG5zMjpyb290PjIuMTYuODQwLjEuMTEzODgzLjMuNDQyNC4xLjEuNjE2PC9uczI6cm9vdD48L2lkZW50eWZpa2F0b3JCaXpuZXNvd3k+PGltaWU+UEFXRcWBPC9pbWllPjxuYXp3aXNrbz5LT1dBTFNLSTwvbmF6d2lza28+PC91cG93YXpuaW9ueT48d2F6bm9zY05hRGF0YUN6YXM+MjAyMi0xMC0wNFQxODo0NTowNS4wMDArMDI6MDA8L3dhem5vc2NOYURhdGFDemFzPjwvaW5mb3JtYWNqYU9VZHppZWxvbmVqWmdvZHppZU1UPjxTaWduYXR1cmUgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvMDkveG1sZHNpZyMiPjxTaWduZWRJbmZvPjxDYW5vbmljYWxpemF0aW9uTWV0aG9kIEFsZ29yaXRobT0iaHR0cDovL3d3dy53My5vcmcvVFIvMjAwMS9SRUMteG1sLWMxNG4tMjAwMTAzMTUiLz48U2lnbmF0dXJlTWV0aG9kIEFsZ29yaXRobT0iaHR0cDovL3d3dy53My5vcmcvMjAwMC8wOS94bWxkc2lnI3JzYS1zaGExIi8+PFJlZmVyZW5jZSBVUkk9IiI+PFRyYW5zZm9ybXM+PFRyYW5zZm9ybSBBbGdvcml0aG09Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvMDkveG1sZHNpZyNlbnZlbG9wZWQtc2lnbmF0dXJlIi8+PC9UcmFuc2Zvcm1zPjxEaWdlc3RNZXRob2QgQWxnb3JpdGhtPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwLzA5L3htbGRzaWcjc2hhMSIvPjxEaWdlc3RWYWx1ZT5walJZbzZLRHhqWXJSd2VNTzdhYjVhWEdqTm89PC9EaWdlc3RWYWx1ZT48L1JlZmVyZW5jZT48L1NpZ25lZEluZm8+PFNpZ25hdHVyZVZhbHVlPmdmeUcyaTgrSlNkaWluYXRKSGtGZFBCa1BLNU5ldzRYcnZ0a3JTT2NtYkRCekJTbk5nYXNSWkdLcHpUVUJTaEdjT0xxOUhubHk1blMKOFJ2RVowT1ZQcGhXREdRQmFBaFVMa2Q3c1lrRFlyQnJPOEVrY2Z1SHYrdkdNbEtqSjFCaWs0TzhvbWZ5V0hFcE5VejVsSWdLODU2RQptR2lRYUFIanA4Wm5HcjNqSnBBVEJ3a0tyalk5enJEb2p0RVRKNWh3Q0g5WkJoeHdqdldjSTR5cy9UV0lxN05XblpRY2cxb0llTnRQCmdCTHl2a1MwMDd2eWNGSHdLUHFReTNtRDdxaVFHRWs4MUtwVGlHNUpna1NGOVZyTFc5YkU2cWhWcUIrd0VybS8zMklvMlV1eDh2c0YKUFFPVWlQOUQxakRBUzlMZGF3MTdHZ3EyOVBjZjcxdmhZMDVjV1E9PTwvU2lnbmF0dXJlVmFsdWU+PEtleUluZm8+PFg1MDlEYXRhPjxYNTA5U3ViamVjdE5hbWU+Qz1QTCxPPUNTSU9aLE9VPVAxIFRlc3Rvd2UsMi41LjQuNT0jMTMxMjUwNDU1MzQ1NGMzYTIwMzIzMjMyMzIzMjMyMzIzMjMyMzIzMyxDTj1rd2FsaWZpa293YW55LWthcmd1bDwvWDUwOVN1YmplY3ROYW1lPjxYNTA5Q2VydGlmaWNhdGU+TUlJRTlqQ0NBdDZnQXdJQkFnSUlSMEI0dWk0eGhXTXdEUVlKS29aSWh2Y05BUUVOQlFBd1R6RWJNQmtHQTFVRUF3d1NRME1nVURFZwpVM1ZpUTBFZ1VHOWtjR2x6TVJNd0VRWURWUVFMREFwUU1TQlVaWE4wYjNkbE1RNHdEQVlEVlFRS0RBVkRVMGxQV2pFTE1Ba0dBMVVFCkJoTUNVRXd3SGhjTk1UZ3dNVEF6TVRFek1ESTVXaGNOTWpBd01UQXpNVEV6TURJNVdqQnVNUjB3R3dZRFZRUUREQlJyZDJGc2FXWnAKYTI5M1lXNTVMV3RoY21kMWJERWJNQmtHQTFVRUJSTVNVRVZUUlV3NklESXlNakl5TWpJeU1qSXpNUk13RVFZRFZRUUxEQXBRTVNCVQpaWE4wYjNkbE1RNHdEQVlEVlFRS0RBVkRVMGxQV2pFTE1Ba0dBMVVFQmhNQ1VFd3dnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCCkR3QXdnZ0VLQW9JQkFRQ2E2bWZIMUg4cFpuTDEyT0o5eDNjYkpnMHU2T3JVOTF0OVgzS0pUVGZ4K2twSUphUHNKaStwVWR6bkFXeWQKRVpnZ01PQzAyWVN6M2ZnNFZaaUI4YU16Q2J2N2dmTXZLTUZUbjZEek5DdUFMbVNuQWIybmMwSXlzcHZJdGpHdnVWa0lBZEtPOW5zRApGTkloajhSRjZMTnM3TmlDVE1Dc3hyZ1E5MWNDU1lQcExZeG1iaDdvL2huc1NIc051TUhIZjFjd2pza2FBcHdNcXFhQWJEcFdKejdQClFHYWY5UGdDdXVzVHFvelNlaHYyWkJzbndaNzk5RTZiL2FjcGs4QU03dWd6c2VyMkZ1M1JkQ0NCVkRrUCtwVDBFVTRhWmU2blhmV04KdzRJWHVBN1dhaFRSSFBRdWl2MExRVmlHT01Ha1ZZK0FPRlFhd2ViY0VDMlBJZkk2TmpVeEFnTUJBQUdqZ2JZd2diTXdIUVlEVlIwTwpCQllFRko3b245d09HeVd3a2ZNNTVjeC92c2ZYTHVQRU1Bd0dBMVVkRXdFQi93UUNNQUF3SHdZRFZSMGpCQmd3Rm9BVU8zcUpWL0d5CnBackdjQzJzMldzU0JGcVk3Z0F3VXdZRFZSMGZCRXd3U2pCSW9FYWdSSVpDYUhSMGNEb3ZMM2R6TFhSbGMzUXRjREV1WTNOcGIzb3UKWjI5MkxuQnNMMk5qY0RFdlkzSnNMME5EVURGVVpYTjBiM2RsVTNWaVEwRlFiMlJ3YVhNdVkzSnNNQTRHQTFVZER3RUIvd1FFQXdJSApnREFOQmdrcWhraUc5dzBCQVEwRkFBT0NBZ0VBUkY5N2ZYL0JGK0VoSVorOWRld2puY05Ga0lseXJ3MURkbkRTdFMrYzZzOUdiK1RFCkRnY3hyTTFieFl5OXpmK3gxMWtQK0k1QlFtdUZvak9vSFBzVGQxRk8wYzZta1BYaThYK3JKWGpnVGhPbXR2V2VFYUlpZm0xb0lwbWUKRzdYajBuRmpoK1R6eDFOTEZ6Z0VUWVJuTUZlMG85aUEzQjVESUI4aEJ6eWVPSTBMUGRiQmR2QTdBckZvNjY1Mjk1angxZ0p5bGU4YQo0enp2TUx5c3p6TDlHeUJCWUthbVVSZ3YzOXpEM1k3djhDR2JKQU9jVFIxOXgybVhKUG1tVStSZnluZXIzb2V0eU9pU3hPUmphejl2CnlzRzVrRHNNdWhPRHBTSG85VU1zUTVUb2RvNlZlYUxIR1lKRk9vanY3VDhsOEd4dnR4NU9GVGJuSmordDRGQVpxQytCcmdpc3pUVFkKOWYrV29wRnBwZUZPbjJGVXpWSFFSOVlsaUF4ZEV0SG5DbFhzWUJCa1ljaHMySWJWTWpONTZpNjdjR0JxbVlCbUNkeVRCRW9RUnBhdApjTEZ4TWJmcmFlczlMTEMzZWt0emRmS2tQYU1RdERTWGhLOU0xbkNWcEQyT3l6OExqbUNKZ0JaTmhSUjY1RTAxSnNhWVlRcEFWQzg2Ck1VUGUreG5Ob3VZYTVCOTlmSjY4T2s3SUEwMmt0VVVTeTNDR1FxbkRXTEhwdHZEejBKMUhaUTFySFBjWS9mZ21vM01WUlcvZExpa0oKYnRIZ3FzMXNMSmpwTENOblNoVk9rdEs1QVRFejFabDkrZzZ1dzNQa3MzNnA5bUE3d0pzWHVFTmR0WExDRzNMSHZ1dDFCQkdyaUR2QQpqYVVGbXpzNVBzS3RsNDI2S1liWE0zQW9UL0k9PC9YNTA5Q2VydGlmaWNhdGU+PC9YNTA5RGF0YT48L0tleUluZm8+PC9TaWduYXR1cmU+PC9wb2RwaXNhbmFJbmZvcm1hY2phT1VkemllbG9uZWpaZ29kemllTVQ+"
  }
}
```
