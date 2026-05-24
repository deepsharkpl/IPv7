# IPv7 — autorski projekt adresacji IP i protokołu sieciowego

**Status dokumentu:** projekt koncepcyjny / specyfikacja techniczna
**Nazwa projektu:** IPv7
**Wersja dokumentacji:** 1.5
**Cel:** zaprojektowanie nowoczesnego, bezpiecznego, skalowalnego systemu adresacji i przesyłania pakietów, który może działać równolegle z IPv4 oraz IPv6, a docelowo umożliwiać budowę oddzielnych sieci testowych, prywatnych, laboratoryjnych lub eksperymentalnych.

> Uwaga: nazwa „IPv7” była historycznie używana dla eksperymentalnych propozycji następcy IPv4. Ten dokument opisuje **nowy, autorski projekt IPv7**, niezależny od dawnych propozycji.

---

## Jak czytać ten dokument

Ten plik jest jednocześnie opisem koncepcji, szkicem specyfikacji technicznej i praktycznym przewodnikiem wdrożeniowym. Nie opisuje istniejącego standardu internetowego zatwierdzonego przez IETF. Opisuje projekt, który można rozwijać jako prototyp, symulator, prywatną sieć laboratoryjną albo bazę do dalszej formalnej specyfikacji.

W dokumencie używane są następujące słowa:

| Słowo | Znaczenie |
| ----- | --------- |
| **musi** | wymaganie obowiązkowe; bez tego elementu implementacja nie powinna być uznawana za zgodną z projektem |
| **nie wolno** | zakaz; złamanie tej zasady oznacza błąd bezpieczeństwa, zgodności albo stabilności |
| **powinien** | silne zalecenie; można odstąpić tylko wtedy, gdy istnieje świadomy, udokumentowany powód |
| **zaleca się** | dobra praktyka; nie zawsze obowiązkowa, ale zwykle korzystna operacyjnie |
| **może** | funkcja opcjonalna; implementacja może ją wspierać, ale nie jest to wymagane w profilu minimalnym |

Jeżeli dana funkcja dotyczy bezpieczeństwa, należy zakładać bezpieczne ustawienie domyślne. Oznacza to, że nowo uruchomiony host, router albo brama powinny domyślnie ograniczać ruch, walidować źródła, chronić prywatność i wymagać jawnej konfiguracji dla zachowań bardziej otwartych.

---

## Spis treści skrócony

1. Założenia projektowe
2. Główne cele IPv7
3. Format adresu IPv7
4. Struktura adresu IPv7
5. Klasy adresów IPv7
6. Prefiksy i notacja CIDR w IPv7
7. Przykładowe adresy
8. Nagłówek pakietu IPv7
9. Nagłówki rozszerzeń
10. ICMPv7
11. Autokonfiguracja IPv7
12. DNSv7
13. Routing IPv7
14. Bezpieczeństwo IPv7
15. NAT w IPv7
16. Fragmentacja i MTU
17. Mobilność IPv7
18. QoS i klasy ruchu
19. Telemetria i obserwowalność
20. Zarządzanie adresacją
21. Plan adresacji dla organizacji
22. Polityka bezpieczeństwa dla przykładowej firmy
23. Mechanizmy przejściowe
24. Protokół sąsiedztwa IPv7-ND
25. Wymagania dla urządzeń IPv7
26. Bogony i zakresy niedozwolone
27. Przykładowe reguły firewall IPv7
28. Standardy kodowania adresu
29. Przykładowy algorytm generowania adresu hosta
30. Trust Challenge
31. Obsługa IoT
32. Obsługa data center
33. Obsługa chmury
34. Monitoring i logowanie
35. Procedura wdrożenia IPv7 w organizacji
36. Minimalna implementacja referencyjna
37. Pseudokod parsera adresu IPv7
38. Pseudokod dopasowania prefiksu
39. Ryzyka projektu
40. Zalecenia projektowe
41. Zakres stosowania zabezpieczeń
42. Profile bezpieczeństwa IPv7
43. Macierz zabezpieczeń według warstwy
44. Hardening hosta IPv7
45. Hardening routera IPv7
46. Kryptografia i zarządzanie kluczami
47. Szczegółowy model zagrożeń
48. Bezpieczeństwo DNSv7
49. Bezpieczeństwo DHCPv7 i SLAACv7
50. Bezpieczeństwo routingu i prefiksów
51. Ochrona przed DoS i DDoS
52. Prywatność i minimalizacja danych
53. Wydajność i optymalizacja IPv7
54. Zasady implementacji parserów i serializerów
55. Operacje, utrzymanie i procedury awaryjne
56. Testowanie zgodności IPv7
57. Przykładowe polityki gotowe do użycia
58. Checklisty wdrożeniowe
59. Szczegółowe rozwinięcie punktów specyfikacji
60. Słownik pojęć
61. Podsumowanie
62. Naukowe rozwinięcie punktów 1-61

---

## 1. Założenia projektowe

IPv7 ma być protokołem warstwy sieciowej, czyli odpowiednikiem IPv4/IPv6, odpowiedzialnym za adresowanie urządzeń, enkapsulację danych w pakiety, routowanie, kontrolę integralności metadanych, separację domen sieciowych oraz mechanizmy bezpieczeństwa na poziomie sieci.

Projekt IPv7 zakłada:

1. Bardzo dużą przestrzeń adresową.
2. Czytelniejszą strukturę adresów niż w IPv6.
3. Wbudowany podział adresu na logiczne pola.
4. Obsługę sieci publicznych, prywatnych, lokalnych, laboratoryjnych i izolowanych.
5. Domyślne wsparcie dla kryptograficznej identyfikacji hostów.
6. Mechanizmy antyspoofingowe.
7. Lepszą separację ruchu między użytkownikami, organizacjami i usługami.
8. Możliwość tunelowania przez IPv4 i IPv6.
9. Łatwiejszą automatyczną konfigurację.
10. Możliwość stosowania krótkich aliasów adresów.
11. Obsługę routingu hierarchicznego i segmentowego.
12. Możliwość działania w sieciach IoT, enterprise, data center i operatorów ISP.

IPv7 nie jest projektowany jako prosty „większy IPv4”. Jest projektowany jako protokół, który łączy adresację, identyfikację, polityki bezpieczeństwa i segmentację w jedną spójną architekturę.

---

## 2. Główne cele IPv7

### 2.1. Skalowalność

IPv7 używa adresów o długości **192 bitów**. Taka długość daje ogromną przestrzeń adresową, ale nadal jest prostsza operacyjnie niż ekstremalnie duże adresy 256-bitowe.

Adres 192-bitowy pozwala na podział na logiczne części:

* globalny typ adresu,
* region,
* operatora lub organizację,
* domenę sieciową,
* podsieć,
* hosta,
* identyfikator bezpieczeństwa.

### 2.2. Bezpieczeństwo domyślne

IPv7 zakłada, że sieć nie jest zaufana. Dlatego bezpieczeństwo nie jest dodatkiem, tylko częścią architektury:

* każdy host może posiadać kryptograficzną tożsamość,
* pakiety mogą być podpisywane,
* źródła mogą być weryfikowane,
* routery mogą odrzucać pakiety z niepoprawnym prefiksem,
* ruch może być szyfrowany end-to-end,
* adresy mogą być czasowe, prywatne lub rotowane.

### 2.3. Czytelność operacyjna

IPv7 ma umożliwiać administratorowi szybkie rozpoznanie, do jakiej klasy należy adres. Adres nie powinien być jedynie losowym ciągiem znaków.

### 2.4. Kompatybilność przejściowa

IPv7 powinien obsługiwać:

* tunelowanie IPv7-over-IPv6,
* tunelowanie IPv7-over-IPv4,
* translację adresów IPv7 ↔ IPv6,
* bramy aplikacyjne,
* dual-stack IPv6/IPv7,
* prywatne sieci laboratoryjne.

---

## 3. Format adresu IPv7

### 3.1. Długość adresu

Adres IPv7 ma długość:

```text
192 bity = 24 bajty
```

Jest zapisywany w 8 blokach po 24 bity albo w 12 blokach po 16 bitów. Zalecany format tekstowy to 12 bloków szesnastkowych po 4 znaki.

Przykład pełnego adresu:

```text
7a01:0001:00ab:0010:0000:0000:04f2:91aa:0000:0000:abcd:0123
```

Przykład skrócony:

```text
7a01:1:ab:10::4f2:91aa:0:0:abcd:123
```

Zasady skracania są podobne do IPv6:

* zera wiodące w bloku można pominąć,
* jeden ciąg bloków zerowych można zastąpić `::`,
* `::` może wystąpić tylko raz w adresie,
* wielkość liter nie ma znaczenia,
* zalecany zapis używa małych liter.

---

## 4. Struktura adresu IPv7

Adres IPv7 dzieli się na pola logiczne:

```text
| VER | CLS | GEO | ORG | NET | SUB | HOST | SEC |
```

Proponowany podział bitowy:

| Pole |  Długość | Znaczenie                                     |
| ---- | -------: | --------------------------------------------- |
| VER  |  8 bitów | identyfikator wersji i rodziny adresu         |
| CLS  |  8 bitów | klasa adresu                                  |
| GEO  |  24 bity | region geograficzny lub logiczny              |
| ORG  |  32 bity | organizacja, operator lub właściciel prefiksu |
| NET  |  32 bity | domena sieciowa                               |
| SUB  |  24 bity | podsieć lub segment                           |
| HOST | 48 bitów | identyfikator hosta/interfejsu                |
| SEC  | 16 bitów | skrót bezpieczeństwa lub profil polityki      |

Razem:

```text
8 + 8 + 24 + 32 + 32 + 24 + 48 + 16 = 192 bity
```

### Jak czytać adres od lewej do prawej

Adres IPv7 należy czytać jak uporządkowaną informację, a nie jak losowy identyfikator. Pierwsze pola mówią, z jaką rodziną i klasą adresu mamy do czynienia. Kolejne pola zawężają lokalizację logiczną: region, organizację, sieć i segment. Dopiero końcowe pola wskazują konkretnego hosta oraz profil bezpieczeństwa.

Prosty opis pól:

| Pole | Pytanie, na które odpowiada |
| ---- | --------------------------- |
| VER | czy to jest adres z rodziny IPv7? |
| CLS | jaki to typ adresu: publiczny, prywatny, lokalny, testowy, usługowy? |
| GEO | gdzie logicznie lub geograficznie znajduje się prefiks? |
| ORG | do jakiej organizacji albo operatora należy prefiks? |
| NET | do której domeny sieciowej należy adres? |
| SUB | do którego segmentu bezpieczeństwa albo podsieci należy host? |
| HOST | który host, interfejs, kontener albo punkt usługi jest adresowany? |
| SEC | jaki profil bezpieczeństwa, skrót albo znacznik polityki jest powiązany z adresem? |

Najważniejsza zasada operacyjna: im bardziej na lewo znajduje się pole, tym bardziej powinno być stabilne i użyteczne dla routingu. Im bardziej na prawo znajduje się pole, tym częściej może być zależne od hosta, prywatności, rotacji i polityki bezpieczeństwa.

### 4.1. Pole VER

Pole `VER` identyfikuje adres jako należący do rodziny IPv7.

Zalecana wartość:

```text
0x7a
```

Przykład:

```text
7a01:0001:00ab:0010:0000:0000:04f2:91aa:0000:0000:abcd:0123
^^^^
część pola VER/CLS w zapisie tekstowym
```

### 4.2. Pole CLS — klasa adresu

Pole `CLS` określa typ adresu.

|  CLS | Nazwa           | Przeznaczenie                         |
| ---: | --------------- | ------------------------------------- |
| 0x00 | Reserved        | zarezerwowane                         |
| 0x01 | Global Unicast  | publiczne adresy globalne             |
| 0x02 | Private Realm   | prywatne adresy organizacji           |
| 0x03 | Local Link      | adresy lokalnego łącza                |
| 0x04 | Loopback        | adresy pętli zwrotnej                 |
| 0x05 | Multicast       | transmisja do grupy                   |
| 0x06 | Anycast         | najbliższy z wielu odbiorców          |
| 0x07 | Servicecast     | adresowanie usługi, nie hosta         |
| 0x08 | Ephemeral       | adresy tymczasowe                     |
| 0x09 | Secure Identity | adresy powiązane z kluczem publicznym |
| 0x0a | Lab/Test        | sieci testowe i dokumentacyjne        |
| 0xff | Experimental    | eksperymenty                          |

### 4.3. Pole GEO

Pole `GEO` może oznaczać region geograficzny albo logiczny. W sieciach prywatnych może być użyte jako identyfikator lokalizacji, data center, kampusu albo kraju.

Przykładowy podział:

```text
GEO = 24 bity
8 bitów  — kontynent lub region główny
8 bitów  — kraj / obszar
8 bitów  — miasto / strefa / lokalizacja
```

Przykład:

```text
01:PL:WA
```

W zapisie binarnym używa się oczywiście wartości liczbowych, ale dokumentacja administracyjna może przechowywać mapowanie kodów.

### 4.4. Pole ORG

`ORG` identyfikuje organizację, operatora, dostawcę usług, firmę, instytucję lub właściciela prefiksu.

W sieciach publicznych pole powinno być przydzielane przez rejestr IPv7. W sieciach prywatnych można używać własnej numeracji.

### 4.5. Pole NET

`NET` określa domenę sieciową w ramach organizacji. Może odpowiadać:

* oddziałowi firmy,
* regionowi chmurowemu,
* środowisku produkcyjnemu,
* środowisku testowemu,
* sieci użytkowników,
* sieci serwerów,
* sieci zarządzania,
* sieci IoT,
* sieci DMZ.

### 4.6. Pole SUB

`SUB` oznacza podsieć lub segment bezpieczeństwa. To pole jest przeznaczone do routingu wewnętrznego oraz mikrosegmentacji.

Przykładowy podział w firmie:

|    SUB | Segment              |
| -----: | -------------------- |
| 000001 | użytkownicy biurowi  |
| 000002 | serwery aplikacyjne  |
| 000003 | bazy danych          |
| 000004 | monitoring           |
| 000005 | sieć administracyjna |
| 000006 | IoT                  |
| 000007 | goście               |
| 000008 | backup               |

### 4.7. Pole HOST

`HOST` identyfikuje hosta lub interfejs. Ma 48 bitów, czyli tyle samo co klasyczny adres MAC, ale nie musi być z nim tożsamy.

Możliwe tryby generowania HOST:

1. Z adresu MAC.
2. Losowo.
3. Z klucza publicznego.
4. Z identyfikatora urządzenia.
5. Z systemu DHCPv7.
6. Z kontrolera SDN.
7. Czasowo, z rotacją.

Zalecenie bezpieczeństwa: nie używać bezpośrednio adresu MAC w publicznych adresach IPv7, ponieważ może to ujawniać informacje o urządzeniu.

### 4.8. Pole SEC

`SEC` to 16-bitowy identyfikator profilu bezpieczeństwa albo skrócony znacznik integralności adresu.

Możliwe zastosowania:

* profil polityki firewall,
* tryb szyfrowania,
* typ tożsamości hosta,
* poziom zaufania,
* oznaczenie adresu tymczasowego,
* skrót z klucza publicznego,
* wykrywanie błędów konfiguracji.

SEC nie zastępuje kryptografii. To pole jest pomocnicze i nie powinno być traktowane jako pełny podpis cyfrowy.

---

## 5. Klasy adresów IPv7

### 5.1. Global Unicast

Global Unicast służy do komunikacji publicznej w Internecie IPv7.

Przykład prefiksu:

```text
7a01::/16
```

Zastosowania:

* serwery publiczne,
* usługi chmurowe,
* operatorzy ISP,
* sieci firmowe z publicznym routingiem,
* komunikacja między organizacjami.

### 5.2. Private Realm

Private Realm to odpowiednik prywatnych zakresów IPv4, ale większy i uporządkowany.

Przykładowy prefiks:

```text
7a02::/16
```

Adresy Private Realm nie są routowane globalnie.

Zastosowania:

* LAN,
* sieci domowe,
* firmy,
* laboratoria,
* środowiska testowe,
* klastry lokalne,
* sieci offline.

### 5.3. Local Link

Local Link działa tylko w obrębie jednego segmentu warstwy drugiej.

Przykład:

```text
7a03::/16
```

Użycie:

* autokonfiguracja,
* discovery sąsiadów,
* komunikacja bez routera,
* uruchamianie urządzeń bez serwera DHCPv7.

### 5.4. Loopback

Loopback to adres lokalny hosta.

Przykład głównego loopbacka:

```text
7a04::1
```

### 5.5. Multicast

Multicast służy do komunikacji jeden-do-wielu.

Przykład:

```text
7a05::/16
```

Zakresy multicast:

| Zakres     | Opis                  |
| ---------- | --------------------- |
| node-local | tylko wewnątrz hosta  |
| link-local | w obrębie łącza       |
| site-local | w obrębie lokalizacji |
| org-local  | w obrębie organizacji |
| global     | globalnie             |

### 5.6. Anycast

Anycast pozwala, aby wiele węzłów używało tego samego adresu, a routing prowadził do najbliższego lub najlepszego węzła.

Zastosowania:

* DNS,
* CDN,
* bramy API,
* serwery czasu,
* punkty wejścia VPN,
* usługi bezpieczeństwa.

### 5.7. Servicecast

Servicecast to rozszerzenie koncepcji anycast. Adres wskazuje nie konkretny host, ale usługę.

Przykład:

```text
7a07:0001:0000:0000:0000:0000:0000:0053
```

Może oznaczać usługę DNS w danym realmie.

Servicecast wymaga katalogu usług IPv7-SD albo integracji z DNSv7.

### 5.8. Ephemeral

Adresy Ephemeral są tymczasowe i mogą rotować automatycznie.

Zastosowania:

* prywatność użytkowników,
* urządzenia mobilne,
* połączenia jednorazowe,
* sesje VPN,
* krótkotrwałe kontenery,
* funkcje serverless.

### 5.9. Secure Identity

Adres Secure Identity jest powiązany z kluczem publicznym hosta.

Idea:

```text
HOST = skrót_klucza_publicznego[0..47]
SEC  = skrót_klucza_publicznego[48..63]
```

Dzięki temu odbiorca może sprawdzić, czy host faktycznie posiada klucz odpowiadający adresowi.

### 5.10. Lab/Test

Adresy Lab/Test służą do dokumentacji, przykładów i laboratoriów.

Przykład:

```text
7a0a::/16
```

Nie powinny być routowane w sieciach produkcyjnych.

---

## 6. Prefiksy i notacja CIDR w IPv7

IPv7 używa notacji prefiksowej podobnej do IPv4/IPv6.

Przykład:

```text
7a01:1:ab:10::/64
```

Ponieważ adres ma 192 bity, długość prefiksu może wynosić od `/0` do `/192`.

Typowe długości prefiksów:

| Prefiks | Zastosowanie    |
| ------: | --------------- |
|      /8 | rodzina IPv7    |
|     /16 | klasa adresu    |
|     /40 | region          |
|     /72 | organizacja     |
|    /104 | domena sieciowa |
|    /128 | podsieć         |
|    /176 | host bez SEC    |
|    /192 | pełny adres     |

Zalecany podział dla organizacji:

```text
/72   — prefiks organizacji
/104  — sieć logiczna
/128  — podsieć
/176  — host
/192  — pełny adres z profilem SEC
```

---

## 7. Przykładowe adresy

### 7.1. Adres publicznego serwera WWW

```text
7a01:0101:0001:0042:0000:0001:0000:0001:0000:0000:0000:0080
```

Interpretacja:

* `7a` — IPv7,
* `01` — Global Unicast,
* region `010001`,
* organizacja `00420000`,
* sieć `00010000`,
* host `000000000080`,
* SEC `0080`.

### 7.2. Adres prywatnego laptopa

```text
7a02:0000:0000:0001:0000:1000:0000:0007:1b2c:3d4e:5f60:0101
```

### 7.3. Adres loopback

```text
7a04::1
```

### 7.4. Adres testowy do dokumentacji

```text
7a0a:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0001
```

### 7.5. Adres usługi DNS Servicecast

```text
7a07::53
```

---

## 8. Nagłówek pakietu IPv7

### 8.1. Założenia

Nagłówek IPv7 powinien być stałej długości w części podstawowej, aby routery mogły szybko przetwarzać pakiety. Opcje rozszerzone powinny znajdować się w osobnych nagłówkach rozszerzeń.

### 8.2. Podstawowy nagłówek IPv7

Proponowana długość podstawowego nagłówka: **80 bajtów**.

