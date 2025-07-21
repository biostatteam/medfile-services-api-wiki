# Usługa: Słowniki EZWM

### Wyszukiwanie wyrobów medycznych
Wyszukiwanie wyrobów medycznych odbywa się za pomocą poniższego endpoint. 

```http request
GET /nfz/wyroby_medyczne/S?search=&page=&limit=
Authorization: Bearer {{ token }}
```

Parametry:
- rodzaj wyrobu
    - S - wyrób medyczny
    - N - naprawa
- filter:  *soczewki*  ???


#### Wyszukiwanie wyrobów rodzaju N - naprawa

```http request
GET /nfz/wyroby_medyczne/N?search=P.100&page=&limit=
```

#### Szukaj wyrobów dotyczących soczewek

```http request
GET /nfz/wyroby_medyczne/S?search=&page=&limit=&filter=soczewki
```

### Pokaż kryteria medyczne wg danego wyrobu medycznego
Endpoint zwraca kryteria medyczne potrzebne do uzupełnienia w ramach wystawiania zlecenia na wyrób medyczny.  
Jako parametr należy wskazać kod wyrobu medycznego.

```http request
GET /nfz/wyroby_medyczne/{{kod_wyrobu_medycznego}}/kryteria_medyczne
Authorization: Bearer {{ token }}
```

Przykład - dla soczewek??   // powinno być kod produktu - np. P.101
```http request
GET /nfz/wyroby_medyczne/soczewki/kryteria_medyczne
Authorization: Bearer {{ token }}
```

## Dodatkowe słowniki do obsługi danych ubezpieczenia pacjenta
### Instytucje właściwe UE

Wyszukiwanie instytucji właściwych.
W przypadku kodu kraju - należy go wskazać w formacie ISO.

```http request
GET /nfz/instytucje_wlasciwe_ue?search=&countrycode=
Authorization: Bearer {{ token }}
```

#### Lista krajów dla zdefiniowanych instytucji właściwych UE

```http request
GET /nfz/instytucje_wlasciwe_ue/kraje
Authorization: Bearer {{ token }}
```

### Uprawnienia dodatkowe
Endpoint umożliwia wyszukanie kodu uprawnienia dodatkowego używanego w ramach wniosków EZWM.
```http request
GET /nfz/dodatkowe_uprawnienia?search=&page=&limit=
Authorization: Bearer {{ token }}
```

### Typy dokumwntów dla wybranego uprawnienia dodatkowego -- sprawdzić
Endpoint zwraca typy dokumentów potwierdzających wybrane uprawnienie przekazywane w request
```http request
GET /nfz/dodatkowe_uprawnienia/typy_dokumentow
Authorization: Bearer {{ token }}
```

niepotrzebne dla ezwm:



### Pokaż listę grup uprawnień

```http request
GET /nfz/dodatkowe_uprawnienia/grupy_uprawnien
Authorization: Bearer {{ token }}
```

### Pokaż listę obszarów stosowania

```http request
GET /nfz/dodatkowe_uprawnienia/obszary_stosowania
Authorization: Bearer {{ token }}
```
