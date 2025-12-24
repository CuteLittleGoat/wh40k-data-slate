# WH40k Data-Slate

Dwustronicowa aplikacja webowa do szybkiego prezentowania komunikatów inspirowanych uniwersum WH40k. **GM.html** to panel prowadzącego (MG), a **Infoczytnik.html** to ekran graczy, który nasłuchuje Firestore i natychmiast odświeża layout, treść oraz audio.

## Wymagania
- Projekt Firebase z włączonym Firestore.
- Serwer statyczny (np. GitHub Pages, Firebase Hosting lub lokalny serwer HTTP), ponieważ pliki korzystają z modułów ES i Firestore.

## Instalacja i uruchomienie
1. Skopiuj `config/firebase-config.template.js` do `config/firebase-config.js`.
2. Wklej konfigurację Firebase (Web) z konsoli: **Project settings → Your apps → Firebase SDK snippet (Config)**.
3. W Firestore utwórz kolekcję `dataslate` i dokument `current` (jeśli go nie ma, aplikacja utworzy go przy pierwszym zapisie).
4. Ustaw reguły Firestore, aby umożliwić odczyt i zapis dokumentu `dataslate/current` dla urządzeń, które mają korzystać z aplikacji.
5. Uruchom aplikację:
   - Lokalnie: `npx http-server .` lub dowolny prosty serwer HTTP.
   - Zdalnie: wrzuć repo do hostingu statycznego (np. GitHub Pages).
6. Otwórz:
   - `GM.html` (panel MG)
   - `Infoczytnik.html` (ekran graczy)

## Użytkowanie (skrót)
- **GM.html**
  - Wybierz frakcję/layout, kolory i rozmiary tekstów.
  - Zdecyduj, czy prefiksy/sufiksy są losowane automatycznie, czy wybierasz je ręcznie.
  - Wpisz treść i kliknij **Wyślij** (typ `message`).
  - **Ping** wysyła sygnał dźwiękowy bez zmiany treści.
  - **Wyczyść ekran** czyści treść (typ `clear`) bez zmiany layoutu.
- **Infoczytnik.html**
  - Po uruchomieniu kliknij raz na ekran, aby odblokować dźwięk (wymóg przeglądarki).
  - Ekran nasłuchuje `dataslate/current` i od razu renderuje nowy stan.

## Aktualizacja aplikacji
### Aktualizacja assetów (tła, logotypy, audio)
1. Podmień pliki w `assets/`:
   - Tła: `assets/layouts/<frakcja>/*.png`
   - Logotypy: `assets/logos/<frakcja>/*.png`
   - Audio globalne: `assets/audio/global/Ping.mp3`, `assets/audio/global/Message.mp3`
2. Zmień wersję cache w `Infoczytnik.html`:
   - `INF_VERSION` (górny skrypt) oraz `ASSET_VERSION` (skrypt modułowy). W praktyce jest to ta sama wartość (`window.__dsVersion`).
3. Odśwież przeglądarkę na urządzeniach graczy. Infoczytnik wymusza cache-busting parametrem `?v=<INF_VERSION>`.

### Aktualizacja konfiguracji Firebase
- Gdy zmienisz projekt Firebase, uaktualnij `config/firebase-config.js` (lub ponownie wygeneruj go z template).

### Aktualizacja logiki / layoutów
- Po zmianach w `GM.html` lub `Infoczytnik.html` zawsze zwiększ `INF_VERSION`, aby urządzenia z cache pobrały świeżą wersję.

## Disclaimer
To narzędzie jest nieoficjalnym, fanowskim projektem stworzonym jako pomoc dla MG w systemie Wrath & Glory. Aplikacja jest udostępniana za darmo wyłącznie do prywatnego, niekomercyjnego użytku. Projekt nie jest licencjonowany, nie jest powiązany ani wspierany przez Games Workshop, Cubicle 7 Entertainment Ltd. ani Copernicus Corporation.
Warhammer 40,000 oraz powiązane nazwy i znaki towarowe są własnością Games Workshop Limited; Wrath & Glory jest własnością odpowiednich właścicieli praw.

## Więcej informacji
Szczegółowy opis kodu, struktury plików i działania każdego elementu aplikacji znajduje się w `docs/Documentation.md`.