Stałe 80 bajtów obejmuje pola funkcjonalne oraz pole rezerwowe/wyrównujące. Bez pola rezerwowego widoczna część funkcjonalna ma 68 bajtów. Rezerwa 12 bajtów utrzymuje stały rozmiar nagłówka, ułatwia wyrównanie pamięci i zostawia miejsce na przyszłe flagi bez zmiany formatu bazowego.

| Pole                |  Długość | Opis                                     |
| ------------------- | -------: | ---------------------------------------- |
| Version             |   4 bity | wersja protokołu, wartość 7              |
| Traffic Class       |  8 bitów | klasa ruchu/QoS                          |
| Flow ID             | 20 bitów | identyfikator przepływu                  |
| Payload Length      |  32 bity | długość danych po nagłówku               |
| Next Header         |  8 bitów | typ następnego nagłówka                  |
| Hop Limit           |  8 bitów | limit przeskoków                         |
| Security Flags      | 16 bitów | flagi bezpieczeństwa                     |
| Reserved/Alignment  |  96 bitów | rezerwa, wyrównanie i przyszłe użycie    |
| Source Address      | 192 bity | adres źródłowy                           |
| Destination Address | 192 bity | adres docelowy                           |
| Header MAC          |  64 bity | skrót integralności nagłówka, opcjonalny |

Schemat:

```text
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Ver| Traffic  |              Flow ID                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         Payload Length                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Next Header  | Hop Limit    |        Security Flags            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                    Reserved / Alignment 96b                  |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                       Source Address 192b                     |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                    Destination Address 192b                   |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         Header MAC 64b                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### 8.3. Version

Wartość pola Version:

```text
7
```

### 8.4. Traffic Class

Pole służy do oznaczania jakości usług:

* ruch zwykły,
* VoIP,
* streaming,
* sterowanie przemysłowe,
* monitoring,
* krytyczna infrastruktura,
* backup,
* ruch niskiego priorytetu.

### 8.5. Flow ID

Flow ID identyfikuje przepływ między hostami. Routery mogą używać go do:

* load balancingu,
* QoS,
* telemetrii,
* wykrywania anomalii,
* szybkiego klasyfikowania pakietów.

### 8.6. Payload Length

32 bity pozwalają opisać bardzo duże pakiety, ale realny MTU nadal powinien być ograniczony przez warstwę drugą i politykę sieci.

### 8.7. Next Header

Next Header wskazuje, jaki protokół lub nagłówek występuje dalej:

| Kod | Znaczenie                    |
| --: | ---------------------------- |
|   0 | brak następnego nagłówka     |
|   6 | TCP                          |
|  17 | UDP                          |
|  58 | ICMPv7                       |
| 100 | nagłówek routingu IPv7       |
| 101 | nagłówek fragmentacji IPv7   |
| 102 | nagłówek bezpieczeństwa IPv7 |
| 103 | nagłówek mobilności          |
| 104 | nagłówek telemetryczny       |

### 8.8. Hop Limit

Odpowiednik TTL. Każdy router zmniejsza wartość o 1. Gdy osiągnie 0, pakiet jest odrzucany.

### 8.9. Security Flags

Przykładowe flagi:

| Bit | Nazwa       | Znaczenie                            |
| --: | ----------- | ------------------------------------ |
|   0 | SIG         | pakiet zawiera podpis                |
|   1 | ENC         | payload jest szyfrowany              |
|   2 | AID         | adres powiązany z tożsamością        |
|   3 | TEMP        | adres tymczasowy                     |
|   4 | NO-FWD      | nie przekazywać poza lokalny segment |
|   5 | TRACE-BLOCK | blokada śledzenia trasy              |
|   6 | ROUTE-PIN   | wymuszona ścieżka                    |
|   7 | ZERO-TRUST  | wymagaj pełnej weryfikacji           |

---

## 9. Nagłówki rozszerzeń

IPv7 obsługuje nagłówki rozszerzeń, aby podstawowy nagłówek pozostał prosty.

### 9.1. Security Extension Header

Zawiera:

* identyfikator algorytmu,
* identyfikator klucza,
* nonce,
* podpis,
* znacznik uwierzytelnienia,
* informacje o szyfrowaniu.

### 9.2. Routing Extension Header

Pozwala określić dodatkowe informacje routingu:

* segment routing,
* lista punktów pośrednich,
* wymagana domena zaufania,
* routing przez usługę bezpieczeństwa,
* routing przez bramę zgodności.

### 9.3. Fragmentation Extension Header

Fragmentacja jest dozwolona tylko na hoście źródłowym. Routery pośrednie nie powinny fragmentować pakietów.

### 9.4. Mobility Extension Header

Dla urządzeń mobilnych:

* identyfikator stały,
* adres tymczasowy,
* token mobilności,
* czas ważności,
* poprzednia lokalizacja.

### 9.5. Telemetry Extension Header

Opcjonalny nagłówek do observability:

* identyfikator domeny,
* znacznik czasu,
* klasa usługi,
* identyfikator ścieżki,
* licznik przeskoków,
* znacznik polityki.

Telemetry Header powinien być wyłączony dla ruchu prywatnego, chyba że polityka organizacji stanowi inaczej.

---

## 10. ICMPv7

IPv7 wymaga własnego protokołu kontrolnego ICMPv7.

### 10.1. Zadania ICMPv7

ICMPv7 obsługuje:

* komunikaty błędów,
* diagnostykę,
* discovery sąsiadów,
* autokonfigurację,
* wykrywanie MTU ścieżki,
* informację o braku trasy,
* kontrolę multicast,
* wykrywanie konfliktu adresu.

### 10.2. Typy komunikatów ICMPv7

| Typ | Nazwa                   | Opis                     |
| --: | ----------------------- | ------------------------ |
|   1 | Destination Unreachable | cel nieosiągalny         |
|   2 | Packet Too Big          | pakiet za duży           |
|   3 | Time Exceeded           | przekroczony Hop Limit   |
|   4 | Parameter Problem       | błąd pola nagłówka       |
| 128 | Echo Request            | ping                     |
| 129 | Echo Reply              | odpowiedź ping           |
| 130 | Neighbor Query          | zapytanie o sąsiada      |
| 131 | Neighbor Reply          | odpowiedź sąsiada        |
| 132 | Router Advertise        | ogłoszenie routera       |
| 133 | Router Solicit          | prośba o router          |
| 134 | Address Conflict        | konflikt adresu          |
| 135 | Trust Challenge         | wyzwanie bezpieczeństwa  |
| 136 | Trust Response          | odpowiedź bezpieczeństwa |

---

## 11. Autokonfiguracja IPv7

IPv7 obsługuje trzy modele konfiguracji:

1. SLAACv7 — samodzielna konfiguracja adresu.
2. DHCPv7 — centralna konfiguracja.
3. SDN-Controlled Addressing — konfiguracja przez kontroler sieci.

### 11.1. SLAACv7

Host otrzymuje Router Advertisement, który zawiera:

* prefiks sieci,
* długość prefiksu,
* czas ważności,
* flagi bezpieczeństwa,
* adres routera,
* politykę DNS,
* politykę prywatności.

Następnie host generuje część HOST.

Tryby generowania HOST:

```text
random
stable-private
public-key-derived
mac-derived
controller-assigned
```

Zalecane tryby:

* dla użytkowników: `stable-private` lub `random`,
* dla serwerów: `controller-assigned`,
* dla systemów zero-trust: `public-key-derived`,
* dla IoT: `controller-assigned`.

### 11.2. DHCPv7

DHCPv7 przydziela:

* adres IPv7,
* prefiks,
* DNS,
* NTP,
* politykę bezpieczeństwa,
* profil firewall,
* certyfikat hosta,
* czas dzierżawy,
* klasę urządzenia,
* reguły rotacji adresu.

### 11.3. SDN-Controlled Addressing

W dużych środowiskach adresy mogą być nadawane przez kontroler SDN.

Kontroler może wymuszać:

* segment użytkownika,
* VLAN/VXLAN,
* profil Zero Trust,
* ACL,
* routing,
* ścieżkę przez firewall,
* izolację mikrosegmentów.

---

## 12. DNSv7

IPv7 wymaga rozszerzenia DNS lub nowego typu rekordu.

### 12.1. Rekord A7

Proponowany rekord DNS dla IPv7:

```text
A7 example.com 7a01:1:ab:10::80
```

### 12.2. Rekord PTR7

Odwrotne mapowanie adresów:

```text
PTR7 80.0.0.0....7a.ip7.arpa example.com
```

### 12.3. Rekord SVC7

Rekord usługi IPv7:

```text
SVC7 _https._tcp.example.com service=web priority=10 target=7a07::443
```

### 12.4. DNSSEC obowiązkowy dla stref publicznych

Dla publicznego IPv7 zaleca się obowiązkowe DNSSEC albo równoważny mechanizm podpisywania rekordów.

---

## 13. Routing IPv7

### 13.1. Routing globalny

Routing globalny powinien być hierarchiczny:

```text
VER/CLS → GEO → ORG → NET → SUB → HOST
```

Dzięki temu tablice routingu mogą agregować prefiksy.

### 13.2. Prefiksy operatorów

Operatorzy powinni otrzymywać duże prefiksy, np.:

```text
/40, /48, /56 lub /72
```

Organizacje końcowe mogą otrzymywać:

```text
/72 lub /80
```

### 13.3. Routing wewnętrzny

Wewnątrz organizacji można używać:

* OSPFv7,
* IS-ISv7,
* BGPv7,
* routingu SDN,
* segment routingu,
* statycznych tras.

### 13.4. BGPv7

BGPv7 musi obsługiwać:

* adresy 192-bitowe,
* agregację prefiksów,
* filtrowanie po klasie adresu,
* podpisywanie ogłoszeń,
* walidację pochodzenia trasy,
* polityki anty-hijacking,
* RPKI-v7.

### 13.5. RPKI-v7

RPKI-v7 wiąże prefiks IPv7 z uprawnionym systemem autonomicznym.

Elementy:

* certyfikat prefiksu,
* Route Origin Authorization,
* podpis operatora,
* czas ważności,
* lista dozwolonych AS,
* status wycofania.

Router powinien odrzucać ogłoszenia:

* bez ważnego podpisu,
* od nieuprawnionego AS,
* z prefiksem spoza przydziału,
* z podejrzanie długim prefiksem,
* łamiące politykę organizacji.

---

## 14. Bezpieczeństwo IPv7

### 14.1. Model zagrożeń

IPv7 musi chronić przed:

* spoofingiem adresów,
* podszywaniem się pod router,
* przejęciem prefiksu,
* atakami MITM,
* skanowaniem sieci,
* śledzeniem użytkowników,
* atakami replay,
* fragmentacją złośliwą,
* fałszywym DHCPv7,
* fałszywym DNS,
* nieautoryzowanym routingiem,
* wyciekiem danych telemetrycznych,
* atakami DoS/DDoS.

### 14.2. Antyspoofing

Router brzegowy musi weryfikować, czy adres źródłowy należy do prefiksu, z którego pakiet faktycznie pochodzi.

Reguła:

```text
Jeżeli źródłowy adres IPv7 nie pasuje do przypisanego prefiksu interfejsu, pakiet odrzuć.
```

### 14.3. Source Address Validation

Każdy router dostępowy powinien mieć włączoną walidację źródła.

Tryby:

* strict,
* loose,
* policy-based,
* cryptographic.

### 14.4. Podpisywanie pakietów

Pakiety mogą zawierać podpis w Security Extension Header.

Podpis obejmuje:

* adres źródłowy,
* adres docelowy,
* Flow ID,
* Hop Limit początkowy,
* payload hash,
* nonce,
* timestamp.

Nie należy podpisywać pól zmienianych przez routery, chyba że istnieje mechanizm normalizacji.

### 14.5. Szyfrowanie

IPv7 powinien obsługiwać szyfrowanie na poziomie sieci, ale nie powinien zastępować TLS na poziomie aplikacji.

Tryby:

* host-to-host,
* site-to-site,
* service-to-service,
* mesh encryption,
* opportunistic encryption.

### 14.6. Ochrona prywatności

Adresy stałe mogą śledzić użytkownika. Dlatego dla urządzeń końcowych zaleca się:

* adresy tymczasowe,
* rotację HOST,
* ukrywanie identyfikatora urządzenia,
* unikanie MAC-derived addressing,
* minimalizację telemetrii,
* osobne adresy dla różnych usług,
* separację adresu tożsamości od adresu lokalizacji.

### 14.7. Secure Neighbor Discovery

Neighbor Discovery musi być zabezpieczony.

Mechanizmy:

* podpisane ogłoszenia routerów,
* Trust Challenge/Response,
* kontrola duplikatów,
* lista zaufanych routerów,
* certyfikaty lokalne,
* rate limiting.

### 14.8. Ochrona przed skanowaniem

IPv7 ma dużą przestrzeń adresową, ale samo to nie wystarcza.

Zalecenia:

* losowe HOST ID,
* firewall domyślnie deny inbound,
* service discovery tylko dla autoryzowanych klientów,
* rate limiting ICMPv7,
* wykrywanie wzorców skanowania,
* adresy usług przez Servicecast zamiast bezpośredniego ujawniania hostów.

### 14.9. Firewall IPv7

Firewall powinien filtrować po:

* klasie adresu,
* prefiksie,
* GEO,
* ORG,
* NET,
* SUB,
* SEC,
* Flow ID,
* Next Header,
* stanie połączenia,
* tożsamości kryptograficznej,
* reputacji prefiksu.

Przykładowa reguła:

```text
allow src.cls=PrivateRealm src.sub=Users dst.sub=WebServices proto=tcp dst.port=443 identity=valid
 deny src.cls=PrivateRealm dst.sub=Databases proto=any unless identity.group=DBAdmins
