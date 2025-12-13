# WH40k Data-Slate (GM + Infoczytnik)
**Wersja:** 2025-12-13  
**Hosting:** GitHub Pages  
**Synchronizacja:** Firebase Firestore (`dataslate/current`)

---

## TL;DR — Setup
1) Skopiuj `config/firebase-config.template.js` → `config/firebase-config.js`  
2) Wklej swój config Firebase do `config/firebase-config.js`  
3) W Firebase włącz **Firestore Database** (na start może być *Test mode*)  
4) Włącz GitHub Pages (Repo → **Settings → Pages**)  
5) Otwórz:
- `GM.html`
- `Infoczytnik.html`
6) W Infoczytniku kliknij raz ekran, żeby odblokować dźwięk

---

# 🇵🇱 Dokumentacja (PL)

## 1) Opis w 2 zdaniach
To prosta aplikacja webowa z 2 stronami:
- **GM.html** — panel dla Mistrza Gry do wysyłania wiadomości/pingów i ustawiania wyglądu.
- **Infoczytnik.html** — ekran dla graczy (tablet/laptop), który wyświetla wiadomość na tle layoutu i odtwarza dźwięki.

Wszystko synchronizuje się w czasie rzeczywistym przez **Firebase Firestore** (dokument: `dataslate/current`).

---

## 2) Instrukcja użytkownika (krok po kroku)

### 2.1 Wymagania
- Nowoczesna przeglądarka (Chrome / Chromium rekomendowane).
- Internet (Firebase + Google Fonts; assety PNG/MP3 są z GitHub Pages).
- GM i Infoczytnik mogą działać:
  - na jednym urządzeniu (2 karty), albo
  - na dwóch urządzeniach (np. laptop GM + tablet graczy).

---

### 2.2 Uruchomienie
1) Otwórz panel GM: `GM.html`  
2) Otwórz Infoczytnik: `Infoczytnik.html`  
3) Infoczytnik pokaże overlay:
   **„Kliknij raz, aby odblokować dźwięk”**  
   Kliknij w overlay (to wymóg przeglądarki — inaczej audio nie ruszy).

> Po odświeżeniu strony Infoczytnika trzeba kliknąć ponownie.

---

### 2.3 Wysyłanie wiadomości (GM → Infoczytnik)
1) W GM wybierz **Frakcja / layout**.  
2) Ustaw **styl treści wiadomości**:
   - **Kolor fontu (treść wiadomości)** — picker lub szybkie kolory (Zielony/Czerwony/Złoty/Biały)
   - **Wielkość fontu (treść wiadomości)** — w px (np. 28)
3) Ustaw **styl Prefix + Suffix**:
   - **Kolor Prefix + Suffix (wspólny)** — wpisz `#ffffff` albo `rgba(...)` lub użyj pickera / szybkich kolorów
   - **Wielkość fontu Prefix + Suffix (wspólna)** — w px (np. 14)
4) Prefix/Suffix (fillery):
   - Zaznaczone **Losuj automatycznie** → GM losuje prefix i suffix
   - Odznaczone → wpisujesz ręcznie numer Prefix (lewa strona) i Suffix (prawa strona)
5) Wpisz **Treść komunikatu**.  
6) Kliknij **Wyślij**.

Efekt w Infoczytniku:
- zmienia się layout i font frakcji,
- ustawiają się kolory i rozmiary,
- wyświetla się prefix + (opcjonalnie logo) + treść + suffix,
- odtwarza się dźwięk **Message**.

---

### 2.4 Ping (GM → Infoczytnik)
1) Kliknij **Ping**.  
Efekt: Infoczytnik odtwarza dźwięk **Ping** (bez zmiany tekstu).

---

### 2.5 Wyczyść ekran (GM → Infoczytnik)
1) Kliknij **Wyczyść ekran**.  
Efekt: znika prefix/treść/suffix, ale tło zostaje.

---

