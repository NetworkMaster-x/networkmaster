# NetworkMaster — dokumentacja programu

**Wersja:** 2.5.x — dokładny numer i pełna historia zmian: [`CHANGELOG.md`](../CHANGELOG.md)
**Rodzaj:** terminalowe narzędzie do diagnostyki i zarządzania siecią lokalną
**Platformy:** Windows, Linux, macOS (amd64, arm64; Windows i Linux dodatkowo 386/32-bit)
**Model dystrybucji:** pojedynczy plik wykonywalny, bez instalatora, portable (dane zapisywane obok pliku programu). Zero zależności uruchomieniowych — jedyne trzy zależności to biblioteki Go wkompilowane statycznie na etapie budowania (SSH/SFTP, patrz §5).
**Status prawny:** oprogramowanie zamknięte, wszelkie prawa zastrzeżone — patrz [`../LICENSE.md`](../LICENSE.md) i [`Regulamin.md`](Regulamin.md)

Ten dokument opisuje program taki, jaki jest — funkcje, sposób działania, architekturę kodu i
znane ograniczenia — z myślą głównie o użytkowniku końcowym. Kto ocenia projekt technicznie
(np. przed zakupem) i chce dużo głębszego spojrzenia — opis każdego modułu, model danych,
proces wydawniczy, dług techniczny — znajdzie je w osobnym dokumencie:
[`Dokumentacja-Techniczna-NetworkMaster.md`](Dokumentacja-Techniczna-NetworkMaster.md).

---

## 1. Czym jest NetworkMaster

NetworkMaster to menu tekstowe (TUI) uruchamiane w terminalu, które zbiera w jednym miejscu
narzędzia do diagnostyki sieci, skanowania LAN, zarządzania konfiguracją sieciową urządzenia,
zarządzania VPN (systemowym oraz WireGuard/OpenVPN) i monitorowania połączenia — bez potrzeby
instalowania osobnych narzędzi wiersza poleceń ani zapamiętywania ich składni.

Program nie wymaga instalacji: to jeden plik wykonywalny na platformę. Wszystkie dane własne
(profile IP, baza WOL, ustawienia, dziennik, raporty) zapisywane są w folderach `core_data/` i
`Reports/` tworzonych obok pliku programu — przeniesienie folderu z programem przenosi też jego
dane.

## 2. Funkcje — pełny spis menu głównego

### Diagnostyka
| Skrót | Funkcja |
|---|---|
| `T` | Testy PING — pomiar opóźnień do wskazanego hosta |
| `R` | Traceroute — śledzenie trasy pakietów do celu |
| `D` | DNS Lookup — odpytywanie rekordów DNS wskazanej domeny |
| `Q` | DNS Benchmark — test szybkości odpowiedzi wielu serwerów DNS |
| `SP` | Test prędkości łącza — pomiar download/upload |
| `Y` | SSL Audit — weryfikacja certyfikatu TLS wskazanego hosta |
| `X` | WHOIS — informacje rejestracyjne domeny/adresu IP |
| `G` | GeoIP — lokalizacja własnego adresu IP lub dowolnego wskazanego |
| `V` | Szybka diagnoza sieci — automatyczny zestaw testów i zbiorczy werdykt |

### Skanowanie
| Skrót | Funkcja |
|---|---|
| `P` | Port Skaner — skanowanie usług na wskazanym hoście (wielowątkowo) |
| `C` | Calculator — kalkulator podsieci (maski, zakresy, liczba hostów) |
| `S` | Skaner Sieci — wykrywanie urządzeń w LAN |
| `A` | ARP Tabela — lista adresów fizycznych (MAC) widocznych w sieci |
| `M` | MAC Lookup — ustalanie producenta karty sieciowej po adresie MAC |