```

### 14.10. Zero Trust w IPv7

IPv7 może wspierać Zero Trust przez:

* adresy powiązane z tożsamością,
* mikrosegmentację,
* polityki SUB/SEC,
* obowiązkowe uwierzytelnienie hosta,
* krótkie czasy ważności adresów,
* rotację kluczy,
* ciągłą ocenę ryzyka.

---

## 15. NAT w IPv7

IPv7 nie wymaga NAT do oszczędzania adresów, ponieważ przestrzeń adresowa jest ogromna.

Dopuszczalne formy translacji:

* NAT7 dla środowisk prywatnych,
* NPT7, czyli translacja prefiksów,
* IPv7-to-IPv6 gateway,
* IPv7-to-IPv4 gateway,
* translacja usługowa na poziomie aplikacji.

Zalecenie: unikać klasycznego NAT ukrywającego wielu hostów za jednym adresem, ponieważ utrudnia bezpieczeństwo end-to-end.

---

## 16. Fragmentacja i MTU

IPv7 stosuje zasadę:

```text
Routery nie fragmentują pakietów.
```

Jeśli pakiet jest zbyt duży, router odsyła ICMPv7 Packet Too Big.

Host źródłowy powinien wykonać Path MTU Discovery.

Minimalny MTU dla IPv7:

```text
1280 bajtów
```

Zalecany MTU w sieciach data center:

```text
9000 bajtów
```

---

## 17. Mobilność IPv7

IPv7 rozróżnia:

* identyfikator hosta,
* adres lokalizacji,
* adres tymczasowy,
* adres usługi.

Dzięki temu urządzenie mobilne może zmieniać sieć bez utraty tożsamości.

Mechanizm:

1. Host posiada stałą tożsamość kryptograficzną.
2. W nowej sieci otrzymuje adres lokalizacji.
3. Aktualizuje Mobility Registry.
4. Aktywne sesje mogą być przekierowane.
5. Stare adresy wygasają po czasie przejściowym.

---

## 18. QoS i klasy ruchu

Traffic Class może zawierać następujące klasy:

| Klasa | Nazwa        | Zastosowanie             |
| ----: | ------------ | ------------------------ |
|     0 | Best Effort  | zwykły ruch              |
|     1 | Low Priority | backup, aktualizacje     |
|     2 | Interactive  | SSH, RDP, terminale      |
|     3 | Streaming    | audio/wideo              |
|     4 | Voice        | VoIP                     |
|     5 | Control      | routing, zarządzanie     |
|     6 | Critical     | infrastruktura krytyczna |
|     7 | Emergency    | awaryjne sterowanie      |

Routery mogą stosować kolejki priorytetowe, shaping i policing.

---

## 19. Telemetria i obserwowalność

IPv7 powinien wspierać nowoczesne operacje sieciowe:

* flow logs,
* metryki opóźnień,
* identyfikację ścieżki,
* wykrywanie anomalii,
* korelację z tożsamością,
* eksport do SIEM,
* eksport do systemów NDR.

### 19.1. Minimalizacja danych

Telemetria nie może naruszać prywatności. Należy zbierać tylko dane potrzebne do bezpieczeństwa i utrzymania sieci.

### 19.2. Ochrona telemetryczna

Logi powinny być:

* podpisywane,
* chronione przed modyfikacją,
* retencjonowane zgodnie z polityką,
* pseudonimizowane tam, gdzie to możliwe.

---

## 20. Zarządzanie adresacją

### 20.1. Rejestr centralny

Dla publicznego IPv7 potrzebny jest rejestr prefiksów.

Rejestr przechowuje:

* przydział ORG,
* prefiksy operatorów,
* dane kontaktowe,
* certyfikaty RPKI-v7,
* polityki routingu,
* status aktywności,
* historię zmian.

### 20.2. IPAM dla IPv7

Organizacja powinna używać systemu IPAM obsługującego:

* adresy 192-bitowe,
* prefiksy IPv7,
* klasy adresów,
* segmenty SUB,
* profile SEC,
* integrację z DNSv7,
* integrację z DHCPv7,
* audyt zmian,
* automatyzację API.

### 20.3. Konwencja nazewnictwa

Przykład:

```text
REGION-ORG-NET-SUB-HOST
EU-ACME-PROD-WEB-001
EU-ACME-PROD-DB-001
EU-ACME-DEV-API-014
```

---

## 21. Plan adresacji dla organizacji — przykład

Organizacja otrzymuje prefiks:

```text
7a01:0101:00ac:me00::/72
```

Przykładowy podział:

| Środowisko | Prefiks                          |
| ---------- | -------------------------------- |
| Produkcja  | `7a01:0101:00ac:me00:0001::/104` |
| Test       | `7a01:0101:00ac:me00:0002::/104` |
| Dev        | `7a01:0101:00ac:me00:0003::/104` |
| Management | `7a01:0101:00ac:me00:0004::/104` |
| Backup     | `7a01:0101:00ac:me00:0005::/104` |
| IoT        | `7a01:0101:00ac:me00:0006::/104` |
| Goście     | `7a01:0101:00ac:me00:0007::/104` |

Uwaga: `me00` jest przykładem czytelnej notacji administracyjnej; w realnym adresie musi być wartością szesnastkową.

---

## 22. Polityka bezpieczeństwa dla przykładowej firmy

### 22.1. Segmenty

| Segment |    SUB | Dostęp                      |
| ------- | -----: | --------------------------- |
| Users   | 000001 | Internet, wybrane aplikacje |
| Admin   | 000002 | zarządzanie systemami       |
| Web     | 000003 | publiczny HTTPS             |
| App     | 000004 | tylko z Web i Admin         |
| DB      | 000005 | tylko z App i Admin         |
| Backup  | 000006 | tylko z serwerów            |
| IoT     | 000007 | tylko broker IoT            |
| Guest   | 000008 | tylko Internet              |

### 22.2. Domyślne zasady

1. Ruch między segmentami jest domyślnie blokowany.
2. Dostęp administracyjny wymaga tożsamości kryptograficznej.
3. Serwery baz danych nie mają bezpośredniego dostępu do Internetu.
4. IoT nie może inicjować połączeń do segmentów użytkowników.
5. Goście nie mogą komunikować się z siecią firmową.
6. Wszystkie prefiksy są logowane w SIEM.
7. ICMPv7 jest ograniczony, ale nie całkowicie blokowany.
8. Router Advertisements są akceptowane tylko od zaufanych routerów.
9. DHCPv7 działa tylko z autoryzowanych serwerów.
10. DNSv7 wymaga walidacji podpisów dla stref publicznych.

---

## 23. Mechanizmy przejściowe

### 23.1. Dual-stack IPv6/IPv7

Host posiada równocześnie adres IPv6 i IPv7.

Zalety:

* proste wdrożenie,
* brak translacji,
* możliwość stopniowej migracji.

Wady:

* dwie polityki bezpieczeństwa,
* większa złożoność monitoringu,
* ryzyko niespójnych reguł firewall.

### 23.2. IPv7-over-IPv6

Pakiet IPv7 jest przenoszony w payload IPv6.

Zastosowanie:

* testy,
* połączenia między lokalizacjami,
* sieci operatorów,
* VPN.

### 23.3. IPv7-over-IPv4

Pakiet IPv7 jest tunelowany przez IPv4.

Zastosowanie:

* laboratoria,
* środowiska bez IPv6,
* przejściowe połączenia.

### 23.4. Brama IPv7 ↔ IPv6

Brama wykonuje translację adresów i protokołów.

Wymagania:

* mapowanie DNS A7 ↔ AAAA,
* translacja ICMPv7 ↔ ICMPv6,
* zachowanie logów,
* kontrola bezpieczeństwa,
* obsługa MTU.

### 23.5. Brama IPv7 ↔ IPv4

Bardziej skomplikowana, ponieważ IPv4 ma znacznie mniejszą przestrzeń adresową.

Najlepsze podejście:

* translacja na poziomie usług,
* reverse proxy,
* load balancer,
* NAT brzegowy,
* brama aplikacyjna.

---

## 24. Protokół sąsiedztwa IPv7-ND

IPv7-ND zastępuje ARP i rozszerza funkcje Neighbor Discovery.

### 24.1. Funkcje

* wykrywanie adresu warstwy drugiej,
* wykrywanie routerów,
* wykrywanie prefiksów,
* wykrywanie konfliktów,
* weryfikacja tożsamości routera,
* ochrona przed spoofingiem,
* wykrywanie usług lokalnych.

### 24.2. Bezpieczne ogłoszenia routerów

Router Advertisement powinien zawierać:

* prefiks,
* czas ważności,
* klucz publiczny routera,
* podpis,
* numer sekwencyjny,
* politykę hostów,
* listę dozwolonych klas adresów.

Host akceptuje RA tylko wtedy, gdy:

* podpis jest poprawny,
* router znajduje się na liście zaufanych,
* prefiks jest zgodny z polityką,
* komunikat nie jest przestarzały.

---

## 25. Wymagania dla urządzeń IPv7

### 25.1. Host IPv7 musi obsługiwać

* adresy 192-bitowe,
* ICMPv7,
* SLAACv7,
* DHCPv7 client,
* Path MTU Discovery,
* podstawowe nagłówki rozszerzeń,
* walidację RA,
* firewall lokalny,
* adresy tymczasowe,
* DNS A7.

### 25.2. Router IPv7 musi obsługiwać

* routing prefiksów IPv7,
* ICMPv7 errors,
* Hop Limit,
* filtrowanie prefiksów,
* antyspoofing,
* ACL IPv7,
* agregację tras,
* logowanie przepływów,
* kontrolę RA,
* rate limiting.

### 25.3. Router brzegowy powinien obsługiwać

* BGPv7,
* RPKI-v7,
* filtrowanie bogonów,
* DDoS filtering,
* tunelowanie IPv7,
* translację IPv7/IPv6,
* telemetry export,
* polityki Zero Trust.

---

## 26. Bogony i zakresy niedozwolone

Bogony to adresy, które nie powinny pojawiać się w publicznym routingu.

Przykłady:

* Reserved,
* Private Realm,
* Local Link,
* Loopback,
* Lab/Test,
* Experimental,
* nieprzydzielone prefiksy,
* wycofane prefiksy,
* prefiksy z błędną walidacją RPKI-v7.

Routery brzegowe powinny automatycznie aktualizować listy bogonów.

---

## 27. Przykładowe reguły firewall IPv7

### 27.1. Blokada ruchu prywatnego z Internetu

```text
deny src=7a02::/16 in=wan
```

### 27.2. Zezwolenie na HTTPS do serwerów WWW

```text
allow dst.sub=WEB proto=tcp dst.port=443 state=new,established
```

### 27.3. Blokada dostępu gości do sieci firmowej

```text
deny src.sub=GUEST dst.org=LOCAL_ORG
allow src.sub=GUEST dst.cls=GlobalUnicast proto=tcp dst.port=80,443
```

### 27.4. Dostęp administratorów do zarządzania

```text
allow src.identity.group=Admins dst.sub=MGMT proto=tcp dst.port=22,443 require=signed
```

### 27.5. Ochrona ICMPv7

```text
allow icmpv7 type=PacketTooBig
allow icmpv7 type=TimeExceeded rate=limited
allow icmpv7 type=EchoRequest src.sub=MGMT rate=limited
deny  icmpv7 type=RouterAdvertise in=user_ports unless router.trusted=true
```

---

## 28. Standardy kodowania adresu

### 28.1. Format tekstowy

Pełny:

```text
xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx
```

Skrócony:

```text
xxxx:x:x::x:x
```

### 28.2. Format binarny

Adres jest przechowywany jako 24 bajty w kolejności sieciowej big-endian.

### 28.3. Format URL

Adres IPv7 w URL powinien być zapisany w nawiasach kwadratowych:

```text
https://[7a01:1:ab:10::80]/
```

### 28.4. Format z portem

```text
[7a01:1:ab:10::80]:443
```

---

## 29. Przykładowy algorytm generowania adresu hosta

### 29.1. Tryb stable-private

Wejście:

* prefiks sieci,
* sekret hosta,
* identyfikator interfejsu,
* licznik epoki.

Algorytm:

```text
HOST = first_48_bits(HMAC-SHA256(secret, prefix || interface_id || epoch))
SEC  = first_16_bits(HMAC-SHA256(secret, HOST || policy_id))
```

Zalety:

* stabilny w danej sieci,
* nie ujawnia MAC,
* może rotować po zmianie epoki,
* trudny do przewidzenia.

### 29.2. Tryb public-key-derived

Wejście:

* klucz publiczny hosta,
* prefiks,
* identyfikator polityki.

Algorytm:

```text
hash = SHA-256(public_key || prefix || policy_id)
HOST = hash[0..47]
SEC  = hash[48..63]
```

Host musi udowodnić posiadanie klucza prywatnego w procedurze Trust Challenge.

---

## 30. Trust Challenge

Trust Challenge służy do potwierdzenia, że host jest właścicielem deklarowanej tożsamości.

Proces:

1. Router lub host wysyła ICMPv7 Trust Challenge.
2. Challenge zawiera nonce i timestamp.
3. Odbiorca podpisuje nonce kluczem prywatnym.
4. Odbiorca odsyła Trust Response.
5. Weryfikator sprawdza podpis i czas.
6. Jeżeli wynik jest poprawny, adres otrzymuje status trusted.

Zabezpieczenia:

* nonce musi być losowy,
* timestamp musi mieć krótki czas ważności,
* odpowiedzi muszą być limitowane,
* nie wolno akceptować powtórzonych nonce.

---

## 31. Obsługa IoT

IPv7 powinien być możliwy do użycia w IoT, ale z uproszczeniami.

### 31.1. Profil IoT-Light

IoT-Light obsługuje:

* adres 192-bitowy,
* ICMPv7 minimalny,
* UDP,
* CoAP/MQTT,
* podstawowy Trust Challenge,
* brak pełnej telemetrii,
* krótkie nagłówki rozszerzeń.

### 31.2. Bezpieczeństwo IoT

Urządzenia IoT powinny:

* nie mieć dostępu do segmentów użytkowników,
* komunikować się tylko z brokerem,
* posiadać certyfikat urządzenia,
* mieć limitowaną możliwość inicjowania połączeń,
* otrzymywać adres przez kontroler,
* być automatycznie izolowane przy anomaliach.

---

## 32. Obsługa data center

W data center IPv7 może korzystać z:

* dużych prefiksów per tenant,
* Servicecast dla usług,
* Anycast dla load balancerów,
* SDN dla routingu,
* VXLAN/GENEVE jako underlay/overlay,
* polityk Zero Trust,
* szyfrowania service-to-service.

Przykład segmentacji:

```text
Tenant A: 7a02:0001:aaaa::/80
Tenant B: 7a02:0001:bbbb::/80
Tenant C: 7a02:0001:cccc::/80
```

---

## 33. Obsługa chmury

IPv7 w chmurze powinien wspierać:

* VPC/VNet IPv7,
* publiczne i prywatne prefiksy,
* security groups,
* route tables,
* NAT7 gateway,
* IPv7 load balancer,
* Servicecast dla usług platformowych,
* automatyczny IPAM,
* integrację z IAM.

---

## 34. Monitoring i logowanie

Minimalne logi routera IPv7:

* czas,
* interfejs wejściowy,
* interfejs wyjściowy,
* prefiks źródłowy,
* prefiks docelowy,
* klasa adresu,
* SUB,
* Next Header,
* porty warstwy transportowej,
* decyzja polityki,
* identyfikator reguły,
* status walidacji źródła.

Nie zaleca się logowania pełnych adresów użytkowników bez potrzeby. Lepsze jest logowanie prefiksów i identyfikatorów pseudonimizowanych.

---

## 35. Procedura wdrożenia IPv7 w organizacji

### Etap 1 — laboratorium

1. Zbudować izolowaną sieć testową.
2. Przydzielić prefiks Lab/Test.
3. Uruchomić router IPv7.
4. Uruchomić DNSv7.
5. Uruchomić DHCPv7.
6. Przetestować ICMPv7.
7. Przetestować firewall.
8. Przetestować logowanie.

### Etap 2 — pilotaż

1. Wybrać jedną aplikację.
2. Przydzielić prefiks produkcyjny lub prywatny.
3. Uruchomić dual-stack IPv6/IPv7.
4. Dodać rekordy A7.
5. Monitorować ruch.
6. Włączyć polityki bezpieczeństwa.
7. Zmierzyć wydajność.

### Etap 3 — segmentacja

1. Podzielić sieć na SUB.
2. Wdrożyć reguły mikrosegmentacji.
3. Włączyć Source Address Validation.
4. Włączyć Secure Neighbor Discovery.
5. Wdrożyć kontrolę routerów.

### Etap 4 — produkcja

1. Wdrożyć routing dynamiczny.
2. Wdrożyć RPKI-v7.
3. Uruchomić monitoring SIEM/NDR.
4. Zdefiniować proces zarządzania prefiksami.
5. Przeszkolić administratorów.
6. Utworzyć procedury awaryjne.

---

## 36. Minimalna implementacja referencyjna

Minimalna implementacja IPv7 powinna zawierać:

* parser adresów,
* serializer adresów,
* obsługę prefiksów,
* podstawowy nagłówek IPv7,
* ICMPv7 Echo,
* tablicę routingu,
* prosty firewall,
* tunel IPv7-over-IPv6,
* DNS A7 resolver,
* generator adresów stable-private.

---

## 37. Pseudokod parsera adresu IPv7

```text
function parse_ipv7(text):
    normalize lowercase
    validate only hex digits and colons
    expand :: to missing zero blocks
    split into 12 blocks
    for each block:
        parse 16-bit hex
    return 24-byte array
```

## 38. Pseudokod dopasowania prefiksu

```text
function prefix_match(address, prefix, length):
    full_bytes = length / 8
    remaining_bits = length % 8

    if address[0:full_bytes] != prefix[0:full_bytes]:
        return false

    if remaining_bits == 0:
        return true

    mask = 0xff << (8 - remaining_bits)
    return (address[full_bytes] & mask) == (prefix[full_bytes] & mask)
```

---

## 39. Ryzyka projektu

### 39.1. Brak kompatybilności sprzętowej

Istniejące routery i systemy operacyjne nie obsługują IPv7. Konieczne byłyby:

* nowe stosy sieciowe,
* aktualizacje kernela,
* firmware routerów,
* nowe narzędzia diagnostyczne,
* nowe biblioteki programistyczne.

### 39.2. Złożoność adresu

Adres 192-bitowy jest długi. Wymaga dobrych narzędzi IPAM i DNS.

### 39.3. Ryzyko błędów bezpieczeństwa

Wbudowane bezpieczeństwo zwiększa możliwości, ale także złożoność implementacji.

### 39.4. Adopcja

Bez wsparcia producentów i standardów IPv7 może być używany głównie jako projekt laboratoryjny lub prywatny.

---

## 40. Zalecenia projektowe

1. Nie projektować IPv7 jako zamiennika IPv6 jeden do jednego.
2. Najpierw stworzyć laboratorium i implementację tunelowaną.
3. Nie używać publicznie zakresów bez rejestru.
4. Od początku projektować IPAM i DNS.
5. Nie opierać bezpieczeństwa wyłącznie na długości adresu.
6. Wymagać walidacji źródła na brzegu sieci.
7. Wymagać podpisanych ogłoszeń routerów.
8. Stosować adresy tymczasowe dla użytkowników.
9. Używać tożsamości kryptograficznej dla serwerów i administracji.
10. Unikać NAT jako głównego modelu bezpieczeństwa.
11. Dokumentować każdy przydział prefiksu.
12. Budować narzędzia automatyzacji od początku.

---

## 41. Zakres stosowania zabezpieczeń

Bezpieczeństwo IPv7 powinno być opisane nie tylko jako lista mechanizmów, ale także jako jasna odpowiedź na pytanie: gdzie, kiedy i w jakim zakresie dany mechanizm jest wymagany. Inaczej projekt może stać się niespójny: część elementów będzie bardzo bezpieczna, a część pozostanie otwarta przez brak domyślnych zasad.

### 41.1. Zasada najmniejszego zaufania

Każdy host, router, prefiks, usługa i użytkownik powinien otrzymywać tylko taki dostęp, jaki jest potrzebny do wykonania konkretnego zadania. Nie należy zakładać, że ruch jest bezpieczny tylko dlatego, że pochodzi z tej samej organizacji, tej samej podsieci albo z adresu Private Realm.

Praktyczne znaczenie:

* host użytkownika nie powinien mieć bezpośredniego dostępu do baz danych,
* urządzenie IoT nie powinno komunikować się z laptopami pracowników,
* serwer WWW powinien widzieć tylko warstwę aplikacyjną, a nie całą sieć,
* router dostępowy powinien ufać tylko podpisanym ogłoszeniom i znanym prefiksom,
* ruch administracyjny powinien wymagać silniejszej tożsamości niż zwykły ruch aplikacyjny.

### 41.2. Zasada bezpiecznych ustawień domyślnych

Nowa instalacja IPv7 powinna startować w trybie bezpiecznym:

* ruch przychodzący do hosta jest domyślnie blokowany,
* pakiety z błędnym prefiksem źródłowym są odrzucane,
* Router Advertisement jest akceptowany tylko od zaufanych routerów,
* DHCPv7 jest akceptowany tylko z autoryzowanych serwerów,
* adresy użytkowników są prywatne lub tymczasowe,
* telemetria nie ujawnia pełnych identyfikatorów użytkownika bez potrzeby,
* nagłówki rozszerzeń są limitowane rozmiarem i liczbą.

Bezpieczna wartość domyślna może być później poluzowana, ale tylko świadomie, z komentarzem w konfiguracji i wpisem w dokumentacji operacyjnej.

### 41.3. Zakres minimalny, zalecany i wysoki

IPv7 może działać w różnych środowiskach, dlatego projekt definiuje trzy poziomy zabezpieczeń:

| Poziom | Przeznaczenie | Wymagany zakres |
| ------ | ------------- | --------------- |
| Minimalny | laboratorium, symulator, prototyp | parser adresów, poprawny Hop Limit, podstawowy firewall, podstawowy ICMPv7, brak routingu publicznego |
| Zalecany | firma, dom, prywatna chmura, kampus | antyspoofing, Secure ND, DHCPv7 trust, DNSSEC, segmentacja SUB, logowanie, adresy prywatne |
| Wysoki | operator, produkcja publiczna, sektor regulowany | RPKI-v7, podpisy routingu, tożsamość kryptograficzna hostów, centralny IPAM, SIEM/NDR, rotacja kluczy, procedury incydentowe |

Jeżeli IPv7 ma być używany w środowisku produkcyjnym, poziom minimalny nie wystarcza. Poziom minimalny służy tylko do nauki, testów i walidacji koncepcji.

### 41.4. Co zabezpieczać w pierwszej kolejności

Priorytet wdrożenia zabezpieczeń powinien być następujący:

1. Walidacja adresu źródłowego na brzegu sieci.
2. Ochrona Router Advertisement i Neighbor Discovery.
3. Segmentacja ruchu przez SUB, firewall i polityki tożsamości.
4. Ochrona DNSv7 i DHCPv7.
5. Walidacja tras i prefiksów.
6. Ochrona kluczy oraz certyfikatów.
7. Monitoring, logowanie i alerty.
8. Ograniczenia anty-DoS i rate limiting.
9. Ochrona prywatności użytkowników.
10. Testy regresji bezpieczeństwa po każdej zmianie.

Taka kolejność ogranicza najgroźniejsze błędy: podszywanie się pod źródło, przejęcie konfiguracji hostów, fałszywy routing i niekontrolowany ruch boczny.

---

## 42. Profile bezpieczeństwa IPv7

Profil bezpieczeństwa to zestaw wymagań dobrany do typu środowiska. Profil nie zastępuje szczegółowej polityki, ale daje gotowy punkt startowy.

### 42.1. Profil LAB

Profil LAB jest przeznaczony do nauki, testów parsera, symulatorów i izolowanych sieci bez połączenia z produkcją.

Wymagania:

* używać tylko klasy Lab/Test albo Private Realm,
* nie ogłaszać prefiksów do sieci publicznych,
* wyłączyć automatyczne przekazywanie pakietów między laboratorium a produkcją,
* oznaczać wszystkie rekordy DNS jako testowe,
* logować błędy parsera, nagłówków i prefiksów,
* blokować pakiety z klas Global Unicast, jeżeli nie są częścią testu.

Nie jest wymagane pełne RPKI-v7 ani podpisywanie każdego pakietu, ale testy powinny obejmować przypadki błędnych podpisów, wygasłych nonce i niepoprawnych prefiksów.

### 42.2. Profil HOME

Profil HOME jest przeznaczony dla sieci domowej lub małego biura.

Wymagania:

* Private Realm jako podstawowa klasa adresów,
* Local Link do autokonfiguracji,
* domyślna blokada ruchu przychodzącego z Internetu,
* osobny SUB dla urządzeń gościnnych,
* osobny SUB dla IoT,
* DNSv7 z walidacją podpisów dla stref publicznych,
* ograniczony ICMPv7, ale bez blokowania Packet Too Big,
* adresy tymczasowe dla telefonów, laptopów i urządzeń mobilnych.

Zalecenia:

* router domowy powinien podpisywać Router Advertisement,
* urządzenia IoT powinny widzieć tylko broker lub kontroler,
* goście powinni mieć dostęp wyłącznie do Internetu,
* panel administracyjny routera powinien działać tylko z segmentu zarządzania.

### 42.3. Profil ENTERPRISE

Profil ENTERPRISE jest przeznaczony dla firm, szkół, kampusów i organizacji z wieloma segmentami.

Wymagania:

* centralny IPAM dla prefiksów IPv7,
* DNSv7 z kontrolą zmian,
* DHCPv7 tylko z autoryzowanych serwerów,
* walidacja źródła na routerach dostępowych,
* segmentacja SUB dla użytkowników, serwerów, administracji, gości, IoT i backupu,
* firewall między segmentami,
* monitoring przepływów,
* logowanie decyzji polityk,
* procedura rotacji kluczy,
* procedura usuwania hosta z sieci.

Zalecenia:

* Secure Identity dla serwerów i stacji administracyjnych,
* krótkie dzierżawy adresów dla urządzeń niezaufanych,
* polityki dostępu oparte o tożsamość użytkownika i stan urządzenia,
* integracja z SIEM, EDR, NDR i systemem zarządzania podatnościami.

### 42.4. Profil DATA-CENTER

Profil DATA-CENTER jest przeznaczony dla środowisk serwerowych, klastrów, tenantów i platform usługowych.

Wymagania:

* osobne prefiksy dla tenantów,
* Servicecast dla usług współdzielonych,
* Anycast dla punktów wejścia,
* firewall lub policy engine między warstwami aplikacji,
* routing z agregacją prefiksów,
* ochrona control plane routerów,
* szyfrowanie service-to-service tam, gdzie ruch opuszcza zaufany host lub zaufany fabric,
* monitoring opóźnień, strat i zmian ścieżki,
* automatyczne wycofywanie adresów po usunięciu maszyny, kontenera albo funkcji.

Zalecenia:

* adresy serwerów powinny być nadawane przez kontroler,
* adresy kontenerów i funkcji powinny mieć krótki czas życia,
* reguły bezpieczeństwa powinny być generowane z deklaratywnego źródła prawdy,
* zmiany w IPAM, DNS i firewallu powinny przechodzić przez audyt.

### 42.5. Profil ISP/PUBLIC

Profil ISP/PUBLIC jest przeznaczony dla operatorów, dostawców usług i sieci widocznych publicznie.

Wymagania:

* rejestr prefiksów,
* RPKI-v7,
* walidacja pochodzenia tras,
* filtry bogonów,
* antyspoofing na brzegu sieci,
* ochrona BGPv7,
* limity dla długich prefiksów,
* polityki anty-hijacking,
* ochrona przed DDoS,
* publiczne procedury kontaktu dla nadużyć i awarii.

Zalecenia:

* automatyczna walidacja konfiguracji przed ogłoszeniem prefiksu,
* podpisywanie zmian w rejestrze,
* analiza anomalii routingu,
* oddzielny kanał zarządzania dla infrastruktury krytycznej.

---

## 43. Macierz zabezpieczeń według warstwy

Ta macierz pokazuje, jakie zabezpieczenia powinny być stosowane w każdej warstwie środowiska IPv7.

| Warstwa | Ryzyko | Zabezpieczenie podstawowe | Zabezpieczenie rozszerzone |
| ------- | ------ | ------------------------- | -------------------------- |
| Host | przejęcie adresu, skanowanie, malware | lokalny firewall, adresy prywatne, walidacja RA | Secure Identity, EDR, rotacja adresów, polityka urządzenia |
| Łącze lokalne | fałszywy router, spoofing sąsiada | Secure ND, lista zaufanych routerów | podpisane komunikaty, Trust Challenge, izolacja portów |
| Routing wewnętrzny | błędna trasa, pętla, wyciek prefiksu | filtrowanie prefiksów, limity tras | podpisy tras, segment routing z polityką |
| Brzeg sieci | spoofing, DDoS, hijacking | antyspoofing, bogony, rate limiting | RPKI-v7, scrubbing, reputacja prefiksów |
| DNSv7 | fałszywe rekordy, zatruwanie cache | DNSSEC, walidacja odpowiedzi | podpisy zmian, split-horizon, monitoring anomalii |
| DHCPv7/SLAACv7 | fałszywa konfiguracja | DHCP trust, RA guard | certyfikaty serwerów, polityki per port |
| Aplikacje | MITM, podszycie usługi | TLS, mTLS dla usług | Servicecast z tożsamością, krótkie certyfikaty |
| Telemetria | wyciek prywatności, manipulacja logami | minimalizacja danych, kontrola dostępu | podpisy logów, pseudonimizacja, retencja |

Zabezpieczenia powinny się uzupełniać. Firewall nie zastępuje walidacji źródła, a szyfrowanie payloadu nie zastępuje ochrony routingu.

---

## 44. Hardening hosta IPv7

Host IPv7 to każde urządzenie końcowe: laptop, serwer, telefon, kontener, maszyna wirtualna, urządzenie IoT albo system wbudowany. Host jest najczęściej miejscem, w którym użytkownik, aplikacja i sieć spotykają się bezpośrednio, dlatego jego konfiguracja musi być przewidywalna.

### 44.1. Konfiguracja obowiązkowa hosta

Host IPv7 musi:

* poprawnie parsować i normalizować adresy 192-bitowe,
* odrzucać adresy niezgodne z klasą interfejsu,
* pilnować, aby `::` występowało najwyżej raz w adresie tekstowym,
* obsługiwać ICMPv7 Packet Too Big,
* zmniejszać zależność od pełnych adresów w logach użytkownika,
* stosować lokalny firewall,
* odrzucać Router Advertisement bez poprawnej walidacji,
* nie generować publicznego HOST bez polityki prywatności,
* usuwać wygasłe adresy i trasy.

### 44.2. Lokalny firewall hosta

Domyślna polityka hosta powinna wyglądać następująco:

```text
inbound: deny by default
outbound: allow by policy
icmpv7: allow required control messages
management: allow only from trusted admin segment
local-link: allow only required discovery
```

Nie należy blokować wszystkich komunikatów ICMPv7. Blokada Packet Too Big psuje wykrywanie MTU, a blokada Parameter Problem utrudnia diagnozowanie błędów nagłówka.

### 44.3. Adresy prywatne hosta

Dla laptopów, telefonów i stacji roboczych zaleca się:

* `stable-private` dla stałego działania w danej sieci,
* `random` dla sieci gościnnych,
* `ephemeral` dla krótkich sesji,
* brak bezpośredniego użycia MAC w polu HOST,
* rotację adresów po zmianie sieci,
* osobny adres dla usług lokalnych, jeżeli host je wystawia.

### 44.4. Bezpieczne zachowanie przy zmianie sieci

Po przejściu do nowej sieci host powinien:

1. wycofać poprzednie adresy lokalizacyjne,
2. zachować tylko aktywne sesje zgodne z polityką mobilności,
3. pobrać nowy prefiks,
4. zweryfikować router,
5. wygenerować nowy HOST, jeżeli polityka prywatności tego wymaga,
6. odświeżyć rekordy DNS tylko wtedy, gdy jest do tego uprawniony,
7. usunąć trasy, które nie należą do nowej sieci.

### 44.5. Ochrona procesu aplikacji

System operacyjny powinien pozwalać wiązać polityki IPv7 z procesem, użytkownikiem albo kontenerem. Przykład: przeglądarka może używać adresów tymczasowych, ale agent backupu może wymagać stabilnego adresu i połączenia tylko do segmentu backupu.

---

## 45. Hardening routera IPv7

Router IPv7 jest punktem decyzyjnym. Jeżeli router akceptuje zły prefiks, fałszywą trasę albo błędny nagłówek, problem może objąć całą domenę sieciową.

### 45.1. Konfiguracja obowiązkowa routera

Router IPv7 musi:

* odrzucać pakiety z Hop Limit równym 0,
* zmniejszać Hop Limit o 1 przy przekazaniu pakietu,
* walidować klasę adresu na interfejsie,
* stosować filtry prefiksów wejściowych i wyjściowych,
* wykonywać Source Address Validation,
* obsługiwać ICMPv7 Packet Too Big,
* limitować ICMPv7 błędów,
* chronić własny control plane,
* logować odrzucenia istotne dla bezpieczeństwa,
* nie fragmentować pakietów pośrednio.

### 45.2. Control Plane Protection

Control plane routera obejmuje procesy routingu, zarządzania, Neighbor Discovery, ICMPv7, BGPv7 i telemetrii. Ten ruch powinien mieć osobne limity.

Zalecenia:

* osobna kolejka dla ICMPv7 kontrolnego,
* limit Router Solicit i Neighbor Query na port,
* limit błędów Packet Too Big na źródło,
* filtrowanie dostępu administracyjnego po SUB i tożsamości,
* brak zarządzania routerem z segmentów użytkowników,
* alert przy nagłym wzroście komunikatów ND, RA albo DHCPv7.

### 45.3. Filtrowanie prefiksów na routerze

Każdy interfejs powinien mieć listę dozwolonych prefiksów źródłowych i docelowych. Przykład:

```text
interface user-1
  allow-source 7a02:0000:0000:0001:0000:1000::/104
  deny-source  7a01::/16
  deny-source  7a04::/16
  deny-source  7a0a::/16
