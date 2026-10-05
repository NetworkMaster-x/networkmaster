# Polityka Prywatności — NetworkMaster

**To jest szablon przygotowany bez udziału prawnika/specjalisty RODO.** Przed jakimkolwiek
komercyjnym wykorzystaniem tego dokumentu skonsultuj go z prawnikiem specjalizującym się w
ochronie danych osobowych. Ten dokument sam w sobie nie stanowi porady prawnej.

Wersja: 1.0 · Dotyczy programu: NetworkMaster (od wersji 2.4.0)

## 1. Ogólna zasada: Program działa lokalnie

NetworkMaster **nie ma konta użytkownika, nie zawiera telemetrii ani analityki i nie wysyła
żadnych danych do Właściciela programu.** Cała logika działa lokalnie, na urządzeniu
Użytkownika. Właściciel programu nie posiada żadnego serwera zbierającego dane z działania
Programu i nie ma wglądu w to, jak i do czego Program jest używany.

## 2. Połączenia sieciowe wychodzące z Programu

Program łączy się z internetem **wyłącznie wtedy, gdy Użytkownik sam aktywnie uruchomi
konkretną funkcję tego wymagającą** z menu głównego. Poniżej pełna lista takich połączeń:

| Funkcja w programie | Usługa zewnętrzna | Jakie dane są wysyłane |
|---|---|---|
| Sprawdzanie aktualizacji (`UP`) | `api.github.com` | standardowe zapytanie HTTP do publicznego API GitHub; bez danych osobowych |
| WHOIS (`X`) | `rdap.org` | sprawdzana domena/adres IP |
| GeoIP (`G`) | `ipwho.is`, `ip-api.com` | sprawdzany adres IP (własny lub dowolny wskazany przez Użytkownika) |
| Test prędkości łącza (`SP`) | `speed.cloudflare.com` | dane testowe (transfer), bez danych osobowych |
| Szybka diagnoza sieci (`V`) | `www.cloudflare.com/cdn-cgi/trace`, `connectivitycheck.gstatic.com` | zapytanie sprawdzające kondycję połączenia |
| MAC Lookup (`M`) | `api.macvendors.com` | sprawdzany adres MAC — **tylko** gdy lokalna, wbudowana baza producentów (ok. 39,7 tys. wpisów IEEE) nie zna danego adresu |

Każde z powyższych połączeń idzie **bezpośrednio** z urządzenia Użytkownika do wskazanej usługi
zewnętrznej. Właściciel programu nie pośredniczy w tej transmisji, nie ma do niej dostępu ani
wglądu. Korzystanie z tych funkcji podlega odpowiednio politykom prywatności: GitHub, dostawców
usługi RDAP, ipwho.is, ip-api.com, Cloudflare, Google oraz macvendors.com — Użytkownik powinien
się z nimi zapoznać, jeśli ma to dla niego znaczenie.

## 3. Dane przechowywane lokalnie

Program zapisuje dane wyłącznie lokalnie, w folderach obok pliku programu:

- **`core_data/`** — ustawienia programu, baza zapisanych profili konfiguracji IP, baza urządzeń
  Wake-on-LAN, dziennik zdarzeń programu, flaga pierwszego uruchomienia.
- **`Reports/`** — wygenerowane przez Użytkownika raporty diagnostyczne.
- **Opcjonalne pliki `.zip`** — kopie zapasowe konfiguracji VPN (systemowego, WireGuard,
  OpenVPN), tworzone wyłącznie na wyraźne żądanie Użytkownika (funkcja backupu w menu VPN).

Użytkownik ma pełną kontrolę nad tymi danymi — usunięcie odpowiednich plików/folderów usuwa dane
bez pozostawiania kopii gdziekolwiek indziej.

## 4. Ważne ostrzeżenie: kopie zapasowe VPN zawierają hasła w postaci jawnej

Pliki kopii zapasowych VPN (`.zip`) mogą zawierać **hasła, klucze PSK oraz klucze prywatne
WireGuard zapisane w postaci jawnej (niezaszyfrowanej)**. To celowy wybór projektowy — backup ma
być kompletny i pozwalać na pełne odtworzenie konfiguracji. W praktyce oznacza to, że taki plik
należy traktować z taką samą ostrożnością jak plik zawierający hasła: nie wysyłać go
niezaszyfrowanym kanałem (np. zwykłym mailem), nie przechowywać w niezabezpieczonej chmurze i
zabezpieczyć dostęp do niego tak, jak do menedżera haseł.

## 5. Podstawa prawna i zgodność z RODO

Ponieważ Właściciel programu nie prowadzi żadnego przetwarzania danych osobowych Użytkowników
(brak serwera, brak konta, brak zbierania danych po stronie Właściciela), większość obowiązków
administratora danych osobowych wynikających z RODO nie znajduje tu zastosowania w klasycznym
sensie. Należy jednak mieć na uwadze, że:

- funkcje opisane w §2 (GeoIP, WHOIS, sprawdzanie aktualizacji, MAC Lookup) mogą przekazywać
  adres IP Użytkownika **bezpośrednio** do wskazanych usług trzecich — to te podmioty, nie
  Właściciel programu, stają się wówczas administratorem tych danych w zakresie własnego
  przetwarzania,
- Użytkownik korzystający z tych funkcji robi to świadomie, uruchamiając je z menu programu.

## 6. Zmiany Polityki Prywatności

Właściciel zastrzega sobie prawo do aktualizacji niniejszej Polityki Prywatności, w
szczególności w związku ze zmianami w funkcjonalności Programu lub modelu jego udostępniania
(patrz [Regulamin, §8](Regulamin.md)).

## 7. Kontakt

W sprawach dotyczących prywatności i danych: **pxware@pxware.pl**.
