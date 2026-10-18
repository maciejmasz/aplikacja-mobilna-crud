# Sprawozdanie 1 – Budowa aplikacji mobilnych z użyciem technologii frontendowych

**Imię i nazwisko studenta: Maciej Masztalerz**  
**Grupa: 43**  
**Data: 18.10.2026**  


## Temat zajęć
**Główne technologie i frameworki frontendowe do budowy aplikacji mobilnych**

Omówienie kluczowych technologii frontendowych wykorzystywanych w tworzeniu aplikacji mobilnych, takich jak **React Native**, **Flutter**, **NativeScript**.  
Porównanie i kontrastowanie tych technologii pod kątem **wydajności**, **łatwości użycia** i **wsparcia społeczności**.  
Omówienie najważniejszych **narzędzi deweloperskich** i **systemów kontroli wersji** (VS Code, Android Studio, Xcode, Git).


## Cele ćwiczenia
1. Zrozumienie różnic między frameworkami React Native, Flutter i NativeScript.  
2. Poznanie podstawowych narzędzi deweloperskich wykorzystywanych w tworzeniu aplikacji mobilnych.  
3. Poznanie zasad pracy zespołowej z wykorzystaniem systemu Git.  
4. Wyciągnięcie wniosków dotyczących doboru technologii do konkretnego typu projektu mobilnego.


## Pytania kontrolne i odpowiedzi

### Technologie i frameworki frontendowe

1. **Jakie są główne różnice między React Native, Flutter i NativeScript?**  
_Odpowiedź:_ React Native uruchamia JavaScript/React w silniku Hermes. Komponenty mapuje na prawdziwe widoki przez renderer C++ (Fabric) i interfejs JSI.
Flutter kompiluje Darta do natywnego kodu maszynowego. Każdy piksel rysuje sam własnym silnikiem Impeller
NativeScript uruchamia JavaScript w osadzonym silniku V8. Cały natywny SDK udostępnia w JS dzięki metadanym generowanym podczas budowania, a interfejs to prawdziwe natywne widoki sterowane bezpośrednio z JS.

2. **Jak każdy z tych frameworków renderuje interfejs użytkownika na urządzeniach mobilnych?**  
_Odpowiedź:_ React Native: kod JS buduje drzewo elementów. Fabric zamienia je na drzewo w C++, Yoga oblicza układ, diff w C++ wyznacza zmiany, a wątek UI nanosi je na prawdziwe obiekty UIView/android.view.View.
Flutter: widżety zamieniają się w elementy, a potem w obiekty renderujące.
NativeScript: @nativescript/core tworzy natywne widoki, wywołując z JS API platformy na wątku UI. Nie ma osobnej warstwy renderującej.

3. **Kiedy warto wybrać Fluttera, a kiedy React Native lub NativeScript?**  
_Odpowiedź:_ Flutter jesli chce się identycznego, własny interfejs na każdej platformie, dużo animacji lub grafiki, albo platformy poza mobilnymi (web i desktop).
React Native jeśli zna się Reacta, chce się natywnego wyglądu i zachowania platformy oraz dużego ekosystemu JavaScript.
NativeScript jeśli potrzeba bezpośredniego dostępu z JS do dowolnego natywnego API, także najnowszych API systemu, bez pisania pluginów, albo chce się używać Angulara, Vue, Svelte lub Solid.

4. **Jakie są mocne i słabe strony React Native w porównaniu do Fluttera?**  
_Odpowiedź:_ React Native używa prawdziwych kontrolek platformy i JavaScriptu/Reacta, a dzięki JSI szybko komunikuje się z kodem natywnym. Jego słabością jest jeden wątek JS, który przy ciężkim renderowaniu gubi klatki. Flutter kompiluje Darta do kodu maszynowego i sam rysuje cały interfejs, co daje wysoką wydajność i identyczny wygląd wszędzie. Płaci za to kontrolkami, które nie są systemowe, oraz większym rozmiarem aplikacji.

5. **Jak Flutter radzi sobie z wydajnością w porównaniu do innych rozwiązań hybrydowych?**  
_Odpowiedź:_ Dokumentacja Fluttera argumentuje, że unika on narzutu frameworków, które uruchamiają interpretowany JS i komunikują się z bibliotekami UI platformy. Używa kodu maszynowego, renderuje bez przełączania się do kodu platformy i liczy układ w czasie O(n). Jego słabe punkty to pojedynczy główny izolat, serializacja w platform channels, koszt osadzania widoków platformy i rozmiar aplikacji. Inne, nienatywne rozwiązania korzystają z mostków lub widoków webowych, ograniczając wydajnośc.

