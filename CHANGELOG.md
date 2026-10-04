# Lista zmian

## 2.4.2

Wydanie testowe – wyłącznie zmiana numeru wersji, oznaczone jako **krytyczne**, żeby
sprawdzić w praktyce mechanizm obowiązkowych aktualizacji (banner, brak opcji „Pomiń tę
wersję”, wymuszone `T`/`N`). Brak zmian w kodzie ani funkcjach.

## 2.4.1

### Poprawki błędów
- **Wycofano obfuskację garble wprowadzoną w 2.4.0.** Powód: w praktyce, na realnym
  wydaniu, Google Chrome (Safe Browsing) blokował pobieranie `NetworkMaster-windows-amd64.exe`
  jako "wirus" – nie tylko Windows Defender na maszynie deweloperskiej (to już wiedzieliśmy),
  ale też przeglądarka blokująca pobranie dla zwykłych użytkowników, czyniąc plik praktycznie
  niepobieralnym. `build.ps1` wrócił do zwykłego `go build` (bez obfuskacji) – pliki są
  znowu ~8 MB, bez zaciemniania nazw/stałych tekstowych. **Jedyny trwały sposób na ochronę
  przed dekompilacją bez tego efektu to podpisanie binarek certyfikatem Authenticode** –
  kosztowe (~100-400$/rok) i wymaga weryfikacji tożsamości/firmy u wystawcy, ale buduje
  realną reputację w SmartScreen/Safe Browsing. Nie wdrożone w tej wersji.
- Wydanie v2.4.0 na GitHubie zostało usunięte i zastąpione przez v2.4.1 z czystymi
  (nie-garble) binarkami – ten sam powód.

## 2.4.0