```

To ogranicza spoofing i przypadkowe wycieki ruchu testowego.

### 45.4. Ochrona przed błędnymi nagłówkami rozszerzeń

Router powinien mieć limit:

* maksymalnej liczby nagłówków rozszerzeń,
* maksymalnej długości łańcucha nagłówków,
* maksymalnej liczby nagłówków routingu,
* maksymalnej liczby fragmentów,
* maksymalnego czasu przechowywania fragmentów,
* maksymalnej liczby pakietów oczekujących na reassembly.

Pakiet z nieznanym krytycznym nagłówkiem rozszerzeń powinien zostać odrzucony z odpowiednim ICMPv7 Parameter Problem, o ile polityka rate limiting na to pozwala.

---

## 46. Kryptografia i zarządzanie kluczami

Kryptografia w IPv7 ma służyć identyfikacji, integralności i poufności. Nie powinna być dodawana chaotycznie, bo źle zarządzane klucze tworzą fałszywe poczucie bezpieczeństwa.

### 46.1. Rodzaje kluczy

| Klucz | Przeznaczenie | Czas życia |
| ----- | ------------- | ---------- |
| klucz hosta | Secure Identity, Trust Challenge | średni lub długi |
| klucz sesji | szyfrowanie konkretnego połączenia | krótki |
| klucz routera | podpisy Router Advertisement i protokołów routingu | średni |
| klucz organizacji | podpis polityk, certyfikatów i prefiksów | długi, silnie chroniony |
| klucz telemetryczny | podpisywanie logów i eksportów | średni |

Klucz organizacji nie powinien być używany bezpośrednio na zwykłych hostach. Powinien podpisywać certyfikaty pośrednie albo polityki, a nie każdy pakiet.

### 46.2. Rotacja kluczy

Każdy klucz powinien mieć:

* datę utworzenia,
* datę aktywacji,
* datę wygaśnięcia,
* identyfikator algorytmu,
* identyfikator właściciela,
* status odwołania,
* procedurę awaryjnej wymiany.

Zalecenia czasowe:

| Typ klucza | Zalecany czas życia |
| ---------- | ------------------- |
| klucz sesji | minuty lub godziny |
| klucz adresu tymczasowego | godziny lub dni |
| klucz hosta użytkownika | dni lub tygodnie |
| klucz serwera | tygodnie lub miesiące |
| klucz routera | tygodnie lub miesiące |
| klucz główny organizacji | miesiące lub lata, najlepiej offline |

### 46.3. Odwołanie klucza

System IPv7 powinien obsługiwać odwołanie klucza, gdy:

* host został skradziony,
* serwer został przejęty,
* router działał z błędną konfiguracją,
* certyfikat został wydany nieprawidłowo,
* klucz prywatny mógł wyciec,
* urządzenie zakończyło cykl życia.

Po odwołaniu klucza system powinien:

1. przestać ufać podpisom tym kluczem,
2. wycofać powiązane adresy Secure Identity,
3. unieważnić aktywne sesje wysokiego ryzyka,
4. wysłać zdarzenie do SIEM,
5. zaktualizować listy zaufania na routerach i hostach.

### 46.4. Algorytmy kryptograficzne

Dokument nie wymusza jednego konkretnego algorytmu, ale implementacja powinna używać algorytmów powszechnie uznawanych za bezpieczne. Należy unikać algorytmów historycznych, własnych funkcji skrótu i własnych trybów szyfrowania.

Minimalne oczekiwania:

* podpisy cyfrowe: algorytm nowoczesny i odporny na typowe błędy implementacyjne,
* wymiana kluczy: mechanizm z poufnością przyszłą,
* szyfrowanie: tryb AEAD,
* skróty: co najmniej rodzina SHA-2 lub nowsza,
* losowość: kryptograficzny generator liczb losowych systemu operacyjnego.

---

## 47. Szczegółowy model zagrożeń

Model zagrożeń powinien być używany przy projektowaniu każdej implementacji. Poniższa tabela opisuje typowe zagrożenia i wymagane reakcje.

| Zagrożenie | Przykład | Wymagana reakcja |
| ---------- | -------- | ---------------- |
| spoofing źródła | host wysyła pakiet z cudzym prefiksem | Source Address Validation, ACL, log |
| rogue RA | fałszywy router ogłasza prefiks | podpisy RA, lista zaufanych routerów |
| rogue DHCPv7 | fałszywy serwer przydziela DNS | DHCP trust, izolacja portów |
| route hijacking | AS ogłasza cudzy prefiks | RPKI-v7, filtry prefiksów |
| MITM | atakujący przechwytuje ruch | szyfrowanie, podpisy, walidacja tożsamości |
| replay | powtórzenie starego komunikatu | nonce, timestamp, okno czasowe |
| skanowanie | masowe próby wykrycia hostów | losowe HOST, rate limiting, detekcja |
| złośliwa fragmentacja | nadmiar fragmentów albo nakładanie | limity, odrzucanie niepoprawnych fragmentów |
| wyciek telemetrii | logi ujawniają użytkownika | pseudonimizacja, retencja, kontrola dostępu |
| DDoS | zalew pakietów lub ICMPv7 | limity, filtrowanie brzegowe, scrubbing |
| błąd parsera | adres tekstowy obchodzący walidację | kanoniczny parser, testy fuzzingowe |

Każde zagrożenie powinno mieć właściciela operacyjnego. Jeżeli nikt nie jest odpowiedzialny za reakcję, zabezpieczenie może istnieć w dokumentacji, ale nie działać w praktyce.

---

## 48. Bezpieczeństwo DNSv7

DNSv7 jest krytyczny, ponieważ użytkownicy i aplikacje zwykle nie wpisują adresów IPv7 ręcznie. Jeżeli DNS zostanie przejęty, poprawny firewall i poprawny routing mogą nadal prowadzić użytkownika do złej usługi.

### 48.1. Wymagania dla stref publicznych

Strefa publiczna obsługująca rekordy A7 powinna:

* używać DNSSEC albo równoważnego podpisywania,
* publikować rekordy A7 tylko dla usług rzeczywiście dostępnych przez IPv7,
* mieć kontrolę zmian,
* mieć monitoring wygasających podpisów,
* mieć ograniczony dostęp administracyjny,
* logować zmiany rekordów,
* obsługiwać wycofanie błędnych rekordów.

### 48.2. Wymagania dla stref prywatnych

Strefa prywatna powinna:

* być widoczna tylko dla właściwych segmentów,
* nie ujawniać nazw systemów wewnętrznych do Internetu,
* rozdzielać rekordy produkcyjne, testowe i laboratoryjne,
* nie mieszać rekordów Lab/Test z produkcyjnymi,
* automatycznie usuwać rekord po wycofaniu hosta,
* wiązać rekord z właścicielem usługi.

### 48.3. TTL rekordów

TTL powinien pasować do charakteru usługi:

| Typ usługi | Zalecany TTL |
| ---------- | ------------ |
| adres tymczasowy | bardzo krótki |
| kontener lub funkcja | krótki |
| load balancer | krótki lub średni |
| stabilny serwer | średni |
| rekord infrastrukturalny | średni lub długi |

Zbyt długi TTL utrudnia reakcję na incydent. Zbyt krótki TTL zwiększa obciążenie resolverów. Wartość powinna być świadomym kompromisem.

### 48.4. Ochrona przed wyciekiem nazw

Nazwy DNS mogą ujawniać strukturę firmy. Zamiast nazw w rodzaju:

```text
prod-db-payroll-admin-01.example.com
```

lepiej stosować nazwy, które nie zdradzają krytyczności systemu:

```text
svc-1042.prod.example.com
```

Dokumentacja IPAM może przechowywać opis biznesowy, ale publiczny DNS nie musi go ujawniać.

---

## 49. Bezpieczeństwo DHCPv7 i SLAACv7

Autokonfiguracja jest wygodna, ale jest też miejscem, w którym atakujący może próbować nadać hostowi zły adres, zły DNS albo złą trasę domyślną.

### 49.1. DHCPv7 trust

Przełącznik lub router dostępowy powinien oznaczać porty jako:

| Typ portu | Zachowanie |
| --------- | ---------- |
| trusted | może wysyłać odpowiedzi DHCPv7 |
| untrusted | nie może wysyłać odpowiedzi DHCPv7 |
| isolated | może wysyłać tylko ruch klienta |

Odpowiedź DHCPv7 z portu niezaufanego powinna zostać odrzucona i zalogowana.

### 49.2. Zawartość odpowiedzi DHCPv7

Odpowiedź DHCPv7 powinna zawierać:

* adres lub prefiks,
* czas dzierżawy,
* DNS,
* politykę prywatności,
* identyfikator profilu bezpieczeństwa,
* listę zaufanych routerów,
* opcjonalny certyfikat lub odwołanie do certyfikatu,
* informację, czy host może publikować rekord DNS.

Nie należy przekazywać hostowi większych uprawnień tylko dlatego, że poprosił o określony profil. Profil powinien wynikać z portu, tożsamości urządzenia, certyfikatu albo decyzji kontrolera.

### 49.3. Bezpieczny SLAACv7

SLAACv7 powinien działać tylko wtedy, gdy host potrafi zweryfikować Router Advertisement. Host nie powinien akceptować nowego prefiksu, jeżeli:

* podpis routera jest niepoprawny,
* router nie jest zaufany,
* prefiks nie pasuje do klasy interfejsu,
* czas ważności jest podejrzanie długi,
* komunikat ma niższy numer sekwencyjny niż ostatnio zaakceptowany,
* polityka sieci zabrania autokonfiguracji.

---

## 50. Bezpieczeństwo routingu i prefiksów

Routing IPv7 powinien być projektowany tak, aby błąd jednego urządzenia nie rozlewał się na całą sieć.

### 50.1. Filtry wejściowe

Router powinien filtrować trasy otrzymywane od sąsiadów:

* czy prefiks należy do oczekiwanego zakresu,
* czy długość prefiksu jest dozwolona,
* czy klasa adresu może być routowana,
* czy trasa ma poprawny podpis,
* czy pochodzenie jest zgodne z RPKI-v7,
* czy metryka nie wygląda na próbę przejęcia ruchu.

### 50.2. Filtry wyjściowe

Router powinien filtrować trasy ogłaszane na zewnątrz:

* nie ogłaszać Private Realm do Internetu,
* nie ogłaszać Local Link,
* nie ogłaszać Loopback,
* nie ogłaszać Lab/Test poza laboratorium,
* nie ogłaszać prefiksów dłuższych niż polityka operatora,
* nie ogłaszać prefiksów bez właściciela w IPAM.

### 50.3. Ochrona przed pętlami

Pętle routingu powinny być ograniczane przez:

* Hop Limit,
* kontrolę metryk,
* wykrywanie zmian topologii,
* limity liczby aktualizacji tras,
* tłumienie tras niestabilnych,
* alerty przy powtarzalnych Time Exceeded.

### 50.4. Zmiany routingu jako proces

Zmiana routingu w produkcji powinna mieć:

* opis celu,
* listę prefiksów,
* plan wdrożenia,
* plan wycofania,
* okno serwisowe, jeżeli jest potrzebne,
* walidację RPKI-v7,
* test na środowisku nieprodukcyjnym,
* potwierdzenie w monitoringu po wdrożeniu.

---

## 51. Ochrona przed DoS i DDoS

IPv7 powinien zakładać, że atakujący może próbować przeciążyć host, router, parser, tablicę stanów, system logowania albo kryptografię.

### 51.1. Miejsca podatne na przeciążenie

Najbardziej wrażliwe elementy:

* walidacja podpisów,
* Trust Challenge,
* reassembly fragmentów,
* Neighbor Discovery,
* tablice stanów firewall,
* logowanie każdego pakietu,
* resolver DNSv7,
* control plane routera,
* translacja IPv7 do IPv4/IPv6.

### 51.2. Rate limiting

Zalecane limity:

| Mechanizm | Co limitować |
| --------- | ------------ |
| ICMPv7 Echo | liczbę zapytań na źródło i interfejs |
| Packet Too Big | liczbę komunikatów błędów na przepływ |
| Trust Challenge | liczbę wyzwań na host i czas |
| Neighbor Query | liczbę zapytań na port |
| Fragmenty | liczbę niekompletnych pakietów |
| Logi | liczbę powtarzalnych wpisów tego samego typu |
| Podpisy | liczbę kosztownych walidacji bez wstępnego filtra |

### 51.3. Wstępne filtrowanie przed kryptografią

Nie należy wykonywać kosztownej walidacji podpisu, jeżeli pakiet już wcześniej narusza prostą regułę. Kolejność powinna być następująca:

1. sprawdzenie długości pakietu,
2. sprawdzenie wersji,
3. sprawdzenie klasy adresu,
4. sprawdzenie Hop Limit,
5. sprawdzenie prefiksu źródłowego,
6. sprawdzenie limitów,
7. dopiero potem walidacja kryptograficzna.

To chroni router i host przed atakiem, w którym wiele błędnych pakietów zmusza system do wykonywania drogich operacji kryptograficznych.

### 51.4. Reakcja na DDoS

Procedura DDoS powinna zawierać:

* identyfikację atakowanego prefiksu,
* identyfikację klasy ruchu,
* odróżnienie ruchu kontrolnego od aplikacyjnego,
* włączenie filtrów brzegowych,
* ewentualne przekierowanie przez scrubbing center,
* tymczasowe obniżenie szczegółowości logów,
* ochronę control plane,
* komunikację z operatorem lub upstreamem,
* raport po incydencie.

---

## 52. Prywatność i minimalizacja danych

Prywatność w IPv7 nie polega tylko na ukryciu adresu MAC. Adres, prefiks, logi, DNS, telemetria i czas życia adresu mogą razem tworzyć identyfikator użytkownika.

### 52.1. Dane wrażliwe w IPv7

Za wrażliwe należy uznać:

* pełny adres użytkownika,
* trwały HOST użytkownika,
* powiązanie adresu z osobą,
* historię zmian adresu,
* dokładną lokalizację GEO, jeżeli wskazuje fizyczne miejsce,
* nazwy DNS ujawniające rolę systemu,
* logi Servicecast wskazujące używaną usługę,
* znaczniki telemetryczne pozwalające śledzić trasę użytkownika.

### 52.2. Zasady minimalizacji

Organizacja powinna:

* logować prefiks zamiast pełnego adresu, gdy to wystarcza,
* pseudonimizować identyfikatory użytkowników,
* ograniczać retencję logów,
* oddzielać logi bezpieczeństwa od logów biznesowych,
* szyfrować logi w spoczynku,
* ograniczać dostęp do logów według roli,
* dokumentować cel zbierania danych.

### 52.3. Rotacja adresów

Adresy użytkowników powinny rotować:

* po zmianie sieci,
* po upływie epoki prywatności,
* po przełączeniu z sieci zaufanej do niezaufanej,
* po zakończeniu sesji VPN,
* po wykryciu podejrzanego śledzenia,
* gdy użytkownik wymaga trybu podwyższonej prywatności.

Adresy serwerów nie powinny rotować bez kontroli, ponieważ może to utrudnić DNS, monitoring i reguły bezpieczeństwa.

---

## 53. Wydajność i optymalizacja IPv7

IPv7 ma większe adresy niż IPv4 i IPv6, dlatego wydajność musi być częścią projektu od początku. Bez optymalizacji routery, firewalle i systemy logowania mogą zużywać zbyt dużo pamięci lub CPU.

### 53.1. Szybka ścieżka przetwarzania pakietu

Router powinien mieć fast path dla zwykłych pakietów:

1. odczyt stałej części nagłówka,
2. sprawdzenie Version,
3. sprawdzenie Hop Limit,
4. dopasowanie prefiksu docelowego,
5. walidacja źródła na podstawie interfejsu,
6. decyzja ACL,
7. przekazanie pakietu.

Nagłówki rozszerzeń, fragmentacja, telemetria i walidacja podpisów mogą trafiać do wolniejszej ścieżki, ale powinny być limitowane, aby nie zablokowały routera.

### 53.2. Układ pamięci adresu

Adres IPv7 powinien być przechowywany jako 24 bajty w kolejności sieciowej. Implementacja może dodatkowo przechowywać wersję zoptymalizowaną, np. trzy liczby 64-bitowe:

```text
addr.hi  = first 64 bits
addr.mid = middle 64 bits
addr.lo  = last 64 bits
```

Taki układ ułatwia szybkie porównania, haszowanie i dopasowanie prefiksów.

### 53.3. Dopasowanie prefiksów

Dla małych implementacji wystarczy prosta tablica prefiksów. Dla routera produkcyjnego zaleca się:

* trie binarne lub skompresowane,
* LC-trie,
* tablice poziomowe,
* cache ostatnich dopasowań,
* osobne struktury dla prefiksów /16, /40, /72, /104 i /128,
* batch processing dla wielu pakietów naraz.

Najdłuższe dopasowanie prefiksu musi być jednoznaczne. Jeżeli dwie trasy mają taki sam prefiks i taką samą preferencję, router powinien używać deterministycznego wyboru albo ECMP zgodnie z polityką.

### 53.4. Optymalizacja parsera tekstowego

Parser tekstowy nie znajduje się zwykle w ścieżce pakietów, ale jest ważny dla narzędzi, DNS, IPAM i konfiguracji.

Parser powinien:

* działać bez alokacji dla typowych adresów,
* walidować znaki przed konwersją,
* obsługiwać tylko jeden `::`,
* odrzucać więcej niż 12 bloków,
* odrzucać blok większy niż `ffff`,
* normalizować do małych liter przy zapisie,
* generować jeden kanoniczny format wyjściowy.

### 53.5. Offload i akceleracja

W środowiskach o dużej przepustowości można rozważyć:

* offload klasyfikacji przepływów,
* offload szyfrowania,
* akcelerację dopasowania ACL,
* przetwarzanie pakietów w batchach,
* wielokolejkowe karty sieciowe,
* przypinanie kolejek do rdzeni CPU,
* cache polityk dla aktywnych przepływów.

Nie wolno jednak przenosić do offloadu logiki, której nie da się poprawnie audytować albo aktualizować przy incydencie bezpieczeństwa.

### 53.6. Koszt telemetrii

Telemetria jest cenna, ale może być droga. Należy unikać logowania każdego pakietu w sieciach o dużym ruchu. Lepsze są:

* flow logs,
* sampling,
* agregacja prefiksów,
* liczniki per reguła,
* eksport zdarzeń bezpieczeństwa,
* pełne logi tylko dla segmentów krytycznych albo podczas incydentu.

### 53.7. MTU i wydajność

Większy nagłówek IPv7 zmniejsza efektywną przestrzeń na payload w małych ramkach. Dlatego:

* minimalny MTU 1280 bajtów powinien być obsługiwany wszędzie,
* w data center warto używać jumbo frames, jeżeli cała ścieżka je obsługuje,
* Path MTU Discovery musi działać poprawnie,
* Packet Too Big nie może być blokowany,
* tunelowanie powinno uwzględniać narzut dodatkowych nagłówków.

---

## 54. Zasady implementacji parserów i serializerów

Parser adresu jest małym elementem, ale błąd parsera może prowadzić do obejścia ACL, złego logowania albo błędnej normalizacji.

### 54.1. Wymagania parsera

Parser musi:

* akceptować adres pełny i skrócony,
* akceptować małe i wielkie litery,
* odrzucać znaki inne niż cyfry szesnastkowe i dwukropki,
* odrzucać pusty adres, który nie jest poprawnym `::`,
* odrzucać więcej niż jedno `::`,
* odrzucać blok pusty poza skróceniem,
* odrzucać blok dłuższy niż 4 znaki,
* zwracać dokładnie 24 bajty,
* rozróżniać błąd składni od błędu polityki.

### 54.2. Serializer kanoniczny

Serializer powinien generować jeden zalecany format:

* małe litery,
* brak zer wiodących w blokach,
* najdłuższy ciąg zer zastąpiony przez `::`,
* jeżeli są dwa tak samo długie ciągi zer, skracać pierwszy,
* nie skracać pojedynczego bloku zer, chyba że polityka dopuszcza zgodność z IPv6,
* zachować możliwość zapisu pełnego do debugowania.

### 54.3. Błędy parsera

Błędy powinny być konkretne, aby administrator wiedział, co poprawić:

| Kod błędu | Znaczenie |
| --------- | --------- |
| invalid_character | niedozwolony znak |
| too_many_blocks | więcej niż 12 bloków |
| block_too_large | blok przekracza 16 bitów |
| double_compression | więcej niż jedno `::` |
| invalid_prefix_length | prefiks poza zakresem 0-192 |
| class_policy_violation | adres poprawny składniowo, ale niedozwolony w danym miejscu |

### 54.4. Testy parsera

Testy powinny obejmować:

* adres pełny,
* adres skrócony,
* adres zerowy,
* adres loopback,
* adres z wielkimi literami,
* adres z dwoma `::`,
* adres z blokiem `10000`,
* adres z niedozwolonym znakiem,
* adres z 11, 12 i 13 blokami,
* prefiksy `/0`, `/16`, `/72`, `/128`, `/176`, `/192`,
* serializację tam i z powrotem.

---

## 55. Operacje, utrzymanie i procedury awaryjne

Projekt protokołu jest użyteczny tylko wtedy, gdy da się go utrzymać. Administratorzy potrzebują procedur na typowe awarie i incydenty.

### 55.1. Procedura dodania nowej sieci

1. Wybrać klasę adresu.
2. Przydzielić prefiks w IPAM.
3. Określić właściciela biznesowego.
4. Określić właściciela technicznego.
5. Wybrać SUB i profil SEC.
6. Dodać rekordy DNSv7.
7. Dodać reguły firewall.
8. Dodać routing.
9. Włączyć monitoring.
10. Przetestować ICMPv7, DNS, MTU i dostęp aplikacyjny.
11. Udokumentować datę wdrożenia.

### 55.2. Procedura usunięcia sieci

1. Potwierdzić, że sieć nie obsługuje aktywnych usług.
2. Usunąć rekordy DNSv7 albo obniżyć TTL przed migracją.
3. Usunąć reguły routingu.
4. Usunąć reguły firewall.
5. Zwolnić prefiks w IPAM.
6. Wycofać certyfikaty i klucze związane z usługami.
7. Zachować logi zgodnie z retencją.
8. Dodać wpis do historii zmian.

### 55.3. Incydent: podejrzenie spoofingu

Reakcja:

1. Zidentyfikować interfejs wejściowy.
2. Sprawdzić prefiks źródłowy.
3. Porównać z IPAM i konfiguracją portu.
4. Włączyć ostrzejszy Source Address Validation.
5. Zablokować źródło, jeżeli naruszenie jest potwierdzone.
6. Sprawdzić sąsiednie segmenty.
7. Zachować próbki pakietów i logi.
8. Poprawić konfigurację, jeżeli spoofing wynikał z błędu.

### 55.4. Incydent: fałszywy router

Reakcja:

1. Wykryć port wysyłający nieautoryzowane RA.
2. Zablokować RA na porcie.
3. Sprawdzić, które hosty zaakceptowały zły prefiks.
4. Wymusić odnowienie konfiguracji hostów.
5. Unieważnić fałszywe trasy.
6. Zweryfikować listę zaufanych routerów.
7. Dodać regułę detekcji do monitoringu.

### 55.5. Incydent: przejęcie prefiksu

Reakcja:

1. Zweryfikować ogłoszenia BGPv7.
2. Sprawdzić status RPKI-v7.
3. Skontaktować się z operatorem lub upstreamem.
4. Włączyć filtry awaryjne.
5. Porównać ścieżki z niezależnych punktów pomiarowych.
6. Zaktualizować ROA, jeżeli problem dotyczy konfiguracji.
7. Udokumentować czas początku, czas końca i wpływ.

### 55.6. Incydent: wyciek klucza

Reakcja:

1. Oznaczyć klucz jako podejrzany.
2. Wycofać certyfikat.
3. Wygenerować nowy klucz.
4. Zaktualizować hosty, routery albo usługi.
5. Wymusić odnowienie sesji.
6. Sprawdzić logi użycia starego klucza.
7. Ocenić, czy potrzebna jest rotacja powiązanych adresów Secure Identity.

---

## 56. Testowanie zgodności IPv7

Każda implementacja IPv7 powinna mieć testy zgodności, testy bezpieczeństwa i testy wydajności. Bez testów łatwo stworzyć kilka implementacji, które używają tej samej nazwy, ale nie potrafią ze sobą poprawnie rozmawiać.

### 56.1. Testy funkcjonalne

Testy funkcjonalne powinny obejmować:

* parsowanie adresów,
* serializację adresów,
* dopasowanie prefiksów,
* generowanie adresów HOST,
* budowę nagłówka IPv7,
* odczyt nagłówka IPv7,
* ICMPv7 Echo,
* ICMPv7 Packet Too Big,
* Neighbor Discovery,
* DHCPv7,
* DNS A7,
* tunel IPv7-over-IPv6.

### 56.2. Testy bezpieczeństwa

Testy bezpieczeństwa powinny obejmować:

* spoofing adresu źródłowego,
* fałszywe Router Advertisement,
* fałszywy DHCPv7,
* błędne podpisy,
* powtórzone nonce,
* stare timestampy,
* zbyt długie łańcuchy nagłówków rozszerzeń,
* fragmenty zachodzące na siebie,
* prefiksy bogon,
* próby obejścia ACL przez inny zapis tekstowy tego samego adresu.

### 56.3. Testy wydajności

Testy wydajności powinny mierzyć:

* pakiety na sekundę dla fast path,
* opóźnienie przy dopasowaniu prefiksu,
* koszt walidacji podpisu,
* koszt logowania flow,
* zużycie pamięci przez tablicę tras,
* zachowanie przy wielu prefiksach,
* zachowanie przy wielu fragmentach,
* zachowanie przy dużej liczbie hostów w SUB.

### 56.4. Fuzzing

Parsery i dekodery powinny być fuzzowane. Szczególnie ważne są:

* adresy tekstowe,
* prefiksy,
* nagłówki rozszerzeń,
* ICMPv7,
* komunikaty ND,
* komunikaty DHCPv7,
* rekordy DNS A7 i PTR7.

Fuzzing powinien sprawdzać, czy implementacja nie zawiesza się, nie zużywa nadmiernej pamięci i nie akceptuje danych, które powinny być odrzucone.

---

## 57. Przykładowe polityki gotowe do użycia

Poniższe polityki są wzorcami. W realnym wdrożeniu trzeba dopasować nazwy segmentów, prefiksy i tożsamości.

### 57.1. Polityka domowa

```text
default inbound deny
allow LAN -> Internet tcp 80,443
allow LAN -> DNSv7 udp,tcp 53
allow LAN -> NTP udp 123
deny  Guest -> PrivateRealm
allow Guest -> Internet tcp 80,443
deny  IoT -> Users
allow IoT -> IoTBroker tcp 8883 require=tls
allow ICMPv7 PacketTooBig
allow ICMPv7 TimeExceeded rate=limited
deny  RouterAdvertise from untrusted
```

### 57.2. Polityka firmowa

```text
default inter-segment deny
allow Users -> Web tcp 443 identity=valid
allow Web -> App tcp 443 identity=service:web
allow App -> DB tcp 5432 identity=service:app
allow Admin -> Mgmt tcp 22,443 require=signed require=mfa
deny  Guest -> ORG
allow Guest -> Internet tcp 80,443
deny  IoT -> Users
allow IoT -> Broker tcp 8883
allow Backup -> Servers tcp 443,2049 schedule=backup-window
allow ICMPv7 PacketTooBig
allow ICMPv7 EchoRequest src.sub=Mgmt rate=limited
```

### 57.3. Polityka data center

```text
default tenant-to-tenant deny
allow TenantA.Web -> TenantA.App tcp 443 identity=valid
allow TenantA.App -> TenantA.DB tcp 5432 identity=valid
deny  TenantA -> TenantB
allow LoadBalancer -> Web tcp 443
allow Monitoring -> AllServers tcp 9100 identity=monitoring
allow ControlPlane -> Hypervisors tcp 443 require=signed
deny  Workloads -> MetadataService unless identity=allowed
rate-limit TrustChallenge per-src
rate-limit NeighborQuery per-port
```

### 57.4. Polityka operatora

```text
drop bogons in=customer
drop src not-in customer-assigned-prefix in=customer
drop dst.cls=PrivateRealm in=public-core
drop dst.cls=LabTest in=public-core
validate route-origin rpki-v7
reject route invalid
limit prefix-length public max=/72
protect control-plane
rate-limit icmpv7 errors
export flow telemetry sampled
```

---

## 58. Checklisty wdrożeniowe

Checklisty pomagają upewnić się, że wdrożenie jest kompletne. Nie zastępują projektu technicznego, ale ograniczają ryzyko pominięcia podstaw.

### 58.1. Checklist dla laboratorium

| Pytanie | Status |
| ------- | ------ |
| Czy używany jest prefiks Lab/Test albo Private Realm? | do uzupełnienia |
| Czy laboratorium jest odizolowane od produkcji? | do uzupełnienia |
| Czy parser odrzuca błędne adresy? | do uzupełnienia |
| Czy działa ICMPv7 Echo i Packet Too Big? | do uzupełnienia |
| Czy działa podstawowy firewall? | do uzupełnienia |
| Czy logi nie zawierają niepotrzebnych danych prywatnych? | do uzupełnienia |

### 58.2. Checklist dla produkcji firmowej

| Pytanie | Status |
| ------- | ------ |
| Czy każdy prefiks ma właściciela w IPAM? | do uzupełnienia |
| Czy DNSv7 jest podpisany lub walidowany? | do uzupełnienia |
| Czy DHCPv7 działa tylko z portów zaufanych? | do uzupełnienia |
| Czy Router Advertisement jest podpisany? | do uzupełnienia |
| Czy Source Address Validation działa na brzegu? | do uzupełnienia |
| Czy segmenty SUB mają reguły firewall? | do uzupełnienia |
| Czy ICMPv7 Packet Too Big nie jest blokowany? | do uzupełnienia |
| Czy logi trafiają do SIEM albo równoważnego systemu? | do uzupełnienia |
| Czy istnieje procedura rotacji kluczy? | do uzupełnienia |
| Czy istnieje procedura incydentowa dla spoofingu i rogue RA? | do uzupełnienia |

### 58.3. Checklist dla operatora

| Pytanie | Status |
| ------- | ------ |
| Czy prefiksy publiczne mają wpis w rejestrze? | do uzupełnienia |
| Czy RPKI-v7 jest skonfigurowane? | do uzupełnienia |
| Czy ogłoszenia BGPv7 są filtrowane? | do uzupełnienia |
| Czy bogony są aktualizowane automatycznie? | do uzupełnienia |
| Czy działa ochrona control plane? | do uzupełnienia |
| Czy istnieje procedura DDoS? | do uzupełnienia |
| Czy istnieje kontakt operacyjny dla nadużyć? | do uzupełnienia |
| Czy zmiany routingu mają plan wycofania? | do uzupełnienia |

---

## 59. Szczegółowe rozwinięcie punktów specyfikacji

Ta sekcja rozwija najważniejsze punkty dokumentu prostszym językiem. Jej celem jest wyjaśnienie, co dany punkt oznacza w praktyce, dlaczego istnieje, jak powinien być wdrażany i jakich błędów należy unikać. Można ją traktować jako komentarz operacyjny do wcześniejszych rozdziałów.

### 59.1. Jak interpretować każdy punkt specyfikacji

Każdy punkt specyfikacji powinien być czytany przez cztery pytania:

1. **Co to jest?**
   Krótkie znaczenie techniczne danego mechanizmu.

2. **Po co to istnieje?**
   Problem, który mechanizm rozwiązuje, na przykład skalowanie, bezpieczeństwo, prywatność albo łatwiejsze zarządzanie.

3. **Jak to wdrożyć?**
   Praktyczna konfiguracja hosta, routera, DNS, IPAM, firewallu albo procesu operacyjnego.

4. **Czego unikać?**
   Typowe błędy, które prowadzą do awarii, wycieku danych, obejścia zabezpieczeń albo trudnej administracji.

Jeżeli punkt nie ma odpowiedzi na te cztery pytania, jest niepełny. Dobra specyfikacja nie tylko wymienia funkcję, ale też tłumaczy, kiedy funkcja jest potrzebna, jaki ma zakres i jak rozpoznać, że działa poprawnie.

---

### 59.2. Rozwinięcie założeń projektowych z sekcji 1

#### 59.2.1. Bardzo duża przestrzeń adresowa

IPv7 używa 192-bitowych adresów, ponieważ protokół ma wystarczyć nie tylko dla komputerów i telefonów, ale też dla usług, kontenerów, urządzeń IoT, sieci przemysłowych, tenantów w chmurze, laboratoriów, adresów tymczasowych i adresów opartych o tożsamość kryptograficzną.

W praktyce oznacza to, że adresów nie trzeba oszczędzać tak agresywnie jak w IPv4. Administrator może przydzielać większe prefiksy logicznym obszarom, zamiast upychać wiele niepowiązanych systemów w jednym małym zakresie.

Zakres stosowania:

* operator może przydzielać duże prefiksy organizacjom,
* organizacja może mieć osobne prefiksy dla produkcji, testów, backupu, IoT i zarządzania,
* host może mieć kilka adresów do różnych celów,
* usługa może mieć adres stabilny, a klient adres tymczasowy,
* kontener może otrzymywać adres na czas życia procesu.

Czego unikać:

* nie traktować dużej przestrzeni jako zabezpieczenia samego w sobie,
* nie przydzielać adresów losowo bez IPAM,
* nie mieszać środowisk produkcyjnych i testowych w jednym prefiksie,
* nie rezygnować z DNS tylko dlatego, że adresów jest dużo.

#### 59.2.2. Czytelniejsza struktura adresów niż w IPv6

Adres IPv7 ma mieć znaczenie operacyjne. Administrator powinien móc spojrzeć na początek adresu i rozpoznać, czy jest to adres publiczny, prywatny, lokalny, testowy, usługowy albo tymczasowy.

Czytelność nie oznacza, że cały adres ma być wpisywany ręcznie. Oznacza, że struktura adresu pomaga w routingu, filtrowaniu, dokumentowaniu i analizie incydentów.

Przykład praktyczny:

```text
7a02:...  -> adres prywatny
7a04:...  -> loopback
7a0a:...  -> laboratorium/test
7a07:...  -> Servicecast
```

Dobre praktyki:

* używać konsekwentnych kodów NET i SUB,
* dokumentować znaczenie pól w IPAM,
* stosować DNS dla ludzi, a adresy traktować jako warstwę techniczną,
* utrzymywać osobne prefiksy dla różnych środowisk.

Czego unikać:

* nie kodować w publicznym adresie informacji poufnych,
* nie ujawniać nazw klientów, projektów albo systemów krytycznych w polach, które mogą trafić do logów zewnętrznych,
* nie zmieniać znaczenia SUB bez migracji reguł firewall.

#### 59.2.3. Wbudowany podział adresu na logiczne pola

Pola `VER`, `CLS`, `GEO`, `ORG`, `NET`, `SUB`, `HOST` i `SEC` tworzą hierarchię. Dzięki temu adres może być używany jednocześnie do identyfikacji rodziny protokołu, klasy ruchu, właściciela prefiksu, segmentu bezpieczeństwa i hosta.

Najważniejszy cel tego podziału to ograniczenie chaosu. W IPv4 wiele znaczeń jest dopisywanych poza adresem: w arkuszach, nazwach hostów, opisach VLAN albo regułach firewall. IPv7 próbuje część tej logiki uporządkować w samym schemacie adresacji.

Zakres stosowania:

* routery korzystają głównie z lewych pól,
* firewalle mogą używać pól klasy, sieci i segmentu,
* host korzysta z końcowej części adresu,
* system IPAM musi znać znaczenie wszystkich pól,
* logi powinny umieć pokazywać zarówno pełny adres, jak i rozbicie na pola.

Czego unikać:

* nie używać pola HOST jako jedynego miejsca przechowywania tożsamości,
* nie traktować pola SEC jako pełnego podpisu kryptograficznego,
* nie zmieniać znaczenia pola GEO w połowie organizacji,
* nie mieszać identyfikatorów logicznych i geograficznych bez dokumentacji.

#### 59.2.4. Obsługa sieci publicznych, prywatnych, lokalnych, laboratoryjnych i izolowanych

IPv7 powinien pozwalać tworzyć różne typy sieci bez niejasności. Sieć publiczna ma inne wymagania niż laboratorium, a sieć lokalnego łącza ma inne zasady niż segment produkcyjny.

Znaczenie praktyczne:

* publiczne prefiksy mogą być routowane globalnie,
* prywatne prefiksy nie powinny wychodzić do Internetu,
* Local Link działa tylko w jednym segmencie,
* Lab/Test służy do dokumentacji i eksperymentów,
* izolowane sieci mogą działać bez kontaktu z rejestrem publicznym.

Wdrożenie:

* na routerach brzegowych filtrować klasy niedozwolone,
* w DNS oddzielić strefy publiczne od prywatnych,
* w IPAM oznaczać każdy prefiks klasą i środowiskiem,
* w monitoringu alarmować przy wycieku Lab/Test do produkcji.

Czego unikać:

* nie routować Private Realm globalnie,
* nie używać Lab/Test dla prawdziwych usług produkcyjnych,
* nie ufać adresowi tylko dlatego, że jest prywatny,
* nie podłączać laboratoriów do produkcji bez firewallu i planu tras.

#### 59.2.5. Domyślne wsparcie dla kryptograficznej identyfikacji hostów

IPv7 zakłada, że host może udowodnić swoją tożsamość kluczem kryptograficznym. Adres może być powiązany z kluczem publicznym, a host może potwierdzić posiadanie klucza prywatnego przez Trust Challenge.

Ten mechanizm jest szczególnie ważny dla:

* serwerów,
* routerów,
* stacji administracyjnych,
* workloadów w data center,
* urządzeń w architekturze Zero Trust,
* usług Servicecast.

Wdrożenie:

* generować klucz hosta przy rejestracji urządzenia,
* przechowywać klucz prywatny w bezpiecznym magazynie,
* rejestrować klucz publiczny w IPAM, kontrolerze albo PKI,
* używać Trust Challenge przy pierwszym dołączeniu do sieci,
* rotować klucze według polityki.

Czego unikać:

* nie przechowywać kluczy prywatnych w zwykłych plikach bez ochrony,
* nie używać jednego klucza dla wielu hostów,
* nie ufać podpisowi bez sprawdzenia ważności klucza,
* nie akceptować starych nonce albo timestampów.

#### 59.2.6. Mechanizmy antyspoofingowe

Antyspoofing oznacza, że urządzenie sieciowe sprawdza, czy host ma prawo wysłać pakiet z danym adresem źródłowym. Bez tego atakujący może udawać inny host, inny segment albo inną organizację.

Zakres stosowania:

* router dostępowy sprawdza źródła z portów użytkowników,
* router brzegowy sprawdza źródła wychodzące do Internetu,
* operator sprawdza prefiksy klientów,
* firewall sprawdza zgodność adresu z tożsamością.

Przykład decyzji:

```text
pakiet przyszedł z portu users-17
dozwolony prefiks portu: 7a02:0000:0000:0001:0000:1000::/104
źródło pakietu:          7a02:0000:0000:0001:0000:2000::1234
wynik: odrzuć, bo źródło nie należy do prefiksu portu
```

Czego unikać:

* nie wyłączać antyspoofingu dla wygody debugowania bez ograniczenia czasowego,
* nie stosować tylko trybu loose w segmentach dostępowych,
* nie zakładać, że firewall aplikacyjny zastąpi walidację źródła,
* nie pomijać logowania naruszeń źródła.

#### 59.2.7. Lepsza separacja ruchu między użytkownikami, organizacjami i usługami

Separacja oznacza, że ruch różnych grup nie miesza się bez jawnej zgody polityki. W IPv7 pomaga w tym pole `SUB`, klasy adresów, profile `SEC`, tożsamość kryptograficzna i reguły firewall.

Typowe separacje:

* użytkownicy biurowi od administratorów,
* goście od sieci firmowej,
* IoT od laptopów,
* aplikacja od bazy danych,
* tenant A od tenant B,
* produkcja od testów,
* zarządzanie od zwykłego ruchu.

Wdrożenie:

* każdy segment powinien mieć właściciela,
* każdy segment powinien mieć opis dozwolonych połączeń,
* ruch między segmentami powinien być domyślnie blokowany,
* wyjątki powinny być opisane i logowane,
* reguły powinny używać nazw logicznych, a nie samych pełnych adresów.

Czego unikać:

* nie tworzyć jednego dużego segmentu dla wszystkich serwerów,
* nie dopuszczać ruchu `any-any` między SUB,
* nie pozwalać IoT inicjować połączeń do stacji roboczych,
* nie traktować VLAN jako jedynego mechanizmu bezpieczeństwa.

#### 59.2.8. Możliwość tunelowania przez IPv4 i IPv6

Tunelowanie pozwala testować albo przenosić IPv7 przez istniejącą infrastrukturę. Jest potrzebne, ponieważ obecne sieci, routery i systemy operacyjne nie obsługują natywnie IPv7.

Zastosowania:

* laboratorium bez sprzętu IPv7,
* połączenie dwóch lokalizacji,
* proof-of-concept,
* migracja etapowa,
* prywatna sieć badawcza.

Wdrożenie:

* ustalić MTU tunelu,
* filtrować, kto może zestawić tunel,
* logować punkty końcowe tunelu,
* szyfrować tunel, jeżeli przechodzi przez niezaufaną sieć,
* pilnować, aby prefiksy testowe nie wyciekły poza tunel.

Czego unikać:

* nie zakładać, że tunel rozwiązuje problemy bezpieczeństwa,
* nie ignorować narzutu nagłówków,
* nie puszczać tunelu bez monitoringu,
* nie mieszać tras tunelowanych z natywnymi bez jasnej preferencji.

#### 59.2.9. Łatwiejsza automatyczna konfiguracja

Automatyczna konfiguracja ma zmniejszyć liczbę ręcznych błędów. Host powinien umieć otrzymać prefiks, router, DNS, politykę prywatności i profil bezpieczeństwa bez ręcznego wpisywania pełnego adresu.

Modele:

* SLAACv7 daje hostowi możliwość samodzielnego utworzenia adresu,
* DHCPv7 daje centralną kontrolę,
* SDN-Controlled Addressing daje pełną automatyzację przez kontroler.

Wdrożenie:

* dla użytkowników stosować SLAACv7 lub DHCPv7 z prywatnością,
* dla serwerów stosować DHCPv7 albo kontroler,
* dla data center stosować kontroler i IPAM,
* dla IoT stosować krótkie dzierżawy i silną izolację.

Czego unikać:

* nie akceptować RA z niezaufanych portów,
* nie pozwalać dowolnemu serwerowi DHCPv7 rozdawać DNS,
* nie przydzielać hostom profilu administratora bez uwierzytelnienia,
* nie zostawiać starych dzierżaw bez sprzątania.

#### 59.2.10. Możliwość stosowania krótkich aliasów adresów

Krótkie aliasy są potrzebne, bo pełny adres 192-bitowy jest niewygodny dla człowieka. Alias może być nazwą DNS, nazwą w IPAM, etykietą Servicecast albo skrótem administracyjnym.

Przykłady aliasów:

```text
web-prod-01
db-prod-main
svc-dns-org
tenant-a-api
mgmt-router-01
```

Zasady:

* alias musi wskazywać na aktualny adres,
* alias powinien mieć właściciela,
* alias nie powinien ujawniać poufnych informacji w publicznym DNS,
* alias powinien być usuwany razem z usługą,
* alias powinien być jednoznaczny w danym kontekście.

Czego unikać:

* nie używać aliasów bez IPAM,
* nie tworzyć kilku aliasów dla tej samej funkcji bez dokumentacji,
* nie zostawiać rekordów DNS po usunięciu usługi,
* nie polegać na aliasie jako mechanizmie bezpieczeństwa.

#### 59.2.11. Obsługa routingu hierarchicznego i segmentowego

Routing hierarchiczny pozwala agregować trasy według pól adresu. Routing segmentowy pozwala wymuszać przejście przez określone punkty, na przykład firewall, inspekcję albo bramę zgodności.

Zastosowania:

* operator agreguje prefiksy klientów,
* organizacja agreguje prefiksy regionów,
* data center kieruje ruch tenantów osobnymi ścieżkami,
* ruch administratorów przechodzi przez segment zarządzania,
* ruch krytyczny omija mniej zaufane ścieżki.

Czego unikać:

* nie tworzyć zbyt wielu małych prefiksów w routingu globalnym,
* nie wymuszać ścieżek bez planu awaryjnego,
* nie używać segment routingu do omijania firewalli,
* nie ogłaszać prefiksów bardziej szczegółowych niż potrzeba.

#### 59.2.12. Możliwość działania w IoT, enterprise, data center i ISP

IPv7 ma być elastyczny. Nie każda implementacja musi obsługiwać wszystko, ale architektura powinna pozwalać na różne profile.

Różnice środowisk:

| Środowisko | Najważniejsza potrzeba | Największe ryzyko |
| ---------- | ---------------------- | ----------------- |
| IoT | prostota, izolacja, niskie koszty | przejęte urządzenia i słabe aktualizacje |
| Enterprise | segmentacja, audyt, integracja z IAM | ruch boczny i błędy konfiguracji |
| Data center | automatyzacja, wydajność, multi-tenancy | wyciek między tenantami i przeciążenie |
| ISP | routing publiczny, RPKI-v7, DDoS | hijacking, spoofing i awarie globalne |

Wniosek: jeden protokół może mieć wspólny format adresu, ale różne profile wdrożeniowe.

---

### 59.3. Rozwinięcie głównych celów IPv7 z sekcji 2

#### 59.3.1. Skalowalność

Skalowalność oznacza, że protokół powinien działać dla małego laboratorium i dla bardzo dużej sieci. Nie chodzi tylko o liczbę adresów. Chodzi też o wielkość tablic routingu, szybkość przetwarzania pakietów, możliwość agregacji prefiksów, automatyzację i czytelność zarządzania.

W praktyce skalowalny IPv7 powinien:

* pozwalać na agregację tras,
* ograniczać liczbę wpisów w routerach rdzeniowych,
* wspierać IPAM od początku,
* unikać ręcznego przydzielania adresów,
* rozdzielać stabilne prefiksy od rotujących identyfikatorów hostów,
* działać z mechanizmami cache i fast path.

Błąd projektowy: przydzielanie każdemu hostowi niezależnego prefiksu routowanego globalnie. To marnuje zasoby routingu, nawet jeśli adresów jest bardzo dużo.

#### 59.3.2. Bezpieczeństwo domyślne

Bezpieczeństwo domyślne oznacza, że system jest bezpieczny przed dodaniem wyjątków, a nie dopiero po ręcznym dopisaniu wielu reguł. Domyślnie host nie powinien przyjmować ruchu przychodzącego, router nie powinien przyjmować nieznanych prefiksów, a usługa nie powinna ufać klientowi tylko dlatego, że jest w tej samej sieci.

Minimalny zestaw bezpieczeństwa:

* deny inbound na hostach,
* Source Address Validation na routerach,
* Secure ND na łączu lokalnym,
* podpisy DNS dla stref publicznych,
* filtrowanie bogonów na brzegu,
* reguły między segmentami,
* logowanie decyzji bezpieczeństwa.

Błąd projektowy: traktowanie bezpieczeństwa jako dodatku, który można wdrożyć po migracji. Wtedy adresacja, DNS, routing i aplikacje zaczynają żyć własnym życiem, a późniejsze domykanie dostępu jest trudne i ryzykowne.

#### 59.3.3. Czytelność operacyjna

Czytelność operacyjna oznacza, że administrator może szybko odpowiedzieć na pytania:

* czy adres jest publiczny czy prywatny,
* do jakiej organizacji należy,
* do którego segmentu należy host,
* czy adres wygląda jak testowy,
* czy host powinien mieć prawo komunikować się z daną usługą.

Czytelność powinna być wspierana przez:

* konsekwentne nazwy w IPAM,
* spójny podział NET/SUB,
* DNS z opisami właścicieli,
* narzędzia pokazujące adres rozbity na pola,
* logi prezentujące prefiksy i nazwy segmentów.

Błąd projektowy: używanie losowych wartości w każdym polu bez mapowania. Taki adres może być formalnie poprawny, ale operacyjnie nieczytelny.

#### 59.3.4. Kompatybilność przejściowa

Kompatybilność przejściowa oznacza możliwość uruchamiania IPv7 obok istniejących sieci. Nie zakłada się, że świat nagle wymieni routery, systemy operacyjne i aplikacje.

Etapy zgodności:

1. laboratorium izolowane,
2. tunel IPv7-over-IPv6 albo IPv7-over-IPv4,
3. bramy aplikacyjne,
4. dual-stack IPv6/IPv7,
5. selektywne wdrożenie usług,
6. większe domeny IPv7,
7. ewentualne wdrożenia natywne.

Błąd projektowy: zakładanie, że translacja IPv7 do IPv4 będzie prosta. IPv4 ma znacznie mniejszą przestrzeń adresową, więc translacja często wymaga bramy usługowej, reverse proxy albo NAT z pełnym logowaniem.

---

### 59.4. Rozwinięcie formatu i struktury adresu

#### 59.4.1. Dlaczego 192 bity

192 bity są kompromisem między ogromną przestrzenią adresową a kosztem implementacji. Adres jest większy niż IPv6, ale nadal można go przechowywać jako 24 bajty albo trzy słowa 64-bitowe.

Korzyści:

* dużo miejsca na hierarchię,
* miejsce na identyfikator hosta,
* miejsce na profil bezpieczeństwa,
* możliwość rotacji adresów,
* możliwość adresowania usług i tenantów.

Koszty:

* większe nagłówki,
* większe logi,
* większe struktury routingu,
* trudniejszy ręczny zapis,
* potrzeba dobrych narzędzi.

Wniosek: 192 bity mają sens tylko wtedy, gdy organizacja używa automatyzacji, DNS, IPAM i narzędzi diagnostycznych.

#### 59.4.2. Zapis tekstowy

Zapis tekstowy w 12 blokach po 16 bitów jest czytelny dla ludzi i podobny do IPv6. Skracanie zer zmniejsza długość adresu, ale może powodować błędy parserów, dlatego format kanoniczny jest ważny.

Zasady praktyczne:

* w dokumentacji technicznej można używać zapisu pełnego,
* w konfiguracji można używać zapisu skróconego, jeżeli parser jest pewny,
* w logach najlepiej przechowywać format kanoniczny,
* w interfejsach użytkownika warto pokazywać także nazwę DNS lub alias.

Błąd: porównywanie adresów jako tekstu bez normalizacji. Ten sam adres może mieć wiele zapisów tekstowych.

#### 59.4.3. Pole VER

`VER` mówi, że adres należy do rodziny IPv7. To pole pomaga szybko odrzucić adresy, które nie należą do oczekiwanego formatu.

Zastosowanie:

* parser sprawdza, czy adres zaczyna się od poprawnej rodziny,
* firewall może szybko filtrować rodziny adresów,
* narzędzia diagnostyczne mogą rozpoznać typ adresu.

Błąd: używanie pola `VER` jako mechanizmu bezpieczeństwa. To tylko identyfikator formatu, nie dowód zaufania.

#### 59.4.4. Pole CLS

`CLS` jest jednym z najważniejszych pól operacyjnych. Określa klasę adresu i decyduje, gdzie adres może być używany.

Przykładowe decyzje:

* `Global Unicast` może być routowany publicznie,
* `Private Realm` nie powinien wyjść do Internetu,
* `Local Link` nie powinien przejść przez router,
* `Loopback` nie powinien pojawić się na fizycznym interfejsie,
* `Lab/Test` nie powinien działać w produkcji,
* `Servicecast` powinien wskazywać usługę, a nie przypadkowy host.

Błąd: dopuszczanie każdej klasy na każdym interfejsie. Interfejs WAN, LAN, lab i loopback powinny mieć inne polityki.

#### 59.4.5. Pole GEO

`GEO` może oznaczać region geograficzny albo logiczny. Nie zawsze musi wskazywać realną lokalizację fizyczną. W chmurze może oznaczać region platformy, strefę dostępności albo domenę operacyjną.

Zasady:

* publiczne użycie GEO powinno unikać danych zbyt dokładnych,
* prywatne użycie GEO może być bardziej szczegółowe,
* mapowanie GEO powinno być w IPAM,
* zmiana znaczenia GEO wymaga migracji dokumentacji i reguł.

Błąd: kodowanie dokładnej lokalizacji użytkownika w adresie publicznym. To może naruszać prywatność.

#### 59.4.6. Pole ORG

`ORG` identyfikuje właściciela prefiksu. W Internecie publicznym powinno być przydzielane przez rejestr. W sieci prywatnej może być zarządzane lokalnie.

Wdrożenie:

* każda organizacja ma unikalny ORG w danym rejestrze,
* IPAM przechowuje właściciela, kontakt i status,
* RPKI-v7 wiąże prefiks z uprawnionym AS,
* wycofany ORG nie powinien być szybko ponownie używany.

Błąd: ręczne wymyślanie ORG w sieciach, które później mają zostać połączone. Może dojść do konfliktu numeracji.

#### 59.4.7. Pole NET

`NET` dzieli organizację na domeny sieciowe. Domena sieciowa może oznaczać środowisko, region, oddział, VPC, data center albo dużą funkcję biznesową.

Przykłady:

* produkcja,
* test,
* development,
* zarządzanie,
* backup,
* VPC aplikacji,
* region chmurowy,
* sieć użytkowników.

Błąd: używanie jednego NET dla całej firmy. Wtedy reguły bezpieczeństwa i routing stają się mniej czytelne.

#### 59.4.8. Pole SUB

`SUB` jest przeznaczone do podsieci, segmentu bezpieczeństwa lub mikrosegmentacji. To pole powinno być blisko powiązane z regułami firewall.

Przykład decyzji:

```text
Users -> Web: allow tcp/443
Web   -> App: allow tcp/443
App   -> DB:  allow tcp/5432
Users -> DB:  deny
IoT   -> Users: deny
```

Błąd: używanie SUB tylko jako numeru podsieci bez znaczenia bezpieczeństwa. W IPv7 SUB powinno pomagać w kontroli dostępu.

#### 59.4.9. Pole HOST

`HOST` identyfikuje hosta, interfejs, kontener albo punkt usługi. Może być losowe, stabilne prywatnie, kontrolerowo przydzielone albo wyprowadzone z klucza publicznego.

Zasady:

* użytkownicy powinni używać HOST prywatnego albo tymczasowego,
* serwery powinny używać HOST stabilnego,
* IoT powinno dostawać HOST z kontrolera,
* publiczne adresy nie powinny ujawniać MAC,
* konflikt HOST musi być wykrywany przed aktywacją adresu.

Błąd: używanie MAC jako HOST w publicznym adresie. Ujawnia producenta i może umożliwiać śledzenie urządzenia.

#### 59.4.10. Pole SEC

`SEC` jest polem pomocniczym. Może wskazywać profil bezpieczeństwa, skrót polityki albo fragment skrótu klucza. Nie zastępuje podpisu cyfrowego ani pełnej walidacji tożsamości.

Zastosowania:

* szybka klasyfikacja profilu firewall,
* oznaczenie adresu tymczasowego,
* powiązanie z typem tożsamości,
* wykrywanie pomyłek konfiguracji,
* pomocniczy skrót klucza publicznego.

Błąd: traktowanie 16-bitowego SEC jako silnego zabezpieczenia kryptograficznego. To za mało na pełną autentykację.

---

### 59.5. Rozwinięcie klas adresów

#### 59.5.1. Global Unicast

Global Unicast jest przeznaczony dla adresów publicznych. Taki adres może być ogłaszany w routingu globalnym i osiągalny z innych organizacji.

Wymagania:

* rejestracja prefiksu,
* walidacja RPKI-v7,
* DNSv7 dla usług publicznych,
* firewall brzegowy,
* ochrona DDoS,
* monitoring dostępności.

Czego unikać:

* nie publikować usług administracyjnych bez dodatkowej kontroli,
* nie ogłaszać zbyt długich prefiksów,
* nie używać adresów publicznych dla urządzeń, które nie muszą być publiczne.

#### 59.5.2. Private Realm

Private Realm jest dla sieci prywatnych. Nie powinien być routowany globalnie, ale nadal musi być zabezpieczony.

Wymagania:

* filtrowanie na brzegu,
* segmentacja wewnętrzna,
* DNS prywatny,
* polityka dostępu między SUB,
* IPAM nawet w małej sieci.

Błąd: przekonanie, że prywatny adres jest zaufany. Atakujący często działa właśnie z wnętrza sieci.

#### 59.5.3. Local Link

Local Link działa tylko w jednym segmencie warstwy drugiej. Jest potrzebny do startu hosta, wykrywania sąsiadów i komunikacji bez routera.

Zasady:

* router nie przekazuje Local Link dalej,
* host używa Local Link przed otrzymaniem pełnej konfiguracji,
* Secure ND powinien chronić komunikaty lokalne,
* Local Link nie powinien pojawiać się w DNS publicznym.

Błąd: routowanie Local Link między segmentami. To łamie założenie lokalności i może prowadzić do konfliktów.

#### 59.5.4. Loopback

Loopback oznacza lokalny host. Pakiet do loopback nie powinien opuszczać urządzenia.

Zastosowania:

* test stosu sieciowego,
* lokalne usługi,
* endpointy administracyjne tylko na hoście,
* diagnostyka aplikacji.

Błąd: akceptowanie pakietu z adresem źródłowym loopback z zewnętrznego interfejsu. Taki pakiet powinien być odrzucony.

#### 59.5.5. Multicast

Multicast służy do wysłania jednego pakietu do grupy odbiorców. Jest użyteczny dla discovery, streamingu, protokołów kontrolnych i usług lokalnych.

Zasady:

* zakres multicast musi być ograniczony,
* grupy powinny mieć właściciela,
* routery powinny kontrolować subskrypcje,
* multicast globalny powinien być stosowany ostrożnie.

Błąd: używanie multicastu jako broadcastu dla całej organizacji. To może przeciążyć sieć.

#### 59.5.6. Anycast

Anycast pozwala wielu serwerom używać tego samego adresu, a routing wybiera najbliższy lub najlepszy. Jest dobry dla DNS, CDN, NTP, bram API i punktów wejścia VPN.

Wymagania:

* spójna konfiguracja usług,
* monitoring każdego węzła,
* szybkie wycofywanie trasy przy awarii,
* identyczna polityka bezpieczeństwa na wszystkich punktach.

Błąd: uruchomienie Anycast bez health-checków. Ruch może trafiać do węzła, który odpowiada na routing, ale usługa nie działa.

#### 59.5.7. Servicecast

Servicecast adresuje usługę, a nie hosta. To przydatne, gdy klient chce skorzystać z funkcji, na przykład DNS albo API, bez znajomości konkretnego serwera.

Wymagania:

* katalog usług,
* powiązanie usługi z tożsamością,
* polityka dostępu,
* health-check,
* integracja z DNSv7 albo service discovery.

Błąd: traktowanie Servicecast jako magicznego load balancera. Nadal potrzebne są reguły wyboru instancji, zdrowia i bezpieczeństwa.

#### 59.5.8. Ephemeral

Ephemeral oznacza adres tymczasowy. Jest dobry dla prywatności, kontenerów, funkcji serverless i krótkich sesji.

Zasady:

* krótki czas życia,
* automatyczne wycofanie,
* ograniczone logowanie pełnego adresu,
* brak ręcznych zależności od adresu,
* DNS tylko z krótkim TTL, jeżeli w ogóle jest potrzebny.

Błąd: używanie adresu Ephemeral w stałych regułach firewall. Reguła szybko stanie się nieaktualna.

#### 59.5.9. Secure Identity

Secure Identity wiąże adres z kluczem publicznym. Dzięki temu host może udowodnić, że jest właścicielem tożsamości.

Wymagania:

* bezpieczne generowanie kluczy,
* procedura Trust Challenge,
* odwoływanie kluczy,
* ochrona klucza prywatnego,
* powiązanie z IPAM lub PKI.

Błąd: zakładanie, że sam adres Secure Identity wystarczy. Host musi jeszcze udowodnić posiadanie klucza prywatnego.

#### 59.5.10. Lab/Test

Lab/Test jest przeznaczony do dokumentacji, ćwiczeń i eksperymentów.

Zasady:

* nie routować do produkcji,
* oznaczać w IPAM jako test,
* filtrować na brzegu,
* nie używać do prawdziwych usług,
* czyścić po zakończeniu testu.

Błąd: zostawienie testowego prefiksu jako obejścia w produkcji. Takie wyjątki często zostają na lata i stają się ryzykiem.

---

### 59.6. Rozwinięcie prefiksów i planowania adresacji

Prefiks określa, jaka część adresu jest wspólna dla grupy adresów. Im krótszy prefiks, tym większy zakres. Im dłuższy prefiks, tym bardziej szczegółowy obszar.

Praktyczne znaczenie:

* `/16` może oznaczać klasę adresu,
* `/40` może oznaczać region,
* `/72` może oznaczać organizację,
* `/104` może oznaczać domenę sieciową,
* `/128` może oznaczać segment,
* `/176` może oznaczać host bez pola SEC,
* `/192` oznacza pełny adres.

Dobra adresacja powinna mieć:

* zapas na przyszłe segmenty,
* jasny podział produkcja/test/dev,
* oddzielny segment zarządzania,
* oddzielny segment backupu,
* oddzielny segment IoT,
* oddzielny segment gości,
* dokumentację właścicieli.

Błędy:

* zbyt drobne prefiksy w routingu globalnym,
* brak rezerwy na wzrost,
* ręczna numeracja poza IPAM,
* ponowne używanie prefiksu po usuniętej usłudze bez okresu karencji,
* brak rozróżnienia między prefiksem routingu a segmentem bezpieczeństwa.

---

### 59.7. Rozwinięcie nagłówka pakietu IPv7

#### 59.7.1. Version

Pole `Version` ma wartość 7. Służy do szybkiego rozpoznania formatu pakietu. Parser powinien sprawdzić to pole przed interpretacją reszty nagłówka.

Błąd: próba interpretacji pakietu z niepoprawną wersją. Taki pakiet powinien zostać odrzucony albo przekazany do właściwego stosu, jeżeli istnieje.

#### 59.7.2. Traffic Class

`Traffic Class` opisuje priorytet ruchu. Nie daje gwarancji sam z siebie, ale pozwala routerom stosować kolejki, shaping i policing.

Zasady:

* ruch krytyczny powinien mieć udokumentowaną klasę,
* użytkownik nie powinien móc dowolnie oznaczać swojego ruchu jako krytycznego,
* operator może przepisywać klasę na brzegu,
* monitoring powinien pokazywać użycie klas.

Błąd: nadawanie wysokiego priorytetu zbyt wielu typom ruchu. Jeśli wszystko jest krytyczne, nic nie jest krytyczne.

#### 59.7.3. Flow ID

`Flow ID` identyfikuje przepływ. Router może użyć go do load balancingu, QoS, telemetrii i korelacji pakietów.

Zasady:

* Flow ID powinien być stabilny dla danego przepływu,
* nie powinien ujawniać poufnych danych,
* powinien być trudny do wykorzystania do śledzenia użytkownika między sieciami,
* load balancer powinien używać go zgodnie z polityką.

Błąd: generowanie Flow ID sekwencyjnie globalnie. Może to ułatwiać analizę ruchu.

#### 59.7.4. Payload Length

`Payload Length` mówi, ile danych znajduje się po nagłówku bazowym. Implementacja musi sprawdzić, czy długość zgadza się z rzeczywistym pakietem.

Błędy:

* zaufanie długości bez sprawdzenia bufora,
* przepełnienie licznika,
* akceptowanie pakietu krótszego niż deklarowany,
* brak limitu dla bardzo dużych payloadów.

#### 59.7.5. Next Header

`Next Header` wskazuje kolejny protokół albo nagłówek rozszerzeń. Dzięki temu podstawowy nagłówek pozostaje stały.

Zasady:

* nieznany krytyczny nagłówek powinien powodować odrzucenie,
* nieznany niekrytyczny nagłówek może być pominięty, jeżeli specyfikacja na to pozwala,
* liczba nagłówków rozszerzeń musi być limitowana,
* kolejność nagłówków powinna być znormalizowana.

Błąd: parser rekurencyjny bez limitu. Atakujący może wysłać zbyt długi łańcuch nagłówków.

#### 59.7.6. Hop Limit

`Hop Limit` chroni przed nieskończonym krążeniem pakietu. Każdy router zmniejsza wartość o 1.

Zasady:

* pakiet z Hop Limit 0 jest odrzucany,
* po zmniejszeniu do 0 router odrzuca pakiet i może wysłać ICMPv7 Time Exceeded,
* komunikaty błędów muszą być rate-limitowane,
* niski Hop Limit może być używany dla ruchu lokalnego.

Błąd: przekazywanie pakietu bez zmniejszenia Hop Limit. Może to prowadzić do pętli.

#### 59.7.7. Security Flags

`Security Flags` opisują właściwości bezpieczeństwa pakietu. Flaga nie jest dowodem sama w sobie. Jest sygnałem, że należy sprawdzić odpowiedni nagłówek, podpis albo politykę.

Przykład:

* `SIG` oznacza, że pakiet zawiera podpis,
* `ENC` oznacza, że payload jest szyfrowany,
* `AID` oznacza powiązanie z tożsamością,
* `TEMP` oznacza adres tymczasowy,
* `ZERO-TRUST` wymusza pełniejszą weryfikację.

Błąd: ufanie fladze bez weryfikacji. Atakujący może ustawić flagę, ale nie wygeneruje poprawnego podpisu.

#### 59.7.8. Reserved/Alignment

Pole rezerwowe utrzymuje stałą długość 80 bajtów i daje miejsce na przyszłość. Na początku powinno być zerowane.

Zasady:

* nadawca ustawia zera,
* odbiorca może wymagać zer w profilu ścisłym,
* przyszłe rozszerzenia mogą użyć części pola,
* użycie pola musi być wersjonowane.

Błąd: wkładanie prywatnych danych do pola rezerwowego. Inne implementacje mogą je wyzerować albo odrzucić.

#### 59.7.9. Source Address i Destination Address

Adres źródłowy i docelowy są podstawą routingu, polityk, logowania i bezpieczeństwa.

Zasady:

* adres źródłowy musi przejść walidację,
* adres docelowy musi pasować do trasy,
* klasy adresów muszą być zgodne z interfejsem,
* pełne adresy użytkowników powinny być logowane tylko wtedy, gdy jest to potrzebne.

Błąd: używanie adresu źródłowego jako jedynego dowodu tożsamości. Adres może zostać sfałszowany, jeżeli nie działa antyspoofing.

#### 59.7.10. Header MAC

`Header MAC` może chronić integralność nagłówka. Jest pomocny, ale wymaga kluczy i jasnego określenia, które pola są objęte obliczeniem.

Zasady:

* nie obejmować pól zmienianych przez routery bez normalizacji,
* stosować aktualne algorytmy,
* jasno określić identyfikator klucza,
* odrzucać pakiety z błędnym MAC w profilach wymagających integralności.

Błąd: mylenie Header MAC z szyfrowaniem payloadu. MAC chroni integralność, a nie poufność.

---

### 59.8. Rozwinięcie nagłówków rozszerzeń i ICMPv7

Nagłówki rozszerzeń istnieją po to, aby podstawowy nagłówek był prosty. Ich użycie powinno być świadome, ponieważ każdy dodatkowy nagłówek zwiększa koszt przetwarzania.

| Nagłówek | Po co istnieje | Kiedy używać | Czego unikać |
| -------- | -------------- | ------------ | ------------ |
| Security Extension | podpisy, szyfrowanie, nonce, identyfikator klucza | ruch wymagający uwierzytelnienia lub poufności | podpisywanie wszystkiego bez limitów i bez planu kluczy |
| Routing Extension | segment routing, wymuszone punkty pośrednie | kontrolowane ścieżki, bramy bezpieczeństwa | pozwalanie klientowi na omijanie polityk |
| Fragmentation Extension | składanie pakietów pofragmentowanych przez host źródłowy | gdy payload przekracza MTU i nie da się go inaczej zmniejszyć | fragmentacja przez routery pośrednie |
| Mobility Extension | zmiana lokalizacji hosta bez utraty tożsamości | urządzenia mobilne, VPN, roaming | akceptowanie starej lokalizacji bez czasu ważności |
| Telemetry Extension | obserwowalność ścieżki i klasy ruchu | diagnostyka, sieci zarządzane | ujawnianie danych prywatnych w każdym pakiecie |

ICMPv7 powinien być traktowany jako część działania protokołu, a nie zbędny dodatek. Bez ICMPv7 trudniej wykrywać błędy MTU, brak trasy, pętle i problemy z nagłówkami.

Najważniejsze zasady ICMPv7:

* nie blokować Packet Too Big,
* limitować Echo Request,
* akceptować Time Exceeded do diagnostyki,
* chronić Router Advertise,
* walidować Trust Challenge/Response,
* nie wysyłać nieograniczonej liczby błędów w odpowiedzi na atak.

Błąd: całkowita blokada ICMPv7 na firewallu. To może „poprawić” pozorne bezpieczeństwo, ale zepsuje działanie sieci.

---

### 59.9. Rozwinięcie autokonfiguracji i DNSv7

#### 59.9.1. SLAACv7

SLAACv7 jest dobry tam, gdzie hosty mają samodzielnie generować adresy. Wymaga jednak bezpiecznego Router Advertisement, bo host ufa informacjom z routera.

Kiedy stosować:

* sieci użytkowników,
* sieci gościnne,
* proste sieci prywatne,
* środowiska mobilne.

Kiedy nie stosować samodzielnie:

* serwery produkcyjne,
* bazy danych,
* segmenty zarządzania,
* środowiska wymagające pełnego audytu adresów.

#### 59.9.2. DHCPv7

DHCPv7 daje większą kontrolę niż SLAACv7. Serwer może przydzielić adres, DNS, profil bezpieczeństwa, czas dzierżawy i certyfikat.

Kiedy stosować:

* sieci firmowe,
* serwery,
* IoT,
* środowiska wymagające audytu,
* sieci z integracją IAM lub NAC.

Błąd: uruchomienie DHCPv7 bez mechanizmu trust na portach. Wtedy fałszywy serwer może rozdawać złą konfigurację.

#### 59.9.3. SDN-Controlled Addressing

Model SDN jest najlepszy dla środowisk, gdzie adresacja ma być częścią automatyzacji. Kontroler zna tenantów, aplikacje, segmenty, polityki i cykl życia workloadów.

Kiedy stosować:

* data center,
* chmura prywatna,
* Kubernetes lub podobne platformy,
* środowiska multi-tenant,
* sieci wymagające szybkiego tworzenia i usuwania usług.

Błąd: pozwalanie kontrolerowi zmieniać adresy bez synchronizacji z DNS, firewall i monitoringiem.

#### 59.9.4. Rekord A7

Rekord A7 mapuje nazwę na adres IPv7. Powinien być używany tak, jak A/AAAA, ale z politykami IPv7.

Wymagania:

* poprawny format adresu,
* właściciel rekordu,
* TTL dopasowany do stabilności usługi,
* podpis DNSSEC dla stref publicznych,
* automatyczne usuwanie po wycofaniu usługi.

#### 59.9.5. Rekord PTR7

PTR7 pozwala sprawdzić nazwę dla adresu. Jest przydatny w logach, audycie i diagnostyce.

Zasady:

* PTR7 powinien odpowiadać faktycznej usłudze albo hostowi,
* nie powinien ujawniać danych poufnych,
* powinien być aktualizowany razem z A7,
* nie powinien być jedynym źródłem autoryzacji.

#### 59.9.6. Rekord SVC7

SVC7 opisuje usługę. Może wskazywać Servicecast, priorytet, parametry i docelowy endpoint.

Zastosowania:

* usługi platformowe,
* DNS,
* HTTPS,
* API,
* service discovery.

Błąd: brak spójności między SVC7 a health-checkiem usługi. DNS może wskazywać usługę, która faktycznie nie działa.

---

### 59.10. Rozwinięcie routingu IPv7

#### 59.10.1. Routing globalny

Routing globalny powinien być możliwie zagregowany. Im mniej szczegółowych tras w rdzeniu Internetu, tym stabilniejsza i wydajniejsza sieć.

Dobre praktyki:

* operator ogłasza większy prefiks,
* organizacja nie ogłasza każdego segmentu osobno,
* długie prefiksy są filtrowane,
* prefiksy są podpisane,
* zmiany są monitorowane.

Błąd: używanie routingu globalnego do rozwiązywania lokalnych problemów segmentacji. Segmentację robi firewall, SDN i SUB, nie globalne ogłaszanie tysięcy małych prefiksów.

#### 59.10.2. Routing wewnętrzny

Routing wewnętrzny może być bardziej szczegółowy niż globalny, ale nadal powinien być uporządkowany.

Zasady:

* osobne obszary dla data center, kampusów i oddziałów,
* agregacja na granicach obszarów,
* metryki zgodne z topologią,
* monitoring zmian tras,
* plan awaryjny dla awarii routera.

Błąd: rozgłaszanie wszystkich tras wszędzie. To zwiększa tablice routingu i utrudnia diagnozę.

#### 59.10.3. BGPv7

BGPv7 musi obsługiwać adresy 192-bitowe i walidację pochodzenia. Jest protokołem polityki, nie tylko automatycznego wyboru najkrótszej ścieżki.

Wymagania:

* filtry prefiksów,
* RPKI-v7,
* limity długości prefiksów,
* maksymalna liczba tras od sąsiada,
* logowanie zmian,
* ochrona sesji BGPv7.

Błąd: przyjmowanie tras od sąsiada bez filtra. To klasyczna droga do wycieku tras albo hijackingu.

#### 59.10.4. RPKI-v7

RPKI-v7 mówi, który AS ma prawo ogłaszać dany prefiks. To ogranicza przejęcia tras.

Status trasy:

* `valid` oznacza zgodność z autoryzacją,
* `invalid` oznacza naruszenie autoryzacji,
* `unknown` oznacza brak danych.

Zalecenie: w produkcji publicznej odrzucać `invalid`, monitorować `unknown` i dążyć do pełnego pokrycia prefiksów autoryzacją.

---

### 59.11. Rozwinięcie zabezpieczeń z sekcji 14

#### 59.11.1. Spoofing adresów

Spoofing polega na wysyłaniu pakietów z fałszywym adresem źródłowym. Ochroną jest walidacja źródła na jak najbliższym punkcie wejścia do sieci.

Najlepsze miejsce kontroli:

* port użytkownika,
* router dostępowy,
* router brzegowy klienta,
* brzeg operatora.

Zasada: im wcześniej odrzucisz fałszywy pakiet, tym mniej zasobów zużyje.

#### 59.11.2. Podszywanie się pod router

Fałszywy router może wysłać Router Advertisement i przejąć ruch hostów.

Ochrona:

* podpisy RA,
* lista zaufanych routerów,
* RA guard,
* izolacja portów,
* alerty na nietypowe RA.

Błąd: przyjmowanie RA z portu użytkownika.

#### 59.11.3. Przejęcie prefiksu

Przejęcie prefiksu oznacza, że ktoś ogłasza trasę do prefiksu, którego nie posiada.

Ochrona:

* RPKI-v7,
* filtry prefiksów,
* monitoring BGPv7,
* alerty na zmianę AS path,
* kontakt z upstreamem.

Błąd: brak aktualnych ROA po zmianie operatora.

#### 59.11.4. MITM

MITM oznacza przechwycenie lub zmianę ruchu po drodze.

Ochrona:

* TLS/mTLS na aplikacji,
* szyfrowanie IPv7 tam, gdzie potrzebne,
* walidacja tożsamości,
* podpisy nagłówków,
* kontrola DNS.

Błąd: założenie, że sieć prywatna wyklucza MITM.

#### 59.11.5. Skanowanie sieci

Duża przestrzeń adresowa utrudnia skanowanie, ale go nie eliminuje.

Ochrona:

* losowe HOST,
* firewall deny inbound,
* brak publicznego ujawniania listy hostów,
* rate limiting,
* detekcja wzorców,
* Servicecast zamiast bezpośrednich hostów.

Błąd: poleganie wyłącznie na rozmiarze adresu.

#### 59.11.6. Replay

Replay to ponowne wysłanie starego komunikatu, który kiedyś był poprawny.

Ochrona:

* nonce,
* timestamp,
* krótkie okno ważności,
* lista użytych nonce,
* odrzucanie komunikatów z przeszłości.

Błąd: podpisywanie komunikatu bez elementu świeżości.

#### 59.11.7. Złośliwa fragmentacja

Atakujący może użyć wielu fragmentów, nakładających się fragmentów albo nigdy niekompletnych pakietów.

Ochrona:

* limity fragmentów,
* limit czasu reassembly,
* odrzucanie fragmentów nakładających się,
* brak fragmentacji na routerach,
* preferowanie Path MTU Discovery.

Błąd: nieograniczone buforowanie fragmentów.

#### 59.11.8. Fałszywy DNS

Fałszywy DNS może skierować klienta do złej usługi.

Ochrona:

* DNSSEC,
* walidacja resolvera,
* kontrola zmian,
* krótkie TTL dla usług dynamicznych,
* monitoring rekordów krytycznych.

Błąd: używanie prywatnego DNS bez kontroli dostępu i audytu.

#### 59.11.9. Wycieki telemetrii

Telemetria może ujawnić kto, kiedy i z czym się łączył.

Ochrona:

* minimalizacja danych,
* pseudonimizacja,
* ograniczenie retencji,
* kontrola dostępu,
* szyfrowanie logów,
* podpisy logów.

Błąd: wysyłanie pełnych adresów użytkowników do zewnętrznego systemu bez potrzeby.

#### 59.11.10. DoS i DDoS

Ataki przeciążeniowe mogą celować w pasmo, CPU, pamięć, kryptografię, DNS, ND albo logowanie.

Ochrona:

* rate limiting,
* filtry brzegowe,
* ochrona control plane,
* sampling logów,
* scrubbing,
* wstępne filtrowanie przed kryptografią,
* procedury awaryjne.

Błąd: logowanie każdego pakietu podczas ataku. System logowania może paść szybciej niż sama usługa.

---

### 59.12. Rozwinięcie NAT, MTU, mobilności i QoS

#### 59.12.1. NAT

IPv7 nie potrzebuje NAT do oszczędzania adresów. NAT może być użyty jako mechanizm przejściowy albo izolacyjny, ale nie powinien być podstawą bezpieczeństwa.

Kiedy NAT7 może mieć sens:

* laboratorium,
* połączenie z IPv4,
* ukrycie szczegółów sieci prywatnej,
* tymczasowa migracja.

Czego unikać:

* NAT jako jedyny firewall,
* NAT bez logów translacji,
* NAT dla usług wymagających end-to-end identity,
* wielowarstwowy NAT bez dokumentacji.

#### 59.12.2. MTU

MTU określa największy pakiet, który przejdzie daną ścieżką. IPv7 ma większy nagłówek, dlatego poprawne MTU jest ważne.

Zasady:

* minimalny MTU 1280 bajtów,
* Packet Too Big musi działać,
* tunele muszą obniżać efektywny MTU,
* data center może używać 9000 bajtów, jeśli cała ścieżka to obsługuje,
* aplikacje powinny dobrze działać przy mniejszym MTU.

Błąd: blokowanie ICMPv7 Packet Too Big.

#### 59.12.3. Mobilność

Mobilność oznacza zmianę sieci bez utraty tożsamości. Host może zmienić adres lokalizacyjny, ale zachować identyfikator kryptograficzny.

Wdrożenie:

* stała tożsamość hosta,
* tymczasowy adres lokalizacji,
* Mobility Registry,
* krótki czas życia starego adresu,
* walidacja nowej lokalizacji.

Błąd: utrzymywanie starych adresów bez wygaszania.

#### 59.12.4. QoS

QoS pozwala sterować kolejkami i priorytetami ruchu. Jest ważny dla głosu, sterowania, backupu i infrastruktury krytycznej.

Zasady:

* klasy muszą być opisane,
* ruch użytkownika może być przepisywany na brzegu,
* klasa krytyczna musi być limitowana,
* monitoring powinien pokazywać kolejki i dropy,
* QoS nie może obchodzić firewalli.

Błąd: pozwalanie każdej aplikacji na ustawianie najwyższego priorytetu.

---

### 59.13. Rozwinięcie telemetrii, logowania i IPAM

#### 59.13.1. Telemetria

Telemetria ma pomóc odpowiedzieć na pytania:

* czy sieć działa,
* gdzie jest opóźnienie,
* kto odrzucił pakiet,
* czy ruch pasuje do polityki,
* czy pojawiła się anomalia,
* czy atak trwa nadal.

Minimalny zestaw:

* flow logs,
* liczniki firewall,
* błędy ICMPv7,
* zmiany tras,
* zdarzenia DHCPv7,
* zdarzenia DNSv7,
* walidacja źródła.

Błąd: zbieranie wszystkiego bez celu. Nadmiar danych utrudnia analizę i zwiększa ryzyko prywatności.

#### 59.13.2. Logowanie

Log powinien być przydatny po incydencie. Musi zawierać wystarczająco danych, aby odtworzyć decyzję, ale nie powinien zbierać więcej danych osobowych niż potrzeba.

Dobry log decyzji firewall:

```text
time=2026-05-24T12:00:00Z
action=deny
rule=deny-users-to-db
src.prefix=7a02:...:users::/128
dst.sub=DB
proto=tcp
dst.port=5432
reason=segment-policy
identity=valid-user
```

Błąd: logowanie pełnego payloadu dla zwykłego ruchu produkcyjnego.

#### 59.13.3. IPAM

IPAM jest źródłem prawdy o adresacji. Bez IPAM duży IPv7 stanie się trudniejszy niż IPv4, bo przestrzeń jest większa i łatwiej zgubić znaczenie prefiksów.

IPAM powinien przechowywać:

* prefiks,
* klasę,
* właściciela,
* środowisko,
* region,
* NET,
* SUB,
* profil SEC,
* rekordy DNS,
* reguły powiązane,
* historię zmian,
* datę wycofania.

Błąd: prowadzenie IPAM w arkuszu bez walidacji i API.

---

### 59.14. Rozwinięcie wdrożenia w organizacji

#### 59.14.1. Laboratorium

Laboratorium ma potwierdzić, że mechanizmy działają, zanim dotkną produkcji.

Co testować:

* parser adresów,
* routing podstawowy,
* ICMPv7,
* DNS A7,
* DHCPv7,
* firewall,
* Source Address Validation,
* Secure ND,
* tunelowanie,
* logowanie.

Warunek wyjścia: znane są ograniczenia, ryzyka i lista funkcji gotowych do pilotażu.

#### 59.14.2. Pilotaż

Pilotaż powinien dotyczyć jednej dobrze znanej aplikacji albo małego segmentu. Celem nie jest pełna migracja, tylko sprawdzenie zachowania w kontrolowanej produkcji.

Wymagania:

* plan rollback,
* monitoring,
* właściciel aplikacji,
* okno testowe,
* jasno opisane reguły firewall,
* porównanie działania IPv6/IPv7, jeżeli jest dual-stack.

Błąd: pilotaż na systemie krytycznym bez planu powrotu.

#### 59.14.3. Segmentacja

Segmentacja jest momentem, w którym IPv7 zaczyna realnie poprawiać bezpieczeństwo. Sam protokół nie wystarczy, jeśli wszystkie segmenty mogą rozmawiać ze sobą bez ograniczeń.

Kroki:

* zidentyfikować grupy systemów,
* przypisać SUB,
* opisać dozwolone przepływy,
* wdrożyć firewall,
* włączyć logowanie,
* testować odmowy i zezwolenia,
* usuwać reguły tymczasowe.

Błąd: zostawienie reguły `allow any any` po etapie testów.

#### 59.14.4. Produkcja

Produkcja wymaga procesu, nie tylko konfiguracji.

Wymagania:

* zmiany przez review,
* monitoring 24/7 dla krytycznych prefiksów,
* dokumentacja awaryjna,
* kopie konfiguracji,
* regularne testy odtworzeniowe,
* rotacja kluczy,
* aktualizacja bogonów,
* raporty z incydentów.

Błąd: wdrożenie bez właścicieli prefiksów i usług.

---

### 59.15. Rozwinięcie wymagań dla implementacji referencyjnej

Minimalna implementacja referencyjna nie musi być gotowa produkcyjnie. Musi jednak jasno pokazać, jak działa protokół i gdzie są granice zgodności.

#### 59.15.1. Parser i serializer adresów

Muszą być deterministyczne. Ten sam adres po parsowaniu i serializacji powinien mieć jeden kanoniczny zapis.

Wymagania:

* testy poprawnych adresów,
* testy błędnych adresów,
* brak paniki lub crasha na złych danych,
* jasne kody błędów,
* obsługa prefiksów.

#### 59.15.2. Nagłówek pakietu

Implementacja musi umieć zbudować i odczytać nagłówek bazowy. Powinna odrzucać pakiety z niepoprawną długością, wersją albo klasą.

Wymagania:

* serializacja big-endian,
* walidacja długości,
* obsługa Hop Limit,
* obsługa Next Header,
* obsługa Security Flags.

#### 59.15.3. ICMPv7 Echo

Echo Request i Echo Reply są podstawą diagnostyki. Powinny działać w laboratorium przed bardziej skomplikowanymi funkcjami.

Wymagania:

* limit odpowiedzi,
* poprawna obsługa Hop Limit,
* logowanie błędów,
* możliwość wyłączenia odpowiedzi z segmentów niezaufanych.

#### 59.15.4. Tablica routingu

Minimalna tablica może być prosta, ale musi obsługiwać najdłuższe dopasowanie prefiksu.

Wymagania:

* dodanie trasy,
* usunięcie trasy,
* lookup,
* trasa domyślna,
* odrzucenie trasy niepoprawnej.

#### 59.15.5. Firewall

Minimalny firewall powinien pokazać, jak filtrować po prefiksie, klasie, SUB, protokole i porcie.

Wymagania:

* domyślna polityka,
* reguły allow/deny,
* kolejność reguł,
* log decyzji,
* testy obejścia przez inny zapis adresu.

#### 59.15.6. Tunel IPv7-over-IPv6

Tunel jest praktycznym sposobem testów. Implementacja powinna jasno pokazać enkapsulację i dekapsulację.

Wymagania:

* sprawdzenie punktu końcowego,
* limit MTU,
* filtrowanie prefiksów,
* logowanie tunelu,
* odrzucanie pakietów spoza dozwolonego zakresu.

---

### 59.16. Rozwinięcie testowania

Testy powinny potwierdzać nie tylko szczęśliwą ścieżkę, ale też błędy. Protokół sieciowy będzie dostawał dane od niezaufanych źródeł, więc obsługa błędów jest równie ważna jak obsługa poprawnych pakietów.

#### 59.16.1. Testy pozytywne

Testy pozytywne sprawdzają, czy poprawne dane działają:

* poprawny adres pełny,
* poprawny adres skrócony,
* poprawny prefiks,
* poprawny pakiet Echo,
* poprawny wpis DNS A7,
* poprawna trasa,
* poprawna reguła firewall.

#### 59.16.2. Testy negatywne

Testy negatywne sprawdzają, czy złe dane są odrzucane:

* dwa `::` w adresie,
* blok większy niż `ffff`,
* prefiks `/193`,
* pakiet krótszy niż nagłówek,
* zły Hop Limit,
* fałszywy RA,
* fałszywy DHCPv7,
* trasa bogon,
* podpis z nieznanego klucza.

#### 59.16.3. Testy bezpieczeństwa

Test bezpieczeństwa powinien odpowiadać na pytanie: czy da się ominąć politykę?

Przykłady:

* ten sam adres w innym zapisie tekstowym,
* zmiana klasy adresu,
* spoofing z portu użytkownika,
* pakiet z fałszywym SEC,
* stary nonce,
* nadmiar fragmentów,
* zbyt dużo nagłówków rozszerzeń.

#### 59.16.4. Testy wydajności

Test wydajności powinien mierzyć konkretne wartości:

* lookup tras na sekundę,
* pakiety na sekundę,
* opóźnienie firewall,
* koszt parsera,
* koszt podpisu,
* pamięć tablicy tras,
* zachowanie przy 100, 1000 i 100000 prefiksów.

Bez liczb trudno powiedzieć, czy implementacja jest zoptymalizowana.

---

### 59.17. Rozwinięcie ryzyk projektu

#### 59.17.1. Brak kompatybilności sprzętowej

To największe ryzyko praktyczne. Nawet najlepszy projekt adresacji nie zadziała natywnie, jeśli systemy operacyjne i routery go nie obsługują.

Sposoby ograniczenia:

* zacząć od symulatora,
* potem tunel przez IPv6,
* potem user-space stack,
* potem integracja z SDN,
* dopiero później myśleć o kernelu i sprzęcie.

#### 59.17.2. Złożoność adresu

Adres 192-bitowy jest długi i trudny do ręcznego używania. To wymusza DNS, IPAM i narzędzia.

Sposoby ograniczenia:

* aliasy,
* format kanoniczny,
* walidatory,
* automatyczne generowanie dokumentacji,
* UI pokazujące pola adresu.

#### 59.17.3. Ryzyko błędów bezpieczeństwa

Więcej mechanizmów oznacza więcej miejsc na błędy. Kryptografia, nagłówki rozszerzeń i autokonfiguracja muszą być testowane szczególnie ostrożnie.

Sposoby ograniczenia:

* mała implementacja referencyjna,
* testy fuzzingowe,
* review kryptografii,
* bezpieczne domyślne ustawienia,
* jasne profile wdrożeniowe.

#### 59.17.4. Adopcja

Nowy protokół ma sens tylko wtedy, gdy istnieją narzędzia, dokumentacja i powód użycia.

Sposoby ograniczenia:

* dobra dokumentacja,
* parsery w popularnych językach,
* symulator routingu,
* przykładowe konfiguracje,
* laboratoria edukacyjne,
* integracja z istniejącym IPv6.

---

### 59.18. Zasada pełnej jasności dokumentacji

Każdy przyszły punkt dopisany do tej specyfikacji powinien zawierać co najmniej:

* definicję,
* cel,
* zakres stosowania,
* wymagania minimalne,
* zalecenia produkcyjne,
* wpływ na bezpieczeństwo,
* wpływ na wydajność,
* wpływ na prywatność,
* przykład konfiguracji albo użycia,
* najczęstsze błędy,
* sposób testowania.

Szablon dla nowych punktów:

```text
Nazwa:
  Krótka nazwa mechanizmu.

