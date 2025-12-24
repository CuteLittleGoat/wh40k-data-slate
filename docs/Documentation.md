# Dokumentacja techniczna WH40k Data-Slate

## 1. Architektura i przepływ danych
- **Model aplikacji:** dwie strony HTML działające równolegle.
  - `GM.html` (panel MG) zapisuje stan do Firestore.
  - `Infoczytnik.html` (ekran graczy) subskrybuje dokument i renderuje widok.
- **Firestore:** kolekcja `dataslate`, dokument `current`.
  - `type` określa akcję: `message`, `ping`, `clear`.
  - Dokument zawiera kompletny stan wizualny (frakcja, kolory, rozmiary, indeksy fillerów, logo, flicker) oraz treść.
- **Cache-busting:** Infoczytnik automatycznie dodaje parametr `?v=<INF_VERSION>` do URL-a i wymusza przeładowanie, aby urządzenia nie trzymały starej wersji.

## 2. Struktura repozytorium (pliki i katalogi)
- `index.html` — strona startowa z linkami do GM i Infoczytnika.
- `GM.html` — panel MG (UI + zapis do Firestore).
- `Infoczytnik.html` — ekran graczy (UI + subskrypcja Firestore + audio).
- `config/`
  - `firebase-config.js` — faktyczna konfiguracja Firebase (globalna zmienna `window.firebaseConfig`).
  - `firebase-config.template.js` — szablon do wypełnienia własną konfiguracją.
- `assets/`
  - `layouts/<frakcja>/` — tła PNG dla frakcji (mapowane w `LAYOUT_BG`).
  - `logos/<frakcja>/` — logotypy PNG (mapowane w `FACTION_LOGO`).
  - `audio/global/` — `Ping.mp3`, `Message.mp3` (domyślne dźwięki).
- `docs/`
  - `README.md` — instrukcja użytkownika.
  - `Documentation.md` — ten dokument.
- `Draft/` — szkice/alternatywne layouty PNG (nieużywane w runtime).
- `backup-PrzedConfigiemJS/` — kopie starych wersji HTML (archiwum, nieużywane w runtime).

## 3. `index.html` — strona startowa
**Cel:** szybki wybór trybu.
- Dwa główne linki:
  - `Otwórz GM` → `GM.html`
  - `Otwórz Infoczytnik` → `Infoczytnik.html`
- Dodatkowe linki „debug” przekazują parametry query (`debug`, `logs`, `mockWs`, `mockAudio`), ale w aktualnym kodzie **nie są obsługiwane** — parametry nie zmieniają działania.
- Krótki komunikat o odblokowaniu dźwięku i przykładowy adres hostingu.

## 4. `config/firebase-config.js` i `firebase-config.template.js`
**Rola:** dostarczenie konfiguracji Firebase do obu stron.
- Pliki wystawiają globalną zmienną `window.firebaseConfig` (bez `export`, aby działało z `<script>`).
- `GM.html` i `Infoczytnik.html` **wymagają** tej zmiennej do połączenia z Firestore.
- Template opisuje, skąd skopiować dane z konsoli Firebase.

## 5. `GM.html` — panel MG (UI + logika zapisu)
### 5.1. HTML i CSS
- Layout w dwóch kolumnach (`.row` + `.col`).
- Sekcje:
  - **Frakcja / layout** (`#faction`).
  - **Kolor treści** (`#fontColor`) + szybkie „chipy”.
  - **Rozmiar treści** (`#msgFontSize`).
  - **Kolor prefix/suffix** (`#psColorText`, `#psColorPicker`) + szybkie „chipy”.
  - **Rozmiar prefix/suffix** (`#psFontSize`).
  - **Losowość fillerów** (`#randomFillers`, `#prefixIndex`, `#suffixIndex`).
  - **Logo** (`#showLogo`) i **Flicker** (`#flicker`).
  - **Live preview** (`#livePreview`) — podgląd wynikowej kompozycji tekstu.
  - **Treść komunikatu** (`#message`).
  - Przyciski akcji (`#sendBtn`, `#pingBtn`, `#clearBtn`, `#clearTextBtn`).
- `#prefixPreview` i `#suffixPreview` pokazują wylosowane/wybrane fillery (pomoc dla MG).

### 5.2. Logika JS (kluczowe elementy)
- **Firebase (compat v9.6.8):**
  - `firebase.initializeApp(firebaseConfig)`
  - `firebase.firestore()`
  - `currentRef = db.collection("dataslate").doc("current")`
