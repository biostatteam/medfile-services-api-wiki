# Komunikacja asynchroniczna

(dokumentacja w budowie)

## Webhook

Aby otrzymywać powiadomienia o wykonanych zadaniach w trybie asynchronicznych, 
musisz zarejestrować webhook, na który będziemy wysyłać wyniki zadań.

```http request
POST /webhook/callback
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}

{
  "callbackUrl": "http:\/\/yourdomain.com\/callback.php"
}
```

## Dostarczalność

Każdy wynik zadania jest wysyłany na webhook do 3 razy w 15-minutowych odstępach.

### Ponowienie wysyłki jednej niedostarczonej wiadomości

Jeżeli z jakiegoś powodu Państwa skrypt nie był w stanie poprawnie odebrać pojedynczej wiadomości, można ponowić raport poniższym poleceniem:

```http request
GET /services/resend/{requestId}
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}
```

### Ponowienie wysyłki wszystkich niedostarczonych wiadomości

W przypadku dłuższej awarii po Państwa stronie można wymusić wysyłkę wszystkich zaległych powiadomień poniższym poleceniem:

```http request
GET /services/resend/
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}
```

## Zlecanie zadań w trybie asynchronicznym

Aby zadanie zostało wykonane asynchronicznie wystarczy 
do tokenu JWT dopisać parametr `requestId` i nadać mu unikalny identyfikator/kod/uri. 
Tak, aby byli Państwo w stanie zidentyfikować odpowiedź i przypisać do konkretnego żądania.

Wprowadzony do tokenu JWT parametr `requestId` zostanie Państwu zwrócony na Państwa `callbackUrl` do każdego wyniku zadania.

> __Uwaga__
> 
> Nie wszystkie zadania obsługują asynchroniczność. Jeżeli pomimo umieszczenia requestId w tokenie odpowiedź nie będzie zawierała odpowiedzi o treści "Asynchronous request created" to oznacza, że nie wspiera ona asynchroniczności i otrzymali Państwo wynik synchronicznie. 