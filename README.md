WH40k Data-Slate (GM + Infoczytnik)
Dokumentacja / Instrukcja (PL)
Wersja: 2025-12-13
Repo: wh40k-data-slate (GitHub Pages)

============================================================
1) OPIS W 2 ZDANIACH
============================================================
To prosta aplikacja webowa składająca się z dwóch stron:
- GM.html — panel dla Mistrza Gry do wysyłania komunikatów, pingów i sterowania wyglądem.
- Infoczytnik.html — ekran dla graczy (tablet/laptop), który wyświetla wiadomość na tle wybranego layoutu oraz odtwarza dźwięki.

Synchronizacja działa w czasie rzeczywistym przez Firebase Firestore (dokument: dataslate/current).

============================================================
2) JAK TO DZIAŁA — PERSPEKTYWA UŻYTKOWNIKA (KROK PO KROKU)
============================================================

2.1. Wymagania
- Dowolny nowoczesny Chrome (na laptopie i/lub tablecie).
- Dostęp do internetu (Firebase + Google Fonts; same PNG/MP3 są z GitHub Pages).
- Obie strony mogą działać na tym samym urządzeniu w dwóch kartach lub na dwóch różnych urządzeniach.

2.2. Uruchomienie
1) Otwórz panel GM:
   - Wejdź na stronę GM (GitHub Pages) i otwórz GM.html.
2) Otwórz Infoczytnik:
   - W drugiej karcie (lub na tablecie) otwórz Infoczytnik.html.
3) Odblokuj dźwięk w Infoczytniku:
   - Po wejściu w Infoczytnik pojawi się nakładka „Kliknij raz, aby odblokować dźwięk”.
   - Kliknij w dowolne miejsce na nakładce (Chrome blokuje audio bez interakcji użytkownika).
   - To trzeba wykonać 1x po odświeżeniu strony.

Uwaga:
- Infoczytnik po uruchomieniu może pokazać ostatnio wysłaną wiadomość (to jest OK i akceptowane).

2.3. Wysyłanie wiadomości (GM → Infoczytnik)
1) W GM wybierz „Frakcja / layout”.
2) Ustaw styl treści wiadomości:
   - „Kolor fontu (treść wiadomości)” — wybierz z picker’a lub z szybkich kolorów (Zielony/Czerwony/Złoty/Biały).
   - „Wielkość fontu (treść wiadomości)” — wpisz rozmiar w px (np. 28).
3) Ustaw styl Prefix + Suffix:
   - „Kolor Prefix + Suffix (wspólny)”:
     - Możesz wpisać w polu tekstowym np. #ffffff albo rgba(255,255,255,.88),
     - albo użyć picker’a (ustawia #RRGGBB),
     - albo użyć szybkich kolorów (Zielony/Czerwony/Złoty/Biały).
   - „Wielkość fontu Prefix + Suffix (wspólna)” — w px (np. 14).
4) Ustaw prefix/suffix (fillery):
   - Zaznaczone „Losuj automatycznie”:
     - GM losuje prefix i suffix z listy dla danej frakcji.
     - Podgląd prefixu i suffixu aktualizuje się automatycznie.
   - Odznaczone „Losuj automatycznie”:
     - Wpisujesz ręcznie numer Prefix (po lewej) i Suffix (po prawej).
     - Podgląd prefixu i suffixu zmieni się po zmianie wartości.
5) Wpisz „Treść komunikatu” (textarea).
6) Kliknij „Wyślij”.
Efekt:
- Infoczytnik zmienia layout na odpowiedni dla frakcji.
- Ustawia font (z Google Fonts + fallback do Calibri/Arial).
- Ustawia kolory i rozmiary według ustawień z GM.
- Wyświetla prefix + logo (dla wybranych frakcji) + treść + suffix.
- Odtwarza dźwięk Message (MP3).

2.4. Ping (GM → Infoczytnik)
1) Kliknij „Ping”.
Efekt:
- Infoczytnik dostaje sygnał ping i odtwarza dźwięk Ping (MP3).
- Dodatkowo wysyłane są ustawienia stylu (kolory/rozmiary), żeby Infoczytnik nie „gubił” wyglądu.

2.5. Wyczyść ekran (GM → Infoczytnik)
1) Kliknij „Wyczyść ekran”.
Efekt:
- Infoczytnik usuwa prefix, treść i suffix, ale zostawia tło layoutu.