**To jest pierwsze publiczne wydanie** – opublikowane jako GitHub Release
([NetworkMaster-x/networkmaster](https://github.com/NetworkMaster-x/networkmaster/releases/tag/v2.4.0)),
razem ze stroną WWW ([networkmaster-x.github.io/networkmaster-site](https://networkmaster-x.github.io/networkmaster-site/)).

### Bezpieczeństwo i dystrybucja
- **Ekran zgody przy pierwszym uruchomieniu** – program wymaga teraz wpisania `TAK`/`T`,
  jasno informując o Licencji, Regulaminie i Polityce Prywatności (link `LegalURL`).
- **Krytyczne (obowiązkowe) aktualizacje** – wydanie z tytułem zaczynającym się od
  `[KRYTYCZNA]`/`[CRITICAL]` nie może zostać trwale pominięte (opcja „Pomiń tę wersję”
  jest wtedy zablokowana); banner w menu głównym staje się czerwony i bardziej naglący.
- **Komunikaty od twórcy** – program ściąga `announcements.json` z publicznego repo przy
  starcie; każdy nieprzeczytany komunikat wymaga potwierdzenia Enterem, historia
  potwierdzeń jest dostępna z menu Ustawień (`US` → `5`).
- ~~**Ochrona przed dekompilacją: `build.ps1` teraz domyślnie buduje przez garble**~~ –
  **WYCOFANE w 2.4.1** (patrz wyżej) – na realnym wydaniu Chrome blokował pobieranie jako
  "wirus". W momencie wprowadzenia potwierdzone testem, że nazwy funkcji takie jak
  `systemVPNSecrets`/`rasDialParams`, widoczne jawnie w zwykłej binarce, nie występowały w
  ogóle w binarce zbudowanej przez garble – ale koszt (fałszywe alarmy u użytkowników)
  okazał się zbyt wysoki w praktyce, nie tylko w teorii.
- **Repozytorium zmieniło nazwę z `update` na `networkmaster`** (GitHub:
  `NetworkMaster-x/networkmaster`) – program sam o tym wie (`UpdateRepo`/`RepoURL` w
  `version.go`). Struktura: `networkmaster` (publiczne – wydania, dokumentacja, Licencja,
  Regulamin, Polityka Prywatności), `networkmaster-core` (prywatne – kod źródłowy),
  `networkmaster-site` (publiczne – strona WWW, w przygotowaniu).

### Nowości
- **Wbudowany VPN systemu teraz działa na wszystkich trzech platformach, nie tylko na
  Windows** – dodawanie/usuwanie/edycja/połączenie/rozłączenie/backup/przywracanie z tym
  samym menu (`VPN`) co dotychczas:
  - **Linux**: profile L2TP i PPTP przez NetworkManager (`nmcli`), wymaga zainstalowanej
    wtyczki `NetworkManager-l2tp` / `NetworkManager-pptp` (opcjonalne pakiety – program
    jasno zgłasza ich brak zamiast cichego niepowodzenia). Hasła/PSK do backupu są czytane
    wprost z plików `/etc/NetworkManager/system-connections/*` jako root.
  - **macOS**: połączenie/rozłączenie/status dla profili VPN, które użytkownik już
    skonfigurował w Ustawieniach Systemu (Sieć > VPN), przez `scutil --nc`. Tworzenie,
    edycja i usuwanie profili z poziomu programu **nie jest możliwe** – to twarde,
    udokumentowane ograniczenie samego macOS (Apple nie udostępnia takiego API poza
    podpisanymi aplikacjami z odpowiednim uprawnieniem), program tłumaczy to wprost w menu
    VPN. `scutil` w ogóle nie widzi profili IKEv2 (tylko starsze L2TP/PPP) – kolejne
    ograniczenie samego narzędzia Apple, nie tego programu. Hasła profili systemowych leżą
    w Keychainie i nie da się ich odczytać do backupu bez ręcznej zgody dla każdego wpisu.
  - Menu VPN pokazuje teraz dokładnie, co jest obsługiwane na danej platformie (baner z
    wyjaśnieniem ograniczeń zamiast po prostu ukrywania opcji).
  - **Windows**: bez zmian funkcjonalnych – wszystko, co działało w 2.3.4, działa dalej
    identycznie (Ctrl+C/Ctrl+X, RAS, Menedżer Poświadczeń, test samotestujący `T`).
- WireGuard i OpenVPN bez zmian – działały identycznie na wszystkich platformach już wcześniej.
- **Nowe wersje 32-bit (386): `NetworkMaster-windows-386.exe` i `NetworkMaster-linux-386`** –
  `build.ps1` buduje teraz 8 plików zamiast 6 (macOS zostaje tylko amd64/arm64, bo Apple
  porzuciło 32-bit x86 w 2017, a 32-bit w ogóle w 2019 – nie ma czego budować).

### Poprawki błędów
- **System aktualizacji nigdy nie znajdywał pliku dla macOS** – `PickAsset` budował
  oczekiwaną nazwę z dosłownego `runtime.GOOS` (`darwin`), a pliki wydań nazywają się
  `NetworkMaster-macos-*` (patrz `build.ps1`); rozmyte dopasowanie zapasowe też nie miało
  wpisu dla `darwin` w mapie tokenów systemu. W efekcie `--check-update` na macOS zawsze
  kończyłby się błędem „wydanie nie zawiera pliku dla darwin/…”, nawet gdyby wydanie
  zawierało właściwy plik. Naprawione przez `assetOSName()` (tłumaczy `darwin`→`macos`) i
  dodanie tokenów `macos/darwin/osx/mac` do rozmytego dopasowania; pokryte nowym testem w
  `update_test.go`. Błąd nie mógł ujawnić się wcześniej testami na Windows – wykryty przy
  przeglądzie kodu, nie przez użytkownika.

### Porządki w repozytorium (przygotowanie pod GitHub)
- Usunięto zarchiwizowany kod źródłowy wersji 1.x (`source_code_linux/`,
  `source_code_windows/`, Python) i przestarzały `NetworkMaster_Technical_Documentation_PL.pdf`
  (dotyczył tamtej wersji) – oba zastąpione tą bazą kodu (Go) i nową dokumentacją.
  Historia obu pozostaje dostępna w historii gita.
- Repozytorium zainicjalizowane w git (`git init`) z `.gitignore` wykluczającym dane
  wykonawcze (`core_data/`, `Reports/`) – nie trafiają do repo, bo i tak odtwarzają się
  same przy pierwszym uruchomieniu.
- Katalog główny zawiera już tylko kod, dokumentację i licencję – wszystkie gotowe binarki
  (8 platform) i `checksums.txt` są wyłącznie w `dist/`, zgodnie z tym, co opisuje
  `source_code/README.md` i `RELEASING.md`.

### Ważne zastrzeżenie
- Wsparcie dla Linuksa i macOS w tej wersji przeszło kompilację i `go vet` na wszystkich
  architekturach, ale **nie zostało uruchomione na żywym Linuksie ani macOS** (środowisko,
  w którym powstało, ma tylko Windows) – składnia `nmcli`/`scutil` została zestawiona z kilku
  niezależnych źródeł, nie z jednego autorytatywnego testu. Przetestuj przed poleganiem na
  tym produkcyjnie.
- Na 32-bit Windows (`windows-386`) odczyt/zapis zapamiętanych haseł i PSK VPN
  (`rasapi_windows.go`) prawdopodobnie nie zadziała – struktura `RASDIALPARAMSW` była
  zweryfikowana tylko dla 64-bit. Reszta programu nie ma tego ograniczenia.

## 2.3.4

### Nowości
- **Druga, niezależna droga odczytu zapamiętanego loginu/hasła VPN: Menedżer Poświadczeń
  Windows** (`CredEnumerateW`/`wincred.h`), używana automatycznie jako zapasowa, gdy klasyczne
  API RAS nic nie zwróci. Test samotestujący (`VPN` → `T`) teraz sprawdza obie drogi i pokazuje,
  która zadziałała.
- **Ważne zastrzeżenie:** test wciąż nie łączy się z prawdziwym serwerem VPN (bo go nie ma),
  a Windows może zapisywać hasło do Menedżera Poświadczeń dopiero PO realnym, udanym
  połączeniu – nie tylko po zaznaczeniu „zapamiętaj”. Ujemny wynik testu w tej wersji może więc
  nie być ostatecznym dowodem, że ta droga nie działa dla prawdziwych, używanych połączeń.

## 2.3.3

### Poprawki błędów
- **Błąd 632 nadal występował po poprawce struktury w 2.3.2** – wygląda na to, że ten konkretny
  komputer/wersja Windows nie akceptuje najnowszego (Windows 8+) kształtu `RASDIALPARAMSW`.
  Zamiast zgadywać kolejną pojedynczą wartość, program teraz **próbuje po kolei cztery znane
  warianty rozmiaru struktury** (2128/2120/2112/2096 bajtów, odpowiadające różnym wersjom
  Windows) i używa pierwszego, który Windows zaakceptuje. Jeśli żaden nie zadziała, komunikat
  błędu pokazuje wynik wszystkich prób, żeby dało się to dalej zdiagnozować.

## 2.3.2

### Poprawki błędów
- **Odczyt/zapis zapamiętanego loginu i hasła VPN dawał błąd 632 (`ERROR_INVALID_SIZE`)** –
  struktura `RASDIALPARAMSW` w Go miała złe pola: `dwCallbackId` to `ULONG_PTR` (8 bajtów na
  64-bit), a nie zwykły `DWORD` (4 bajty), i brakowało pól `dwIfIndex`/`szEncPassword`
  dodanych w Windows 7/8. Poprawiono na podstawie autentycznego nagłówka Windows SDK; test
  na żywo (`VPN` → `T`) wskazał ten błąd. **Wymaga ponownego potwierdzenia** – zweryfikuj
  jeszcze raz przez `VPN` → `T`.
- Odczyt PSK dla profili L2TP utworzonych przez `Add-VpnConnection -L2tpPsk` prawdopodobnie
  nie jest możliwy przez klasyczne API `RasGetCredentials` – dokumentacja Microsoftu sugeruje,
  że ten PSK trafia do nowszego magazynu profili VPN, a nie do klasycznego magazynu
  poświadczeń RAS. Zaktualizowano komunikaty w programie, żeby to jasno tłumaczyły zamiast
  sugerować nieznaną usterkę.

## 2.3.1

### Nowości
- **Test odczytu haseł VPN (`VPN` → `T`):** samotestująca się diagnostyka, która sprawdza, czy zapis
  i odczyt zapamiętanych haseł/PSK działa na danym komputerze – **bez potrzeby prawdziwego serwera
  VPN ani internetu**. Tworzy dwa jednorazowe profile testowe, zapisuje w nich znaną wartość,
  odczytuje ją z powrotem, porównuje i zawsze sprząta po sobie. Uruchom to po aktualizacji z 2.3.0,
  żeby sprawdzić, czy backup z hasłami będzie działać na Twoim komputerze.

## 2.3.0

### Nowości
- **Pełny backup VPN teraz obejmuje hasła i PSK profili systemowych Windows.** Program odczytuje
  zapamiętane dane logowania i klucz PSK tą samą udokumentowaną metodą Win32 (`rasapi32.dll`),
  której od lat używają narzędzia takie jak NirSoft do odzyskiwania zapamiętanych haseł
  dial-up/VPN – to nie jest obejście zabezpieczeń, tylko API, po które sam Windows sięga, żeby
  wypełnić okno logowania przy kolejnym połączeniu. Działa dla profilu **bieżącego użytkownika**,
  wyłącznie gdy zaznaczono „Zapamiętaj dane logowania” i choć raz udało się połączyć.
  - **Ograniczenie tej wersji:** sam odczyt/zapis poświadczeń nie mógł zostać przetestowany na
    żywo w środowisku, w którym program powstawał (zablokowane przez wewnętrzny klasyfikator
    bezpieczeństwa Claude Code jako „eksploracja poświadczeń”) – zaimplementowano na podstawie
    dokumentacji Microsoft, a układ pamięci struktur zweryfikowano niezależnie przez porównanie
    z `Marshal.SizeOf` w .NET (zgodność co do bajtu). **Przetestuj u siebie** przed poleganiem na
    tym w produkcji: utwórz profil VPN, zaznacz „Zapamiętaj dane logowania”, połącz się raz,
    zrób backup i sprawdź plik `manifest.json` w archiwum.
  - Klucz PSK w formularzu edycji profilu L2TP jest teraz pokazywany wprost (o ile da się go
    odczytać), zamiast komunikatu, że Windows go nie ujawnia.

## 2.2.0

### Nowości
- **Wersja macOS** (`darwin/amd64`, `darwin/arm64`): karty sieciowe, konfiguracja IP/DHCP,
  ARP, połączenia sieciowe (`lsof`), DNS, WHOIS, SSL, skanery, test prędkości, WireGuard i
  OpenVPN działają tak samo jak na Windows/Linuksie. Zbudowana i przechodzi `go vet`, ale
  **nie była uruchomiona na prawdziwym Macu** (brak takiego komputera przy tworzeniu) –
  przed poleganiem na niej produkcyjnie przetestuj podstawowe funkcje.
  - Ograniczenia specyficzne dla macOS: brak wbudowanego VPN systemowego (jak w Windows) –
    dostępne są WireGuard/OpenVPN; siła sygnału WiFi jest niedostępna z terminala na
    nowoczesnym macOS (Apple usunęło do tego proste, niewymagające sudo API); hasła WiFi
    z Keychaina wymagają zgody użytkownika w oknie systemowym dla każdej sieci z osobna.

### Poprawki błędów
- `normalizeMAC` odrzucał poprawne adresy MAC w formacie BSD/macOS, gdzie `arp -a` nie
  dopełnia zerami pojedynczych oktetów (np. `8:0:27:0:0:1` zamiast `08:00:27:00:00:01`) –
  dotyczyłoby to również ręcznie wpisanych adresów w tym stylu na każdej platformie.

## 2.1.0

### Nowości
- **Skróty klawiszowe w całym programie:** **Ctrl+C** cofa o jedno okno (w menu głównym kończy program),
  **Ctrl+X** natychmiast zamyka cały program – z menu, kreatorów, pól wpisywania, skanów, dashboardu,
  testu prędkości, a nawet w trakcie pingu ciągłego. Działa od razu, bez Entera. Nad każdym menu jest przypomnienie.
- **Menedżer VPN (`VPN`):** pełne dodawanie / edycja / usuwanie oraz backup i przywracanie dla:
  - **wbudowanego VPN Windows** (PPTP/L2TP/SSTP/IKEv2) przez natywne API systemu — łącznie z listą, statusem,
    łączeniem/rozłączaniem (`rasdial`) i profilami dla wszystkich użytkowników (wymaga Administratora);
  - **WireGuard** — kreator (z generowaniem pary kluczy, jeśli zainstalowane `wireguard-tools`) lub wklejenie
    gotowego `.conf`, edycja adresu/DNS/MTU i zarządzanie peerami bez naruszania nieznanych dyrektyw/komentarzy,
    połączenie jako usługa systemowa (Windows, WireGuard for Windows) lub przez `wg-quick` (Linux);
  - **OpenVPN** — import istniejącego `.ovpn` razem z plikami certyfikatów, na które wskazuje, albo wklejenie
    treści; połączenie na pierwszym planie (`openvpn --config`).
  - **Pełny backup VPN** pakuje wszystkie trzy rodzaje profili do jednego pliku `.zip` (`Backups\VPN_*.zip`)
    wraz z kluczami WireGuard i certyfikatami OpenVPN; **przywracanie** odtwarza je selektywnie, z opcją
    nadpisania. Hasła/PSK profili systemowych nie trafiają do kopii (Windows ich nie ujawnia, podobnie jak haseł WiFi).
  - Na Linuksie wbudowany VPN systemu nie jest dostępny (brak jednego uniwersalnego API jak w Windows) –
    program jasno to komunikuje i kieruje do WireGuard/OpenVPN.

### Poprawki błędów
- `psRun` (silnik zapytań PowerShell) nie przechwytywał `stderr`, więc każdy błąd PowerShell docierał jako
  bezużyteczne „exit status 1” bez treści. Teraz komunikat błędu PowerShell trafia bezpośrednio do użytkownika.
- Skaner dyrektyw plików OpenVPN (`ca`/`cert`/`key`/...) źle dzielił linie z cudzysłowem, więc ścieżka pliku
  ze spacją w nazwie (`key "client key.key"`) nigdy nie była rozpoznawana i nie trafiała do kopii/importu.

## 2.0.0

Program został przepisany w języku **Go**: jeden przenośny plik (Windows i Linux), bez rozpakowywania
się do folderu tymczasowego przy każdym starcie, mniejszy (8 MB zamiast 12,5 MB) i dużo szybszy.
Dane z wersji 1.x (profile IP, baza WOL) są w pełni zgodne.

### Nowości
- **Skróty klawiszowe w całym programie:** **Ctrl+C** cofa o jedno okno (w menu głównym kończy program),
  **Ctrl+X** natychmiast zamyka cały program – z menu, kreatorów, pól wpisywania, skanów, dashboardu,
  testu prędkości, a nawet w trakcie pingu ciągłego. Działa od razu, bez Entera. Nad każdym menu jest przypomnienie.
- **Aktualizacje z GitHuba:** program sprawdza nowe wersje, pokazuje listę zmian i pyta, czy zaktualizować
  (Tak / Nie teraz / Pomiń tę wersję). Pobrany plik jest weryfikowany sumą SHA-256. Komenda `UP`.
- **Szybka diagnoza sieci (`V`):** sprawdza kartę, bramę, Internet, DNS, HTTPS i portal logowania,
  po czym podaje werdykt i wskazówkę, co naprawić.
- **Test prędkości łącza (`SP`):** opóźnienie, pobieranie i wysyłanie.
- **Skaner sieci LAN:** prawdziwa maska podsieci, nazwy hostów, producent z wbudowanej bazy offline, oznaczenie
  własnego urządzenia i bramy, ponowne skanowanie, raport.
- **Skaner portów:** wielowątkowy (sekundy zamiast minut), zakresy i listy portów, banery usług.
- **DNS:** zapytania o rekordy A/AAAA/MX/NS/TXT/CNAME/PTR do wybranego serwera; benchmark mierzy prawdziwe
  zapytania DNS, a nie ping.
- **Netstat z nazwami procesów** i filtrem.
- **Audyt SSL:** pokazuje także certyfikaty wygasłe i samopodpisane, SAN, protokół, szyfr, zaufanie łańcucha.
- **WHOIS/RDAP** także dla adresów IP, **GeoIP** dla dowolnego adresu.
- **Kalkulator podsieci:** wildcard, maska binarna, klasa, typ adresu, podział na podsieci.
- **Edytor hosts:** wpisy numerowane, usuwanie, automatyczne kopie zapasowe i przywracanie.
- **Wykrywanie uprawnień administratora** i ponowne uruchomienie jako Administrator (`ADM`).
- **Przeglądarka raportów** (`RA`), ustawienia i dziennik błędów (`US`), dashboard z wykresem opóźnienia.

### Poprawki błędów
- WHOIS na Windows nigdy nie odczytywał danych: wyszukiwał fragmenty JSON w tekście, który PowerShell zwraca jako
  sformatowany obiekt (zawsze „Brak danych”). Teraz używa API RDAP bezpośrednio.
- Polecenia budowane z tekstu wpisanego przez użytkownika trafiały do powłoki (`shell=True`), np. `ping 8.8.8.8 & ...`.
  Teraz argumenty są przekazywane bezpośrednio i walidowane.
- Kalkulator podsieci (Windows) dla dużych sieci, np. /8, budował listę ~16 mln hostów.
- „Zerowanie sieci” zawsze wyświetlało `[OK]`, nawet gdy krok się nie powiódł (np. brak uprawnień). Teraz sprawdza
  kody wyjścia i wymaga uprawnień administratora.
- Raporty WiFi (z hasłami), ARP, ipconfig i szybkiego DNS były zapisywane bez pytania – teraz tylko na życzenie.
- Nazwy plików raportów z niedozwolonymi znakami (np. adres URL wpisany zamiast domeny) kończyły się błędem zapisu.
- Odczyt list kart/wyników poleceń zależał od przecinków w CSV, apostrofów i języka systemu (`ping`, `netsh`) –
  teraz używa JSON i rozpoznaje odpowiedzi niezależnie od języka.
- Linux: hasło WiFi zawierające znak `=` było ucinane; brakujący `import time` w edytorze hosts (błąd wykonania);
  `cls` zamiast `clear` przy pierwszym uruchomieniu; nieprawidłowa sekwencja ucieczki w wyrażeniu regularnym.
- WOL: dla urządzeń dodanych ze skanera pakiet szedł tylko na adres hosta zamiast na broadcast; puste linie w bazie
  tworzyły puste profile. Teraz pakiet wychodzi na broadcast każdego aktywnego interfejsu.
- Zapisy baz danych są atomowe (przerwanie zapisu nie uszkodzi pliku); adresy IP/maski są walidowane przed zastosowaniem.
- Edytor hosts dopisywał pustą linię przed każdym wpisem i przy braku uprawnień kończył się nieobsłużonym wyjątkiem.
  Teraz robi kopię zapasową i jasno informuje o braku uprawnień.
- Eksport profili WiFi na Windows działa także wtedy, gdy system ukrywa nazwy (brak zgody na lokalizację).
