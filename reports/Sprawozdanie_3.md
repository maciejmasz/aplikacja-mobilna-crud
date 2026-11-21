# Laboratorium 3 — sprawozdanie teoretyczne

**Imię i nazwisko: Maciej Masztalerz**  
**Grupa: 43**  
**Data: 21.11.2026**

## T1. Ewolucja technologii frontendowych

1. Wybierz **trzy istotne zmiany** w rozwoju interfejsów: od klasycznych stron WWW do współczesnych aplikacji mobilnych.
2. Dla każdej wskaż: problem wcześniejszego podejścia, nowe rozwiązanie oraz nowy koszt/ograniczenie.
3. Wskaż jedną zmianę, która Twoim zdaniem miała największy wpływ na sposób projektowania aplikacji CRUD. Uzasadnij wybór.

| Zmiana / okres | Problem wcześniejszego podejścia | Co się poprawiło? | Jaki nowy problem powstał? |
| -------------- | -------------------------------- | ----------------- | -------------------------- |
| Ajax [5]          | Każda zmiana danych wymagała pobrania i narysowania całej strony.                      | Pobierane są tylko potrzebne dane i odświeża się tylko zmieniona część strony [5].       | Kod sam musi wiedzieć, co odświeżyć po odpowiedzi serwera, i obsłużyć asynchroniczność.                |
| SPA [6]           | Każda nawigacja to kolejne pełne żądanie do serwera. | Po pierwszym załadowaniu widoki są podmieniane, a aplikacja działa płynniej [6]. | Aplikacja sama zarządza stanem i routingiem. Gorsze indeksowanie przez wyszukiwarki i trudniejszy pomiar wydajności [6]. |
| Deklaratywny UI sterowany stanem [1][8][10] | W podejściu imperatywnym każdy widok zależny od danych trzeba zaktualizować ręcznie [10]. | Opisuje się wygląd ekranu dla danego stanu, a framework odświeża widok [1][10]. React Native (wykorzystywany w projekcie) tworzy przy tym prawdziwe natywne widoki [8]. | Trzeba wskazywać właściciela stanu i unikać stanu zduplikowanego [1] |

**Uzasadnienie:** Trzecia zmiana podana powyżej, deklaratywny UI sterowany stanem. Na przykładzie mojego problemu, dodanie, edycja lub usunięcie sesji zmienia dane widoczne naraz na liście, w sumie godzin (BR-02) i w postępie względem celu. Imperatywnie każde z tych miejsc trzeba poprawiać osobno, deklaratywnie wystarczy zmienić dane. Ajax i SPA były warunkiem wstępnym, ale to deklaratywność ukształtowała podział na stan, komponenty i warstwę danych.

## T2. Imperatywne i deklaratywne budowanie UI

1. Wyjaśnij różnicę między tymi podejściami, **nie ograniczając się do definicji**.
2. Wymyśl przykład zmiany listy po dodaniu elementu i opisz przebieg w obu podejściach (możesz użyć pseudokodu).
3. Zidentyfikuj **dwie potencjalne trudności** deklaratywnego podejścia.

**Różnica.** Imperatywnie programista pisze kroki (znajdź element, zmień tekst, ukryj komunikat); dokumentacja Reacta porównuje to do podawania kierowcy trasy zakręt po zakręcie i zauważa, że błędy rosną wraz ze złożonością formularza [1]. Deklaratywnie opisuje wynik dla danego stanu, a framework dobiera zmiany [1]. W Compose aktualizacja UI to ponowne wywołanie funkcji z nowymi argumentami, a nie wołanie setterów [10]. Różnica polega na tym, kto odpowiada za zgodność ekranu ze stanem: programista czy framework.

**Przykład: dodanie sesji 05.11, 45 min do listy przedmiotu „Bazy danych”** (nagłówek „Razem: 2,25 h”).

*Imperatywnie:*
```
onSave(session):
  repo.insert(session)
  listView.addChild(createRow(session))      // dodaj wiersz
  emptyLabel.hide()                          // ukryj "brak sesji"
  total.setText(format(totalMinutes()))      // przelicz sumę
  progressBar.setValue(percent())            // popraw postęp
```
Pominięty ostatni lub przedostatni krok zostawia ekran niespójny z danymi.

*Deklaratywnie:*
```tsx
const [sessions, setSessions] = useState<Session[]>([]);
function onSave(s) { repo.insert(s); setSessions([...sessions, s]); }

const visible = sessions.filter(matchesFilter).sort(byDate);
const totalMin = sum(visible.map(s => s.czasMin));   // wartość pochodna
return visible.length === 0 ? <EmptyState/> : <><Header total={totalMin}/><FlatList data={visible} .../></>;
```
Handler zmienia tylko stan, a lista, komunikat, suma i postęp wynikają z niego.

