# Usługa: Słowniki EZWM
Słowniki używane podczas tworzenia dokumentu EZWM (oraz inne słowniki udostępniane przez NFZ znajdują się w zbiorze endpointów /dictionary/nfz.
Każy z endpointów autoryzowany jest poprzez bearer.
Poniżej przedstawiona jest lista endpointów wraz z opisem funkcjonalności.

### Wyszukiwanie wyrobów medycznych
Wyszukiwanie wyrobów medycznych odbywa się za pomocą poniższego endpoint: 

```http request
GET /dictionary/nfz/wyroby_medyczne/S?search=&page=
```

Parametry:
- rodzaj wyrobu
    - S - wyrób medyczny
    - N - naprawa
- ```search``` - wyszukiwanie po kod/nazwa
- ```page``` - numer strony z wynikiem (domyślnie 1)
- użycie ```&filter=soczewki``` umożliwi wyświetlenie w śród wyników wyłącznie soczewek


#### Wyszukiwanie wyrobów rodzaju N - naprawa

```http request
GET /dictionary/nfz/wyroby_medyczne/N?search=A.03.01.N&page=
```

#### Szukaj wyrobów dotyczących soczewek

```http request
GET /dictionary/nfz/wyroby_medyczne/S?search=&page=&limit=&filter=soczewki
```

### Pokaż kryteria medyczne wg danego wyrobu medycznego
Endpoint zwraca kryteria medyczne potrzebne do uzupełnienia w ramach wystawiania zlecenia na wyrób medyczny.  
Jako parametr należy wskazać kod wyrobu medycznego.

```http request
GET /dictionary/nfz/wyroby_medyczne/{{kod_wyrobu_medycznego}}/kryteria_medyczne
```

Poniższy przykład wyszukiwanie kryteriów dla produktu A.003 Proteza ostateczna w obrębie stawu skokowego z kikutem oporowym:

```http request
GET /dictionary/nfz/wyroby_medyczne/A.003/kryteria_medyczne
```

## Dodatkowe słowniki do obsługi danych ubezpieczenia pacjenta
### Instytucje właściwe UE

Wyszukiwanie instytucji właściwych.
Dostępne wyszukiwanie:
- ```search``` - nazwa/kod instytucji
- ```countrycode``` - kod kraju (w formacie ISO)

```http request
GET /dictionary/nfz/instytucje_wlasciwe_ue?search=&countrycode=
```

#### Lista krajów dla zdefiniowanych instytucji właściwych UE

```http request
GET /dictionary/nfz/instytucje_wlasciwe_ue/kraje
```

### Uprawnienia dodatkowe
Endpoint umożliwia wyszukanie kodu uprawnienia dodatkowego używanego w ramach wniosków na wyroby medyczne.  

Wyszukiwanie dostępne po:
- ```search``` - nazwa/kod uprawnienia
- ```area``` - umożliwia ograniczenie zwracanych kodów do odpowiedniego obszaru (ZPOSP/SWIAD)

```http request
GET /dictionary/nfz/dodatkowe_uprawnienia?search=&page=&limit=
```

### Typy dokumentów dla wybranego uprawnienia dodatkowego
Endpoint zwraca typy dokumentów potwierdzających wybrane uprawnienie możliwe do przekazania ze wskazanym uprawnieniem dodatkowym.  
Należy wskazać poprawny kod uprawnienia dodatkowego. 

```http request
GET /dictionary/nfz/dodatkowe_uprawnienia/typy_dokumentow?dodatkowe_uprawnienie=
```
---
Dodatkowe endpointy, które nie są wymagane do wystawiania wniosków na zaopatrzenie:

### Pokaż listę grup uprawnień
Endpoint zwraca pełny słownik grup uprawnień udostępniony przez NFZ.

```http request
GET /dictionary/nfz/dodatkowe_uprawnienia/grupy_uprawnien
```

### Pokaż listę obszarów stosowania
Endpoint zwraca pełny słownik opszarów stosowania uprawnień dodatkowych udostępniony przez NFZ.

```http request
GET /dictionary/nfz/dodatkowe_uprawnienia/obszary_stosowania
```
