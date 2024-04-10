
### Szukaj wyrobów rodzaju S

```http request
GET /nfz/wyroby_medyczne/S?search=&page=&limit=
Authorization: Bearer {{ token }}
```

### Szukaj wyrobów rodzaju N

```http request
GET /nfz/wyroby_medyczne/N?search=P.100&page=&limit=
Authorization: Bearer {{ token }}
```

### Szukaj wyrobów dotyczących soczewek

```http request
GET /nfz/wyroby_medyczne/S?search=&page=&limit=&filter=soczewki
Authorization: Bearer {{ token }}
```

### Pokaż kryteria medyczne wg danego wyrobu medycznego

```http request
GET /nfz/wyroby_medyczne/{{kod_wyrobu_medycznego}}/kryteria_medyczne
Authorization: Bearer {{ token }}
```

### Pokaż kryteria medyczne wg wyrobów medycznych dotyczących soczewek

```http request
GET /nfz/wyroby_medyczne/soczewki/kryteria_medyczne
Authorization: Bearer {{ token }}
```

### Szukaj instytucji właściwych UE

> countrycode ISO Alphw 2 (DE, PL, DK,...)

```http request
GET /nfz/instytucje_wlasciwe_ue?search=&countrycode=
Authorization: Bearer {{ token }}
```

### Pokaż listę krajów dla zdefiniowanych instytucji właściwych UE

> the list of countries found in the data

```http request
GET /nfz/instytucje_wlasciwe_ue/kraje
Authorization: Bearer {{ token }}
```

### Szukaj uprawnień dodatkowych

```http request
GET /nfz/dodatkowe_uprawnienia?search=&page=&limit=
Authorization: Bearer {{ token }}
```

### Pokaż listę typów dokumentów

```http request
GET /nfz/dodatkowe_uprawnienia/typy_dokumentow
Authorization: Bearer {{ token }}
```

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