2.6. Wyczyść pole (tylko GM)
1) Kliknij „Wyczyść pole”.
Efekt:
- Czyści wyłącznie pole tekstowe w GM (textarea), nic nie wysyła do Infoczytnika.

2.7. Podgląd prefix/suffix (GM)
- Nad polem „Treść komunikatu” jest podgląd prefixu (ramka preview).
- Pod polem „Treść komunikatu” jest podgląd suffixu (ramka preview).
- Podgląd jest tylko do odczytu, a zmiana odbywa się przez losowanie lub numer ręczny.

============================================================
3) JAK TO DZIAŁA — TECHNICZNIE (KOD, SKRYPTY, FIREBASE)
============================================================

3.1. Struktura aplikacji (logika)
Są 2 niezależne strony HTML:
- GM.html:
  - UI + logika wyboru frakcji, kolorów, rozmiarów, losowania prefix/suffix.
  - Zapisuje stan do Firestore: dataslate/current.
- Infoczytnik.html:
  - Nasłuchuje Firestore: dataslate/current (real-time).
  - Na podstawie danych z dokumentu aktualizuje:
    - layout (tło PNG),
    - font frakcji,
    - kolory i rozmiary tekstów,
    - prefix/suffix (z tekstu lub indeksów),
    - logo frakcji (opcjonalnie),
    - dźwięk (Ping/Message),
    - czyszczenie ekranu.

3.2. Firebase / Firestore — kluczowa część
Używamy Firebase Firestore jako „kanału komunikacji” w czasie rzeczywistym.
Najważniejszy element:
- Kolekcja: dataslate
- Dokument: current
- Ścieżka: dataslate/current

GM wykonuje currentRef.set(...).
Infoczytnik wykonuje onSnapshot(currentRef, callback).

To działa jak „wspólny notatnik”:
- Każde kliknięcie w GM zapisuje nowy stan dokumentu.
- Infoczytnik natychmiast dostaje zmianę przez onSnapshot.

3.3. Pola w dokumencie dataslate/current (kontrakt danych)
Najważniejsze pola:

A) Sterowanie typem zdarzenia
- type: "message" | "ping" | "clear"

B) Meta / anty-dublowanie
- nonce: losowy identyfikator zdarzenia
- ts: serverTimestamp()

Infoczytnik trzyma lastNonce i ignoruje powtórzenia.

C) Wygląd i treść
- faction: np. "mechanicus", "inquisition"…
- text: treść wiadomości (dla type="message")
- color / fontColor: kolor treści wiadomości (hex)

D) Prefix/Suffix (dwa tryby)
- prefixIndex / suffixIndex (1..N) — wybór z list fillerów po stronie Infoczytnika
ALBO (opcjonalnie w przyszłości):
- prefix / suffix (gotowy tekst)

E) Styl fontów (z GM)
- msgFontSize: np. "28px"
- prefixFontSize: np. "14px"
- suffixFontSize: np. "14px"
- prefixColor: np. "#ffffff" lub "rgba(...)"
- suffixColor: jw.

F) Audio (opcjonalne)
- pingUrl (jeśli kiedyś będziesz nadpisywać globalny ping)
- msgUrl / messageUrl (jeśli kiedyś będziesz nadpisywać globalny message)

Aktualnie Infoczytnik domyślnie korzysta z:
assets/audio/global/Ping.mp3
assets/audio/global/Message.mp3

3.4. GM.html — co robi kod
GM:
- Definiuje listy fillerów (LAYOUTS) dla frakcji:
  LAYOUTS[frakcja].prefixes[]
  LAYOUTS[frakcja].suffixes[]
- Funkcja computePreview():
  - Wylicza bieżący prefix/suffix:
    - losowo, gdy randomFillers checked,
    - albo według numeru, gdy randomFillers odznaczony.
  - Uzupełnia podglądy prefixPreview i suffixPreview.
  - Blokuje/odblokowuje pola ręczne (disabled) gdy losowanie włączone.

- Funkcja sendMessage(isClear):
  - Składa paczkę danych do Firestore:
    - type = "message" lub "clear"
    - faction, color/fontColor
    - msgFontSize, prefix/suffix style
    - prefixIndex, suffixIndex
    - text
    - nonce, ts
  - Wysyła przez currentRef.set(..., { merge:false })

