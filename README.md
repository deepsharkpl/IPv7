# IPv7 — autorski projekt adresacji IP i protokołu sieciowego

**Status dokumentu:** projekt koncepcyjny / specyfikacja techniczna
**Nazwa projektu:** IPv7
**Wersja dokumentacji:** 1.0
**Cel:** zaprojektowanie nowoczesnego, bezpiecznego, skalowalnego systemu adresacji i przesyłania pakietów, który może działać równolegle z IPv4 oraz IPv6, a docelowo umożliwiać budowę oddzielnych sieci testowych, prywatnych, laboratoryjnych lub eksperymentalnych.

> Uwaga: nazwa „IPv7” była historycznie używana dla eksperymentalnych propozycji następcy IPv4. Ten dokument opisuje **nowy, autorski projekt IPv7**, niezależny od dawnych propozycji.

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

| Pole                |  Długość | Opis                                     |
| ------------------- | -------: | ---------------------------------------- |
| Version             |   4 bity | wersja protokołu, wartość 7              |
| Traffic Class       |  8 bitów | klasa ruchu/QoS                          |
| Flow ID             | 20 bitów | identyfikator przepływu                  |
| Payload Length      |  32 bity | długość danych po nagłówku               |
| Next Header         |  8 bitów | typ następnego nagłówka                  |
| Hop Limit           |  8 bitów | limit przeskoków                         |
| Security Flags      | 16 bitów | flagi bezpieczeństwa                     |
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

## 41. Słownik pojęć

| Pojęcie     | Znaczenie                                        |
| ----------- | ------------------------------------------------ |
| IPv7        | autorski protokół i system adresacji 192-bitowej |
| CLS         | klasa adresu                                     |
| GEO         | pole regionu geograficznego lub logicznego       |
| ORG         | identyfikator organizacji                        |
| NET         | domena sieciowa                                  |
| SUB         | podsieć lub segment                              |
| HOST        | identyfikator hosta                              |
| SEC         | profil bezpieczeństwa lub skrót pomocniczy       |
| Servicecast | adresowanie usługi zamiast hosta                 |
| RPKI-v7     | infrastruktura walidacji pochodzenia tras        |
| SLAACv7     | automatyczna konfiguracja adresów IPv7           |
| DHCPv7      | centralna konfiguracja adresów IPv7              |
| ICMPv7      | protokół kontrolny IPv7                          |
| DNSv7       | rozszerzenia DNS dla IPv7                        |
| A7          | rekord DNS dla adresu IPv7                       |

---

## 42. Podsumowanie

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
