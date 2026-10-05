# Regulamin korzystania z programu NetworkMaster

**To jest szablon przygotowany bez udziału prawnika.** Przed jakimkolwiek komercyjnym
wykorzystaniem tego dokumentu skonsultuj go z prawnikiem — szczególnie w zakresie zgodności z
prawem polskim i unijnym (w tym RODO oraz przepisami o ochronie praw konsumenta). Ten dokument
sam w sobie nie stanowi porady prawnej.

Wersja Regulaminu: 1.0 · Dotyczy programu: NetworkMaster (od wersji 2.4.0)

## 1. Definicje

- **Program** — oprogramowanie NetworkMaster, obejmujące pliki wykonywalne, kod źródłowy (jeśli
  udostępniony), dokumentację i wszelkie materiały towarzyszące.
- **Właściciel** — **Paweł Mościbrodzki, NIP 5372682524**, podmiot posiadający pełne
  prawa autorskie i majątkowe do Programu.
- **Użytkownik** — każda osoba fizyczna lub prawna pobierająca, instalująca lub uruchamiająca
  Program.

## 2. Status Programu

Program jest oprogramowaniem **zamkniętym** (nie open source) — pełne warunki licencyjne znajdują
się w [`../LICENSE.md`](../LICENSE.md). Aktualnie Program jest udostępniany Użytkownikom
**nieodpłatnie**. Właściciel zastrzega sobie prawo do zmiany tego modelu w przyszłości (patrz §8).

## 3. Zakres licencji użytkowania

Właściciel udziela Użytkownikowi niewyłącznej, nieprzenoszalnej licencji na uruchamianie
Programu na własny, prywatny lub firmowy użytek. Licencja **nie obejmuje** prawa do:

- dalszej dystrybucji, sprzedaży, podnajmu ani publicznego udostępniania Programu lub jego
  kodu źródłowego osobom trzecim,
- tworzenia i rozpowszechniania utworów pochodnych (zmodyfikowanych wersji, forków),
- dekompilacji, deasemblacji lub innej formy inżynierii wstecznej Programu, poza zakresem,
  w jakim jest to wyraźnie dozwolone bezwzględnie obowiązującymi przepisami prawa (np. art. 75
  ustawy o prawie autorskim i prawach pokrewnych w zakresie interoperacyjności),
- usuwania lub zmiany oznaczeń praw autorskich, niniejszego Regulaminu ani `LICENSE.md`.

## 4. Charakter Programu i ryzyko korzystania

Program wykonuje operacje na konfiguracji sieciowej urządzenia, plikach systemowych (m.in. plik
hosts) oraz — w module VPN — na profilach połączeń sieciowych, w tym profilach systemowego VPN.
Użytkownik korzysta z tych funkcji **na własną odpowiedzialność** i we własnym zakresie ocenia
skutki ich użycia, w szczególności:

- zmian konfiguracji sieciowej i naprawy/resetu ustawień sieci,
- edycji pliku hosts,
- tworzenia, edycji, usuwania i łączenia z profilami VPN (systemowymi, WireGuard, OpenVPN).

## 5. Kopie zapasowe VPN — jawne przechowywanie haseł

Program celowo zapisuje w plikach kopii zapasowych VPN (`.zip`) hasła, klucze PSK oraz klucze
prywatne WireGuard **w postaci jawnej, niezaszyfrowanej** — jest to świadomy wybór projektowy,
mający zapewnić pełną, kompletną kopię konfiguracji. Użytkownik przyjmuje do wiadomości to
zachowanie i ponosi pełną odpowiedzialność za bezpieczne przechowywanie i przesyłanie takich
plików (m.in. unikanie wysyłki niezaszyfrowanym kanałem, przechowywania w niezabezpieczonej
chmurze itp.).

## 6. Brak gwarancji

Program dostarczany jest **„tak jak jest” (AS IS)**, bez jakiejkolwiek gwarancji, wyraźnej lub
dorozumianej, w tym bez gwarancji przydatności do określonego celu czy nienaruszania praw osób
trzecich. Właściciel nie ponosi odpowiedzialności za szkody wynikłe z korzystania z Programu, w
najszerszym zakresie dopuszczalnym przez obowiązujące prawo.

## 7. Aktualizacje

Program może łączyć się z GitHub w celu sprawdzenia dostępności nowszej wersji i pobrania opisu
zmian. Instalacja aktualizacji wymaga każdorazowo świadomej decyzji Użytkownika — Program nie
aktualizuje się samoczynnie bez zgody.

## 8. Zmiana warunków i planowana sprzedaż Programu

Właściciel zastrzega sobie prawo do zmiany niniejszego Regulaminu oraz modelu udostępniania
Programu w przyszłości, w tym do wprowadzenia odpłatności. Właściciel rozważa również odpłatne
zbycie pełnych praw do Programu — wraz z kodem źródłowym — na rzecz wybranego nabywcy. Taka
transakcja stanowiłaby sprzedaż całości praw (aktywów), regulowaną odrębną umową między
Właścicielem a nabywcą, i **nie jest przedmiotem niniejszego Regulaminu** ani nie zmienia
automatycznie warunków korzystania obowiązujących dotychczasowych Użytkowników w momencie jej
zawarcia, chyba że nabywca postanowi inaczej i poinformuje o tym Użytkowników.

## 9. Prawo właściwe

Niniejszy Regulamin podlega prawu polskiemu. Wszelkie spory będą rozstrzygane przez sąd właściwy
zgodnie z obowiązującymi przepisami.

## 10. Kontakt

W sprawach dotyczących Programu i niniejszego Regulaminu: **pxware@pxware.pl**.