- Funkcja ping():
  - Zapisuje type="ping" + nonce + (merge:true)
  - Dodatkowo dokleja style, żeby utrzymać spójny wygląd po stronie Infoczytnika.

3.5. Infoczytnik.html — co robi kod
Infoczytnik:
- Ładuje fonty Google Fonts (z fallbackiem do Calibri/Arial).
- Definiuje CSS zmienne (variables) do wyglądu:
  --accent, --font
  --screen-top/right/bottom/left (bezpieczne marginesy)
  --msg-font-size, --prefix-font-size, --suffix-font-size
  --prefix-color, --suffix-color

- Layout i pole tekstu:
  - Tło (layout) jest <img class="layout-img"> z object-fit: contain.
  - Na nim leży przez absolute-position element .screen,
    czyli „bezpieczny obszar” na tekst.
  - To .screen ma overflow:auto => przewijanie tylko w tym obszarze.
  - Tło się nie rusza, przewija się tylko tekst.

- Wyliczanie „bezpiecznego pola” na tekst:
  Zrobione przez zestaw wartości procentowych:
    --screen-top, --screen-right, --screen-bottom, --screen-left

  Dlaczego procenty?
  - Ponieważ panel ma zmienny rozmiar na różnych urządzeniach,
    a tło jest skalowane proporcjonalnie.
  - Procenty powodują, że obszar tekstu skaluje się wraz z tłem.

  Jak zostało dobrane?
  - Metoda praktyczna: patrzymy na PNG i przyjmujemy „zapas” tak,
    żeby tekst nie nachodził na ozdobne elementy ramki.
  - Dla DataSlate_Inq.png ramka jest inna (cieńsza) — osobny preset.
  - Dla DataSlate_04.png ramka jest grubsza — osobny preset.
  - Presety są w SCREEN_INSETS:
      inquisition: {top:"14%", right:"14%", bottom:"26%", left:"18%"}
      default04:   {top:"14%", right:"14%", bottom:"22%", left:"18%"}
  To jest celowo konserwatywne (większy margines), żeby było bezpiecznie.

- Dopasowanie rozmiaru panelu do ekranu:
  Funkcja fitPanel(ar):
    - bierze aspect ratio layoutu (AR),
    - wylicza maksymalny rozmiar panelu tak, by mieścił się w viewport (vw/vh),
    - ustawia panel.style.width/height.
  AR jest w LAYOUT_AR:
    inquisition: 707/1023
    default04:   1131/1600

- Nasłuch Firestore:
  onSnapshot(doc(db,"dataslate","current"), callback)

  callback:
  1) sprawdza nonce (żeby nie odtwarzać tego samego eventu ponownie)
  2) wyciąga faction i color
  3) applyLayout(faction,color) — zmienia tło i font frakcji, ustawia insets i AR
  4) applyTextStyleFromDoc(d) — ustawia CSS zmienne rozmiarów i kolorów
  5) reaguje na type:
     - clear: czyści teksty, zostawia tło
     - ping: odtwarza Ping
     - message: buduje prefix/suffix (z tekstu lub indeksów) i wyświetla, odtwarza Message

- Fillery po stronie Infoczytnika:
  FILLERS zawiera listy prefix/suffix (dla mechanicus i inquisition itd).
  Jeśli GM wysyła prefixIndex/suffixIndex, Infoczytnik wybiera z listy.
  Jeśli brak — bierze domyślnie 1.

- Logo frakcji:
  FACTION_LOGO mapuje:
    inquisition → assets/logos/inquisition/Inquisition.png
    mechanicus  → assets/logos/mechanicus/Mechanicus.png
  Logo ma stały kontener logoBox (54x54) i object-fit: contain (bez deformacji).
  Logo jest po prawej od prefixu i przewija się razem z prefixem, bo jest w tej samej .screen.

3.6. Audio — dlaczego był „klik do odblokowania”
Chrome (i większość przeglądarek) blokuje automatyczne odtwarzanie dźwięku,
dopóki użytkownik nie wykona akcji w tej karcie (klik/klawisz).
Dlatego Infoczytnik ma overlay unlock, który po kliknięciu ustawia audioArmed=true.

============================================================
4) ZASOBY (ASSETS) I CACHE / WERSJONOWANIE
============================================================

