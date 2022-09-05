# Usługa: Podpis XaDES-BES

Usługa umożliwiająca podpisanie dokumentu XML za pomocą certyfikatu ZUS practitionera.

## Nagłowki
- Content-type: application/xml
- Authorization: Bearer {token}

W tokenie musi znaleźć się informacja, czyim certyfikatem należy podpisać dokument (*practitioner*).

## Endpoint
- POST `/tools/sign` - Podpis dokumentu XML

## Dane wejściowe
Dokument przeznaczony do podpisu powinien zostać przesłany w body żądania (POST), z nagłówkiem
`Content-type: application/xml`.

## Dane wyjściowe
W odpowiedzi zwrócony zostanie podpisany dokument XML (jako `application/xml`) lub informacja o błędzie (`application\json`).
