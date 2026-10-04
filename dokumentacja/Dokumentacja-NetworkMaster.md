# NetworkMaster — dokumentacja programu

**Wersja:** 2.4.0
**Rodzaj:** terminalowe narzędzie do diagnostyki i zarządzania siecią lokalną
**Platformy:** Windows, Linux, macOS (amd64, arm64; Windows i Linux dodatkowo 386/32-bit)
**Model dystrybucji:** pojedynczy plik wykonywalny, bez instalatora, bez zależności zewnętrznych, portable (dane zapisywane obok pliku programu)

Ten dokument opisuje program taki, jaki jest — funkcje, sposób działania, architekturę kodu i
znane ograniczenia — z myślą zarówno o użytkowniku końcowym, jak i o kimś oceniającym produkt
technicznie (np. przed zakupem).

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

### Pozostałe
| Skrót | Funkcja |
|---|---|
| `UP` | Aktualizacje — sprawdzenie/instalacja nowej wersji z GitHub Releases |
| `US` | Ustawienia i informacje o programie |
| `ADM` | (tylko Windows, gdy brak uprawnień) ponowne uruchomienie jako Administrator |

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

Przy pierwszym uruchomieniu program prosi o potwierdzenie i tworzy obok siebie foldery
`core_data/` (dane własne) i `Reports/` (raporty). Przeniesienie/skopiowanie folderu z plikiem
programu przenosi też cały jego stan.

Przełączniki wiersza poleceń: `--version`, `--check-update`, `--help`.

## 5. Budowanie ze źródeł

Wymagany [Go](https://go.dev/dl/) 1.22 lub nowszy. Program nie używa CGO ani zależności spoza
biblioteki standardowej.

```powershell
cd source_code
.\build.ps1 -Version 2.4.0
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

## 7. Znane ograniczenia

Poniższe punkty są świadomie i w pełni ujawnione — żadne z nich nie jest ukryte ani pomniejszone:

1. **Linux i macOS nie były uruchomione na żywym systemie.** Cały kod dla tych platform
   kompiluje się i przechodzi statyczną analizę (`go vet`) na wszystkich architekturach, ale był
   tworzony i weryfikowany wyłącznie na Windows — autor nie miał dostępu do maszyny z Linuksem
   ani macOS do testów na żywo. Zanim uruchomisz program produkcyjnie na tych systemach,
   przetestuj podstawowe funkcje.
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

---

Pełna historia zmian: [`../CHANGELOG.md`](../CHANGELOG.md).