4.1. Zasoby w repo (przykład)
- assets/audio/global/Ping.mp3
- assets/audio/global/Message.mp3
- assets/layouts/inquisition/DataSlate_Inq.png
- assets/layouts/<faction>/DataSlate_04.png (tymczasowo)
- assets/logos/inquisition/Inquisition.png
- assets/logos/mechanicus/Mechanicus.png

4.2. ASSET_VERSION (cache-busting)
W Infoczytniku jest stała:
ASSET_VERSION = "2025-12-13-1";

Każdy asset jest ładowany z dopiskiem:
...?v=ASSET_VERSION

Po co?
- GitHub Pages + Chrome potrafią trzymać stary obrazek/audio w cache.
- Zmiana ASSET_VERSION wymusza pobranie nowej wersji.

Kiedy zmieniać?
- Gdy podmienisz PNG lub MP3 i urządzenia dalej widzą starą wersję.
- Najprościej: zwiększ numer na końcu, np. "2025-12-13-2".

============================================================
5) JAK ROZWIJAĆ APLIKACJĘ (DOKŁADNE PROCEDURY)
============================================================

5.1. Dodawanie nowych plików audio (globalnie)
Cel: podmienić Ping.mp3 i/lub Message.mp3.

Kroki:
1) W repo wrzuć nowe pliki do:
   assets/audio/global/
   i nazwij:
   - Ping.mp3
   - Message.mp3
   (najprościej: nadpisz istniejące)

2) Zacommituj zmiany.

3) W Infoczytniku zwiększ ASSET_VERSION, np.:
   "2025-12-13-2"
   Zacommituj.

4) Odśwież Infoczytnik hard refresh (Ctrl+Shift+R).
   Jeśli tablet trzyma cache: dopisz do URL Infoczytnika ?v=123 lub zmień ASSET_VERSION.

5.2. Dodawanie osobnych dźwięków dla frakcji (przyszłość)
Założenie:
- Każda frakcja może mieć własny Ping i Message.

Proponowana struktura:
assets/audio/factions/<faction>/Ping.mp3
assets/audio/factions/<faction>/Message.mp3

Zmiany w kodzie (koncepcja):
- W Infoczytniku dodać mapę:
  FACTION_AUDIO = { mechanicus:{ping:"...", msg:"..."}, ... }
- W obsłudze ping/message wybierać:
  - jeśli istnieje FACTION_AUDIO[faction] → użyj go,
  - w przeciwnym wypadku → DEFAULT_*.

Alternatywa:
- Z GM wysyłać pingUrl/msgUrl zależnie od frakcji (już jest wspierane w Infoczytniku),
  wtedy GM „decyduje” jaki dźwięk zagra.

5.3. Dodawanie nowej frakcji
To wymaga zmian w 2 plikach: GM.html i Infoczytnik.html.

A) GM.html
1) Dodaj <option> w select#faction:
   <option value="new_faction">Nazwa frakcji</option>

2) Dodaj wpis do LAYOUTS:
   LAYOUTS.new_faction = { prefixes:[...], suffixes:[...] }

B) Infoczytnik.html
1) Dodaj tło layoutu:
   - wrzuć PNG do assets/layouts/new_faction/...
2) Dodaj mapowanie w LAYOUT_BG:
   new_faction: `assets/layouts/new_faction/<plik>.png?v=${ASSET_VERSION}`
3) Jeśli layout ma inny AR:
   - dodaj do LAYOUT_AR i ustaw w applyLayout warunek podobnie jak inquisition.
4) Jeśli layout wymaga innych marginesów:
   - dodaj preset do SCREEN_INSETS i użyj go w applyLayout.
5) Dodaj font frakcji (opcjonalnie):
   - dopisz font do linka Google Fonts (w head)
   - dopisz FONT_STACK.new_faction = '"NazwaFontu", Calibri, Arial, sans-serif'
6) Dodaj fillery w FILLERS (opcjonalnie, jeśli GM wysyła tylko index):
   FILLERS.new_faction = { prefixes:[...], suffixes:[...] }
7) Dodaj logo (opcjonalnie):
   - wrzuć PNG do assets/logos/new_faction/Logo.png
   - dodaj do FACTION_LOGO:
     new_faction: `assets/logos/new_faction/Logo.png?v=${ASSET_VERSION}`

Na końcu:
- Zwiększ ASSET_VERSION (jeśli doszły nowe grafiki) i zacommituj.

5.4. Dodawanie nowego layoutu (PNG) i ustawienie „bezpiecznego pola”
Cel: inny wygląd ramki => trzeba dopasować marginesy.

