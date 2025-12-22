# Dokumentacja techniczna

## Architektura i przepływ danych
- **Strony:** `GM.html` zapisuje stan do Firestore, `Infoczytnik.html` nasłuchuje Firestore i aktualizuje widok.
- **Ścieżka synchronizacji:** kolekcja `dataslate`, dokument `current` (`dataslate/current`).
- **Payload:** informacje o layoucie, kolorach, rozmiarach fontu, treści komunikatu, opcjonalnym logo oraz flagach sterujących dźwiękiem.

## Konfiguracja Firebase (`config/firebase-config.js`)
1. Skopiuj `config/firebase-config.template.js` i uzupełnij go danymi z konsoli Firebase (`apiKey`, `projectId`, itd.).
2. Plik ładowany jest przez obie strony, dlatego każda zmiana kluczy wymaga przebudowania/odświeżenia hostingu.
3. Firestore musi mieć włączone odczyt/zapis (np. reguły testowe na czas developmentu).

## Warstwa wizualna Infoczytnika
- Tło to obraz `<img>` z `object-fit: contain`; tekst nakładany jest w absolutnie pozycjonowanym overlayu `.screen`.
- Marginesy bezpieczeństwa definiują zmienne CSS `--screen-top/right/bottom/left` w procentach, dzięki czemu treść skaluje się proporcjonalnie do tła.
- Każdy layout może mieć własny stos fontów (`FONT_STACK`), proporcje (`LAYOUT_AR`) i insets (`SCREEN_INSETS`).

## Audio i odblokowanie dźwięku
- Przeglądarki wymagają interakcji użytkownika, dlatego Infoczytnik wyświetla overlay „Kliknij raz, aby odblokować dźwięk”.
- Po kliknięciu uruchamiane są `Ping.mp3` i `Message.mp3` z `assets/audio/global/` lub niestandardowe ścieżki ustawione w stanie Firestore.

## Struktura assetów i wersjonowanie
- Layouty: `assets/layouts/<frakcja>/*.png`
- Logotypy: `assets/logos/<frakcja>/*.png`
- Dźwięki globalne: `assets/audio/global/Ping.mp3` oraz `Message.mp3`
- Stała `ASSET_VERSION` doklejana jako query param wymusza odświeżenie cache (np. `?v=2025-12-13-1`).

## Rozszerzanie projektu
- **Nowa frakcja:** dodaj pliki layoutu/logo w `assets/layouts/<frakcja>/` i `assets/logos/<frakcja>/`, uzupełnij mapy `LAYOUT_BG`, `FACTION_LOGO`, `FONT_STACK`, opcjonalnie `SCREEN_INSETS` i `LAYOUT_AR` w `GM.html`/`Infoczytnik.html`.
- **Nowy layout:** dodaj PNG do właściwego folderu, dopisz wpis w `LAYOUT_BG`, zaktualizuj proporcje i marginesy, podbij `ASSET_VERSION`.
- **Zamiana dźwięków:** podmień pliki w `assets/audio/global/` lub wskaż niestandardowe ścieżki w stanie wysyłanym z GM; pamiętaj o `ASSET_VERSION`.
- **Modyfikacja UI:** komponenty panelu GM sterują właściwościami Firestore (treść, prefiksy/sufiksy, kolory, rozmiary fontów, tryb losowania). Zmiany w formularzu należy mapować na aktualizacje dokumentu `dataslate/current`.

## Troubleshooting
- Brak nowych assetów: zwiększ `ASSET_VERSION`, wykonaj twarde odświeżenie przeglądarki lub wyczyść cache urządzenia.
- Brak dźwięku: upewnij się, że overlay został kliknięty i ścieżki do plików audio są poprawne.
- Brak synchronizacji: sprawdź `config/firebase-config.js`, dostępność Firestore oraz reguły odczytu/zapisu.
