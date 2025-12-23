# Dokumentacja techniczna

## Architektura i przepływ danych
- **Strony:** `GM.html` zapisuje stan do Firestore, `Infoczytnik.html` subskrybuje dokument i renderuje widok (HTML/CSS/Audio). `[GM.html → Firestore ← Infoczytnik.html]`
- **Firestore:** kolekcja `dataslate`, dokument `current`. Zdarzenia `message`, `ping` i `clear` wymieniają kompletny stan wizualny (frakcja, kolory, rozmiary fontu, indeksy fillerów, flaga logotypu, flicker, treść, nonce, timestamp).
- **Cache-busting:** Infoczytnik wymusza parametr `?v=<INF_VERSION>` w adresie i ponownie ładuje stronę, dzięki czemu nowe assety tła/logo/audio są widoczne bez czyszczenia cache.

## Konfiguracja Firebase (`config/firebase-config.js`)
1. Skopiuj `config/firebase-config.template.js` do `config/firebase-config.js` i wklej konfigurację z konsoli Firebase (Web app settings).
2. Plik jest ładowany jako zwykły `<script>` w obu stronach i udostępnia `window.firebaseConfig`.
3. Firestore musi zezwalać na odczyt i zapis dokumentu `dataslate/current` (w dev mogą to być reguły testowe).

## Warstwa wizualna Infoczytnika
- Tło to `<img.layout-img>` z `object-fit: contain`; overlay `.screen` jest absolutnie pozycjonowany w insetach zależnych od frakcji.
- Marginesy overlayu są trzymane w zmiennych CSS `--screen-top/right/bottom/left` i ustawiane funkcją `setInsets`. Współczynniki AR layoutu (np. 707/1023 dla inkwizycji) pilnują proporcji panela (`fitPanel`).
- Font i kolor akcentu są aplikowane przez zmienne CSS (`--font`, `--accent`), a wielkości i kolory prefix/suffix/treści przez `--prefix-font-size`, `--suffix-font-size`, `--msg-font-size`, `--prefix-color`, `--suffix-color`.
- Efekt CRT/flicker pochodzi z pseudo-elementów `.crt::before/::after` oraz animacji `flickerBg`; flaga `flicker` z Firestore dodaje/usuwa klasę `.no-flicker` na `.screen`.
- Logo (jeśli dostępne) renderowane jest w `.logo` i ukrywane, gdy frakcja nie ma wpisu w `FACTION_LOGO` lub `showLogo` jest `false`.

## Audio i odblokowanie dźwięku
- Overlay `#unlock` reaguje na click/touch/pointer/keydown i ustawia `window.__dsAudioArmed = true`. Dopóki użytkownik nie kliknie, audio nie jest odtwarzane.
- Funkcja globalna `window.__dsPlayUrlOnce(url)` tworzy nowy obiekt `Audio` i odtwarza wskazany plik tylko gdy audio jest odblokowane; błędy są ignorowane, by nie blokować UI.
- Domyślne ścieżki w Infoczytniku: `assets/audio/global/Ping.mp3?v=<ASSET_VERSION>` oraz `Message.mp3`. GM może nadpisać URL-e przez pola `pingUrl`/`msgUrl` w dokumencie (jeśli doda je do zapisu).

## Struktura assetów i wersjonowanie
- Layouty: `assets/layouts/<frakcja>/…png` — mapowane w `LAYOUT_BG` (Infoczytnik).
- Logotypy: `assets/logos/<frakcja>/…png` — mapowane w `FACTION_LOGO`; brak wpisu = brak loga nawet gdy przełącznik „Dodaj logo” jest włączony.
- Dźwięki globalne: `assets/audio/global/Ping.mp3` oraz `Message.mp3`.
- Stała `ASSET_VERSION`/`INF_VERSION` w Infoczytniku doklejana jest jako query param do plików, co wymusza odświeżenie cache po podmianie assetów.