### 2.6 Wyczyść pole (tylko GM)
1) Kliknij **Wyczyść pole**.  
Efekt: czyści tylko textarea w GM, nic nie wysyła.

---

## 3) Jak to działa (technicznie)

### 3.1 Architektura
Są 2 niezależne strony:
- **GM.html** zapisuje stan do Firestore
- **Infoczytnik.html** nasłuchuje Firestore i aktualizuje ekran

Kanał synchronizacji:
- Kolekcja: `dataslate`
- Dokument: `current`
- Ścieżka: `dataslate/current`

---

### 3.2 Firebase config — plik `config/firebase-config.js`
W repo jest template:

`config/firebase-config.template.js`  
Skopiuj jako:

`config/firebase-config.js`

i wklej:

```js
window.firebaseConfig = {
  apiKey: "…",
  authDomain: "…",
  projectId: "…",
  storageBucket: "…",
  messagingSenderId: "…",
  appId: "…"
};

GM.html ładuje config jako zwykły <script> i korzysta z window.firebaseConfig.
Infoczytnik.html też ładuje config jako <script> i korzysta z window.firebaseConfig.

3.3 Kontrakt danych w Firestore (dataslate/current)

Najważniejsze pola:

A) Typ zdarzenia

type: "message" | "ping" | "clear"

B) Dedup / meta

nonce: unikalny identyfikator zdarzenia (zapobiega ponownemu odtworzeniu)

ts: serverTimestamp()

C) Treść i wygląd

faction: np. mechanicus, inquisition…

text: treść wiadomości

color / fontColor: kolor treści (hex)

msgFontSize: np. "28px"

D) Fillery

prefixIndex, suffixIndex (1..N)

(opcjonalnie w przyszłości) prefix, suffix jako gotowe teksty

E) Styl prefix/suffix

prefixColor, suffixColor (np. #fff lub rgba(...))

prefixFontSize, suffixFontSize (np. "14px")

F) Audio (opcjonalne)

pingUrl

msgUrl / messageUrl

Domyślnie Infoczytnik używa:

assets/audio/global/Ping.mp3

assets/audio/global/Message.mp3

3.4 GM.html — co robi kod

Trzyma listy fillerów dla frakcji w obiekcie LAYOUTS.

computePreview():

losuje lub wybiera indeksy,

pokazuje preview prefix/suffix,

blokuje ręczne pola gdy losowanie włączone.

sendMessage(isClear):

zapisuje do dataslate/current pełny stan (type, faction, style, indeksy, text, nonce).

ping():

zapisuje type=ping,

też wysyła style, żeby Infoczytnik nie „gubił” wyglądu.

3.5 Infoczytnik.html — co robi kod

Renderuje tło layoutu jako <img> z object-fit: contain.

Warstwa .screen leży na obrazie i jest „bezpiecznym polem” na tekst.

Bezpieczne pole jest wyznaczone procentami:

--screen-top/right/bottom/left
Dzięki temu obszar skaluje się razem z layoutem na różnych ekranach.

fitPanel(ar) dopasowuje rozmiar panelu do ekranu, zachowując aspect ratio.

onSnapshot() nasłuchuje dataslate/current:

ignoruje duplikaty po nonce,

ustawia layout/font/kolory/rozmiary,

reaguje na type:

clear → czyści tekst

ping → odtwarza dźwięk Ping

message → składa prefix/suffix i wyświetla + odtwarza Message

3.6 Dlaczego jest „Kliknij raz, aby odblokować dźwięk”

Chrome i inne przeglądarki blokują autoplay audio, dopóki użytkownik nie wykona akcji.
Infoczytnik ma overlay, który po kliknięciu:

chowa overlay,

uzbraja audio,

od tego momentu można odtwarzać MP3.

4) Zasoby i cache (ważne na tabletach)
4.1 Struktura assets

assets/audio/global/Ping.mp3

assets/audio/global/Message.mp3

assets/layouts/<faction>/...png

assets/logos/<faction>/...png

4.2 Wersjonowanie assetów (cache-busting)

Infoczytnik ma stałą:

ASSET_VERSION = "2025-12-13-1" (lub podobną)

Do URLi assetów jest dodawane:

...?v=${ASSET_VERSION}

Zmieniaj ASSET_VERSION zawsze, gdy podmienisz PNG/MP3, żeby tablety nie trzymały starej wersji w cache.

5) Jak rozwijać aplikację (procedury)
5.1 Podmiana globalnych dźwięków

Nadpisz pliki:

assets/audio/global/Ping.mp3

assets/audio/global/Message.mp3

Commit

Zmień ASSET_VERSION w Infoczytniku

Commit

Odśwież (na tabletach czasem trzeba wyczyścić cache / zmienić wersję)

5.2 Dodanie nowej frakcji

Wymaga zmian w 2 plikach: GM.html i Infoczytnik.html.

GM.html

Dodaj <option value="new_faction">Nazwa</option> w select.

Dodaj wpis w LAYOUTS.new_faction = { prefixes:[...], suffixes:[...] }.

Infoczytnik.html

Dodaj layout w assets/layouts/new_faction/...png

Dodaj mapowanie w LAYOUT_BG

Jeśli layout ma inne proporcje → dodaj AR do LAYOUT_AR i obsłuż w applyLayout()

Jeśli layout ma inną ramkę → dodaj preset do SCREEN_INSETS i obsłuż w applyLayout()

(Opcjonalnie) dodaj font w FONT_STACK i w linku Google Fonts

(Opcjonalnie) dodaj fillery w FILLERS (jeśli Infoczytnik ma je wyliczać po indeksach)

(Opcjonalnie) dodaj logo w FACTION_LOGO + wrzuć PNG do assets/logos/new_faction/

Na koniec:

Zmień ASSET_VERSION, jeśli dodałeś nowe pliki PNG/MP3.

5.3 Dodanie nowego layoutu lub zmiana tła istniejącego

Wrzuć/Podmień PNG w assets/layouts/<faction>/...

Zmień ścieżkę w LAYOUT_BG jeśli zmieniła się nazwa pliku

Jeśli layout ma inne proporcje:

policz AR = szerokość / wysokość

dodaj do LAYOUT_AR

Dopasuj marginesy w SCREEN_INSETS:

ustaw duże marginesy,

wyślij długi tekst,

zmniejszaj aż będzie idealnie,

zostaw zapas, żeby nie dotykało ramki

Zwiększ ASSET_VERSION (cache!)

5.4 Zmiana fontu

Dodaj font do <head> (Google Fonts) lub użyj fontu lokalnego.

Ustaw w FONT_STACK[faction].

Commit i odśwież.

5.5 Dodanie/zmiana logo

Wrzuć logo do assets/logos/<faction>/...png

Dodaj wpis do FACTION_LOGO

Zwiększ ASSET_VERSION (żeby odświeżyć cache)

6) Diagnostyka (najczęstsze problemy)
6.1 „Nie widzę zmian / tablet widzi starą wersję”

Zwiększ ASSET_VERSION

Hard refresh (Ctrl+Shift+R)

Na mobile: wyczyść cache strony / użyj nowszej wersji linku

6.2 „Zawiesza się na ‘Kliknij aby odblokować dźwięk’”

Kliknij w overlay (spróbuj też dotknąć i przytrzymać).

Jeśli nadal nie znika, problem jest zwykle w:

cache starego JS,

błędzie JS blokującym wykonanie (sprawdź konsolę),

nietypowych ustawieniach przeglądarki na tablecie.

6.3 „Nie wysyła z GM / nie odbiera w Infoczytniku”

Sprawdź czy config/firebase-config.js istnieje i jest poprawny.

Sprawdź czy Firestore jest włączony w Firebase Console.

Sprawdź reguły Firestore (na start test mode ok).

🇬🇧 Documentation (EN)
1) Short description

WH40k Data-Slate is a lightweight web app made of two pages:

GM.html — Game Master control panel for sending messages/pings and changing the visual style.

Infoczytnik.html — Player display screen (tablet/laptop) showing the message on a themed Data-Slate background and playing audio cues.

They sync in real time via Firebase Firestore (dataslate/current).

2) User guide (step by step)
2.1 Requirements

Modern browser (Chrome/Chromium recommended)

Internet connection

One device for GM and one for players (or two tabs on one device)

2.2 Startup

Open GM.html

Open Infoczytnik.html

On Infoczytnik click the overlay:
“Click once to unlock audio”
(Browsers block audio until user interaction.)

2.3 Sending a message

Choose Faction / layout

Set message style (color + font size)

Set Prefix+Suffix style (shared color + shared font size)

Configure fillers:

Randomize automatically, or manual indices

Enter message text

Click Send

Result:

Layout, fonts, colors update

Prefix + (optional logo) + message + suffix are displayed

Message sound plays

2.4 Ping

Click Ping → plays Ping sound.

2.5 Clear screen

Click Clear screen → clears text but keeps background.

3) Technical overview
3.1 Architecture

GM.html writes state to Firestore

Infoczytnik.html listens with onSnapshot() and updates UI

Firestore path:

Collection: dataslate

Document: current

Path: dataslate/current

3.2 Firebase config (config/firebase-config.js)

Create from template and paste your Firebase web config:
window.firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};

Both pages read it from window.firebaseConfig.

3.3 Firestore document schema (data contract)

Key fields:

type: "message" | "ping" | "clear"

nonce: unique event id (dedup)

faction

text

color / fontColor

msgFontSize

prefixIndex, suffixIndex

prefixColor, suffixColor

prefixFontSize, suffixFontSize

Optional audio overrides:

pingUrl

msgUrl / messageUrl

3.4 Safe text area (why it never goes outside the frame)

The background is an <img> with object-fit: contain.
Text lives in an absolutely positioned .screen overlay.

The safe area margins are percentage-based CSS variables:

--screen-top/right/bottom/left

Percentages scale with the background image across devices, ensuring the text stays inside the frame.

3.5 Audio unlock overlay

Mobile/desktop browsers block autoplay.
Infoczytnik shows an overlay and unlocks audio after a click/tap.

4) Assets & cache busting
4.1 Assets structure

assets/audio/global/Ping.mp3

assets/audio/global/Message.mp3

assets/layouts/<faction>/...png

assets/logos/<faction>/...png

4.2 ASSET_VERSION

Infoczytnik uses:

ASSET_VERSION = "YYYY-MM-DD-X"

All assets are loaded with ?v=ASSET_VERSION to force refresh on tablets/browsers.

5) Extending the project
5.1 Replace global audio

Replace:

assets/audio/global/Ping.mp3

assets/audio/global/Message.mp3

Commit

Increase ASSET_VERSION

Commit again

5.2 Add a new faction

GM.html: add <option> + add LAYOUTS.new_faction

Infoczytnik.html: add layout in LAYOUT_BG, optional FONT_STACK, FILLERS, FACTION_LOGO, custom SCREEN_INSETS/LAYOUT_AR

5.3 Add/replace a layout

Put PNG in assets/layouts/<faction>/

Update LAYOUT_BG

If aspect ratio differs, update LAYOUT_AR

Tune safe margins in SCREEN_INSETS

Increase ASSET_VERSION

5.4 Change fonts

Add font to Google Fonts link

Update FONT_STACK[faction]

5.5 Add/replace logos

Put PNG in assets/logos/<faction>/

Update FACTION_LOGO

Increase ASSET_VERSION

6) Troubleshooting

Changes not visible: increase ASSET_VERSION, hard refresh, clear cache on mobile.

No audio: click/tap to unlock audio overlay.

GM doesn’t send / Infoczytnik doesn’t receive: verify config/firebase-config.js, Firestore enabled, rules allow read/write.