Kroki:
1) Wrzuć PNG do:
   assets/layouts/<faction>/<nazwa>.png

2) (Opcjonalnie) zmierz aspect ratio:
   - AR = szerokość / wysokość (w px).
   Przykład:
   707/1023 dla Inkwizycji.

3) Ustaw w Infoczytnik:
   - w LAYOUT_BG wskaż ten plik
   - w LAYOUT_AR dodaj AR (jeśli inny niż default)
   - w SCREEN_INSETS dodaj preset marginesów

Jak dobrać SCREEN_INSETS (praktyczna metoda):
- Ustaw tymczasowo bardzo duże marginesy (np. top 20%, left 20%, right 20%, bottom 25%),
- Wyślij długi tekst z GM,
- Zmniejszaj marginesy stopniowo, aż tekst będzie “blisko” czarnego pola,
  ale nadal nie dotyka ozdobnych elementów ramki.
- Zostaw zapas — tekst nie musi wypełniać całego czarnego obszaru.

Dlaczego to działa stabilnie?
- Ponieważ panel ma stały AR dopasowany do tła, a marginesy są procentowe,
  więc „bezpieczne pole” skaluje się idealnie z obrazkiem.

5.5. Dodawanie nowego logo (różne rozmiary)
Kroki:
1) Wrzuć logo do:
   assets/logos/<faction>/<nazwa>.png
2) Dodaj wpis w FACTION_LOGO:
   <faction>: `assets/logos/<faction>/<nazwa>.png?v=${ASSET_VERSION}`
3) Zwiększ ASSET_VERSION i zacommituj.

Logo się nie deformuje, bo:
- kontener ma stały rozmiar,
- logo ma object-fit: contain.

5.6. Zmiany UI w GM (np. nowe kontrolki)
Zasada:
- Wszystko co ma wpływać na Infoczytnik musi zostać zapisane do Firestore.
- Infoczytnik musi te pola odczytać i zastosować w CSS/DOM.

Wzorzec:
GM:
- dodajesz input w HTML,
- dodajesz go do el = {...},
- w sendMessage/ping dopisujesz pole do obiektu set(),
Infoczytnik:
- w applyTextStyleFromDoc(d) dopisujesz obsługę pola
  i mapujesz na CSS variable albo bezpośrednią zmianę elementu.

============================================================
6) DIAGNOSTYKA / TROUBLESHOOTING
============================================================

6.1. „Nie widzę zmian na GitHub Pages”
- Upewnij się, że commit poszedł do repo.
- Zrób Ctrl+Shift+R.
- Dopisz do URL ?v=123.
- Sprawdź, czy otwierasz właściwy plik (GM.html vs index.html).

6.2. „Audio nie gra”
- W Infoczytniku kliknij overlay odblokowania audio.
- Sprawdź czy pliki MP3 istnieją pod ścieżką assets/audio/global/.
- Sprawdź czy w konsoli nie ma błędów 404.

6.3. „Tekst nachodzi na ramkę”
- Zwiększ marginesy SCREEN_INSETS dla danego layoutu.
- Upewnij się, że AR jest poprawny.

============================================================
7) SŁOWNIK / NAZYWNICTWO
============================================================
- GM — panel nadawczy (Mistrz Gry)
- Infoczytnik — ekran odbiorczy (gracze)
- Layout — tło PNG (ramka Data-Slate)
- Screen / safe area — bezpieczny obszar na tekst (z marginesami)
- Fillery — prefixy i suffixy
- nonce — unikat zdarzenia do ignorowania duplikatów
- ASSET_VERSION — “cache buster” dla PNG/MP3

============================================================
8) CHECKLISTA „CO ZMIENIAM, GDY…”
============================================================

- Podmieniam MP3/PNG:
  1) wrzuć plik (nadpisz lub dodaj)
  2) commit
  3) zwiększ ASSET_VERSION
  4) commit
  5) Ctrl+Shift+R / ?v=...

- Dodaję nową frakcję:
  1) GM: option + LAYOUTS (prefix/suffix)
  2) Inf: LAYOUT_BG + (opcjonalnie) FONT_STACK + FILLERS + FACTION_LOGO + insets/ar
  3) assets: layout + logo + (opcjonalnie) audio
  4) ASSET_VERSION++

============================================================
KONIEC DOKUMENTU (PL)
============================================================