## Payload Firestore (`dataslate/current`)
| Pole | Typ | Opis |
| --- | --- | --- |
| `type` | `"message"` \| `"ping"` \| `"clear"` | Rodzaj akcji; `clear` usuwa tylko tekst, `ping` odpala dźwięk, `message` ustawia treść. |
| `faction` | `string` | Klucz frakcji zgodny z mapami `LAYOUT_BG`/`FILLERS`/`FONT_STACK`. |
| `color`/`fontColor` | `string` | Kolor akcentu/treści wiadomości (HEX/CSS). |
| `msgFontSize` | `string` (`"28px"`…) | Rozmiar treści, trafia do `--msg-font-size`. |
| `prefixColor`/`suffixColor` | `string` | Kolory prefixu i suffixu. |
| `prefixFontSize`/`suffixFontSize` | `string` | Rozmiary fontu prefixu/suffixu. |
| `prefixIndex`/`suffixIndex` | `number` | Indeksy 1-based do fillerów; używane gdy nie podano `prefix`/`suffix`. |
| `prefix`/`suffix` (`prefixText`/`suffixText`) | `string` | Bezpośredni tekst linii nad/pod wiadomością. |
| `text` | `string` | Treść wiadomości; puste stringi czyszczą widok, ale `type="clear"` jest preferowany. |
| `showLogo` | `boolean` | Czy pokazać logo frakcji. |
| `flicker` | `boolean` | Czy włączyć animację CRT. |
| `pingUrl`/`msgUrl` (`messageUrl`) | `string` | Opcjonalne ścieżki niestandardowych dźwięków. |
| `nonce` | `string` | Wartość unikalna z GM (timestamp + losowa część), zapobiega powtórnym aplikacjom tego samego snapshota. |
| `ts` | `firebase.firestore.FieldValue.serverTimestamp()` | Znacznik czasu ustawiany po stronie serwera. |

## Zachowanie panelu GM (`GM.html`)
- Używa Firebase 9 compat (`initializeApp`, `firestore`). Zapis do dokumentu następuje przez `currentRef.set(..., { merge:false })` dla wiadomości/clear i `{ merge:true }` dla ping.
- Filler prefix/suffix są pobierane z mapy `LAYOUTS` (9 wpisów na frakcję). Checkbox „Losuj automatycznie” blokuje ręczne pola numerów; przy zapisie do Firestore numery są losowane w zakresie długości tablicy.
- Funkcje `normalizePx` i `getPrefixSuffixColor` pilnują walidacji pól liczbowych/kolorów. Live preview (`updateLivePreview`) odzwierciedla aktualną kompozycję.
- Przycisk „Ping” wysyła zdarzenie typu `ping`, zachowując ostatnio wybrane ustawienia stylu, dzięki czemu Infoczytnik nie traci fontu/kolorów przy samym pingu.

## Rozszerzanie projektu
- **Nowa frakcja:** dodaj pliki PNG do `assets/layouts/<frakcja>/` i ewentualnie logo do `assets/logos/<frakcja>/`, uzupełnij mapy `LAYOUT_BG`, `FILLERS`, `FONT_STACK`, opcjonalnie proporcje w `LAYOUT_AR` i insets w `SCREEN_INSETS`. Jeśli ma mieć logo, dodaj wpis w `FACTION_LOGO`.
- **Nowy layout dla istniejącej frakcji:** dodaj plik PNG i podmień ścieżkę w `LAYOUT_BG`. Jeżeli zmienia się układ pola tekstowego, zaktualizuj wpis w `SCREEN_INSETS`/`LAYOUT_AR` i podnieś `ASSET_VERSION`.
- **Niestandardowe dźwięki:** podmień pliki w `assets/audio/global/` albo dodaj pola `pingUrl`/`msgUrl` w zapisie z GM wskazujące nowe ścieżki (pamiętaj o `?v=...`).
- **Integracja z innym backendem:** zachowaj format dokumentu `dataslate/current` lub stwórz kompatybilną warstwę API, która publikuje identyczne pola; Infoczytnik reaguje na aktualizacje snapshotu Firestore.

## Troubleshooting
- **Brak nowych assetów:** zwiększ `ASSET_VERSION`/`INF_VERSION`, wykonaj twarde odświeżenie lub wyczyść cache urządzenia.
- **Brak dźwięku:** upewnij się, że overlay został kliknięty (sprawdź `window.__dsAudioArmed` w konsoli) i że ścieżki do plików audio są prawidłowe.
- **Brak synchronizacji:** sprawdź `config/firebase-config.js`, dostępność Firestore i reguły odczytu/zapisu; w razie braku `firebaseConfig` Infoczytnik wyświetli statyczny layout bez live-updatu.
