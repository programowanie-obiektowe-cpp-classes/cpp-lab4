# Instrukcja IV
## Wstęp
Na dzisiejszych zajęciach zajmiemy się szablonami (template'ami) klas oraz funkcji (od C\+\+14 istnieją także szablony zmiennych, ale zaznajomienie się z nimi pozostawiamy dla chętnych).
Template'y stanowią fundament C\+\+ oraz są głównym powodem, dla którego język ten nie jest tylko "C z klasami" (choć stwierdzenie to można znaleźć w wielu miejscach w sieci).
Ich obecność pozwala na pisanie generycznego kodu o maksymalnie szerowkiej gamie zastosowań.
Przykładem takiego podejścia jest sama biblioteka standardowa (STL - *Standard Template Library*), w której nie znajdziemy prawie żadnych funkcji i klas, lecz *szablony* funkcji i klas.
Szablony definiujemy zgodnie z następującą składnią:
```C++
template< /* lista parametrów */ >
// Tutaj normalna definicja klasy/funkcji/aliasu/obiektu(C++14), wewnątrz której korzystamy z parametrów
```
Dalej możemy korzystać ze zdefiniowego szablonu w następujący sposób:
```C++
/* nazwa szablonu */ < /* konkretne argumenty zgodne z rodzajem zadeklarowanych parametrów */ >
// Powyższa linijka jest nazwą klasy/funkcji/etc., z której możemy korzystać jak z każdej innej klasy/funkcji/etc.
```
Dzięki zastosowaniu template'ów możemy zdefiniować ciało danej klasy/funkcji/etc. tylko raz, a następnie instancjonować dany szablon dla dowolnych (zgodnych z deklaracją) parametrów, w zależności od potrzeby.
Podkreślmy, że klasą/funkcją/etc. jest dopiero instancja szablonu, nie sam szablon.
Proces instancjonowania template'ów odbywa się w czasie kompilacji, także możemy mieć pewność, że wykorzystanie tej funkcjonalności języka nie pociąga za sobą żadnego kosztu w wydajności programu.

## Rodzaje parametrów szablonów
Zanim napiszemy pierwszy szablon, powiedzmy, jakie może on w ogóle mieć rodzaje parametrów.
Ich pełną listę możemy oczywiście znaleźć [w dokumentacji](https://en.cppreference.com/w/cpp/language/template_parameters), tutaj ograniczymy się do dwóch najważniejszych: typów oraz parametrów niebędących typami (tłumaczenie z angielskiego jest niestety mało wdzięczne).

### Typ jako parametr
Szablony w C\+\+ mogą być sparametryzowanane typami, zgodnie ze składnią
```C++
template<typename T /* ... */>
/* definicja szablonu */
```
Zamiast `typename` możemy zamiennie użyć `class`, `typename` jest jednak zgodne z powszechną konwencją.
Wszędzie, gdzie w definicji danego sparametryzowanego bytu występuje typ, możemy teraz użyć `T`.
Możemy zatem użyć `T` m.in. jako:
- typ pola klasy
- typ argumentu funkcji (w tym metody klasy)
- typ zwracany przez funkcję
- argument innego szablonu (template'y możemy dowolnie zagnieżdżać)

Wyobraźmy sobie teraz, że mamy napisać funkcję, która przyjmuje 2 liczby i zwraca ich sumę.
Gdyby nie template'y, musielibyśmy pisać osobną funkcję dla `int`ów, `double`'i, `float`ów, `bool`i itd.
Teraz wystarczy, że napiszemy jeden szablon, sparametryzowany typem argumentu i odpowiednio go zainstancjonujemy (jak dowiemy się niebawem jawne podanie parametrów szablonu nie będzie konieczne).
Przykład ten stanowi jedynie wierzchołek góry lodowej zastosowań szablonów w C\+\+.

### NTTP
Parametrem szablonu może być także byt inny niż typ.
W języku angielskim mówimy *non-type template parameter*, w dalszej części tego tekstu korzystać będziemy właśnie ze skrótu NTTP.
NTTP mogą być:
- typy całkowite (`int`, `char`, `bool`, etc.)
- enumeracje (`enum`)
- referencje lvalue do obiektu lub funkcji (poza zakresem tej instrukcji)
- wskaźniki do obiektu lub funkcji (poza zakresem tej instrukcji)
- `std::nullptr_t` (poza zakresem tej instrukcji)
- od C\+\+20 wymagania NTTP zostały poluzowane, dopuszczane są teraz także typy strukturalne i zmiennoprzecinkowe (zdecydowanie poza zakresem tej instrukcji)

NTTP dają szablonowi dostęp do wartości liczbowych (lub im podobnym) *w czasie kompilacji*.
W konsekwencji w pełni legalna jest np. statyczna alokacja pamięci uzależniona od liczby całkowitej będącej parametrem szablonu (`int tab[N]` jest w tym kontekście dopuszczalne).

## Szablony klas
Możemy teraz napisać nasz pierwszy szablon klasy.
Zacznijmy od rzeczy trywialnych.

#### Zadanie 1
Napisz szablon klasy `Para`, który trzyma 2 obiekty typu, którym jest sparametryzowany.

Szablonu `Para` możemy teraz użyć wszędzie tam, gdzie potrzebujemy trzymać razem parę obiektów pewnego typu.

#### Zadanie 2
Dodaj do szablonu klasy `Para` metodę `suma`, która zwraca sumę trzymanych obiektów (użyj operatora `+`)

Widzimy już pierwszy problem towarzyszący szablonom - aby napisany kod był poprawny, dla typu, którym zainstancjonujemy szablon, musi istnieć zdefiniowany poprawnie operator `+`.
W czasie pisania szablonu nie mamy kontroli nad typem, który zostanie do tego użyty<sup>1</sup>.
Na szczęście wszystkie błędy tego typu zostaną wykryte w czasie kompilacji.

Przećwiczmy także klasy sparametryzowane NTTP.

#### Zadanie 3
Napisz szablon klasy `TablicaPar` sparametryzowany 1 typem oraz 1 NTTP typu `unsigned int`.
Niech przechowuje ona statycznie zaalokowaną tablicę obiektów typu `Para<T>` o długości `N`, gdzie `T` i `N` to parametry klasy `TablicaPar<T, N>`.

#### Zadanie 4
Przeciąż dla klasy `TablicaPar` operator `[]` tak, aby umożliwić indeksowanie po trzymanej tablicy.

#### Zadanie 5
Przećwicz działanie napisanych szablonów na typie `double` (np. wypełnij tablicę a następnie policz sumę wszystkich trzymanych par liczb).
Teraz wykonaj to samo zadanie dla typu `int`.
W ilu miejscach musiałaś/musiałeś zmodyfikować kod?

### Specjalizacje szablonów klas
Jeżeli chcemy, aby nasz szablon zachowywał się w szczególny sposób dla jakiejś grupy parametrów, możemy dodać do niego specjalizację.
Klasy specjalizujemy wg. następującego schematu:
```C++
// Definicja
template</* parametry */>
class Klasa { /* ... */ };

// Specjalizacja
template</* lista parametrów specjalizacji, w szczególności może być pusta */>
class Klasa</* konkretne typy/wartości/etc. wynikające z parametrów specjalizacji */>
{ /* ... */ };
```

Pokażmy to na konkretnym przykładzie:
```C++
template <typename T>
struct S {
    void print() { puts("Szablon ogólny"); }
};

template <>
struct S<double> {
    void print() { puts("Specjalizacja dla double"); }
};
```
#### Zadanie 6
Skopiuj powyższy kod i stwórz w funkcji `main` obiekty typu `S<int>` i `S<double>`.
Zweryfikuj, że metoda `print` działa zgodnie z oczekiwaniami.

Nie musimy jednak specjalizować klas dla konkretnych parametrów.
Zwróćmy uwagę, że sama specjalizacja także posiada listę parametrów, którą możemy wykorzystać.
Na przykład:
```C++
// Ogólna definicja
template <typename T>
struct S { /* ... */ };

// Specjalizacja dla wskaźników
template <typename T>
struct S<T*> { /* ... */ };

// Specjalizacja dla referencji
template <typename T>
struct S<T&> { /* ... */ };
```
Zauważmy też, że nie musimy podawać ogólnej definicji szablonu, wystarczy ogólna *deklaracja*.
W takim wypadku, gdy spróbujemy zainstancjonować szablon dla parametrów, które nie pasują do żadnej z jego specjalizacji, nasz program się nie skompiluje.
Jest to swego rodzaju sposób nakładania więzów na szablony (choć niezbyt elegancki, *vide* przypis<sup>1</sup>).
Możemy także postąpić odwrotnie: deklarując specjalizację bez definicji "wyłączamy" ją.

#### Zadanie 7
Zadeklaruj specjalizację dla klasy `TablicaPar`, która "wyłączy" puste tablice (czyli klasy `TablicaPar<T, 0>` dla każdego `T`).

---

<sup>1</sup> Nie jest to do końca prawda, istnieją sposoby nakładania więzów na typy, którymi parametryzujemy szablon.
Zainteresowane osoby odsyłamy do haseł: SFINAE (dawniej), `if constexpr` (C\+\+17) oraz koncepty (*concepts*, C\+\+20).
Techniki te wykraczają jednak poza zakres bieżących zajęć.

## Szablony funkcji
Szablony funkcji definiujemy zgodnie z tym samym schematem, co szablony klas.
Główne różnice stanowią fakt, że nie możemy ich specjalizować (tę rolę spełnia przeciążanie funkcji) oraz dedukcja typów argumentów<sup>2</sup> (opisana niżej).
Główną siłą szablonu funkcji jest fakt, że można wykorzystać jego parametry jako typy argumentów (lub wartości zwracanej).
Możemy zatem napisac w jednym miejscu dowolnie skomplikowaną implementację pewnego algorytmu działającego na argumentach nie konkretnego typu, ale całej *rodziny typów*, spełniającej jakieś minimalne założenia tej implementacji.
Na przykład, pisząc funkcję
```C++
template<typename T>
T add(const T& a, const T& b)
{ return a + b; }
```
jesteśmy przy jej pomocy w stanie dodać 2 obiekty każdego typu należacego do rodziny typów, dla których zdefiniowany jest operator `+` zwrcający obiekt tego samego typu co jego argumenty.
Działa więc ona równie dobrze dla typu `double`, jak dla typu `Wektor2D` z pierwszego laboratorium.
Jest to swego rodzaju statyczny polimorfizm - mamy wspólny interfejs dla różnych klas.
Jeżeli spróbujemy zainstancjonować szablon z typem nie spełniającym naszych założeń, nasz program się nie skompiluje.

#### Zadanie 8
Napisz funkcję `iloczyn`, która przyjmuje tablicę typu, którym jest sparametryzowana oraz liczbę całkowitą będącą rozmiarem tablicy.
Niech zwraca ona iloczyn elementów tej tablicy, liczony operatorem `*`.
Zastanów się, jakie założenia czynisz na temat typu tablicy?

### Dedukcja typów argumentów
Napisaną powyżej funkcję możemy zawołać np. w następujący sposób:
```C++
int tab[]    = {1, 2, 3};
int silnia_3 = iloczyn<int>(tab, 3);
```
W drugiej linijce jawne podanie parametru funkcji `iloczyn` jest niepotrzebne.
C\+\+ jest statycznie typowany, a zatem podanie `tab` jako argumentu jednoznacznie determinuje parametr, z jakim ma zostać zainstancjonowany szablon.

#### Zadanie 9
Napisz wolnostojącą funkcję `sumaPary`, która przyjmuje parę (w znaczeniu szablonu `Para` napisanego wyżej) obiektów typu, którym jest sparametryzowana i zwraca ich sumę (użyj metody `suma`).
Stwórz parę liczb całkowitych i policz ich sumę przy użyciu funkcji `sumaPary`.
Ile razy musiałaś/musiałeś użyć słowa kluczowego `int`?
Dzięki dedukcji typów argumentów odpowiedź powinna wynosić 1!

---

<sup>2</sup> Od C\+\+17 istnieje dedukcja parametrów typu obiektu na podstawie typu argumentów jego konstruktora (CTAD), jednak temat ten wykracza poza zakres bieżącego kursu.

## Wybrane szablony z STL
W tej części instrukcji pokażemy działanie kilku podstawowych szablonów biblioteki standardowej.
Pierwsze 4 dotyczą tzw. *smart pointers*, czyli klas, które pozwalają nam korzystać ze wskaźników w prostszy i bezpieczniejszy sposób: `std::unique_ptr` i `std::shared_ptr`.
Istnieje także 3. rodzaj smart pointera - `std::weak_ptr`, lecz zaznajomienie się z nim pozostawiamy dla chętnych.
Dalej poznamy `std::variant` i `std::visit`, które pozwolą nam drastycznie uprościć kod z zajęć dotyczących polimorfizmu.
Dodajmy, że celem tego rozdziału nie jest nauczenie czytelnika każdego niuansu omawianych szablonów (po takowe odsyłamy do dokumentacji), tylko przedstawienie ich filozofii i podstaw użytkowania, tak, aby w przyszłości czytelnik wiedział po jakie rozwiązanie sięgnąć w obliczu konkretnego problemu.

### [`std::unique_ptr`](https://en.cppreference.com/w/cpp/memory/unique_ptr)
Klasa `std::unique_ptr<T>` to smart pointer ("inteligentny wskaźnik") posiadający wyłączną własność nad zasobem typu `T` i niszczy ten zasób w swoim destruktorze (zakres życia zasobu jest ograniczony zakresem życia smart pointera).
Wypunktujmy najważniejsze cechy tego szablonu:
1. Jeden z konstruktorów `std::unique_ptr<T>` przyjmuje obiekt typu `T*` i zarządza zasobem, na który wskazuje podany wskaźnik.
Od C\+\+14 nie korzystamy z tego konstruktora, lecz zamiast tego z funkcji `std::make_unique<T>`.
2. `std::unique_ptr` posiada konstruktor domyślny, który tworzy obiekt, który niczym nie zarządza.
3. `std::unique_ptr` ma usunięty konstruktor kopiujący i kopiujący operator przypisania.
4. `std::unique_ptr` ma dobrze zdefiniowany konstruktor przenoszący i przenoszący operator przypisania.
Te dwie metody specjalne "przejmują" zasób, którym zarządzał argument konstruktora/operatora przenoszącego.
5. `std::unique_ptr` posiada zdefiniowane operatory `*` oraz `->`, które działają analogicznie jak dla zwykłego wskaźnika.
6. Destruktor `std::unique_ptr` niszczy zasób, którym dany obiekt zarządza.

Szablon klasy `std::unique_ptr` także posiada specjalizację dla typów będących tablicami (`std::unique_ptr<T[]>`), która reprezentuje wyłączną własność nad *tablicą* obiektów.
Działa ona nieco inaczej niż ogólny szablon:
1. `std::unique_ptr<T[]>` nie ma przeciążonych operatorów `*` i `->`.
Zamiast nich posiada operator `[]`, który pozwala na indeksowanie po tablicy, którą zarządza.
2. `std::unique_ptr<T[]>` niszczy trzymane zasoby przy użyciu `delete[]`, a nie `delete` (poprawnie usuwa każdy element tablicy).

Wymieniowne powyżej cechy pozwalają nam korzystać z obiektów `std::unique_ptr` dokładnie tak samo, jak z wbudowanych wskaźników, nie musimy się za to martwić o zwalnianie pamięci.
Dodatkowo mamy pewność, że nigdy nie wykonamy nieumyślnej kopii zasobu, ani nie spróbujemy odnieść się do zasobu, który został zniszczony.
Warto też zaznaczyć, że dynamiczny polimorfizm opisany w instrukcji nr 3 działa w niezmienionej formie dla `std::unique_ptr`!
W konsekwencji, jeżeli mamy istenijący kod, w którym korzystamy z wbudowanych wskaźników, to możemy zamienić deklarację wszystkich `T*` na `std::unique_ptr<T>` oraz usunąć wszystkie zawołania operatora `delete` (pod warunkiem, że wbudowane wskaźniki reprezentowały wyłączną własność).
Taka operacja pozwoli nam skrócić kod (nie musimy wołać `delete`) oraz zagwarantuje nam jego poprawność (nigdy nie zapomnimy już zwolnić pamięci, próba kopiowania wskaźników teraz kończy się błędem kompilacji).
Przyjrzyjmy się, jak może to wyglądać.
Rozważmy następujący kod:
```C++
bool  warunek = sprawdzWarunek();
Baza* wsk_baza;

if (warunek)
    wsk_baza = new Pochodna1{};
else
    wsk_baza = new Pochodna2{};

wsk_baza->metodaWirtualna();
delete wsk_baza;
```
Możemy go przepisać jako:
```C++
bool                  warunek = sprawdzWarunek();
std::unique_ptr<Baza> wsk_baza; // konstruktor domyślny

if (warunek)
    wsk_baza = std::unique_ptr<Pochodna1>{new Pochodna1{}};
else
    wsk_baza = std::unique_ptr<Pochodna2>{new Pochodna2{}};
	
wsk_baza->metodaWirtualna(); // działa dzięki przeciązeniu operatora ->
// nie musimy pamiętać o wołaniu delete, robi to za nas destruktor!
```
W tym przykładzie widzimy, że `std::unique_ptr<KlasaPochodna>` jest konwertowalny na `std::unique_ptr<KlasaBazowa>`.

### [`std::make_unique`](https://en.cppreference.com/w/cpp/memory/unique_ptr/make_unique)
W powyższym przykładzie, mało elegancka mogą wydawać się linijki, w któych tworzymy `std::unique_ptr<PochodnaX>` i przypisujemy je do `wsk_baza`.
Szczęśliwie, od standardu C\+\+14, mamy do dyspozycji szablon funkcji `std::make_unique`.
`std::make_unique<T>(argumenty...)` konstruuje na stercie obiekt typu `T` przy użyciu podanych argumentów<sup>3</sup>, a następnie zwraca `std::unique_ptr<T>` do tego obiektu.
Efektywnie woła on za nas operator `new`.
W konsekwencji, linijkę
```C++
wsk_baza = std::unique_ptr<Pochodna1>{new Pochodna1{}};
```
możemy zamienić na
```C++
wsk_baza = std::make_unique<Pochodna1>();
```
co jest niewątpliwie zwięźlejsze i prostsze w zrozumieniu.
`std::make_unique` jest jednym z szablonów funkcji, przy użyciu których nie używamy dedukcji typów, lecz zawsze jawnie podajemy parametr szablonu funkcji.
Jest to bardzo logiczne - nie jesteśmy w stanie na podstawie typów argumentów stwierdzić typu obiektu, którego konstruktor chcemy zawołać.
Wiele klas może mieć konstruktory, które przyjmują dany zestaw typów!

### [`std::shared_ptr`](https://en.cppreference.com/w/cpp/memory/shared_ptr)


### [`std::make_shared`](https://en.cppreference.com/w/cpp/memory/shared_ptr/make_shared)

### Uwagi nt. smart pointerów

### `std::variant`

### `std::visit`

<sup>3</sup> Mechanizm, który pozwala definiować szablony dla nieznanej *a priori* liczby parametrów, wykracza poza zakres tego kursu.
Zainteresowani mogą szukać hasła *variadic templates*.