### Zarządzanie
| Skrót | Funkcja |
|---|---|
| `I` | IPConfig — pełny podgląd konfiguracji sieciowej urządzenia |
| *(numer karty na liście)* | Menu wybranej karty sieciowej — konfiguracja adresu IP/DHCP tego interfejsu |
| `L` | Sprzęt sieciowy — informacje o kartach/urządzeniach (Menedżer Urządzeń na Windows, `lspci`/`lsusb` na Linuksie, `system_profiler` na macOS) |
| `H` | Edytor pliku hosts — z automatycznymi kopiami zapasowymi |
| `F` | Flush DNS — czyszczenie pamięci podręcznej DNS |
| `Z` | Zerowanie Sieci — zbiorczy reset/naprawa konfiguracji sieciowej |
| `J` | Baza Profili IP — zapisane profile konfiguracji do szybkiego przełączania |
| `K` | Baza WOL — zarządzanie urządzeniami i budzenie przez Wake-on-LAN |
| `U` | Otwarcie natywnego edytora połączeń systemu (Windows: Edytor Połączeń; Linux: `nm-connection-editor`; macOS: Ustawienia Systemu → Sieć) |
| `RA` | Raporty — przeglądarka wcześniej zapisanych raportów |

### VPN
| Skrót | Funkcja |
|---|---|
| `VPN` | Menedżer VPN — WireGuard, OpenVPN i VPN systemowy: dodawanie, edycja, usuwanie, połączenie/rozłączenie, pełny backup i przywracanie z pliku `.zip` |
| `W` | WiFi Vault — podgląd zapisanych haseł sieci WiFi |

WireGuard i OpenVPN działają **identycznie na wszystkich trzech platformach** — pełne
dodawanie/edycja/usuwanie profili, połączenie/rozłączenie, backup i przywracanie.

VPN systemowy (natywna integracja z mechanizmem VPN wbudowanym w system operacyjny) ma
**różny zakres możliwości w zależności od platformy** — to nie jest ograniczenie kodu, tylko
konsekwencja tego, co poszczególne systemy w ogóle udostępniają programom:

| Możliwość | Windows (RAS) | Linux (NetworkManager/`nmcli`) | macOS (`scutil --nc`) |
|---|---|---|---|
| Typy tuneli | PPTP, L2TP, SSTP, IKEv2 | L2TP, PPTP | tylko odczyt istniejących (L2TP/PPP) |
| Tworzenie profilu | ✅ | ✅ (wymaga wtyczki `NetworkManager-l2tp`/`-pptp`) | ❌ niemożliwe |
| Edycja profilu | ✅ | ✅ | ❌ niemożliwe |
| Usuwanie profilu | ✅ | ✅ | ❌ niemożliwe |
| Połączenie / rozłączenie | ✅ | ✅ | ✅ (tylko profile już istniejące) |
| Odczyt zapamiętanego hasła/PSK do backupu | ✅ (najlepszym staraniem, patrz §6) | ✅ (z pliku konfiguracyjnego, jako root) | ❌ (Keychain wymaga zgody per wpis) |
| IKEv2 | ✅ | ❌ (brak natywnego wsparcia) | ❌ (`scutil` w ogóle go nie widzi) |

Na macOS nowe profile VPN trzeba założyć w Ustawieniach Systemu (Sieć → VPN) — program potrafi
się z nimi tylko łączyć/rozłączać i sprawdzać status. To twarde ograniczenie systemu Apple:
tworzenie profili VPN programowo wymaga podpisanej aplikacji z uprawnieniem `NEVPNManager`,
którego zwykły, niezależny program terminalowy nie może uzyskać.

### Monitorowanie
| Skrót | Funkcja |
|---|---|
| `E` | Ekran Dowodzenia — zbiorczy dashboard w czasie rzeczywistym |
| `B` | Bandwidth — bieżący monitor obciążenia łącza |
| `O` | Monitor WiFi — siła sygnału na żywo |
| `N` | Netstat — aktywne porty i połączenia wraz z nazwami procesów |

