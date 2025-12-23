# WH40k Data-Slate

Dwustronicowa aplikacja webowa do szybkiego pokazywania komunikatów inspirowanych WH40k. **GM.html** służy jako panel operatora, a **Infoczytnik.html** to ekran graczy, który nasłuchuje Firestore i natychmiast odświeża layout, tekst i audio.

## Jak działa aplikacja?
- GM zapisuje stan w dokumencie Firestore `dataslate/current` (typ zdarzenia, frakcja, styl, treść, kontrolki UI).
- Infoczytnik subskrybuje ten dokument, aktualizując tło, fonty, marginesy, animację flicker, logo oraz treść z prefixem/suffixem.
- Odtwarzanie dźwięku jest odblokowywane jednorazowym kliknięciem overlayu; kolejne „ping”/wiadomości uruchamiają odpowiednie pliki MP3.

## Szybki start
1. Skopiuj `config/firebase-config.template.js` do `config/firebase-config.js` i wklej tam konfigurację Firebase (Web).
2. Włącz Firestore Database w projekcie Firebase z regułami pozwalającymi na odczyt/zapis dokumentu `dataslate/current`.
3. Uruchom aplikację lokalnie (np. prosty `npx http-server .`) lub hostuj przez GitHub Pages, pamiętając o pliku `config/firebase-config.js`.
4. Otwórz `GM.html` i `Infoczytnik.html` (na tym samym lub różnych urządzeniach). Infoczytnik automatycznie wymusza cache-busting parametrem `?v=<INF_VERSION>`.
5. Kliknij overlay w Infoczytniku („Kliknij raz, aby odblokować dźwięk”), aby przeglądarka zezwoliła na odtwarzanie audio.

## Sterowanie w panelu GM
- **Frakcja / layout** – wybór zestawu tła, logo (opcjonalnie), stacku fontów i fillerów dla prefixu/suffixu.
- **Kolor i rozmiar treści** – input koloru + szybkie chipy, suwak liczbowy px.
- **Kolor i rozmiar prefix/suffix** – wspólny kolor (tekst + picker), px; działa niezależnie od koloru treści.
- **Losowość fillerów** – gdy włączone, prefix/suffix są losowane z mapy frakcji; wyłączone pozwala wpisać numer indeksu ręcznie.
- **Logo** – przełącznik dodawania logotypu frakcji do ekranu gracza (Infoczytnik ukryje logo, jeśli frakcja nie ma mapowania).
- **Flicker** – włącza/wyłącza animację efektu CRT w Infoczytniku.
- **Ping** – wysyła zdarzenie typu `ping` uruchamiające dźwięk ping bez zmiany treści ani layoutu.
- **Wyślij / Wyczyść** – zapisują typ `message` (z treścią) lub `clear` (czyści tekst, zachowując layout i styl).
- **Wyczyść pole** – lokalne czyszczenie textarea bez zapisu do Firestore.
- **Podgląd na żywo** – sekcja Live preview pokazuje aktualną kompozycję prefix/treść/suffix z bieżącymi kolorami i rozmiarami.

## Aktualizacja assetów
- Pliki tła: `assets/layouts/<frakcja>/*.png`
- Logotypy: `assets/logos/<frakcja>/*.png` (tylko frakcje obecne w `FACTION_LOGO`).
- Dźwięki globalne: `assets/audio/global/Ping.mp3` i `Message.mp3`.
- Zmieniaj `ASSET_VERSION`/`INF_VERSION` w Infoczytniku, aby dołączyć `?v=...` i wymusić odświeżenie cache po podmianie plików.

## Więcej szczegółów
Szczegółową dokumentację architektury, payloadu Firestore i sposobów rozszerzania aplikacji znajdziesz w `docs/Documentation.md`.