- **Dane słownikowe:**
  - `FONT_STACK` — mapowanie frakcji do fontów Google.
  - `LAYOUTS` — listy prefixów i suffixów (9 wpisów na frakcję).
- **Walidacja i pomocnicze funkcje:**
  - `normalizePx` — clamp dla rozmiarów fontu.
  - `clampIndex` — bezpieczne przycinanie indeksów fillerów.
  - `rand1to` — losowanie indeksu 1..N.
  - `getPrefixSuffixColor` — pobiera kolor pref/suf, z fallbackiem na `rgba(255,255,255,.88)`.
- **Podgląd:**
  - `computePreview()` losuje/wybiera fillery, ustawia `#prefixPreview`, `#suffixPreview`, blokuje ręczne pola, i wywołuje `updateLivePreview()`.
  - `updateLivePreview()` synchronizuje fonty, kolory i rozmiary w sekcji „Live preview”.
- **Wysyłanie stanu:**
  - `sendMessage(isClear)` zapisuje pełen snapshot do Firestore (`merge:false`).
    - `type` = `message` lub `clear`.
    - `text` jest pusty przy `clear`.
    - `prefixIndex`, `suffixIndex`, kolory, rozmiary, `showLogo`, `flicker` zawsze wysyłane.
  - `ping()` zapisuje tylko styl + `type:"ping"` (`merge:true`), aby nie nadpisywać treści.
- **Unikalność:** `nonce()` (timestamp + losowa końcówka) pozwala Infoczytnikowi ignorować powtórne snapshoty.

## 6. `Infoczytnik.html` — ekran graczy (UI + renderowanie)
### 6.1. Cache-busting i wersje
- `INF_VERSION` ustawiane w skrypcie w `<head>`.
- Jeśli `?v` w URL różni się od `INF_VERSION`, strona wykonuje `window.location.replace(...)` i wymusza reload.
- `window.__dsVersion` jest używane jako wspólna wersja dla wszystkich assetów (`ASSET_VERSION`).

### 6.2. Warstwa wizualna (CSS)
- **Panel**: `.panel` i `.layout-img` tworzą ramkę tła (PNG) z `object-fit: contain`.
- **Obszar tekstu**: `.screen` jest pozycjonowany absolutnie przez CSS variables:
  - `--screen-top/right/bottom/left`
- **Teksty**:
  - `.prefix` i `.suffix` używają `--prefix-color`/`--suffix-color` i własnych font-size.
  - `.msg` używa `--accent` i `--msg-font-size`.
- **CRT/flicker**:
  - `.crt::before/::after` dodają scanlines i vignette.
  - `.screen::after` animuje migotanie (`flickerBg`).
  - Klasa `.no-flicker` wyłącza animację.
- **Logo**:
  - `.logo` jest ukrywane lub pokazywane zależnie od `showLogo` i mapowania w `FACTION_LOGO`.

### 6.3. Audio unlock
- Overlay `#unlock` blokuje dźwięk, dopóki użytkownik nie wykona interakcji.
- Funkcja `window.__dsPlayUrlOnce(url)` tworzy nowy `Audio` i odtwarza dźwięk tylko, gdy `window.__dsAudioArmed === true`.
- Zdarzenia odblokowujące: `click`, `touchstart`, `pointerdown`, `keydown`.

### 6.4. Logika JS (Firestore + render)
- **Firebase (moduł v12.6.0):**
  - `initializeApp(firebaseConfig)`
  - `getFirestore(app)`
  - `onSnapshot(doc(db, "dataslate", "current"), ...)`
- **Mapy i stałe:**
  - `LAYOUT_BG` — mapowanie frakcji → PNG layoutu.
  - `LAYOUT_AR` — proporcje tła (np. inkwizycja 707/1023).
  - `SCREEN_INSETS` — marginesy panelu tekstu dla wybranych layoutów.
  - `FONT_STACK` — fonty per frakcja.
  - `FILLERS` — teksty prefix/suffix.
  - `FACTION_LOGO` — mapowanie frakcji → logo (tylko `mechanicus`, `inquisition`).