### Zdalny dostęp
| Skrót | Funkcja |
|---|---|
| `SSH` | Klient SSH/SFTP — zapisane hosty, sesja interaktywna (pełny zdalny terminal), wykonanie pojedynczej komendy, przeglądarka plików (SFTP) |
| `FTP` | Klient FTP — zapisane hosty, przeglądarka plików (stary, nieszyfrowany protokół — patrz §7) |

Profile SSH obsługują logowanie hasłem i/lub kluczem prywatnym (opcjonalnie zaszyfrowanym
hasłem). Przy pierwszym połączeniu z danym hostem program pokazuje odcisk palca klucza
serwera i prosi o potwierdzenie (TOFU — "zaufaj przy pierwszym połączeniu", ten sam
mechanizm co `known_hosts` w OpenSSH) — zmiana klucza przy kolejnym połączeniu jest
sygnalizowana wprost jako możliwy atak, nie cicho ignorowana.

W przeglądarkach plików SFTP i FTP dostępna jest opcja **[O] Otwórz w eksploratorze
systemu** — wysyła bieżący katalog jako adres `sftp://`/`ftp://` do systemowego mechanizmu
skojarzeń protokołów (Finder na macOS, GVFS/KIO przez `xdg-open` na Linuksie, cokolwiek
zarejestrowane na Windows — na Windows natywnie działa to tylko dla `ftp://`; dla `sftp://`
wymaga zainstalowanego zewnętrznego klienta, np. WinSCP). **Ostrzeżenie:** jeśli profil ma
zapisane hasło, trafia ono jawnie do tego adresu, czyli chwilowo do listy argumentów
uruchamianego procesu — widocznej innym procesom/użytkownikom tej maszyny (np. w
Menedżerze Zadań) — to dodatkowe ryzyko ponad samo przechowywanie w pliku, program
sygnalizuje to przed otwarciem.

### Pozostałe
| Skrót | Funkcja |
|---|---|
| `UP` | Aktualizacje — sprawdzenie/instalacja nowej wersji z GitHub Releases. Wydania oznaczone jako krytyczne/obowiązkowe nie dają się trwale pominąć (patrz §7) |
| `US` | Ustawienia i informacje o programie, w tym historia przeczytanych komunikatów od twórcy (`5`) |
| `ADM` | (tylko Windows, gdy brak uprawnień) ponowne uruchomienie jako Administrator |

Dodatkowo: program przy starcie sam sprawdza, czy twórca opublikował **komunikat** (np.
ogłoszenie, ostrzeżenie) — każdy nieprzeczytany wymaga potwierdzenia Enterem, zanim program
przejdzie do menu głównego. Historia potwierdzonych komunikatów jest zawsze dostępna z menu
Ustawień.

## 3. Skróty klawiszowe

Działają w **każdym** oknie programu:

- **Ctrl+C** — cofnij się o jedno okno (do menu, które wywołało bieżące).
- **Ctrl+X** — natychmiastowe, całkowite zamknięcie programu, z dowolnego miejsca.

## 4. Instalacja i uruchomienie

Program nie ma instalatora. Wystarczy pobrać plik odpowiadający systemowi i architekturze
procesora i uruchomić go w terminalu:

- `NetworkMaster-windows-amd64.exe`, `NetworkMaster-windows-arm64.exe`, `NetworkMaster-windows-386.exe`
- `NetworkMaster-linux-amd64`, `NetworkMaster-linux-arm64`, `NetworkMaster-linux-386`
- `NetworkMaster-macos-amd64`, `NetworkMaster-macos-arm64`

(macOS nie ma wariantu 386 — Apple porzuciło 32-bit x86 w 2017 r., a 32-bit w ogóle w 2019 r.)

**Linux i macOS:** pobrany plik nie ma domyślnie uprawnienia do uruchamiania — trzeba je
nadać przed pierwszym startem:
```
chmod +x NetworkMaster-linux-amd64        # (albo nazwa pliku, który pobrałeś)
./NetworkMaster-linux-amd64
```

