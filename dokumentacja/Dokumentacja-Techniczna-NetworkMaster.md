# NetworkMaster — dokumentacja techniczna (due diligence)

Ten dokument jest przeznaczony dla kogoś, kto **ocenia kod technicznie** — głównie przed
zakupem projektu (patrz [Regulamin §8](Regulamin.md)) — i chce wiedzieć precyzyjnie, z czym
ma do czynienia: jak jest zbudowany, jak jest przechowywany stan, jakie są realne (nie
marketingowe) ograniczenia i dług techniczny. Dokument [`Dokumentacja-NetworkMaster.md`](Dokumentacja-NetworkMaster.md)
opisuje program z perspektywy użytkownika; ten dokument patrzy na niego z perspektywy kogoś,
kto będzie czytał i rozwijał ten kod.

Każde twierdzenie poniżej odpowiada konkretnemu plikowi/funkcji w `source_code/` — to nie jest
ogólny opis, to mapa kodu.

| | |
|---|---|
| **Produkt** | NetworkMaster — terminalowe (TUI) narzędzie do diagnostyki sieci, skanowania LAN, zarządzania VPN i klient SSH/SFTP/FTP |
| **Wersja dokumentu** | odpowiada CHANGELOG.md — dokładny numer wydania: [`CHANGELOG.md`](../CHANGELOG.md) |
| **Platformy** | Windows, Linux, macOS — amd64/arm64 (Windows i Linux dodatkowo 386) |
| **Język** | Go 1.22+, zero CGO |
| **UI** | własny TUI w terminalu (kolory ANSI, surowy tryb klawiatury), zero bibliotek UI |
| **Backend** | brak — w pełni lokalny, portable |
| **Dystrybucja** | pojedynczy plik wykonywalny, GitHub Releases, wbudowany auto-update |
| **Status prawny** | zamknięty, wszelkie prawa zastrzeżone — [`LICENSE.md`](../LICENSE.md) |

---

## Spis treści

