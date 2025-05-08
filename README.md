Ten projekt umożliwia automatyczne pobieranie i aktualizację listy adresów IP należących do Google (np. Google Fonts, gstatic) przy pomocy MikroTika. Dzięki temu możesz dynamicznie tworzyć reguły firewall bazujące na rzeczywistych adresach Google, bez potrzeby ręcznej edycji.
---
### 1. Zainstaluj parser JSON: `mikrotik-json-parser`
W terminalu MikroTika:

```
/tool fetch url="https://raw.githubusercontent.com/Winand/mikrotik-json-parser/master/JParseFunctions" dst-path=JParseFunctions.rsc
/import file-name=JParseFunctions.rsc
```

> Parser zawiera wszystkie niezbędne funkcje w jednym pliku – `JParseFunctions`.

---

### 2. Dodaj skrypt funkcji `fGoogleIpRange`

Utwórz nowy skrypt w **System > Scripts**, nazwij go `fGoogleIpRange` i wklej zawartość z pliku `fGoogleIpRange.script`.

Ten skrypt odpowiada za pobieranie pliku `goog.json`, parsowanie danych i aktualizowanie listy adresowej.

---

### 3. Dodaj wywołujący skrypt `google`

Dodaj nowy skrypt o nazwie `google`:

```
/system script run fGoogleIpRange
:global fGoogleIpRange

$fGoogleIpRange "google" "https://www.gstatic.com/ipranges/goog.json" "https://gist.githubusercontent.com/zarv1k/e1b37a1a5a2c10936fb532302416bed1/raw/google.exceptions.json"
set $fGoogleIpRange
```

> Pierwszy parametr `"google"` oznacza nazwę listy `/ip firewall address-list`.

---

### 4. Uruchomienie testowe

W terminalu MikroTika uruchom ręcznie:

```
/system script run google
```

Sprawdź, czy pojawiły się wpisy w:
```
/ip firewall address-list print where list="google"
```

---

### 5. Dodaj harmonogram (Scheduler)

Aby skrypt był uruchamiany codziennie automatycznie:

```
/system scheduler
add name=update_google_ips interval=1d start-time=04:00:00 on-event="/system script run google"
```

---

### 6. Dodaj regułę firewall (opcjonalnie)

Pozwól na ruch do adresów Google z listy `google`:

```
/ip firewall filter
add chain=forward dst-address-list=google action=accept comment="Allow Google IPs"
```

---

### 7. Lista wyjątków `google.exceptions.json`

Plik wyjątków umożliwia **wyłączenie określonych adresów IP** z listy `google`. Adresy te zostaną dodane jako `disabled` w `address-list`.

#### Przykład zawartości pliku:
```
{
  "8.8.8.8": "Wyklucz DNS",
  "172.217.0.0/16": "Nie zezwalać na Google Fonts"
}
```

Link `raw`, np.:  
`https://raw.githubusercontent.com/jjarosinskium/Googlelist/refs/heads/main/google.exceptions.json`

---

## Autorzy

[@zarv1k](https://gist.github.com/zarv1k) oraz parser JSON [@Winand](https://github.com/Winand/mikrotik-json-parser).
