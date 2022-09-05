# FAQ

## Ile trwa uzyskanie dostępu do środowiska testowego

Dostęp udzielny jest w dniu podpisania umowy o poufności lub dnia następnego, jeżeli podpisanie nastąpiło w godzinach wieczornych.

## Nasz program jest instalowany na komputerach lekarzy, gdzie powinna być zaimplementowana komunikacja z API.

Dostęp do środowiska produkcyjnego jest udostępniany tylko zaufanym urządzeniom.
W takim wypadku należy uruchomić usługę sieciową, która będzie pośredniczyła w komunikacji pomiędzy komputerem lekarza i naszym API.
Usługa ta będzie miała dostęp do API, a oprogramowanie gabinetowe na komputerze lekarza będzie musiało łączyć się z Państwa usługą, aby uzyskać dostęp do naszego API.

## Posiadamy jeden podmiot, ale wiele jednostek albo komórek, jak skonfigurować to w API?

Każda jednostka, komórka lub praktyka to odrębna organizacja w rozumieniu komunikacji z API.
W takim wypadku należy utworzyć tyle organizacji (organization), ile jednostek chcą Państwo obsługiwać.
W przypadku usług P1 dla każdej organizacji wymagane jest wgranie certyfikatów TLS i WSS.

## Czy istnieją limity żądań do API?

Nie ma obecnie limitów na żądania w środowiskach testowych i produkcyjnych. Usługa jest stale monitorowana i skalowana.