6. **Jak NativeScript integruje się z natywnymi API urządzenia?**  
_Odpowiedź:_ Podczas budowania generator metadanych odczytuje każde publiczne API z SDK i pluginów do pliku binarnego dołączanego do aplikacji. W czasie działania wywołania JS są wyszukiwane w tych metadanych, a wartości konwertowane. Wywołanie idzie potem prosto przez JNI na Androidzie lub FFI na iOS. Natywne obiekty wracają jako wrappery JS, a natywne klasy można w JS rozszerzać i implementować ich interfejsy.

7. **Który framework ma obecnie największe wsparcie społeczności i dlaczego?**  
_Odpowiedź:_  Flutter ma na tysiące pluginów w pub.dev i wsparcie Google. React Native ma wsparcie Meta i dostęp do ekosystemu npm/React. NativeScript należy do OpenJS Foundation i wygląda na mniejszy. Wydaje się ze Flutter ma najwieksze wsparcie społeczności.

8. **W jakich przypadkach lepiej wybrać framework natywny zamiast hybrydowego?**  
_Odpowiedź:_  Silna zależność od UI specyficznego dla platformy lub najnowszych kontrolek systemu.
Złożone komponenty natywne, które trzeba by osadzić, jak mapy czy WebView.
Praca krytyczna wydajnościowo, której nie da się wynieść poza główny wątek.
Bardzo ostre limity rozmiaru aplikacji lub czasu uruchamiania.

9. **Jakie języki programowania są używane w poszczególnych frameworkach?**  
_Odpowiedź:_ W React Native to JavaScript/TypeScript oraz Kotlin/Java i Swift/Objective-C dla modułów natywnych. Rdzeń jest w C++.
Flutter to Dart. Embeddery napisane są w Java/C++, Swift/Objective-C i C++, a silnik w C++. Przez FFI można wywoływać C, Rusta lub Go.
NativeScript to JavaScript/TypeScript, opcjonalnie Kotlin/Java i Swift/Objective-C dla kodu natywnego.


10. **Jak wygląda proces budowania i publikacji aplikacji w każdym z frameworków?**  
_Odpowiedź:_ Wszystkie trzy tworzą standardowe paczki do sklepów (AAB/APK dla Google Play, IPA dla App Store)
Flutter: flutter build appbundle / flutter build ipa. --analyze-size mierzy rozmiar, a --split-debug-info go zmniejsza.
React Native: buildy release przez Gradle i Xcode lub CocoaPods albo przez Expo. Dokumentacja podkreśla, by testować wydajność w trybie release, a nie dev.
NativeScript: CLI ns (ns run, potem build release). Metadane generowane są podczas budowania, a --compileSdk określa, jaki poziom API Androida obejmują.


### Techniki i narzędzia programistyczne

11. **Jakie funkcje Visual Studio Code są szczególnie przydatne w tworzeniu aplikacji mobilnych?**  
_Odpowiedź:_  Możliwośc instalowania wszechstronnych rozszerzeń dających wsparcie dla technologii moblinych, pozwalających na debugowanie programów w czasie rzeczywistym. Wbudowany terminal również ułatwia pracę. VSC jest również bardzo lekkim w wykorzystaniu zasobów programem, wieć pozwala na pisanie kodu wszędzie, a poźniejsze jego uruchomienie już w emulatorze.

12. **Czym różni się Android Studio od Xcode?**  
_Odpowiedź:_  Android Studio to oficjalne IDE (integrated development enviorment), służące do programowania aplikacji działających na systemie Android. Xcode to jego odpowiednik dla środowiska Apple, bo służy do pisania aplikacji działających na wszystkich (nie tylko na iOS, ale również na macOS, watchOS, itd.) systemach operacyjnych urządzeń firmy Apple. 

13. **Jakie rozszerzenia VS Code usprawniają pracę z React Native lub Flutterem?**  
_Odpowiedź:_ React Native Tools i oficjalne rozszerzenie Flutter to rozszerzenia pozwalające na kompilowanie, uruchamianie i debuggowanie programów napisanych z użyciem tych technologii. Oprócz tego dodają wsparcie dla Syntaxu, otwieranie projektów i ułatawiają edytowanie i naprawianie kodu.

14. **Jak przebiega proces debugowania aplikacji mobilnej w VS Code lub Android Studio?**  
_Odpowiedź:_  Proces debugowania polega na przerwaniu działania aplikacji w konkretnym momencie, i analizie wartości zmiennych i stosu wywołań. Gdy debugger natrafi na ustawiony wcześniej punkt przerwania (Program będzie działał do tego momentu), zatrzyma działanie i wyrzuci wartości zmiennych w danym momenice i stos wywołań adresów pamięci. Pozwala równiez na testowanie konkretnych fragmentów kodu w czasie działania aplikacji.

15. **Jakie są zalety używania emulatorów i symulatorów w procesie testowania aplikacji?**  
_Odpowiedź:_  Pozwalają na obserwacje rzeczywistego działania programu i testowanie aplikacji bez potrzeby wykorzystywania fizycznych urządzeń mobilnych.