**Dwie trudności deklaratywnego podejścia:**
1. **Nadmiarowy lub zduplikowany stan.** Gdyby suma była osobnym stanem obok listy, każdy handler musiałby aktualizować oba, czyli wracamy do problemu imperatywnego. Dokumentacja zaleca usuwać stan, który da się obliczyć z innego [1].
2. **Efekty uboczne i koszt renderów.** Funkcje opisujące UI mogą być wołane często, w dowolnej kolejności, a ich wykonanie może być pominięte lub powtórzone [10]. Zapis do SQLite w renderowaniu dawałby wielokrotne zapisy, więc takie operacje należą do handlerów i warstwy danych. Przeliczanie sumy z wielu sesji przy każdym renderze trzeba w razie potrzeby przenieść do zapytania SQL.

## T3. Stan, komponenty i przepływ danych

1. Wybierz dwie funkcjonalności z przydzielonego projektu, w których pojawia się stan.
2. Określ, które informacje są tymczasowe, współdzielone między ekranami i trwałe.
3. Zaproponuj podział odpowiedzialności na komponenty / warstwy. Uzasadnij, gdzie powinny znajdować się reguły biznesowe.
4. Opisz sytuację, w której nieprawidłowe zarządzanie stanem powoduje niespójność UI.

| Funkcjonalność | Rodzaj stanu | Kto nim zarządza? | Ryzyko niespójności |
| -------------- | ------------ | ----------------- | ------------------- |
| F1: formularz sesji | Tymczasowy: wartości pól, błędy. Trwały: sesja w tabeli `sesja_nauki`. | Formularz: lokalny `useState`/hook. Zapis: repozytorium po walidacji z domeny. | UI pokazuje sesję jako dodaną mimo nieudanego zapisu, albo lista nie wie o nowej sesji. |
| F2: podsumowanie w okresie | Tymczasowy: tekst wyszukiwania. Współdzielony między ekranami: wybrany przedmiot i okres, załadowane sesje. Trwały: sesje, przedmioty i cel godzin w SQLite. | Stan współdzielony: jeden store/kontekst we wspólnym rodzicu [2]. Sumę liczy czysta funkcja domeny. | Dwa ekrany trzymają własne kopie listy i pokazują różne sumy. |

**Podział odpowiedzialności:** (1) komponenty UI (ekrany Expo Router [11], `FlatList`, formularz) tylko wyświetlają i przekazują zdarzenia; (2) stan (hooki/store) trzyma dane współdzielone, a każdy kawałek stanu ma jednego właściciela [2]; (3) domena (czyste funkcje `validateSession`, `sumMinutesInRange`, `canDeleteSubject`) zawiera reguły biznesowe; (4) dane (repozytorium na expo-sqlite) wykonuje SQL i tłumaczy błędy bazy.

**Reguły biznesowe w domenie** testuje się je w Jest samym wywołaniem funkcji, bez renderowania i bazy [7], działają tak samo z dodawania i edycji, są w jednym miejscu.

**Niespójność UI:** lista sesji „Bazy danych” trzyma własną kopię w `useState`, a formularz zapisuje nową sesję (05.11, 45 min) do SQLite. Lista dalej pokazuje „2,25 h”, więc użytkownik uznaje, że zapis się nie udał, i dodaje sesję ponownie. W bazie są dwa duplikaty, a po odświeżeniu suma skacze do 3,00 h. Przyczyną są dwie kopie tej samej informacji bez jednego właściciela [2]. Rozwiązanie: jeden store czytany przez oba ekrany, aktualizowany dopiero po udanym zapisie.

**Uzasadnienie:** informacja potrzebna jednemu ekranowi zostaje lokalnie, potrzebna wielu ekranom ma jednego właściciela wyżej [2], a to, czego nie wolno stracić po zamknięciu aplikacji, trafia do bazy [3].

## T4. CRUD, walidacja i reguły biznesowe

1. Dla głównej encji własnego projektu opisz po jednym **nietrywialnym** scenariuszu Create, Read, Update i Delete.
2. Zaproponuj **dwie walidacje techniczne** oraz **jedną regułę biznesową**; wyjaśnij różnicę.
3. Zidentyfikuj problem, jaki może wystąpić przy usunięciu rekordu powiązanego z inną encją. Zaproponuj politykę postępowania.