1. [Stos technologiczny i zależności](#1-stos-technologiczny-i-zależności)
2. [Uruchomienie i budowanie](#2-uruchomienie-i-budowanie)
3. [Konfiguracja i rebranding (dane do podmiany)](#3-konfiguracja-i-rebranding-dane-do-podmiany)
4. [Struktura projektu](#4-struktura-projektu)
5. [Architektura](#5-architektura)
6. [Opis komponentów](#6-opis-komponentów)
7. [Model danych i przechowywanie](#7-model-danych-i-przechowywanie)
8. [System aktualizacji i komunikatów](#8-system-aktualizacji-i-komunikatów)
9. [Bezpieczeństwo i prywatność](#9-bezpieczeństwo-i-prywatność)
10. [Testowanie](#10-testowanie)
11. [Wydanie i dystrybucja — proces trzech repozytoriów](#11-wydanie-i-dystrybucja--proces-trzech-repozytoriów)
12. [Znane ograniczenia i dług techniczny](#12-znane-ograniczenia-i-dług-techniczny)
13. [Proponowany rozwój (roadmapa)](#13-proponowany-rozwój-roadmapa)
14. [Słownik pojęć](#14-słownik-pojęć)

---

## 1. Stos technologiczny i zależności

| Obszar | Technologia |
|---|---|
| Język | Go 1.22 (dyrektywa `go 1.22` w `go.mod` — pinowana ręcznie, patrz niżej) |
| Build | brak frameworka budowania — `go build` / `go test` / `go vet` wywoływane przez `build.ps1` (PowerShell) |
| UI | brak biblioteki TUI (nie ncurses/tcell/bubbletea) — własna, ręczna obsługa ANSI i surowego trybu terminala |
| Współbieżność | tylko standardowa biblioteka (`goroutine`/`channel`), brak frameworków async |

**Zależności Go (`go.mod`):**

| Moduł | Wersja | Do czego | Czy konieczna |
|---|---|---|---|
| `golang.org/x/crypto` | v0.33.0 | implementacja protokołu SSH (`ssh.Dial`, PTY, auth) | tak — kryptografia SSH nie jest czymś, co pisze się samemu |
| `golang.org/x/term` | v0.29.0 | tryb surowy terminala przy wpisywaniu haseł/PSK (maskowanie) | pomocnicza, mała |
| `github.com/pkg/sftp` | v1.13.9 | protokół SFTP na już nawiązanej sesji SSH | tak, z tego samego powodu co SSH |
| `golang.org/x/sys` | v0.30.0 | zależność pośrednia `x/crypto`/`x/term` | pośrednia |
| `github.com/kr/fs` | v0.1.0 | zależność pośrednia `pkg/sftp` | pośrednia |

To są **jedyne trzy bezpośrednie zależności w całym projekcie** — reszta (diagnostyka, skanowanie,
VPN, FTP, system aktualizacji) jest napisana bez żadnych zależności zewnętrznych, włącznie z
protokołem FTP (RFC 959) napisanym od zera na `net/textproto`. Program przed wersją 2.5.0 (klient
SSH/SFTP/FTP) miał **zero** zależności — to świadoma zmiana, opisana w CHANGELOG.md przy 2.5.0,
nie przeoczenie.

**Pułapka przy pracy z modułami:** `go get`/`go mod tidy` przy pobieraniu najnowszych wersji
`x/crypto`/`x/term` podbijają automatycznie dyrektywę `go` w `go.mod` do wersji lokalnego
toolchaina (np. 1.26), bo najnowsze wydania tych pakietów tego wymagają. Żeby zachować
zadeklarowane minimum **Go 1.22**, przypięto starsze, konkretne wersje (powyższa tabela) i **po
każdej operacji na modułach trzeba ręcznie przywrócić `go 1.22`** w `go.mod` — inaczej
zadeklarowane minimum przestaje być prawdą bez żadnego ostrzeżenia kompilatora.

## 2. Uruchomienie i budowanie

```powershell
cd source_code
go build -o NetworkMaster.exe .     # szybka budowa na bieżącą platformę (sekundy)
go vet ./...                        # analiza statyczna
go test .                           # 108 testów, kilkanaście-kilkadziesiąt sekund
.\build.ps1 -Version X.Y.Z          # testy + budowa 8 wariantów + checksums.txt
```

`build.ps1` jest jedynym "CI" tego projektu — nie ma GitHub Actions ani innego pipeline'u.
Uruchamia `go vet` i `go test .` na hoście budującym (zawsze Windows w tym projekcie), a
następnie krzyżowo kompiluje (`GOOS`/`GOARCH`) na 8 kombinacji. **Krzyżowa kompilacja weryfikuje
tylko, że kod się kompiluje** na danej platformie — nie uruchamia testów na docelowym systemie
(Go nie pozwala łatwo wykonać testów dla innego `GOOS` bez emulacji/QEMU). W praktyce: pakiet
testów faktycznie wykonuje się tylko na Windows przy każdym budowaniu; poprawność na
Linux/macOS opiera się na (a) plikach z build tagami pisanymi specyficznie pod API każdego
systemu, (b) czystym `go vet` na każdej platformie, (c) testach parserów, które nie wymagają
uruchomienia na danym systemie (działają na nagranych, przykładowych danych wejściowych), i
(d) od niedawna — bezpośrednich testach na żywym serwerze Linux (patrz §10 i §12).

Pierwsze budowanie wymaga internetu (pobranie trzech modułów Go do lokalnego cache'u). Wynikowy
plik jest w pełni samodzielny — moduły są wkompilowane statycznie, zero zależności
uruchomieniowych na maszynie użytkownika.

## 3. Konfiguracja i rebranding (dane do podmiany)

W przeciwieństwie do projektów z jednym plikiem `.env`/`config`, dane specyficzne dla właściciela
są rozproszone w kilku miejscach — to pierwsza rzecz, którą nowy właściciel powinien przejrzeć:

| Element | Plik | Uwagi |
|---|---|---|
| `AppName` | `source_code/version.go` | nazwa używana w UI, ścieżkach danych (`os.UserConfigDir()/<AppName>`) |
| `UpdateRepo`, `RepoURL` | `source_code/version.go` | repozytorium GitHub, z którego program czyta Releases przy aktualizacji — **zmiana wymaga przeniesienia całej historii wydań**, program nie ma mechanizmu "migracji" między repozytoriami |
| `LegalURL` | `source_code/version.go` | link do folderu `dokumentacja/` w publicznym repo, pokazywany w ekranie zgody przy pierwszym uruchomieniu |
| `AnnouncementsURL` | `source_code/version.go` | surowy URL do `announcements.json` na branchu `main` publicznego repo — **musi** tam istnieć, inaczej sprawdzanie komunikatów po cichu nie znajduje nic (nie crashuje, patrz §8) |
| Właściciel, NIP, kontakt | `LICENSE.md`, `dokumentacja/Regulamin.md`, `dokumentacja/Polityka-Prywatnosci.md` | dane prawne — do zmiany przy przeniesieniu praw |
| Adres kontaktowy w kodzie/UI | brak — program nie ma żadnego hardkodowanego adresu e-mail w kodzie źródłowym; adresy (np. `pxware@pxware.pl`) są tylko w dokumentach `.md` i na stronie www, nie w `source_code/` | łatwiejszy rebranding niż gdyby było wkompilowane w binarkę |
| Nazwa plików wydań | `source_code/RELEASING.md`, `build.ps1` | wzorzec `NetworkMaster-<os>-<arch>` — zmiana `AppName` **nie** zmienia automatycznie tego wzorca, trzeba edytować oba miejsca ręcznie |
| Strona WWW | osobne repozytorium `networkmaster-x.github.io` (GitHub Pages) | jeden plik `index.html`, poza tym repozytorium kodu — patrz §11 |

Program **nie ma** ikony/logotypu wkompilowanego w binarkę (to program konsolowy, nie GUI) —
jedyna "marka" widoczna użytkownikowi to nazwa w nagłówkach menu (`drawMenu()` w `main.go`) i
powyższe stałe.

## 4. Struktura projektu

```
NetworkMaster/
├── CHANGELOG.md, LICENSE.md
├── dokumentacja/              – dokumenty użytkownika/prawne/techniczne (ten plik też tu jest)
├── source_code/
│   ├── go.mod, go.sum
│   ├── build.ps1, RELEASING.md, README.md
│   ├── main.go                – menu główne, pierwsze uruchomienie, flagi CLI
│   ├── ui.go, config.go, log.go, exec.go
│   │                           – terminal/kolory, ścieżki i ustawienia (portable), log, uruchamianie poleceń bez powłoki
│   ├── keys.go, terminal_{windows,linux,darwin}.go
│   │                           – Ctrl+C/Ctrl+X: tryb surowy terminala per platforma, edytor linii
│   ├── platform_{windows,linux,darwin,unix}.go
│   │                           – wszystko zależne od systemu: karty sieciowe, ARP, WiFi, uprawnienia, restart
│   ├── netparse.go            – czyste parsery (arp/ip neigh/ss/netsh/ping), w pełni testowalne
│   ├── update.go, update_ui.go, update_test.go
│   │                           – aktualizacje z GitHub Releases
│   ├── announcements.go, announcements_test.go
│   │                           – komunikaty od twórcy
│   ├── db.go, wol.go, reports.go, card.go
│   │                           – profile IP, Wake-on-LAN, raporty, menu karty sieciowej
│   ├── scan.go, dns.go, subnet.go, web.go, speedtest.go, health.go, live.go, mac.go, tools_sys.go
│   │                           – narzędzia diagnostyczne
│   ├── vpn_types.go, vpn_store.go, vpn_ui.go, vpn_unix.go
│   │   vpn_windows.go, vpn_linux_native.go, vpn_darwin_native.go, rasapi_windows.go
│   │                           – menedżer VPN (WireGuard/OpenVPN + VPN systemowy per platforma)
│   ├── remote_types.go, remote_store.go
│   │                           – wspólne dla SSH i FTP: typy profili, magazyn JSON
│   ├── ssh_client.go, ssh_hostkey.go, sftp_client.go, ssh_ui.go
│   │                           – klient SSH/SFTP
│   ├── ftp_client.go, ftp_ui.go
│   │                           – klient FTP (od zera, RFC 959)
│   ├── oui.gz                 – osadzona baza producentów MAC (IEEE, ~39,7 tys. wpisów)
│   ├── tools/mockgithub        – atrapa API GitHuba do testów aktualizacji
│   ├── tools/genoui             – generator oui.gz z pliku CSV
│   └── *_test.go (13 plików)  – patrz §10
└── (gitignored, nigdy nie commitowane: _gotool/ – portable Go, core_data/ – dane
    uruchomieniowe z testów lokalnych, Reports/)
```

**50 plików `.go` nie-testowych, ~12 400 linii; 13 plików testowych, ~2 850 linii; razem ~15 250
linii.** Największe pliki to `vpn_ui.go` (876 linii — menu menedżera VPN, wszystkie platformy
współdzielą ten jeden plik UI), `netparse.go` (627), `scan.go` (584), `platform_linux.go` (555),
`platform_windows.go` (526), `update.go` (512) — żaden plik nie jest na tyle duży, żeby wymagał
pilnego podziału, ale `vpn_ui.go` jest naturalnym kandydatem do rozbicia na mniejsze pliki
(dodawanie/edycja/backup/lista), jeśli funkcjonalność VPN będzie dalej rosła.

## 5. Architektura

### 5.1 Model ogólny

Jedna baza kodu Go, współdzielona przez wszystkie platformy. Różnice systemowe są odseparowane
plikami z tagami budowania (`//go:build windows`, `//go:build linux`, `//go:build darwin`,
`//go:build unix`), **nie** osobnymi gałęziami Git ani osobnymi binarkami o różnym kodzie —
każda platforma kompiluje te same pliki "wspólne" plus swój własny zestaw plików
platformowych. To oznacza, że zmiana w pliku wspólnym (np. `scan.go`) automatycznie obowiązuje
na wszystkich trzech systemach bez ręcznej synchronizacji.

```
┌─────────────────────────────────────────────────────────┐
│  main() → mainLoop()  (main.go)                          │
│  menu tekstowe, drawMenu(), mainActions[klawisz] → fn()  │
└───────────────┬─────────────────────────────┬────────────┘
                │                              │
     ┌──────────▼──────────┐       ┌───────────▼───────────┐
     │  moduły funkcjonalne │       │  warstwa systemowa     │
     │  (diagnostyka,       │──────▶│  platform_<os>.go      │
     │  skanowanie, VPN,    │       │  (ARP, WiFi, karty,    │
     │  SSH/SFTP/FTP, ...)  │       │   uprawnienia, restart)│
     └──────────┬───────────┘       └────────────────────────┘
                │
     ┌──────────▼───────────┐
     │  warstwa terminala    │   keys.go, terminal_<os>.go
     │  (Ctrl+C/Ctrl+X,      │   – surowy tryb terminala,
     │   edytor linii)       │     per platforma
     └────────────────────────┘
```

### 5.2 Mechanizm nawigacji: panic/recover, nie stos stanów

Każde "okno" programu (funkcja menu, formularz, narzędzie) zaczyna się od `defer backOnCtrlC()`
(`keys.go`). Gdy użytkownik naciśnie Ctrl+C w dowolnym miejscu tej funkcji, warstwa odczytu
klawiatury wywołuje `panic(errBack{})`. Ten panic "przebija" cały stos wywołań bieżącego okna i
jest przechwytywany przez najbliższy `defer backOnCtrlC()` nad nim na stosie — czyli przez
funkcję, która to okno otworzyła. Efekt: "wróć do okna, które mnie wywołało", bez żadnego
jawnego stosu stanów UI czy maszyny stanów.

**Nieoczywisty, ale w pełni zamierzony skutek tego mechanizmu:** `main()` samo ma
`defer backOnCtrlC()` na samym początku (przed `mainLoop()`). Menu najwyższego poziomu nie ma
żadnego okna-rodzica — jego "rodzicem" jest właśnie `main()`. Dlatego **Ctrl+C naciśnięty
dokładnie w menu głównym kończy program równie czysto jak Ctrl+X**, mimo że dokumentacja
użytkownika opisuje Ctrl+C jako "wróć o jedno okno" — na najwyższym poziomie "jedno okno wyżej"
to po prostu wyjście z programu. Zweryfikowane bezpośrednio podczas testów na żywym serwerze
Linux (§10): obie kombinacje klawiszy w menu głównym kończą proces czysto, bez zawieszenia i bez
śladu panicu w konsoli (bo recover go przechwytuje, zanim doleci do runtime Go).

### 5.3 Dane: program portable, z fallbackiem

Program nie ma instalatora i domyślnie zapisuje swoje dane (`core_data/`, `Reports/`) **obok
pliku wykonywalnego** (`initPaths()` w `config.go`, oparte na `os.Executable()`). To pozwala
przenosić cały katalog programu wraz z jego stanem. Gdy katalog programu (albo jego podfoldery
danych) nie są zapisywalne, program **wykrywa to i przełącza się** na katalog konfiguracji
użytkownika (`os.UserConfigDir()/<AppName>` — `%AppData%` na Windows, `~/.config` na Linuksie,
`~/Library/Application Support` na macOS), z czytelnym ostrzeżeniem na starcie
(`dataNote`, wyświetlane w `main.go`).

Do wersji 2.5.2 ta detekcja sprawdzała tylko zapisywalność samego katalogu bazowego, nie
podfolderów `core_data/`/`Reports/` — co dawało cichą utratę zapisu, jeśli te podfoldery
istniały już z innymi uprawnieniami (typowo: jednorazowe `sudo ./NetworkMaster...` na
Linuksie/macOS). Naprawione i opisane szczegółowo w §12.

## 6. Opis komponentów

### 6.1 Rdzeń i UI terminala

| Plik | Rola |
|---|---|
| `main.go` | `mainLoop()` — pętla menu; `mainActions map[string]func()` — routing klawisza na funkcję; `handleFlags()` — `--version`/`--check-update`/`--help`; `firstRun()` — ekran zgody z licencją przy pierwszym starcie |
| `ui.go` | `input()`/`inputSecret()` (maskowanie haseł), `runTool()` — `defer backOnCtrlC()` + `recover()` wokół każdego wywołania narzędzia z menu |
| `keys.go` | `errBack` (typ panic-sygnału), `backOnCtrlC()`, obserwator Ctrl+X (natychmiastowe `os.Exit`, podmieniane w testach przez `exitFunc`) |
| `terminal_windows.go` / `_linux.go` / `_darwin.go` | tryb surowy terminala — **trzy zupełnie różne implementacje**, bo Windows (Console API), Linux i macOS (termios, ale z drobnymi różnicami) obsługują to inaczej |
| `config.go`, `log.go`, `exec.go` | ścieżki danych i fallback (§5.3), prosty log tekstowy do pliku, uruchamianie poleceń **bez powłoki** (`exec.Command` z rozbitymi argumentami, nie `sh -c "..."` — ochrona przed wstrzyknięciem poleceń przez dane wejściowe użytkownika, patrz §9) |

### 6.2 Parsery (`netparse.go`)

Cała logika interpretacji wyjścia poleceń systemowych (`arp`/`ip neigh`, `ss`, `netsh`, `ping` w
wielu językach interfejsu systemu) jest wydzielona do czystych funkcji `string → struct`,
niezależnych od tego, czy dane polecenie jest fizycznie dostępne na maszynie, na której toczą
się testy — testy karmią te funkcje nagranymi próbkami tekstu. To jest właśnie to, co pozwoliło
w 100% przetestować logikę parsowania na Windows, mimo że część poleceń (`arp`, `ip neigh`)
dotyczy Linuksa.

### 6.3 System aktualizacji (`update.go`, `update_ui.go`)

Patrz §8 — opisany osobno, bo ma własny model bezpieczeństwa.

### 6.4 Komunikaty od twórcy (`announcements.go`)

`fetchAnnouncements()` ściąga `AnnouncementsURL` (JSON) przy starcie w tle
(`startAnnouncementsCheck()`), `pendingAnnouncements()` filtruje te, których ID nie są jeszcze w
lokalnej historii potwierdzeń (`core_data/announcements_seen.json`,
`loadAckedAnnouncements()`/`saveAckedAnnouncements()`). `handleStartupAnnouncements()` blokuje
przejście do menu głównego, aż użytkownik potwierdzi Enterem każdy nieprzeczytany komunikat
(`showAnnouncement()`). Ten sam wzorzec "GitHub jako lekka baza danych" co system aktualizacji —
brak komunikatów lub błąd sieci po cichu nic nie pokazuje (nie blokuje programu, patrz §8).

### 6.5 Menedżer VPN

| Plik | Rola |
|---|---|
| `vpn_types.go` | typy (`VPNEntry`, `SystemVPNProfile`, `WGSummary`...), parser/serializer formatu `.conf` WireGuard i `.ovpn` OpenVPN (`splitConfBlocks`, `summarizeWG`, `scanOpenVPNFileRefs`...) |
| `vpn_store.go` | magazyn plików profili, backup do `.zip` z `manifest.json`, przywracanie |
| `vpn_ui.go` | menu menedżera — jeden plik wspólny dla wszystkich platform, który pyta warstwę platformową o jej **realne** możliwości (`systemVPNInfo()`) i pokazuje użytkownikowi dokładnie to, czego dana platforma pozwala zrobić, zamiast ukrywać niedostępne opcje za cichym błędem |
| `vpn_unix.go` | WireGuard/OpenVPN na Linuksie i macOS (identyczne na obu — to te samo narzędzia linii poleceń) |
| `vpn_windows.go` | VPN systemowy Windows przez natywne RAS API |
| `vpn_linux_native.go` | VPN systemowy Linux przez `nmcli` (NetworkManager) — zweryfikowane na żywo: gdy `nmcli` nie istnieje, program wyświetla `[i] Nie znaleziono 'nmcli' (NetworkManager) – wbudowany VPN niedostępny. Użyj WireGuard/OpenVPN.` zamiast crashować lub ciche nic nie robić (§10) |
| `vpn_darwin_native.go` | VPN systemowy macOS przez `scutil --nc` — tylko odczyt/połączenie z istniejącymi profilami (twarde ograniczenie API Apple, nie kodu, patrz §12) |
| `rasapi_windows.go` | odczyt zapamiętanych haseł/PSK Windows RAS do pełnego backupu — bezpośrednie wywołania `rasapi32.dll`, struktury zweryfikowane na poziomie bajtów w testach (`unsafe.Sizeof`/`Offsetof`, patrz §10) |

### 6.6 Klient SSH/SFTP/FTP

| Plik | Funkcje | Rola |
|---|---|---|
| `remote_types.go`, `remote_store.go` | `SSHHost`/`FTPHost`, magazyn JSON | typy i przechowywanie profili, wspólne dla SSH i FTP |
| `ssh_client.go` | `sshClientConfig`, `sshDial`, `sshRunCommand`, `sshInteractiveShell` | konfiguracja klienta (hasło i/lub klucz prywatny), sesja z pojedynczą komendą, pełna sesja interaktywna (PTY + surowe przekazywanie klawiatury) |
| `ssh_hostkey.go` | `loadKnownHosts`, `saveKnownHosts`, `tofuVerify` | TOFU — weryfikacja klucza hosta przy pierwszym połączeniu, zapis odcisku w `core_data/ssh_known_hosts.json`, ten sam model co `known_hosts` OpenSSH |
| `sftp_client.go` | `sftpConnect`, `sftpSession.{list,cd,download,upload,remove,mkdir}` | przeglądarka plików SFTP na już nawiązanej sesji SSH |
| `ftp_client.go` | `ftpConnect`, `ftpClient.{Pwd,Cwd,Mkdir,Delete,List,Download,Upload}`, `simple()` | klient FTP od zera (RFC 959, tryb pasywny), na `net/textproto` |
| `ssh_ui.go`, `ftp_ui.go` | — | menu: lista hostów, formularz, przeglądarka plików, `[O]` otwarcie w systemowym eksploratorze |

**Historia błędu w `ftp_client.go` warta odnotowania:** `net/textproto.Pipeline` wymaga, żeby
*każde* wywołanie `Cmd()` było opakowane parą `StartResponse`/`EndResponse` w kolejności —
pierwsza wersja `TYPE I` w `ftpConnect` wywoływała `Cmd()`/`ReadResponse()` bezpośrednio, co po
cichu psuło wewnętrzny sekwencer na *każdym kolejnym* poleceniu (deadlock). Błąd został złapany
przez testy z prawdziwym serwerem FTP przed wydaniem (§10), nie przez użytkownika po wydaniu —
dowód na to, że ten konkretny styl testowania (prawdziwy serwer, nie atrapa) faktycznie łapie
błędy, które testy na atrapach by przepuściły.

## 7. Model danych i przechowywanie

Wszystkie dane własne programu żyją w `core_data/` (obok pliku programu albo w katalogu
fallback, §5.3) — **żadnej bazy danych**, same pliki JSON/binarne, odczytywane/zapisywane przez
`writeFileAtomic()` (zapis do pliku `.tmp` + `os.Rename` — awaria w trakcie zapisu nie
koruptuje istniejących danych).

| Plik | Format | Zawartość |
|---|---|---|
| `installed.flag` | 2 bajty | znacznik "program już widział ekran zgody" |
| `ip_registry.sys` | tekstowy, pola rozdzielane `;` (`nazwa;ip;maska;brama;dns1;dns2;`) — format zgodny z wersją 1.x, mimo nazwy pliku to zwykły tekst, nie binarny format | baza profili IP |
| `wol_registry.sys` | tekstowy, analogiczny format | baza urządzeń Wake-on-LAN |
| `settings.json` | JSON | `Settings` — auto-update, interwał sprawdzania, ostatnio widziana/pominięta wersja |
| `networkmaster.log` | tekst | log zdarzeń startowych i błędów zapisu (`logf()`) |
| `announcements_seen.json` | JSON | ID potwierdzonych komunikatów od twórcy |
| `ssh_hosts.json` | JSON, **jawne hasła** | profile SSH (host, login, hasło i/lub klucz prywatny) |
| `ssh_known_hosts.json` | JSON | odciski kluczy hostów (TOFU) |
| `ftp_hosts.json` | JSON, **jawne hasła** | profile FTP |
| `vpn/` | pliki `.conf`/`.ovpn` + metadane | profile WireGuard/OpenVPN |
| `Reports/` | tekst/Markdown | raporty zapisane na życzenie z narzędzi diagnostycznych |
| backup VPN (`VPN_<znacznik czasu>.zip`) | ZIP z `manifest.json` | pełny backup profili VPN **z jawnymi hasłami/kluczami PSK/kluczami prywatnymi WireGuard** — świadoma decyzja (§9), nie przeoczenie |

**Hasła i klucze są przechowywane jawnie, bez szyfrowania, we wszystkich modułach jednolicie**
(VPN, SSH, FTP) — to udokumentowana, świadoma decyzja projektowa (prostota, przenośność — jeden
plik backupu ma zawierać *wszystko* potrzebne do odtworzenia konfiguracji), nie przeoczenie.
Konsekwencje bezpieczeństwa są opisane w §9 i w [Dokumentacja-NetworkMaster.md §7](Dokumentacja-NetworkMaster.md).

## 8. System aktualizacji i komunikatów

Program nie ma własnego serwera — "backend" to wprost API GitHub Releases repozytorium
wskazanego w `UpdateRepo` (`version.go`). `Updater.Check()` (`update.go`) odpytuje listę
wydań, `PickAsset()` wybiera plik odpowiadający `runtime.GOOS`/`GOARCH` (z tłumaczeniem
`darwin`→`macos` w nazwach plików), `expectedHash()` czyta sumę SHA-256 z pola `digest`
załącznika GitHuba albo z `checksums.txt`.

**Model bezpieczeństwa (z `RELEASING.md`):**
- Pobieranie wyłącznie przez HTTPS; przekierowanie na HTTP jest blokowane (`allowedURL()`).
- Zmienna `NETWORKMASTER_UPDATE_URL` (do testów bez publikowania, `tools/mockgithub`) działa
  **tylko** dla adresów pętli zwrotnej (`127.0.0.1`/`localhost`) — `isLoopbackHost()` —
  więc nie da się jej użyć do przekierowania prawdziwego klienta na obcy serwer (ochrona przed
  SSRF-podobnym nadużyciem tej furtki testowej).
- Suma SHA-256 jest **obowiązkowa** — brak sumy odrzuca aktualizację.
- Przed podmianą plik jest też sprawdzany pod kątem nagłówka wykonywalnego (`hasExecutableMagic` —
  `MZ` dla Windows, `ELF` dla Linuksa) — ochrona przed podmianą na plik, który nie jest wcale
  programem.
- Podmiana jest odwracalna: stary plik trafia do `NetworkMaster.exe.old` i wraca automatycznie,
  jeśli nowy plik nie wystartuje poprawnie (`applyUpdate()`, `cleanupOldBinary()`).
- **To, czego ten model NIE chroni:** przejęcia konta GitHub właściciela repozytorium. Sumy
  kontrolne pochodzą z tego samego wydania co plik — chronią przed uszkodzeniem transferu i
  przypadkowym błędem publikacji, nie przed złośliwym aktorem, który ma dostęp do konta
  publikującego wydania. Pełne zamknięcie tej furtki wymagałoby podpisu kryptograficznego
  (np. Ed25519) z kluczem publicznym wkompilowanym w program, niezależnym od konta GitHub —
  nie zaimplementowane, odnotowane jako możliwy kierunek rozwoju w `RELEASING.md`.
- Wydania oznaczone jako **draft** lub **pre-release** są ignorowane przez klienta.

Komunikaty od twórcy (`announcements.go`) używają tego samego wzorca "GitHub jako płytka,
publiczna baza danych", ale bez weryfikacji integralności — to kanał tylko-informacyjny
(tekst do wyświetlenia), nie wykonywalny kod, więc nie wymaga tego samego poziomu ochrony co
aktualizacje.

## 9. Bezpieczeństwo i prywatność

- **Brak telemetrii i konta użytkownika.** Program łączy się z internetem wyłącznie wtedy, gdy
  użytkownik sam uruchomi funkcję, która tego wymaga (sprawdzenie aktualizacji, komunikaty od
  twórcy, WHOIS, GeoIP, test prędkości, lookup MAC, sesje SSH/SFTP/FTP) — każde połączenie
  dotyczy tylko usługi, z którą dana funkcja jest bezpośrednio związana.
- **Polecenia systemowe uruchamiane bez powłoki** (`exec.Command` z osobnymi argumentami, nigdy
  `sh -c`/`cmd /c` ze sklejonym łańcuchem) — eliminuje klasę błędów wstrzyknięcia poleceń przez
  dane wejściowe użytkownika (np. nazwę hosta zawierającą `; rm -rf`).
- **Hasła/klucze przechowywane jawnie** (VPN, SSH, FTP) — świadoma decyzja, patrz §7. Plik
  backupu VPN wymaga tej samej ostrożności, co plik z hasłami.
- **TOFU dla SSH** (`ssh_hostkey.go`) — zmiana klucza hosta przy kolejnym połączeniu jest
  sygnalizowana wprost jako możliwy atak (man-in-the-middle), nie cicho ignorowana.
- **Ostrzeżenie o ekspozycji hasła przez argumenty procesu:** funkcja `[O] Otwórz w
  eksploratorze systemu` (SFTP/FTP) przekazuje adres `sftp://`/`ftp://` (z hasłem, jeśli profil
  je ma) do zewnętrznego procesu systemowego — hasło trafia chwilowo do listy argumentów tego
  procesu, widocznej innym procesom/użytkownikom tej maszyny (np. w Menedżerze Zadań). Program
  sygnalizuje to przed otwarciem, nie naprawia (nie da się tego naprawić bez zmiany mechanizmu
  skojarzeń protokołów samego systemu operacyjnego).
- **FTP jest nieszyfrowany** (ograniczenie samego protokołu z 1985 r., nie kodu) — hasło i
  transferowane pliki idą jawnym tekstem przez sieć, jeśli użytkownik wybierze FTP zamiast SFTP.
- **Brak podpisu cyfrowego binarek** (Authenticode/Apple Developer ID) — SmartScreen/Gatekeeper
  mogą pokazać ostrzeżenie "nieznany wydawca" przy pierwszym uruchomieniu. Próba obfuskacji
  (`garble`) w 2.4.0 została wycofana w 2.4.1, bo w praktyce dawała gorszy efekt: Google Chrome
  (Safe Browsing) blokował pobieranie pliku, oznaczając go jako wirusa. Podpis cyfrowy jest
  jedynym trwałym rozwiązaniem, nie zaimplementowanym w tej wersji (koszt certyfikatu, nie
  decyzja techniczna).
- Model bezpieczeństwa systemu aktualizacji — patrz §8.

## 10. Testowanie

`go test .` w `source_code/` — **108 testów w 13 plikach**, wykonują się w kilkanaście-kilkadziesiąt
sekund na Windows (jedyna platforma, na której testy faktycznie *uruchamiają się*, patrz §2).

| Plik testowy | Zakres |
|---|---|
| `logic_test.go` | parsery (`arp`, `ip neigh`, `ss`, `netsh`, `ping` w wielu językach systemowych), kalkulator podsieci, pozostała "czysta" logika |
| `keys_test.go` | mechanizm Ctrl+C/Ctrl+X (`errBack`, `recover`, `exitFunc` podmieniany na atrapę) |
| `vpn_test.go`, `vpn_windows_test.go` | parser/serializer WireGuard/OpenVPN, backup/restore, oraz (Windows) weryfikacja **na poziomie bajtów** (`unsafe.Sizeof`/`Offsetof`) układu struktur Win32 RAS/Menedżera Poświadczeń — bez wywoływania samych API kredencjałowych |
| `update_test.go` | cały system aktualizacji na atrapie API GitHuba (`tools/mockgithub`): wybór pliku per platforma/architektura, wykrywanie wydań krytycznych, weryfikacja SHA-256, odrzucanie podmienionych plików, podmiana z wycofaniem |
| `announcements_test.go` | komunikaty od twórcy: filtrowanie nieprzeczytanych, zapis historii potwierdzeń |
| `remote_test.go` | magazyn profili SSH/FTP, logika TOFU, poprawność kodowania URL (`sftp://`/`ftp://` ze specjalnymi znakami w haśle — `net/url.URL`, nie sklejanie stringów) |
| `ssh_server_test.go` + `ssh_integration_test.go` | **prawdziwy, lokalny serwer SSH** (`golang.org/x/crypto/ssh` jako serwer) — testy łączą się z nim prawdziwym klientem programu, nie atrapą wywołań |
| `ftp_server_test.go` + `ftp_integration_test.go` | **prawdziwy, lokalny serwer FTP** z realnym katalogiem tymczasowym — to właśnie te testy znalazły deadlock w `ftp_client.go` (§6.6) przed wydaniem |
| `platform_windows_test.go` | funkcje specyficzne dla Windows |
| `config_test.go` | `dataDirsWritable()` — regresja dla błędu opisanego w §12 |

**Poza automatycznym pakietem testów**, w ramach tej rundy przeglądu technicznego program został
**ręcznie zweryfikowany na żywym, zdalnym serwerze Ubuntu 26.04 LTS** (nie emulacja, nie
kontener) — połączenie przez prawdziwą sesję SSH z PTY, sterowanie tymi samymi klawiszami, które
wpisałby użytkownik. Zweryfikowano: budowanie krzyżowe → działający plik ELF, `--version`/
`--check-update`/`--help`, ekran zgody przy pierwszym uruchomieniu, komunikaty od twórcy, pełne
menu główne, IPConfig (realny `ip addr`/`ip route`/`/etc/resolv.conf`), tabelę ARP (fallback na
`ip neigh`, bo `arp` nie był nawet zainstalowany na tym hoście, plus poprawny lookup producenta
MAC dla dwóch realnych urządzeń w sieci), kalkulator podsieci, oraz menedżer VPN — w tym
poprawne, czytelne zgłoszenie braku `nmcli`/NetworkManager zamiast cichej awarii. Właśnie ta
weryfikacja znalazła i doprowadziła do naprawy błędu opisanego w §12 (cichy brak zapisu danych
przy wcześniejszym uruchomieniu jako root). Nie zweryfikowano na tym samym serwerze: realnego
połączenia WireGuard/OpenVPN (wymaga drugiego końca tunelu), monitora WiFi (serwer nie ma karty
WiFi) i macOS (brak dostępu do maszyny z tym systemem).

`go vet` musi przechodzić czysto na wszystkich 8 kombinacjach system/architektura przy każdym
budowaniu — wymusza to `build.ps1` (przerywa budowanie, jeśli `go vet` zwróci błąd na
jakiejkolwiek platformie).

## 11. Wydanie i dystrybucja — proces trzech repozytoriów

Projekt jest rozbity na **trzy repozytoria GitHub** pod kontem `NetworkMaster-x`:

| Repozytorium | Widoczność | Zawartość |
|---|---|---|
| `networkmaster-core` | prywatne | pełne zwierciadło kodu źródłowego (to, co kupuje nabywca) |
| `networkmaster` | publiczne | **tylko** dokumentacja, `CHANGELOG.md`, `LICENSE.md`, `announcements.json` i GitHub **Releases** (binarki) — zero kodu źródłowego |
| `networkmaster-x.github.io` | publiczne (GitHub Pages) | strona WWW — jeden plik `index.html`, osobna historia Git, musi mieć tę właśnie nazwę (`<konto>.github.io`), żeby Pages serwowało ją pod adresem głównym, plus plik `.nojekyll` (bez niego GitHub Pages próbuje przetworzyć stronę przez Jekyll i może się to nie powiedzie bez żadnego czytelnego błędu) |

**Synchronizacja między nimi jest w pełni ręczna** — nie ma żadnej automatyzacji (brak GitHub
Actions, brak webhooków). Przy każdym wydaniu trzeba ręcznie: skopiować zaktualizowane
`README.md`/`CHANGELOG.md`/`LICENSE.md`/`dokumentacja/*.md`/`announcements.json` do kopii
roboczej publicznego repo, commitować i wypychać osobno do każdego z trzech repozytoriów, a
potem jeszcze zbudować i opublikować wydanie binarek (`gh release create`, patrz
`RELEASING.md`). To jest największy punkt manualnej pracy w całym procesie wydawniczym i
najbardziej oczywisty kandydat do automatyzacji dla nowego właściciela (patrz §13).

**Zasada dotycząca starych wydań:** nigdy nie usuwa się wydania z GitHuba przy publikowaniu
nowego (poza jednorazowymi wydaniami testowymi lub naprawdę szkodliwym wydaniem, jak v2.4.0 —
patrz CHANGELOG.md) — każde wydanie to trwała historia i możliwość rollbacku.

## 12. Znane ograniczenia i dług techniczny

Pełna, opisana z perspektywy użytkownika lista jest w
[Dokumentacja-NetworkMaster.md §7](Dokumentacja-NetworkMaster.md#7-znane-ograniczenia). Tutaj —
to, co jest istotne z perspektywy kogoś oceniającego kod/proces:

1. **Brak CI/CD.** `build.ps1` to cały "pipeline" — uruchamiany ręcznie, na jednej maszynie
   (Windows autora). Nie ma automatycznego uruchamiania testów przy każdym pushu/PR.
2. **Synchronizacja trzech repozytoriów jest ręczna** (§11) — realne ryzyko rozjazdu, jeśli ktoś
   zapomni skopiować zmianę do wszystkich miejsc. Brak automatycznego mechanizmu wykrywania
   takiego rozjazdu.
3. **Testy automatyczne realnie wykonują się tylko na Windows** (§2, §10) — pokrycie
   Linux/macOS to kompilacja + `go vet` + (od niedawna, częściowo) ręczna weryfikacja na żywym
   serwerze Linux, nie automatyczny pakiet testów uruchamiany na tych systemach.
4. **Naprawiony w tej rundzie przeglądu (odnotowane jako precedens, nie jako otwarty problem):**
   cichy brak zapisu danych (ustawienia, log, historia komunikatów, bazy IP/WOL), gdy
   `core_data/`/`Reports/` zostały wcześniej utworzone jako root — mechanizm fallbacku do
   katalogu konfiguracji użytkownika sprawdzał tylko katalog bazowy, nie te podfoldery.
   Znalezione przez ręczny test na żywym serwerze Ubuntu (nie przez automatyczny pakiet
   testów — dowód, że testowanie na realnym, "zabrudzonym" środowisku łapie klasy błędów, które
   testy na czystym, tymczasowym katalogu (`t.TempDir()`) nie złapią sam z siebie, dopóki ktoś
   nie doda testu odtwarzającego właśnie ten scenariusz — co zostało zrobione, `config_test.go`).
   Patrz CHANGELOG.md (wersja 2.5.2).
5. **Brak podpisu cyfrowego** (§9) — SmartScreen/Gatekeeper pokazują ostrzeżenie "nieznany
   wydawca" przy pierwszym uruchomieniu/pobraniu każdej binarki.
6. **`vpn_ui.go` (876 linii)** to największy plik projektu i naturalny kandydat do podziału,
   jeśli funkcjonalność VPN dalej rośnie (np. kolejne protokoły) — obecnie nie jest to pilny
   problem, ale warto mieć to na radarze przy większej rozbudowie.
7. **Model bezpieczeństwa aktualizacji chroni przed uszkodzonym transferem, nie przed
   przejęciem konta GitHub** (§8) — pełne zamknięcie tej furtki wymaga podpisu kryptograficznego
   niezależnego od GitHuba, nieobecnego w tej wersji.
8. **Reszta ograniczeń jest specyficzna funkcjonalnie** (macOS i tworzenie profili VPN, Linux i
   wtyczki NetworkManager, Windows 32-bit i odczyt zapamiętanych haseł VPN, klawiatura w sesji
   SSH na Windows, FTP bez szyfrowania) — opisana szczegółowo w dokumentacji użytkownika, nie
   powtarzana tutaj.

## 13. Proponowany rozwój (roadmapa)

Pomysły w naturalny sposób wynikające z architektury i ograniczeń opisanych wyżej, nie
zobowiązanie — do oceny przez nowego właściciela:

- **Automatyzacja CI/CD** (GitHub Actions: `go vet`/`go test` przy każdym PR, docelowo też
  automatyczne budowanie i publikacja wydań) — największy pojedynczy krok redukujący ryzyko
  błędu ludzkiego w procesie opisanym w §11.
- **Testy Linux/macOS w CI** (np. przez macierz `runs-on` w GitHub Actions) — zamiast polegać
  tylko na kompilacji krzyżowej i ręcznej weryfikacji.
- **Podpis cyfrowy binarek** (Authenticode na Windows, Apple Developer ID na macOS) — usuwa
  ostrzeżenia systemowe, koszt to głównie certyfikat, nie zmiana kodu.
- **GUI** (np. jako alternatywny front-end korzystający z tej samej logiki domenowej) — obecna
  architektura (logika w osobnych plikach od UI terminala) częściowo to ułatwia, ale `vpn_ui.go`
  i inne pliki `*_ui.go` mieszają logikę z prezentacją silniej niż reszta kodu.
- **Dodatkowe protokoły VPN / rozszerzenie klienta SSH** (np. agent SSH, `ProxyJump`) — logika
  klienta SSH jest już wyodrębniona (`ssh_client.go`) na tyle, że rozszerzenie nie wymaga
  przebudowy reszty programu.
- **Podpis kryptograficzny aktualizacji niezależny od GitHuba** (§8, §12 pkt 7).

## 14. Słownik pojęć

| Termin | Znaczenie w tym projekcie |
|---|---|
| **TOFU** | Trust On First Use — ufaj kluczowi serwera przy pierwszym połączeniu, ostrzegaj przy każdej zmianie. Ten sam model co `known_hosts` OpenSSH; tu: `ssh_hostkey.go`. |
| **Build tag** | znacznik `//go:build <warunek>` nad plikiem `.go`, decydujący, czy plik wchodzi do kompilacji dla danego `GOOS`/`GOARCH`. Mechanizm separacji kodu platformowego w tym projekcie. |
| **Portable (program)** | brak instalatora; dane własne programu żyją obok pliku wykonywalnego, nie w systemowych lokalizacjach — przenosisz folder, przenosisz stan. |
| **RAS** | Remote Access Service — natywne API Windows do VPN/dial-up, używane przez `rasapi_windows.go` i `vpn_windows.go`. |
| **`nmcli`** | interfejs linii poleceń NetworkManagera (Linux) — brak tego pakietu = brak VPN systemowego na danym Linuksie, zgłaszane programowi wprost. |
| **`scutil --nc`** | narzędzie systemowe macOS do zarządzania konfiguracjami sieciowymi, w tym VPN — jedyny sposób programowej integracji z VPN systemowym macOS, i to tylko do odczytu/połączenia z istniejącymi profilami. |
| **SSRF (Server-Side Request Forgery)** | klasa ataku polegająca na zmuszeniu programu do wysłania żądania do nieautoryzowanego celu. Tu istotne przy `NETWORKMASTER_UPDATE_URL` (§8) — ograniczone do adresów pętli zwrotnej właśnie w tym celu. |
| **Magic bytes** | pierwsze bajty pliku identyfikujące jego format (`MZ` dla plików `.exe`, `ELF` dla binarek Linuksa) — używane przy weryfikacji pobranej aktualizacji. |
| **Authenticode / Gatekeeper / SmartScreen** | mechanizmy Windows/macOS ostrzegające przed niepodpisanymi cyfrowo plikami wykonywalnymi. |