16. **Jakie problemy mogą wystąpić przy konfiguracji środowiska deweloperskiego i jak je rozwiązać?**  
_Odpowiedź:_  Mogą wystąpić konflikty między wersjami oprogramowania oraz ograniczenia sprzętowe w wirtualizacji emulatorów. Należy dbać o aktualną wersję SDK środowiska, upewnienie się że IDE ma do niego dostęp, oraz, że licencje SDK są zaakceptowane. Jeśli chodzi o emulator, nalezy się upewnić, czy wirtualizacja jest poprawnie włączona, oraz czy komputer posiada wystarczająco dużo zasobów by utrzymać emulator. 

17. **Jak działa system kontroli wersji Git i dlaczego jest istotny w projektach zespołowych?**  
_Odpowiedź:_  Git pozwala na śledzenie zmian w plikach i podział pracy w pracy grupowej na wielu programistów wprwadzających jednocześnie zmiany za pomocą branchowania. Nie wymaga stałego połączenia z internetem, bo lokalnie na dysku znajduje się kopia repozytorium, które póżniej może zostać wysłane do chmury po wprowadzeniu zmian. Jest istotny w projektach zespołowych, bo pokazuje pełną historię prac, pozwala na równocześna pracę wielu programistom, jest niezależny od połączenia internetowego, a serwisy takie jak GitHub czy GitLab ułatwiają wprowadzanie zmian i code review.

18. **Jakie korzyści daje korzystanie z GitHub lub GitLab w pracy zespołowej?**  
_Odpowiedź:_ To jest workflow pozwalający na efektywną pracę w zespole. Branchowanie pozwala na podział obowiązków i dostępów, pull requesty pozwalają na planowanie i rozkładanie pracy w zespole i efektywną komunikacje między członkami zespołu. Code rewiev Pozwala na wychytywanie błedów i edytowanie kodu przed wprowadzeniem jakichkolwiek zmian do rzeczywsitego programu.

19. **Na czym polega workflow oparty o branchowanie, pull requesty i code review?**  
_Odpowiedź:_ Tworzenie branchy izoluje zmiany kodu w jednym miejscu, a póżniejsze pull request i code review pozwala na wychwycenie błedów i wprowadzanie zmian w odizolowanym kodzie przed wprowadzeniem zmian w głównym, produkcyjnym branchu.

20. **Jak zintegrować Git z procesem CI/CD w projekcie mobilnym?**  
_Odpowiedź:_  Należy połączyć zdarzenia w repozytorium (push, pull requesty itd.) ze skryptami budującymi lub testującymi aplikację. Środowisko uruchamiające skrypty musi obsługiwać SDK dla środowiska mobilnego. 


## Wnioski końcowe

Podsumuj najważniejsze wnioski z ćwiczenia:  
- Który framework wybrałbyś do własnego projektu i dlaczego?  
- Jakie narzędzia najbardziej ułatwiły pracę?  
- Jakie problemy napotkałeś i jak je rozwiązałeś?

_Odpowiedź:_  Prawdopodbnie użyję React Native z Expo. Opiera się na JavaScripcie/TypeScripcie i React, więc korzysta z języków które już dobrze znam ze względu na zajęcia w poprzednim semestrze, oraz są to technologie typowo frontendowe. Z jednego kodu powstaje aplikacja na iOS i Androida, a dzięki Expo Go i buildom w chmurze można testować i budować wersję na iPhone’a bez komputera z macOS. Wymagania karty biznesowej realizuje się w nim za pomocą, np. expo-sqlite zapewniając trwały zapis i relację z blokadą usuwania przedmiotu z sesjami, FlatList z zapytaniami SQL obsługując listę, wyszukiwanie, filtrowanie i sortowanie, a Jest pozwala napisać wymagane testy reguły BR-01.

Zważając na to, że na dzisiejszych laboratoriach skupilem się na przeprowadzeniu researchu i porównaniu teoretycznego techonologii, nie jestem w stanie powiedzieć jakie narzędzia ułatwiły mi pracę praktyczną, bo takiej nie wykonałem. Jeśli chodzi o pomoc w badaniu tematu, wykorzystałem oficjalną dokumentację trzech wcześniej wymienonych technologii i model Claude, który przygotował dla mnie charakterystykę na podstawie dokumentacji każdej z tych technologii. Porównanie wykonałem samodzielnie. Resztę informacji zdobyłem wyszukiwaniami w Google, zasobami na subredditach traktujących o konktretnych technologiach i wiedzą własną. Nie napotkałem większych problemów podczas dzisiejszej pracy. Resztę zadań, w tym sprawozdanie LAB-01 wykonam w późniejszym terminie między zajęciami. W tej chwili istnieje działające repo na GitHubie, do którego Prowadzący ma dostęp, oraz to sprawozdanie które zostanie wrzucone na platformę Moodle, na podstawie którego wykonam technology-comparision i decision.