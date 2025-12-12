# WH40k Data-Slate (GM + Infoczytnik)

PL/EN documentation in one file.

Projekt: prosty „komunikator” do sesji RPG (WH40k), gdzie GM wysyła komunikaty z laptopa, a gracze widzą je na tablecie w klimatycznym interfejsie (Data-Slate) z layoutami, fillerami i dźwiękiem.

---

## 🇵🇱 Opis projektu (PL)

### Co to jest?
WH40k Data-Slate to mini-aplikacja działająca w przeglądarce (bez instalacji), składająca się z dwóch stron:

- **GM.html** — panel MG (laptop). Umożliwia:
  - wybór frakcji (layoutu),
  - wybór koloru fontu,
  - wybór prefix/suffix (losowo lub ręcznie),
  - wysyłanie wiadomości,
  - ping (sam dźwięk + efekt),
  - czyszczenie ekranu Infoczytnika,
  - czyszczenie pola tekstowego w GM.

- **Infoczytnik.html** — ekran dla graczy (tablet). Umożliwia:
  - wyświetlanie tła layoutu (DataSlate PNG),
  - nakładanie tekstu (prefix / treść / suffix),
  - przewijanie długiej wiadomości w obrębie „czarnego okna”,
  - delikatne efekty wizualne,
  - odtwarzanie dźwięku przy wiadomości i ping.

Synchronizacja danych odbywa się przez **Firebase Firestore** (w chmurze), dzięki czemu laptop i tablet mogą być w tej samej sieci lub w różnych miejscach — wystarczy internet.

---

## Jak to działa (architektura)

1. **GM.html** zapisuje dokument `dataslate/current` w Firestore.
2. **Infoczytnik.html** nasłuchuje zmian tego dokumentu (`onSnapshot`).
3. Każde zdarzenie ma `nonce` (unikalne ID). Jeśli `nonce` jest nowe:
   - `type=message` → wyświetla tekst + gra dźwięk Message
   - `type=ping` → nie zmienia tekstu, ale gra dźwięk Ping + efekt
   - `type=clear` → chowa tekst i zostawia samo tło layoutu

---

## Hosting i pliki statyczne (GitHub Pages)

Projekt jest hostowany przez **GitHub Pages**. Przykładowo:
- Główna strona: `https://cutelittlegoat.github.io/wh40k-data-slate/`

Wszystkie zasoby (PNG, MP3) trzymamy w repozytorium, dzięki czemu mają stabilne URL-e i działają w `<audio>`.

---

## Struktura katalogów

```text
wh40k-data-slate/
├─ index.html
├─ GM.html
├─ Infoczytnik.html
└─ assets/
   ├─ audio/
   │  ├─ global/
   │  │  ├─ Ping.mp3
   │  │  └─ Message.mp3
   │  ├─ mechanicus/
   │  ├─ inquisition/
   │  ├─ militarum/
   │  ├─ chaos_undivided/
   │  ├─ khorne/
   │  ├─ nurgle/
   │  ├─ tzeentch/
   │  └─ slaanesh/
   └─ layouts/
      ├─ inquisition/
      │  └─ DataSlate_Inq.png
      ├─ mechanicus/
      │  └─ DataSlate_04.png
      ├─ militarum/
      │  └─ DataSlate_04.png
      ├─ chaos_undivided/
      │  └─ DataSlate_04.png
      ├─ khorne/
      │  └─ DataSlate_04.png
      ├─ nurgle/
      │  └─ DataSlate_04.png
      ├─ tzeentch/
      │  └─ DataSlate_04.png
      └─ slaanesh/
         └─ DataSlate_04.png