- **Funkcje kluczowe:**
  - `fitPanel(ar)` — dopasowuje panel do okna, zachowując aspect ratio.
  - `setInsets(p)` — ustawia CSS variables z marginesami.
  - `setFlickerState(shouldFlicker)` — dodaje/usuwa klasę `.no-flicker`.
  - `setLogoForFaction(key, shouldShow)` — pokazuje logo, jeśli istnieje.
  - `getFillerText(key, idx, kind)` — pobiera prefix/suffix po indeksie 1-based.
  - `applyLayout(factionKey, color, showLogo)` — wybiera tło, font, color i marginesy.
  - `applyTextStyleFromDoc(d)` — synchronizuje rozmiary/kolory tekstu.
  - `showMessage`/`clearMessageKeepLayout` — aktualizują widok.
- **Obsługa snapshotu:**
  1. Jeśli `nonce` identyczny jak poprzednio — ignoruj.
  2. Zastosuj layout, kolory, flicker.
  3. `type === "clear"` → wyczyść tekst.
  4. `type === "ping"` → odtwórz dźwięk ping (bez zmiany treści).
  5. `type === "message"` → wstaw prefix/suffix i treść, odtwórz dźwięk wiadomości.
- **Prefix/suffix w `message`:**
  - Najpierw używane są `prefix`/`suffix` (lub `prefixText`/`suffixText`).
  - Jeśli ich brak, ale są indeksy (`prefixIndex`, `suffixIndex`), pobierany jest odpowiedni filler.
  - Jeśli treść istnieje, a prefix/suffix nadal brak — fallback na indeks 1.

## 7. Payload Firestore (`dataslate/current`)
| Pole | Typ | Opis |
| --- | --- | --- |
| `type` | `"message"` \| `"ping"` \| `"clear"` | Rodzaj akcji. `clear` usuwa tylko tekst, `ping` uruchamia dźwięk, `message` ustawia treść. |
| `faction` | `string` | Klucz frakcji zgodny z mapami layoutów (np. `mechanicus`). |
| `color` / `fontColor` | `string` | Kolor treści wiadomości (CSS/HEX). |
| `msgFontSize` | `string` (np. `"28px"`) | Rozmiar treści, trafia do `--msg-font-size`. |
| `prefixColor` / `suffixColor` | `string` | Kolory prefix/suffix. |
| `prefixFontSize` / `suffixFontSize` | `string` | Rozmiary prefix/suffix. |
| `prefixIndex` / `suffixIndex` | `number` | Indeksy (1-based) do fillerów. |
| `prefix` / `suffix` (`prefixText` / `suffixText`) | `string` | Bezpośredni tekst linii nad/pod wiadomością. |
| `text` | `string` | Treść wiadomości. |
| `showLogo` | `boolean` | Czy pokazać logo frakcji. |
| `flicker` | `boolean` | Czy włączyć animację CRT. |
| `pingUrl` / `msgUrl` (`messageUrl`) | `string` | Opcjonalne ścieżki niestandardowych dźwięków. |
| `nonce` | `string` | Unikalny identyfikator snapshota (zapobiega dublowaniu). |
| `ts` | `serverTimestamp` | Znacznik czasu Firestore. |

## 8. Aktualizacja assetów i layoutów
- **Nowa frakcja:** dodaj pliki do `assets/layouts/<frakcja>/`, ewentualne logo do `assets/logos/<frakcja>/`, uzupełnij mapy w `Infoczytnik.html` (`LAYOUT_BG`, `FILLERS`, `FONT_STACK`, opcjonalnie `FACTION_LOGO`, `LAYOUT_AR`, `SCREEN_INSETS`).
- **Nowy layout istniejącej frakcji:** podmień PNG i zaktualizuj mapę `LAYOUT_BG`. Jeśli zmienia się proporcja lub położenie tekstu — zaktualizuj `LAYOUT_AR`/`SCREEN_INSETS`.
- **Audio:** podmień `assets/audio/global/Ping.mp3` i `Message.mp3` lub używaj `pingUrl`/`msgUrl` w dokumencie.
- **Wersjonowanie:** podnieś `INF_VERSION` (a przez to `ASSET_VERSION`), aby wymusić pobranie nowych assetów.

## 9. Troubleshooting
- **Brak dźwięku:** upewnij się, że użytkownik kliknął overlay (sprawdź `window.__dsAudioArmed` w konsoli) i że pliki audio istnieją.
- **Brak synchronizacji:** sprawdź `config/firebase-config.js`, reguły Firestore i dostępność sieci.
- **Nowe assety nie widać:** zwiększ `INF_VERSION` i odśwież urządzenie (cache-busting).