| Operacja | Scenariusz i warunki | Możliwy błąd | Jak powinien zareagować system? |
| -------- | -------------------- | ------------ | ------------------------------- |
| Create | Sesja dla „Bazy danych”: 05.11.2026, 45 min, „Indeksy”. Przedmiot musi istnieć, pola przechodzą walidację; suma w okresie obejmującym 05.11 rośnie. | `czas_min = 0` lub `45,5`; pusty temat; przedmiot usunięty w międzyczasie. | Walidacja blokuje zapis i pokazuje komunikat przy polu, baza nie jest wywoływana. |
| Read | Sesje „Bazy danych” z okresu 01.11–15.11.2026, wyszukiwanie po temacie, sortowanie malejąco po dacie; suma 2,25 h z celu 10 h (90 + 45 min). Granice włącznie. | Okres odwrócony; brak wyników; sesja z 15.11 pominięta przez błąd „o jeden dzień”. | Odwrócony okres: „Data początkowa nie może być późniejsza niż końcowa”. Brak wyników: pusty widok z tekstem dopasowanym do przyczyny. Sesja z granicznej daty wliczona. |
| Update | Zmiana czasu z 90 na 60 min i przeniesienie sesji do „Algorytmy”; przeliczają się sumy obu przedmiotów. | Niepoprawne wartości | Te same reguły co Create. |
| Delete | Usunięcie sesji po potwierdzeniu; suma spada o jej czas. Osobno: usunięcie przedmiotu z sesjami. | Przypadkowe usunięcie. | Dialog „Usunąć sesję? Nie można tego cofnąć.” |

**Walidacje techniczne:** (1) `czas_min` to liczba całkowita z zakresu 1–1440 (BR-01; dodatkowo `CHECK (czas_min > 0)` w bazie); (2) `data` to istniejąca data `YYYY-MM-DD` (odrzucamy np. `2026-02-30`), a temat jest niepusty (do 200 znaków).

**Reguła biznesowa (BR-02):** suma godzin przedmiotu w okresie `[od, do]` to suma `czas_min` sesji z `od ≤ data ≤ do` (granice włącznie), dzielona przez 60 i zaokrąglona do 2 miejsc dopiero przy wyświetlaniu.

**Różnica:** walidacja techniczna sprawdza formę lub zakres pojedynczej wartości, niezależnie od domeny, i zwykle da się ją zapisać jako ograniczenie bazy. Reguła biznesowa wynika z sensu aplikacji.

**Polityka usuwania:** usunięcie przedmiotu z sesjami zostawiłoby sesje bez rodzica albo po cichu skasowało historię nauki. SQLite oferuje `NO ACTION`, `RESTRICT`, `CASCADE` i `SET NULL` [4]. Wybieram `ON DELETE RESTRICT`: `CASCADE` odrzucam, bo aplikacja jest lokalna i sesje to jedyna kopia danych. `SET NULL` odpada, bo `parent_id` jest `NOT NULL`. Klucze obce są w SQLite domyślnie wyłączone i dotyczą pojedynczego połączenia, więc po otwarciu bazy wykonuję `PRAGMA foreign_keys = ON` [4].

## T5. Trwałość, błędy i jakość rozwiązania

1. Porównaj dwa możliwe sposoby przechowywania danych dla niewielkiej aplikacji CRUD (np. magazyn klucz–wartość i relacyjna baza lokalna). Nie zakładaj, że oba są równie dobre dla wszystkich danych.
2. Zaprojektuj test manualny, który **udowodni trwałość** danych po zamknięciu i ponownym uruchomieniu aplikacji.
3. Opisz zachowanie aplikacji przy nieudanym zapisie oraz przy próbie wprowadzenia danych niepoprawnych.
4. Wskaż, co byłoby trudne do przetestowania, gdyby logika biznesowa znajdowała się wyłącznie w kodzie widoku.

| Kryterium                   | Rozwiązanie A: AsyncStorage (klucz–wartość) | Rozwiązanie B: expo-sqlite (baza relacyjna) |
| --------------------------- | ------------------------------------------- | ------------------------------------------- |
| Obsługa relacji             | Brak; relację przedmiot do sesji odtwarza kod, nic nie pilnuje istnienia przedmiotu. | Klucz obcy z `ON DELETE RESTRICT` blokuje usunięcie przedmiotu z sesjami [4]. |
| Złożoność implementacji     | Niska na start (zapis/odczyt po kluczu), rośnie z danymi: kolekcję trzeba wczytać, zmienić i zapisać. | Wyższa na start (schemat, SQL, repozytorium), ale transakcje cofają nieudane zapisy [3]. |
| Aktualizacja i wyszukiwanie | Zmiana jednej sesji nadpisuje całą wartość. | `UPDATE`/`DELETE` po `id`; `BETWEEN`, wyszukiwanie, sortowanie i `SUM` w SQL. |
| Ograniczenia                | Prosty magazyn na małe dane, nieszyfrowany [9], brak typów i kluczy obcych. | Większy narzut początkowy. `execAsync` nie zabezpiecza przed SQL injection [3]

