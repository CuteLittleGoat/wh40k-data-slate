# WH40k Data-Slate

## Jak działa aplikacja?
- Aplikacja składa się z dwóch stron: **GM.html** (panel dla mistrza gry) i **Infoczytnik.html** (ekran graczy).
- Obie strony synchronizują się w czasie rzeczywistym przez Firebase Firestore w dokumencie `dataslate/current`.
- GM wybiera layout frakcji, ustawia formatowanie, wpisuje treść i wysyła ją do Infoczytnika, który od razu aktualizuje wyświetlany komunikat i dźwięki.

## Szybki start
1. Skopiuj `config/firebase-config.template.js` do `config/firebase-config.js` i wklej swój konfigurację Firebase.
2. Włącz Firestore Database w projekcie Firebase.
3. W repo włącz GitHub Pages, aby hostować aplikację.
4. Otwórz `GM.html` i `Infoczytnik.html` (na tym samym lub na różnych urządzeniach).
5. W Infoczytniku kliknij overlay „Kliknij raz, aby odblokować dźwięk”, aby przeglądarka zezwoliła na odtwarzanie audio.

## Jak aktualizować dane i wprowadzać zmiany
- **Aktualizacja treści:** wpisz nowy komunikat w `GM.html` i kliknij „Wyślij”, aby pojawił się w Infoczytniku.
- **Zmiana layoutu lub frakcji:** wybierz opcję w panelu GM, a Infoczytnik natychmiast przełączy tło, logo i fonty dla danej frakcji.
- **Podmiana assetów (audio/grafika):** zastąp pliki w `assets/audio/` lub `assets/layouts/` / `assets/logos/` i zwiększ wartość `ASSET_VERSION`, aby wymusić odświeżenie cache.
- **Dodawanie frakcji lub layoutów:** dopisz nowe wpisy w mapach layoutów/logo/fontów w kodzie (szczegóły w `docs/Documentation.md`) i dodaj odpowiadające im pliki PNG/MP3.
- **Konfiguracja Firebase:** utrzymuj poprawny `config/firebase-config.js`; zmiana projektu Firebase wymaga podmiany kluczy w tym pliku.

## Więcej szczegółów
Dokładny opis kodu, architektury oraz rozszerzania projektu znajdziesz w `docs/Documentation.md`.
