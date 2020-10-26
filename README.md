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
- referencją lvalue do obiektu lub funkcji (poza zakresem tej instrukcji)
- wskaźnikiem do obiektu lub funkcji (poza zakresem tej instrukcji)
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

---

<sup>1</sup> Nie jest to do końca prawda, istnieją sposoby nakładania więzów na typy, którymi parametryzujemy szablon.
Zainteresowane osoby odsyłamy do haseł: SFINAE (dawniej), `if constexpr` (C\+\+17) oraz *concepts* (C\+\+20).
Techniki te leżą jednak poza zakresem bieżących zajęć.

## Szablony funkcji

### Dedukcja typów argumentów

### Uniwersalne referencje

## Szablony aliasów

## Wybrane szablony z STL

### `std::unique_ptr`

### `std::shared_ptr`

### `std::reference_wrapper`

### `std::variant`

### `std::visit`