Oba rozwiązania nie nadają się równie dobrze do wszystkich danych. A pasuje do drobnych ustawień (np. ostatnio wybrany okres) [9], a dla relacji 1 do N, filtrowania po okresie i blokady usuwania właściwe jest B.

**Test trwałości:**
1. Zacząć z pustą bazą (czysta instalacja lub wyczyszczone dane aplikacji).
2. Dodać przedmiot „Bazy danych”, cel 10 h.
3. Dodać sesje: 02.11.2026 / 90 min / „Normalizacja”, 05.11.2026 / 45 min / „Indeksy”, 20.11.2026 / 60 min / „Transakcje”.
4. Wybrać okres 01.11–15.11.2026: suma **2,25 h** (22,5% celu), sesja z 20.11 niewliczona.
5. Spróbować zapisać sesję z czasem `0`: komunikat błędu, brak nowego rekordu.
6. Całkowicie zamknąć aplikację; opcjonalnie zrestartować urządzenie.
7. Uruchomić ponownie i otworzyć „Bazy danych”.
8. Oczekiwane: przedmiot i dokładnie trzy sesje z tymi samymi wartościami, suma dla 01.11–15.11 nadal 2,25 h, brak sesji z czasem `0`, a próba usunięcia przedmiotu daje komunikat o 3 sesjach. Dane przeszły pełny restart procesu, więc nie pochodzą ze stanu w pamięci. Baza SQLite przetrwa restart aplikacji [3], a test sprawdza, że aplikacja z tego korzysta.

**Obsługa błędów i testowalność:**
- Nieudany zapis: zapis idzie w transakcji, a nieudana jest cofana [3]. Store zmienia się dopiero po sukcesie.
- Dane niepoprawne: domena odrzuca je przed wywołaniem bazy. `CHECK` i `FOREIGN KEY` działają jako zabezpieczenie.
- Logika tylko w widoku: BR-01, BR-02 i blokadę usuwania można by sprawdzić jedynie przez uruchomienie całego ekranu (renderowanie, symulacja wpisywania, często prawdziwa baza). Logikę z efektami ubocznymi w renderowaniu trudno też uruchamiać powtarzalnie, bo UI może być wołane wielokrotnie i w dowolnej kolejności [10].

## Źródła i samodzielność opracowania

Podaj **co najmniej trzy źródła**, w tym **minimum dwa fragmenty oficjalnej dokumentacji** (tytuł, adres, data dostępu). Wskaż, do którego zadania każde źródło zostało wykorzystane (w tym AI, jeśli użyto).

1. React, *Reacting to Input with State*, https://react.dev/learn/reacting-to-input-with-state. T1, T2, T3.
2. React, *Sharing State Between Components*, https://react.dev/learn/sharing-state-between-components. T3.
3. Expo, *SQLite*, https://docs.expo.dev/versions/latest/sdk/sqlite/. T4, T5 (transakcje, trwałość, `execAsync`).
4. SQLite, *Foreign Key Support*, https://www.sqlite.org/foreignkeys.html. T4, T5.
5. MDN, *Ajax*, https://developer.mozilla.org/en-US/docs/Glossary/AJAX. T1.
6. MDN, *SPA*, https://developer.mozilla.org/en-US/docs/Glossary/SPA. T1.
7. Jest, *Getting Started*, https://jestjs.io/docs/getting-started. T3, T5.
8. React Native, *Core Components and Native Components*, https://reactnative.dev/docs/intro-react-native-components. T1.
9. Expo, *Store data*, https://docs.expo.dev/develop/user-interface/store-data/. T5.
10. Android Developers, *Thinking in Compose*, https://developer.android.com/develop/ui/compose/mental-model. T1, T2, T5.
11. Expo, *Introduction to Expo Router*, https://docs.expo.dev/router/introduction/. T3.

**Użycie AI** Model Opus 5.5 został użyty do: znalezienia źródeł, streszczenia źródeł oraz redakcji odpowiedzi.

**Które stwierdzenie z materiałów źródłowych zweryfikowano w innym źródle i dlaczego?** Przyjąłem tezę, że klucze obce w SQLite trzeba włączać ręcznie i że `RESTRICT` blokuje usunięcie przedmiotu z sesjami. Sprawdziłem ją w specyfikacji SQLite [4], bo dokumentacja Expo [3] opisuje API biblioteki, a nie semantykę kluczy obcych. Opis AsyncStorage (nieszyfrowany magazyn klucz–wartość na małe dane) oparłem na stronie Expo [9], bo strona instalacyjna biblioteki go nie opisuje, i dlatego nie podaję limitu rozmiaru.