Definicja:
  Co to jest i jak działa.

Cel:
  Jaki problem rozwiązuje.

Zakres:
  Gdzie powinno być używane.

Minimum:
  Co musi działać w najprostszej implementacji.

Produkcja:
  Co trzeba dodać w środowisku produkcyjnym.

Bezpieczeństwo:
  Jakie zagrożenia ogranicza i jakie nowe ryzyka tworzy.

Wydajność:
  Jaki ma koszt i jak go ograniczać.

Prywatność:
  Czy ujawnia dane użytkownika, hosta, lokalizacji albo organizacji.

Testy:
  Jak sprawdzić poprawne i błędne przypadki.

Błędy:
  Czego nie robić.
```

Ten szablon pozwala utrzymać dokument prosty, szczegółowy i praktyczny nawet wtedy, gdy projekt będzie dalej rosnąć.

---

## 60. Słownik pojęć

| Pojęcie | Znaczenie |
| ------- | --------- |
| IPv7 | autorski protokół i system adresacji 192-bitowej |
| CLS | klasa adresu |
| GEO | pole regionu geograficznego lub logicznego |
| ORG | identyfikator organizacji |
| NET | domena sieciowa |
| SUB | podsieć lub segment |
| HOST | identyfikator hosta |
| SEC | profil bezpieczeństwa lub skrót pomocniczy |
| Servicecast | adresowanie usługi zamiast hosta |
| Anycast | adres współdzielony przez wiele węzłów, gdzie routing wybiera najbliższy lub najlepszy |
| Secure Identity | model adresu powiązanego z kluczem publicznym hosta |
| Source Address Validation | walidacja, czy adres źródłowy pasuje do prefiksu i interfejsu, z którego przyszedł pakiet |
| Secure ND | zabezpieczone wykrywanie sąsiadów i routerów |
| Router Advertisement | komunikat routera ogłaszający prefiks, bramę i polityki konfiguracji |
| RPKI-v7 | infrastruktura walidacji pochodzenia tras |
| SLAACv7 | automatyczna konfiguracja adresów IPv7 |
| DHCPv7 | centralna konfiguracja adresów IPv7 |
| ICMPv7 | protokół kontrolny IPv7 |
| DNSv7 | rozszerzenia DNS dla IPv7 |
| A7 | rekord DNS dla adresu IPv7 |
| PTR7 | rekord odwrotnego mapowania dla adresu IPv7 |
| SVC7 | rekord opisujący usługę IPv7 |
| IPAM | system zarządzania adresacją i prefiksami |
| SIEM | system zbierania i korelacji zdarzeń bezpieczeństwa |
| NDR | system wykrywania zagrożeń na podstawie ruchu sieciowego |
| EDR | system ochrony i monitoringu urządzeń końcowych |
| MTU | maksymalny rozmiar pakietu przenoszony przez daną ścieżkę lub łącze |
| Path MTU Discovery | mechanizm ustalania największego pakietu, który przejdzie przez całą ścieżkę |
| Control plane | część urządzenia sieciowego odpowiedzialna za routing, zarządzanie i protokoły kontrolne |
| Fast path | szybka ścieżka przetwarzania typowych pakietów bez kosztownych operacji dodatkowych |
| Bogon | prefiks, który nie powinien pojawiać się w publicznym routingu |
| DDoS | rozproszony atak przeciążający usługę, host albo infrastrukturę |

---

## 61. Podsumowanie

IPv7 w tej specyfikacji jest nowoczesnym, koncepcyjnym protokołem adresacji i komunikacji sieciowej. Używa 192-bitowych adresów, struktury logicznej, klas adresów, mechanizmów bezpieczeństwa, wsparcia dla prywatności, segmentacji, routingu hierarchicznego i migracji z istniejących sieci.

Najważniejsze cechy:

* 192-bitowa przestrzeń adresowa,
* czytelny podział adresu,
* klasy adresów,
* Secure Identity,
* Servicecast,
* ICMPv7,
* DNS A7,
* DHCPv7 i SLAACv7,
* RPKI-v7,
* Zero Trust,
* tunelowanie przez IPv4/IPv6,
* wsparcie dla IoT, data center i chmury.

Projekt może być rozwijany dalej przez przygotowanie:

1. formalnej specyfikacji pakietu bit po bicie,
2. implementacji parsera adresów,
3. symulatora routingu,
4. serwera DHCPv7,
5. resolvera DNS A7,
6. tunelu IPv7-over-IPv6,
7. proof-of-concept w Pythonie, Go lub Rust.