**macOS dodatkowo** zablokuje uruchomienie niepodpisanego pliku komunikatem "nie można
otworzyć, bo pochodzi od niezidentyfikowanego dewelopera" (Gatekeeper) — trzeba jednorazowo
zdjąć flagę kwarantanny:
```
xattr -d com.apple.quarantine NetworkMaster-macos-amd64
```
albo kliknąć plik prawym przyciskiem → Otwórz, i potwierdzić w oknie systemowym.

Przy pierwszym uruchomieniu program prosi o potwierdzenie i tworzy obok siebie foldery
`core_data/` (dane własne) i `Reports/` (raporty). Przeniesienie/skopiowanie folderu z plikiem
programu przenosi też cały jego stan.

Przełączniki wiersza poleceń: `--version`, `--check-update`, `--help`.

## 5. Budowanie ze źródeł

Wymagany [Go](https://go.dev/dl/) 1.22 lub nowszy. Program nie używa CGO. Od wersji z
klientem SSH/SFTP/FTP korzysta z trzech zależności Go (`golang.org/x/crypto`,
`golang.org/x/term`, `github.com/pkg/sftp`) — potrzebny jest dostęp do internetu przy
pierwszym budowaniu (pobranie modułów), zero zależności uruchomieniowych (jeden,
samodzielny plik wykonywalny jak zawsze — moduły są wkompilowane statycznie).

```powershell
cd source_code
.\build.ps1 -Version X.Y.Z
```

Skrypt uruchamia testy (`go vet` + `go test`), a następnie buduje wszystkie 8 wariantów do
`dist/` wraz z plikiem `checksums.txt`. Szybka budowa tylko na bieżącą platformę:
`go build -o NetworkMaster .` z poziomu `source_code/`.

## 6. Architektura kodu (dla czytelnika technicznego)

Cały program to jedna baza kodu w Go, współdzielona przez wszystkie platformy — różnice
systemowe są odseparowane plikami z odpowiednimi tagami budowania (`//go:build windows` itd.),
a nie osobnymi gałęziami kodu.

- **Rdzeń i UI terminala** — główna pętla menu i obsługa flag CLI, wspólne funkcje terminala
  (kolory, wejście, paski postępu). Skróty Ctrl+C/Ctrl+X są zaimplementowane jako mechanizm
  współdzielony: każde „okno” (funkcja menu) rejestruje możliwość powrotu przez `panic`/`recover`,
  a warstwa terminala per-platforma (osobna dla Windows, Linux i macOS — każdy system inaczej
  obsługuje tryb surowy terminala) przechwytuje kombinacje klawiszy.
- **Warstwa systemowa** — cała logika zależna od systemu operacyjnego (karty sieciowe, DHCP/IP
  statyczne, ARP, WiFi, uprawnienia administratora, restart) jest odseparowana w osobnych plikach
  na Windows, Linux i macOS, z jednym plikiem wspólnym dla Linuksa i macOS tam, gdzie mechanizmy
  są analogiczne (np. `sudo`).
- **Parsery** — cała logika parsowania wyjścia poleceń systemowych (np. `arp`, `ip neigh`, `ss`,
  wynik `ping` w różnych językach systemu) jest wydzielona do czystych, w pełni testowalnych
  funkcji, niezależnie od tego, czy dane narzędzie jest fizycznie dostępne na maszynie, na której
  akurat toczą się testy.
- **System aktualizacji** — odpytuje GitHub Releases repozytorium wskazanego w kodzie, dobiera
  właściwy plik dla systemu/architektury, wymaga sumy SHA-256 przed podmianą i potrafi się
  wycofać, jeśli instalacja się nie powiedzie.
- **Menedżer VPN** — warstwa wspólna (typy danych, magazyn plików WireGuard/OpenVPN,
  backup/restore, menu) korzysta z jednego, ujednoliconego opisu możliwości danej platformy
  (jaki tunel obsługuje, czy może tworzyć/edytować/usuwać profile), dzięki czemu interfejs
  pokazuje użytkownikowi dokładnie to, co dana platforma naprawdę potrafi, zamiast po prostu
  ukrywać niedostępne opcje. Sama integracja z VPN systemowym jest już w pełni osobna dla
  każdej platformy: Windows przez natywne API RAS, Linux przez `nmcli`, macOS przez `scutil`.
- **Baza producentów MAC** — osadzona bezpośrednio w binarce (skompresowana baza IEEE, ok. 39,7
  tys. wpisów), z zapytaniem do zewnętrznego serwisu jako uzupełnienie tylko wtedy, gdy adres nie
  znajduje się lokalnie.
- **Komunikaty od twórcy** — program ściąga `announcements.json` z publicznego repo przy
  starcie (ten sam mechanizm "GitHub jako lekka baza danych" co system aktualizacji); lokalna
  historia potwierdzeń w `core_data/announcements_seen.json`.
- **Klient SSH/SFTP/FTP** — SSH/SFTP przez `golang.org/x/crypto/ssh` i `github.com/pkg/sftp`
  (jedyne zależności, których nie da się bezpiecznie napisać samemu — kryptografia SSH to nie
  miejsce na własne implementacje). FTP jest napisany od zera, bez zależności — to stary,
  czysto tekstowy protokół, więc da się to zrobić poprawnie samemu. Oba dzielą ten sam wzorzec
  magazynu profili (`core_data/ssh_hosts.json`, `core_data/ftp_hosts.json`) co reszta programu.

### Jakość kodu i testy

Cały program to ok. 15 000 linii Go w jednej bazie kodu. Automatyczny pakiet testów (`go test`)
obejmuje 108 testów w 13 plikach — parsery (dane z `arp`, `ip neigh`, `ss`, `netsh`, `ping` w wielu
językach systemowych), kalkulator podsieci, system aktualizacji (na atrapie API GitHuba: wybór
pliku per platforma/architektura, weryfikacja SHA-256, odrzucanie podmienionych plików, podmiana
z wycofaniem), system komunikatów od twórcy, weryfikację układu struktur Win32 API (RAS,
Menedżer Poświadczeń) na poziomie bajtów, oraz klienta SSH/SFTP/FTP — uruchamiane są **prawdziwe,
lokalne serwery SSH i FTP** (nie atrapy wywołań) w ramach testów, żeby zweryfikować faktyczny
protokół na drucie, nie tylko logikę. `go vet` przechodzi czysto na wszystkich 8 kombinacjach
system/architektura przy każdym wydaniu (wymusza to `build.ps1`).

## 7. Znane ograniczenia

Poniższe punkty są świadomie i w pełni ujawnione — żadne z nich nie jest ukryte ani pomniejszone:

1. **macOS nie był uruchomiony na żywym systemie.** Kod dla macOS kompiluje się i przechodzi
   statyczną analizę (`go vet`), ale był tworzony i weryfikowany wyłącznie na Windows/Linux —
   autor nie miał dostępu do maszyny z macOS do testów na żywo. Zanim uruchomisz program
   produkcyjnie na macOS, przetestuj podstawowe funkcje.

   **Linux natomiast był zweryfikowany na żywym serwerze (Ubuntu 26.04 LTS):** uruchomienie,
   przełączniki `--version`/`--check-update`, pierwsze uruchomienie i ekran zgody, komunikaty od
   twórcy, menu główne, IPConfig, tabela ARP (z lookupem producenta MAC, przez fallback na
   `ip neigh`, bo `arp` nie był nawet zainstalowany na tym hoście), kalkulator podsieci i
   menedżer VPN — w tym poprawne, czytelne zgłoszenie braku `nmcli`/NetworkManager, zamiast
   cichej awarii. Właśnie na tym teście znaleziono i naprawiono realny błąd: ciche pomijanie
   zapisu danych, gdy `core_data/` był wcześniej utworzony jako root — patrz CHANGELOG.md
   (wersja 2.5.2). Nie zweryfikowano na żywo: rzeczywistego połączenia VPN (WireGuard/OpenVPN
   wymaga drugiego końca tunelu), monitora WiFi (serwer testowy nie ma karty WiFi) i edytora
   pliku hosts/operacji wymagających uprawnień roota (konto testowe miało `sudo`, ale testy
   uruchamiano bez niego, żeby sprawdzić typowy, nieprzywilejowany przypadek).
2. **macOS: brak programowego tworzenia/edycji/usuwania profili VPN.** To ograniczenie samego
   systemu Apple (wymaga podpisanej aplikacji z uprawnieniem `NEVPNManager`), nie luka w kodzie.
   Dodatkowo `scutil` (narzędzie systemowe, na którym opiera się ta funkcja) w ogóle nie pokazuje
   profili typu IKEv2 — to również ograniczenie samego narzędzia Apple.
3. **Linux: VPN systemowy wymaga opcjonalnych wtyczek** `NetworkManager-l2tp` /
   `NetworkManager-pptp`, które nie zawsze są domyślnie zainstalowane — program zgłasza ich brak
   wprost zamiast cicho zawodzić. Domyślnie NetworkManager "z pudełka" nie umie tworzyć połączeń
   VPN typu L2TP/PPTP — potrzebuje do tego dodatkowego modułu. Instalacja (nazwy pakietów różnią
   się między dystrybucjami):
   - Ubuntu/Debian: `sudo apt install network-manager-l2tp network-manager-pptp`
   - Fedora: `sudo dnf install NetworkManager-l2tp NetworkManager-pptp`
   - Arch Linux: `networkmanager-l2tp` / `networkmanager-pptp` (AUR)

   IKEv2 i SSTP nie mają natywnego wsparcia na Linuksie (żadna wtyczka tego nie zmieni) — do nich
   użyj WireGuard lub OpenVPN.
4. **32-bit Windows (`windows-386`): odzyskiwanie zapamiętanych haseł/PSK VPN do backupu
   prawdopodobnie nie zadziała.** Struktura danych używana przez Windows API do tego celu
   (`RASDIALPARAMSW`) została zweryfikowana i przetestowana tylko dla 64-bit Windows. Funkcja nie
   zwróci błędu — po prostu nie znajdzie zapisanego hasła, tak samo jak w sytuacji, gdy
   użytkownik nigdy się wcześniej pomyślnie nie połączył. Reszta programu na 32-bit Windows
   działa bez tego ograniczenia.
5. **Hasła i klucze w kopiach zapasowych VPN są zapisywane jawnie (bez szyfrowania).** To
   świadoma decyzja projektowa, nie błąd — celem jest, żeby jeden plik `.zip` backupu naprawdę
   zawierał wszystko potrzebne do pełnego odtworzenia konfiguracji, łącznie z hasłami, kluczami
   PSK i kluczami prywatnymi WireGuard. Oznacza to, że plik backupu wymaga takiej samej ostrożności
   jak plik z hasłami — nie należy go przesyłać ani przechowywać bez dodatkowego zabezpieczenia.
6. **Binarki nie są podpisane cyfrowo (brak certyfikatu Authenticode/Apple Developer ID).**
   System operacyjny lub antywirus może przy pierwszym uruchomieniu/pobraniu pokazać ostrzeżenie
   ("nieznany wydawca", SmartScreen, Gatekeeper na macOS — patrz §4) — to standardowe zachowanie
   dla każdego niepodpisanego pliku wykonywalnego, nie oznaka realnego zagrożenia. Próba
   obfuskacji kodu (garble) w wersji 2.4.0 została wycofana w 2.4.1, bo w praktyce powodowała
   dużo gorszy efekt: Google Chrome (Safe Browsing) blokował pobieranie pliku jako "wirus" —
   patrz CHANGELOG.md. Jedyny trwały sposób pozbycia się tych ostrzeżeń to podpis cyfrowy, nie
   wdrożony w tej wersji.
7. **Windows: strzałki i klawisze funkcyjne (F1-F12, Page Up/Down, Home/End) nie działają w
   interaktywnej sesji SSH.** Windows dostarcza te klawisze do konsoli jako kody wirtualne, nie
   jako sekwencje ANSI — warstwa odczytu klawiatury tego programu (zbudowana pierwotnie do
   prostej edycji linii) je pomija. Zwykłe pisanie, Enter, Backspace i Ctrl+C (przekazywane do
   zdalnego procesu) działają normalnie, tak samo na Linuksie/macOS, gdzie te klawisze w ogóle
   nie sprawiają problemu (surowy strumień bajtów naturalnie przenosi sekwencje ANSI). Pełna
   naprawa na Windows wymagałaby osobnej warstwy parsowania VT, co wykracza poza rozsądny zakres
   tej funkcji — świadomie przyjęte ograniczenie.
8. **Zwykłe FTP jest protokołem bez szyfrowania** — hasło i transferowane pliki idą jawnym
   tekstem przez sieć. To ograniczenie samego protokołu (RFC 959 z 1985 r.), nie tego programu.
   Jeśli to możliwe, używaj SFTP zamiast FTP.

## 8. FAQ

**Czy program wysyła moje dane gdziekolwiek?**
Nie ma telemetrii ani konta użytkownika. Program łączy się z internetem wyłącznie wtedy, gdy
sam uruchomisz konkretną funkcję, która tego wymaga (sprawdzenie aktualizacji, WHOIS, GeoIP,
test prędkości, lookup producenta MAC) — i tylko z usługą, której dana funkcja bezpośrednio
dotyczy. Pełny wykaz w [Polityce Prywatności](Polityka-Prywatnosci.md).

**Czy program wymaga uprawnień administratora?**
Część funkcji (np. zmiana konfiguracji karty sieciowej, edycja pliku hosts, niektóre operacje
VPN) tego wymaga — program wykrywa brak uprawnień i wyraźnie to sygnalizuje, oferując na
Windows opcję ponownego uruchomienia jako Administrator (`ADM`).

**Czy działa bez internetu?**
Tak — większość funkcji (konfiguracja sieci, skanowanie LAN, ARP, baza profili IP/WOL, edycja
hosts, WireGuard/OpenVPN w sieci lokalnej) działa całkowicie offline. Internetu wymagają tylko
funkcje z natury sieciowe zewnętrznie (WHOIS, GeoIP, aktualizacje, test prędkości).

**Czy program instaluje coś w systemie?**
Nie. To pojedynczy plik wykonywalny bez instalatora; jedyne pliki, jakie tworzy, to własne
foldery danych (`core_data/`, `Reports/`) obok siebie.

**Dlaczego Windows/Chrome/antywirus ostrzega przy pobraniu albo uruchomieniu?**
Bo plik nie jest podpisany cyfrowo (patrz punkt 6 w §7) — to standardowe zachowanie systemu dla
każdego niepodpisanego programu, niezależnie od tego, co kod faktycznie robi. Jedyny sposób,
żeby to ostrzeżenie zniknęło na dobre, to podpis cyfrowy (certyfikat Authenticode) — nie
wdrożony w tej wersji (patrz CHANGELOG.md).

**Znalazłeś błąd albo masz sugestię poprawki?**
Napisz na **pxware@pxware.pl** — opisz, co się stało, i zaznacz w temacie, że chodzi o
NetworkMaster (ten sam adres obsługuje więcej niż jeden projekt).

---

Pełna historia zmian: [`../CHANGELOG.md`](../CHANGELOG.md).
