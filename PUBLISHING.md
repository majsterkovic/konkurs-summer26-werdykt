# Publikacja werdyktu na GitHub Pages — instrukcja dla agenta

## Stan repo

- `index.html` — samodzielna, kompletna strona (owinięty fragment z Artifact:
  `<!doctype html>` + `<head>` z linkiem do Google Fonts i CSS + `<body>` z
  treścią i lightboxem). Waga ok. 8,2 MB — sześć wywołanych zdjęć finałowych
  jest osadzonych jako base64 (1700 px dłuższy bok), reszta miniatur (27
  kandydatów w sekcjach sporu/odrzuceń) w mniejszej rozdzielczości, też base64.
  Strona nie ma żadnych zależności poza dwoma hostami CDN dozwolonymi przez
  Google Fonts (`fonts.googleapis.com`, `fonts.gstatic.com`) — działa offline
  poza czcionkami.
- `images/` — te same sześć zdjęć finałowych jako osobne pliki JPG, dłuższy
  bok 2400 px, jakość 88 (0,5–2,5 MB/szt., razem ~8,2 MB). Nieużywane przez
  `index.html` — trzymane obok jako pliki źródłowe do ewentualnego wykorzystania
  w innym kontekście (np. druk finalistów, e-mail do organizatora). Można je
  pominąć przy publikacji Pages, jeśli zależy na mniejszym repo.
- Repo **nie jest jeszcze zainicjalizowane jako git** w chwili pisania tej
  instrukcji — zrób to jako pierwszy krok poniżej, jeśli `git status` w tym
  katalogu zwróci błąd.

## ⚠️ Zanim opublikujesz — to wymaga potwierdzenia użytkownika

**GitHub Pages jest zawsze publiczny.** Ustawienie repozytorium jako prywatne
NIE ukrywa strony Pages — kontrola dostępu do Pages istnieje wyłącznie w
GitHub Enterprise Cloud, którego to konto (`majsterkovic`, plan zwykły) nie ma.
Publikacja = zdjęcia konkursowe i pełny werdykt trafiają do otwartego
internetu, zanim konkurs zostanie rozstrzygnięty.

Nie sprawdzałem regulaminu konkursu „Summer '26" pod kątem tego, czy dopuszcza
publikację prac przed rozstrzygnięciem — wiele firmowych konkursów tego
zabrania. **Zanim wykonasz krok 3 (push) i krok 4 (włączenie Pages), zapytaj
użytkownika wprost, czy akceptuje, że strona będzie publicznie widoczna** —
chyba że użytkownik już to potwierdził w wiadomości, która zleciła Ci tę
pracę. Kroki 1–2 (git init, commit) są bezpieczne i lokalne, wykonaj je bez
pytania.

Alternatywa bez tego ryzyka: strona jest już opublikowana jako prywatny
Artifact (widoczny tylko dla użytkownika, chyba że sam go udostępni) pod:
`https://claude.ai/code/artifact/b37a9876-7fcb-4ab4-b5bf-93de0640573e`
Warto to przypomnieć użytkownikowi jako opcję, jeśli waha się co do publicznego GitHub Pages.

## Krok 1 — inicjalizacja repo (bezpieczne, lokalne)

```bash
cd /home/mariusz/Desktop/raw/konkurs_publikacja
git init
git add index.html images/ PUBLISHING.md
git commit -m "Werdykt jury: Summer '26 Photo Contest (Nature Seekers / Urban Explorers)"
```

Uwaga: NIE dodawaj do tego repo żadnych innych plików z `/home/mariusz/Desktop/raw/`
poza tym, co już leży w `konkurs_publikacja/` — reszta katalogu (`praga_dng`,
`szklarska_jpgs`, `warianty_konkursowe`, `Elŝutujo/raws` itd.) to prywatne
źródła RAW i pliki robocze, nie jest przeznaczona do publikacji.

## Krok 2 — utworzenie repozytorium na GitHub (wymaga potwierdzenia z sekcji ⚠️ powyżej)

`gh` jest zalogowany jako `majsterkovic` (scope `repo` obecny). Nazwa repo do
ustalenia z użytkownikiem — poniżej przykład:

```bash
gh repo create konkurs-summer26-werdykt --public --source=. --remote=origin
git push -u origin main
```

Jeśli użytkownik zdecyduje się na private repo mimo że Pages i tak będzie
publiczny (np. żeby kod źródłowy strony nie był przeszukiwalny, tylko sama
wersja Pages) — użyj `--private` zamiast `--public`.

## Krok 3 — włączenie GitHub Pages

```bash
gh api -X POST repos/majsterkovic/konkurs-summer26-werdykt/pages \
  -f "source[branch]=main" -f "source[path]=/"
```

Jeśli komenda zwróci błąd (bywa, że trzeba użyć innego builda API albo
poczekać na propagację), alternatywnie w przeglądarce:
Settings → Pages → Source: `Deploy from a branch` → `main` / `/ (root)`.

Strona pojawi się pod adresem:
`https://majsterkovic.github.io/konkurs-summer26-werdykt/`

(zmień nazwę repo w URL, jeśli użytkownik wybrał inną).

## Krok 4 — weryfikacja

- Sprawdź, że strona się wczytuje i że lightbox (kliknięcie w zdjęcie w sekcji
  „Sześć zgłoszeń") otwiera pełną rozdzielczość.
- Sprawdź w DevTools, że czcionki z Google Fonts się ładują (brak CSP errors —
  Pages nie ma restrykcji jak Artifact, więc to tylko kwestia sieci).
- Zgłoś użytkownikowi finalny URL i przypomnij, że strona jest teraz publiczna
  i indeksowalna — usunięcie repo później nie cofnie ewentualnych kopii w
  cache wyszukiwarek.

## Dane źródłowe (dla kontekstu, gdyby trzeba było coś zregenerować)

Strona została zbudowana skryptem Python w katalogu tymczasowym sesji, na
podstawie:
- ocen dwóch niezależnych agentów-krytyków (fotograf/art director + juror
  konkursowy), którzy ocenili 27 kandydatów niezależnie od siebie,
- werdyktu lead-jurora (mnie) rozstrzygającego spory między krytykami,
- wywołanych plików PNG z `/home/mariusz/Desktop/raw/warianty_konkursowe/`
  (m.in. `stacja.png`, `top_picks_graded/mgla.png`, `top_picks_graded/kloda.png`,
  `praga_graded/synagoga.png`, `praga_graded/schody.png`, `praga_graded/katedra.png`).

Skrypt budujący (`build.py`) i wszystkie pośrednie artefakty (miniatury base64,
tabele ocen w Pythonie) zostały w katalogu tymczasowym sesji i nie są częścią
tego repo — jeśli treść trzeba będzie zmienić, najprościej edytować
bezpośrednio `index.html` (jest to zwykły HTML+CSS+JS, sekcje są czytelnie
oznaczone komentarzami klas jak `.dossier`, `.dispute`, `.near`, `.rej`).
