# NetworkMaster

NetworkMaster to narzędzie do diagnostyki sieciowej stworzone, aby uprościć zarządzanie lokalną
siecią. Działa w terminalu, bez instalatora, na Windows, Linuksie i macOS.

## 📥 Pobieranie

Gotowe pliki wykonywalne dla wszystkich platform (Windows/Linux/macOS, amd64/arm64, oraz
386 dla Windows/Linuksa) znajdziesz w zakładce **[Releases](../../releases)** tego repozytorium.
Pobierz plik odpowiadający Twojemu systemowi i uruchom go w terminalu — nie wymaga instalacji.

## 🛠 Funkcje

* **Diagnostyka:** szybka diagnoza sieci, Ping, Traceroute, DNS (rekordy, propagacja, benchmark), test prędkości łącza, SSL/TLS, WHOIS, GeoIP.
* **Skanowanie:** porty (wielowątkowo), podsieci, urządzenia w LAN (nazwy, producenci offline).
* **Zarządzanie:** konfiguracja kart i profile IP, Wake-on-LAN, edycja pliku hosts z kopiami zapasowymi, raporty.
* **VPN:** wbudowany VPN systemu (Windows: PPTP/L2TP/SSTP/IKEv2 przez RAS; Linux: L2TP/PPTP przez
  NetworkManager; macOS: łączenie/rozłączanie z profilami utworzonymi w Ustawieniach Systemu –
  tworzenie nowych profili na macOS nie jest technicznie możliwe spoza podpisanej aplikacji Apple),
  WireGuard i OpenVPN na wszystkich platformach – dodawanie, edycja, usuwanie, łączenie/rozłączanie,
  pełny backup i przywracanie (`.zip`).
* **Monitorowanie:** dashboard w czasie rzeczywistym, pasmo, WiFi, połączenia z nazwami procesów.
* **Aktualizacje:** program sam sprawdza nowe wersje (w tym repozytorium), pokazuje listę zmian
  i pyta o zgodę — z wyjątkiem aktualizacji oznaczonych jako krytyczne/obowiązkowe.
* **Komunikaty od twórcy:** program może pokazać ogłoszenie przy starcie (wymaga potwierdzenia),
  z lokalną historią potwierdzeń dostępną z menu Ustawień.
* **Przenośny:** jeden plik, dane zapisywane obok programu.
* **Skróty:** `Ctrl+C` – wstecz o jedno okno, `Ctrl+X` – zamknięcie programu (działają w każdym oknie).

## 📄 Dokumentacja

Pełna dokumentacja użytkownika i techniczna – folder [`dokumentacja/`](dokumentacja/).
Lista zmian – [`CHANGELOG.md`](CHANGELOG.md).

## ⚖️ Licencja

Oprogramowanie zamknięte (nie open source), wszelkie prawa zastrzeżone – obecnie udostępniane
bezpłatnie na warunkach opisanych w [`LICENSE.md`](LICENSE.md) i
[`dokumentacja/Regulamin.md`](dokumentacja/Regulamin.md). Polityka prywatności:
[`dokumentacja/Polityka-Prywatnosci.md`](dokumentacja/Polityka-Prywatnosci.md).

Kod źródłowy programu jest zamknięty i nie jest udostępniany w tym repozytorium.
