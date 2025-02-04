# Nowości w rdzeniu języka

## Nowe typy danych podstawowych

C++11 wprowadza kilka nowych fundamentalnych typów danych:

- typ: `[unsigned|signed] long long [int]`
  - Gwarancja minimum 64 bitów
  - Literał: `LL`

    ```cpp
    long long int i = 123456789012345LL;
    ```

- typy dla znaków UTF16/UTF32: `char16_t`, `char32_t`
- typ dla pustego wskaźnika `nullptr`: `nullptr_t`

## Typy całkowite o znanym rozmiarze

W C++11 wprowadzono aliasy na typy całkowite o znanym rozmiarze:

- `int8_t`, `int16_t`, `int32_t`, `int64_t` - typy całkowite ze znakiem
- `uint8_t`, `uint16_t`, `uint32_t`, `uint64_t` - typy całkowite bez znaku
- `intptr_t`, `uintptr_t` - typy całkowite dla wskaźników mogące przechować wskaźnik na `void`
- `intmax_t`, `uintmax_t` - największe typy całkowite dostępne w danej architekturze

```cpp
#include <cstdint>

int8_t i8 = 127;
uint16_t ui16 = 65535;
int32_t i32 = 2147483647;
uint64_t ui64 = 18446744073709551615ULL;
uintmax_t umax = 18446744073709551615ULL;
```

## nullptr - uniwersalny pusty wskaźnik

Nowe słowo kluczowe - `nullptr`

- wartość dla wskaźników, które na nic nie wskazują (wartość `0`)
- bardziej czytelny i bezpieczniejszy odpowiednik stałej `NULL/0`
- posiada zdefiniowany przez standard typ - `std::nullptr_t` (zdefiniowany w pliku nagłówkowym `<cstddef>`)

```c++
namespace std
{
    typedef decltype(nullptr) nullptr_t;
}
```

Deklaracja pustych zmiennych wskaźnikowych od C++11 powinna wyglądać:

```c++
int* ptr_1 = nullptr; 
assert(ptr_1 == nullptr);

// or

int* ptr_2{}; // p1 is set to 0
assert(ptr_2 == nullptr);
```

`nullptr` rozwiązuje problem z przeciążeniem funkcji przyjmujących jako argument wskaźnik lub typ całkowity:

```c++
void foo(int);

foo(0); // calls foo(int)
foo(NULL); // calls foo(int)
foo(nullptr); // compile-time error


void bar(int);
void bar(void*);

bar(0); // calls bar(int)
bar(NULL); // calls bar(int) if NULL is 0, ambiguous if NULL is 0L
bar(nullptr); // calls bar(void*)
```

## Raw String Literals

W C++11 możemy uniknąć specjalnego traktowania "znaków ucieczki" (*escape characters*) w literałach znakowych stosując tzw. "Raw String Literals".

"Raw string" zaczyna się od `R"(`, a kończy się `)"`.

```c++
std::string no_newlines = R"(\n\n)";
std::string cmd{R"(cd "C:\new folder\text")"};
```

Literały tekstowe mogą zawierać teraz kilka lini:

```c++
std::string with_newlines = R"(Line 1 of text...
Line 2...
Line 3)";
```

Aby mieć możliwość umieszczenia sekwencji `)"` w literale "raw string", należy użyć sekwencji przestankowej *delim*. W rezultacie kompletna składnia literału "raw string" to: `R"delim()delim"`.

- sekwencja przestankowa może mieć długość do 16 znaków i nie może zawierać białych znaków (spacji itp.).

```c++
std::string str1 = R"raw(a\
b\nc()"
)raw";

std::string str2 = "a\\\n    b\\nc()\"\n    ";
```

Literały "raw string" są szczególnie przydatne przy definiowaniu wyrażeń regularnych lub ścieżek w systemie Windows.

```c++
std::regex re1( R"!("operator\(\)"|"operator->")!" );  // "operator()"|"operator->"
```

## Wsparcie dla Unicode

Od C++11 dozwolone są następujące literały znakowe:

- `u8` - definiuje kodowanie UTF-8
- `u` - definiuje literał ze znakami `char16_t`
- `U` - definiuje literał ze znakami `char32_t`

```c++
u"UTF-16 string literal"  // char16_t (UTF-16)
U"UTF-32 string literal"  // char32_t (UTF-32)
u8"UTF-8 string literal"  // char (UTF-8)
```

Kody znaków są podawane w postaci `\unnnn` oraz `\Unnnnnnnn`:

```c++
u8"G clef: \U0001D11E"
u"Euro: \u0024"
U"Skull and bones: \u2620"
```

### Typy łańcuchowe

Standard definiuje cztery typy łańcuchowe:

- `std::string` - specjalizacja szablonu `std::basic_string<char>`
- `std::wstring` - specjalizacja szablonu `std::basic_string<wchar_t>`
- `std::u16string` - specjalizacja szablonu `std::basic_string<char16_t>`
- `std::u32string` - specjalizacja szablonu `std::basic_string<char32_t>`

## Rozszerzone typy wyliczeniowe

Dla typów wyliczeniowych można od C++11 specyfikować typ całkowity, na którym definiowane jest wyliczenie.

Wartości podawane w wyliczeniu muszą mieścić się w dozwolonym zakresie dla typu.

```c++
enum Coffee : std::uint_8_t { espresso, cappucino, latte };

enum State : unsigned char { opened, closed, unknown = 999 }; // error!
```

Typ na bazie którego definiowane jest wyliczenie jest dostępny za pomocą metafunkcji `std::underlying_type_t<EnumType>`:

```c++
#include <type_traits>

using namespace std;

enum Coffee : uint8_t { espresso, cappuccino, latte };

int main()
{
    Coffee coffee = espresso;

    if (std::is_same_v<uint8_t, std::underlying_type_t<Coffee>>)
        std::cout << "Underlying type is uint8_t\n";
}
```

### std::underlying_type_t<Enum>

Funkcja `std::underlying_type_t<Enum>` zwraca typ całkowity, na którym zdefiniowane jest wyliczenie.

```c++
enum Color : uint8_t { red, green, blue };

std::underlying_type_t<Color> int_value = green; // int_value : uint8_t
asssert(int_value == 1);
```

````{note}
W C++23 wprowadzono funkcję `std::to_underlying`, która zwraca wartość wyliczenia w postaci typu całkowitego.

```c++
auto int_value = std::to_underlying(Color::green); // int_value : uint8_t
```

Jej implementacja jest bardzo prosta:

```c++
template <typename Enum>
constexpr auto to_underlying(Enum e) noexcept
{
    return static_cast<std::underlying_type_t<Enum>>(e);
}
```
````

## Wyliczenia silnie typizowane - Scoped Enumerations

Dla wyliczeń silnie typizowanych (*scoped enums*):

- można specyfikować na bazie jakiego typu definiowane jest wyliczenie
  - domyślnym typem na którym definiowane jest wyliczenie jest `int`
- niejawna konwersja do i z typu całkowitego nie jest dozwolona
- nie ma możliwości porównania obiektów różnych typów wyliczeniowych
- wartości wyliczenia są umieszczone w zakresie typu
- istnieje możliwość użycia deklaracji zapowiadającej (*forward declaration*). Gdy używany jest tylko typ nie ma potrzeby rekompilacji przy dodaniu nowej wartości dla wyliczenia.

```c++
enum class Engine : uint8_t;  // forward declaration

Engine e;

enum class Engine : uint8_t { petrol, diesel, wankel };

e = Engine::petrol; // OK

e = diesel;  // error - no "diesel" in scope

e = 3; // error

e = static_cast<Engine>(1); // OK: e == Engine::diesel

int index = Engine::wankel; // error

std::underlying_type_t<Engine> index = 
    static_cast<underlying_type_t<Engine>>(Engine::wankel);
```

Zamiast `enum class` może być użyte `enum struct` - nie ma żadnej semantycznej różnicy między tymi formami.

## Deklaracje zmiennych z auto

Deklaracje zmiennych z użyciem słowa kluczowego `auto` umożliwiają automatyczną dedukcję typu zmiennej przez kompilator. Dedukcja typu odbywa się na podstawie inicjalizatora.

```{note}
- `auto` jest słowem kluczowym, które w C++11 otrzymało nowe znaczenie
- w poprzednich standardach oznaczało zmienną automatyczną (tworzoną na stosie) - praktycznie nigdy nie było używane
```

```c++
auto i = 42; // i : int

auto f = 3.14f; // f : float

std::set<std::string> spellcheck;
auto it = spellcheck.begin(); // it : std::set<std::string>::iterator
auto const_it = spellcheck.cbegin(); // const_it : std::set<std::string>::const_iterator
```

Definiując zmienną z użyciem `auto` można dodawać modyfikatory `const`, `volatile` oraz stosować referencje lub wskaźniki:

```c++
auto i = 10; // i : int
const auto* ptr1 = &i; // ptr1 : const int*
const auto ptr2 = &i; // ptr2 : int* const

double f();
auto r1 = f();  // r1 : double
const auto& r2 = f();  // r2: const double&

const auto& ref_spellcheck = spellcheck; // ref_spellcheck : const std::set<std::string>&
```

### Iteracja po kontenerach z auto

Używając `auto` można wygodnie iterować po kontenerach zlecając kompilatorowi dedukcję typu iteratora.

Aby uzyskać w trakcie iteracji `const_iterator` należy użyć nowych metod z interfejsu kontenerów standardowych `cbegin()` i `cend()`.

```c++
void do_something(int& x);
void print(const int& x);

vector<int> vec = { 1, 2, 3, 4, 5 };

for(auto it = vec.begin(); it != vec.end(); ++it)
{
    do_something(*it); // ok - type of it - vector<int>::iterator
}

for(auto it = vec.cbegin(); it != vec.cend(); ++it)
{
    print(*it); // ok - type of it - vector<int>::const_iterator
}
```

### Mechanizm dedukcji typu dla auto

Mechanizm dedukcji typu dla `auto` jest praktycznie taki sam jak dla parametrów szablonu.

```c++
template <typename T> void f(ParamType t);

f(expr); // dedukcja typu t na podstawie wyrażenia
auto i = expr; // praktycznie ten sam mechanizm dedukcji typu
```

W mechanizmie automatycznej dedukcji typów specyfikator typu `auto` jest odpowiednikiem `ParamType` w mechanizmie dedukcji typów dla szablonów.

Możemy wyróżnić trzy zasadnicze przypadki:

1. Specyfikator typu nie jest ani referencją ani wskaźnikiem.

    - jeśli wyrażenie inicjujące jest referencją - referencja jest usuwana
    - ignorowane są modyfikatory `const` i `volatile`
    - tablice i funkcje rozpadają się do wskaźników (*decay to pointers*)

    ```c++
    int x = 42;
    const int cx = 665;
    int& ref_x = x;
    const int& cref_x = x;

    auto a1 = x;      // a1 : int
    auto a2 = cx      // a2 : int - const is stripped
    auto a3 = ref_x;  // a3 : int - reference is stripped
    auto a4 = cref_x; // a4 : int - reference and const are stripped

    int data[10];  
    auto a5 = data;   // a5 : int* - array decays to pointer

    int foo();
    auto a6 = foo;    // a6 : int(*)() - function decays to pointer
    ```

2. Specyfikator typu jest referencją lub wskaźnikiem.

    - zachowywane są referencje oraz modyfikatory `const` i `volatile`
    - tablice i funkcje nie rozpadają się do wskaźników

    ```c++
    int x = 42;
    const int cx = 665;
    int& ref_x = x;
    const int& cref_x = x;

    auto& a1 = x;      // a1 : int&
    auto& a2 = cx;     // a2 : const int&
    auto& a3 = ref_x;  // a3 : int&
    auto& a4 = cref_x; // a4 : const int&

    int data[10];  
    auto& a5 = data;   // a5 : int(&)[10]

    int foo();
    auto& a6 = foo;    // a6 : int(&)()
    ```

3. Specyfikator typu jest "uniwersalną referencją"

    - jeśli wyrażenie inicjujące jest `l-value` następuje dedukcja to referencji `l-value`
    - jeśli wyrażenie inicjujące jest `r-value` następuje dedukcja to referencji `r-value`

    ```c++
    int x = 42;

    auto&& ax1 = x;   // ax1 : int&

    auto&& ax2 = 42;  // ax2 : int&&
    ```

````{important}
Jedyna różnica między mechanizmem dedukcji typu w szablonach a w `auto` dotyczy  
dedukcji typu z listy inicjalizacyjnej zdefiniowanej za pomocą nawiasów klamrowych `{ 1, 2, 3 }`.

```c++
auto items = { 1, 2, 3 }; // items : std::initializer_list<int>
```
````

### Składnia definicji zmiennych z auto

Dozwolone są dwie składnie inicjalizacji:

- składnia bezpośredniej inicjalizacji

  - `auto var1(expr);`

  - lub `auto var1{expr};`

- składnia kopiująca - `auto var2 = expr;`

```c++
// direct initialization syntax
int i = 10;
auto a = i;
auto b(i); // b : int
auto c{i}; // c : int - since C++17
auto d = {i}; // d : compiler error since C++17
```

**Obie składnie mają takie samo znaczenie w kontekście dedukcji typów.**


````{note}
Jeśli typ dedukowany ma konstruktor kopiujący zdefiniowany jako `explicit`, to kompiluje się tylko składnia bezpośrednia.

```c++
// copy initialization syntax
struct Expl
{
    Expl() {}
    explicit Expl(const Expl&) {}
};

Expl e;
auto a1 = e; // compile error
auto a2{e}; // OK
```
````

## Pętla for dla zakresów - range-based for

Pętla *range-based for* iteruje po wszystkich elementach zakresu.

Wyrażenie

```c++
for(for-range-declaration : for-range-initializer)
{
    statement;
}
```

jest rozwijane do pętli:

```c++
{
    auto &&__range = for-range-initializer;
    auto __begin = begin-expr ;
    auto __end = end-expr ;
    for ( ; __begin != __end; ++__begin ) {
        for-range-declaration = *__begin;
        statement;
    }
}
```

Wyrażenie `begin-expr` i `end-expr` są ewaluowane do:

- `__range.begin()` i `__range.end()`
- lub z niższym priorytetem do `begin(__range)` i `end(__range)`

W praktyce pętla *range-based for* umożliwia wygodną iterację po:

- kontenerach standardowych:

```c++
std::vector<int> vec = { 1, 2, 3, 4 };

for(const int& item : vec)
    std::cout << item << " ";
```

- tablicach typu *C-style*

```c++
int data[4] = { 1, 2, 3, 4 };

for(auto& n : data)
    n *= 2;
```

- liście inicjalizacyjnej

```c++
for(const auto& item : { 100, 200, 300, 400})
    std::cout << item << " ";
```

### Efektywna wersja range-based for

Kopiowanie elementów w trakcie iteracji może obniżyć wydajność (np. dla typów `std::string`, `std::shared_ptr`) lub może być zabronione np. `std::unique_ptr`. Można uniknąć kopiowania elementów w trakcie iteracji dodając referencję. Opcjonalnie można również stosować modyfikator `const` lub `volatile`.

```c++
std::vector<std::shared_ptr<Gadget>> shared_gadgets;
// ...

for(const auto& ptr : shared_gadgets)
    ptr->do_something();
```

```c++
std::vector<std::unique_ptr<Gadget>> unique_gadgets;
// ...

for(auto ptr : unique_gadgets) // compilation error
    ptr->do_something();

for(const auto& ptr : unique_gadgets) // ok
    ptr->do_something();
```

### Funkcje std::begin() i std::end()

Funkcje wolne `std::begin()` oraz `std::end()` są częścią biblioteki standardowej C++11 i umożliwiają między innymi pętli *range-based for* iterację po tablicach natywnych (*C-array*). 

Są zaprojektowane jako adaptery, które umożliwią iterację po kontenerach, które nie posiadają metod `begin()` i `end()`.

W bibliotece standardowej C++ są zdefiniowane dla:
  
  * kontenerów standardowych

    ```c++
    namespace std
    {
        template <typename Container>
        auto begin(Container& c) -> decltype(c.begin())
        {
            return c.begin();
        }

        template <typename Container>
        auto end(Container& c) -> decltype(c.end())
        {
            return c.end();
        }
    }
    ```

  * tablic natywnych

    ```c++
    template <typename T, size_t N>
    T* begin(T (&array)[N])
    {
        return array;
    }

    template <typename T, size_t N>
    T* end(T (&array)[N])
    {
        return array + N;
    }
    ```

Możliwe jest również zdefiniowanie własnych funkcji `begin()` i `end()` dla własnych kontenerów, które nie posiadają metod `begin()` i `end()`.

````{tip}
Pisząc kod generyczny dla kontenerów warto zawsze używać `std::begin()` i `std::end()`.

```c++
template <typename TContainer, typename TValue>
auto find_value(TContainer& cont, const TValue& value)
{
    using std::begin;
    using std::end;
    auto it = std::find(begin(cont), end(cont), value);
    return it;
}
```
````

### Iteracja po kontenerach zdefiniowanych przez użytkownika

Iteracja po kontenerach zdefiniowanych przez użytkownika wymaga zdefiniowania odpowiednich funkcji składowych `begin()` i `end()`.

```c++
struct SomeContainer
{
    int data[5] = { 1, 2, 3, 4, 5 };

    int* begin() { return data; }
    int* end() { return data + 5; }

    const int* begin() const { return data; }
    const int* end() const { return data + 5; }
};

SomeContainer container;

// we can iterate over the container
for(const auto& item : container)
    std::cout << item << "\n";

int* pos_of_3 = find_value(container, 3);
assert(*pos_of_3 == 3);
```

Można też utworzyć własne implementacje funkcji `begin()` i `end()` adaptując struktury danych, które nie posiadają tych metod.

```c++
struct OtherContainer
{
    int data[5] = { 1, 2, 3, 4, 5 };
};

int* begin(OtherContainer& container) { return container.data; }
int* end(OtherContainer& container) { return container.data + 5; }

OtherContainer container;

// we can iterate over the container
for(const auto& item : container)
    std::cout << item << "\n";

int* pos_of_3 = find_value(container, 3);
assert(*pos_of_3 == 3);
```

## Składnia jednolitej inicjalizacji

Motywacją dla wprowadzenia składni jednolitej inicjalizacji, był fakt, że inicjalizacja w C++98 stwarzała programistom wiele problemów.

Przykłady problemów w C++98:

```c++
int i; // undefined value

int var1(5); // "direct initialization" - from C++98

const int var2 = 10; // "copy initialization" - from C

int var3 = int(); // var3 becomes 0 - from C++98

int var4(); // function declaration!!!

int values[] = { 1, 2, 3, 4 }; // brace initialization

struct Point { int x, y; };
const Point pt1 = { 10, 20 }; // brace initialization

std::complex<double> c(4.0, 2.0); // initialization of classes

std::vector<std::string> cities; // no initialization for list of values
cities.push_back("Warsaw");
cities.push_back("Cracow");
```

Cele jednolitej inicjalizacji w C++11:

- jeden uniwersalny sposób inicjalizacji danych
- powinna istnieć możliwość inicjalizacji kontenerów standardowych
- powinna zabraniać inicjalizacji z wykorzystaniem zawężających konwersji

Składnia jednolitej inicjalizacji jest rozszerzeniem standardu. Prawie cały kod wykorzystujący składnię C++98 jest wciąż poprawny. Jedyny wyjątek od tej reguły - niejawna konwersja, która zawęża typ.

### Inicjalizacja z wykorzystaniem {}

Składnia inicjalizacji z wykorzystaniem nawiasów klamrowych jest teraz dozwolona we wszystkich przypadkach:

```c++
int i; // undefined value - still possible!!!

int var1{5};
const int var2{10};

int arr1[] = { 1, 2, var1, var1 + var2 };

const Point pt1{ 10, 20 };

std::complex<double> c{ 4.0, 2.0 };

const std::vector<std::string> cities = { "Wroclaw", "Cracow" };
std::map<int, std::string> = { {1, "one"}, {2, "two"} };
```

Można używać składni z klamrami do inicjalizacji pól na liście inicjalizacyjnej konstruktora oraz do inicjalizacji tablic dynamicznych.

```c++
class Data
{
    static inline int id_gen_{0};

public:
    Data() : id_{++id_gen_}, data_{ 1, 2, 3 }, names_{"Jan", "Adam", "Ewa"} {}
private:
    const int id_;
    int data_[3];
    std::vector<std::string> names_;
};


int* buffer = new int[5] { 1, 5, var1, var2 + 10, -1 };
```

### Inicjalizacja {} dla agregatów

Inicjalizacja agregatów za pomocą klamer przebiega dokładnie w ten sam sposób jak w C++98 i powoduje inicjalizację składowych wg kolejności ich definicji w agregacie. Liczba elementów na liście musi być równa lub mniejsza ilości elementów w agregacie.

```c++
std::array<int, 3> arr1 = {}; // [0, 0, 0]
std::array<int, 3> arr2 = { 1, 2 }; // [1, 2, 0]
std::array<int, 3> arr3 = { 1, 2, 3 }; // [1, 2, 3]
std::array<int, 3> arr4 = { 1, 2, 3, 4 }; // error
```

### Inicjalizacja {} klas/struktur nie będących agregatami

Inicjalizacja {} klas/struktur nie będących agregatami powoduje wywołanie odpowiedniego konstruktora.

```c++
class Vector2D
{
public:
    Vector2D(int x, int y) : x_{x}, y_{y}
    {}
private:
    int x_, int y_;
};

Vector2D vec1 { 10, 20 };
Vector2D vec2 = { 40, 20 };
Vector2D vec3 { 56, 33, 22 }; // error! too many args

Vector2D versor_x()
{
    return { 1, 0 };
}
```

Składnia "kopiująca" `T variable = expr` nie może wywołać konstruktora zdefiniowanego jako `explicit`:

```c++
std::unique_ptr<int> ptr1 = { new int{10} }; // error
std::unique_ptr<int> ptr2 { new int{12} }; // OK
```

### Konwersja zawężająca typ

C++11 nie zezwala na użycie w inicjalizacji klamrowej niejawnej konwersji, która może doprowadzić do zawężenia typu.

```c++
int arr1[] = { 1, 2, 4.5 }; // OK in C++98; error in C++11

int arr2[] = { 1, 2, static_cast<int>(4.5) }; // OK both in C++98 and C++11

Vector2D vec1(4, 5.5); // OK
Vector2D vec2{4, 5.5}; // error - implicit narrowing
```

## Listy inicjalizacyjne

Aby umożliwić inicjalizację kontenerów standardowych za pomocą składni z nawiasami klamrowymi C++11 wprowadza nowy typ - `std::initializer_list`.

```c++
std::vector<int> v{1, 2, 3};  // creates vector with items: [1, 2, 3]

v.insert(v.end(), { 99, 22, 11, 22, -1 });

v = { 1, 2, 3, 5 }; 
```

### Klasa initializer\_list\<T\>

- Typ zdefiniowany w pliku `<initializer_list>`
  - plik ten jest dołączany przez inne pliki nagłówkowe (np. `<utility>`)
- Przechowuje elementy listy inicjalizującej w tablicy i implementuje ograniczony interfejs umożliwiający dostęp do elementów przy pomocy iteratorów:
  - `size()` - ilość elementów w tablicy
  - `begin()` - wskaźnik (`const T*`) do pierwszego elementu tablicy
  - `end()` - wskaźnik (`const T*`) wskazujący koniec zakresu
- Elementy listy inicjalizującej są niezmienne - *immutable*

Jako argument funkcji `initializer_list` powinien być przesyłany przez wartość:

```c++
#include <initializer_list>

void show_items(std::initializer_list<int> args)
{
    for(const auto& item : args)
        std::cout << item << "\n";
}

show_items({1, 2, 3, 4, 5, 6, 7 });
```

Przykład użycia `initializer_list` w konstruktorze klasy:

```c++
template<typename T>
class Container
{
public:
    Container(std::initializer_list<T> items); //initializer-list constructor
    //...
private:
    size_t size_;
    T* items_;
};

template<typename T>
Container::Container(std::initializer_list<T> items)
    : size_{ items.size() }  //set container's size
{
    reserve(size_);   // get the right amount of space
    uninitialized_copy(items.begin(), items.end(), items_);
        // initialize elements in items_[0:items.size())
}

// ...

Container<int> c1 { 1, 2, 3 }; // OK
Container<double> c2 = { 1.2, 3.14, 5.0 }; // OK
Container<int> c3 { 1, 2, 3.5 }; // error - template parameter T can't be deduced
```

### Listy inicjalizacyjne w przeciążonych konstruktorach

Jeśli klasa posiada wiele wersji konstruktora, przy wywołaniu konstruktora poprzez nawiasy klamrowe preferowany jest konstruktor z `std::initializer_list`.

```c++
class Gadget
{
public:
    Gadget(int, int);
    Gadget(int, std::string);
    Gadget(std::initializer_list<int>);
};

Gadget g1(77, "a"); // calls Gadget::Gadget(int, string)
Gadget g2 {77, "a"}; // calls Gadget::Gadget(int, string)
Gadget g3 = { 77, "a"}; // calls Gadget::Gadget(int, string)

Gadget g5(33, 22);  // calls Gadget::Gadget(int, int)
Gadget g6 { 33, 22 };  // calls Gadget::Gadget(initializer_list)
Gadget g7 = { 33, 22 };  // calls Gadget::Gadget(initializer_list)

auto il = { 77, "a"}; // error - no initializer list deductable
auto il = { 77, 3, 4 };  // OK - initializer_list<int> deduced
Gadget g8(il);  // calls Gadget::Gadget(initializer_list)

Gadget g9 = {};   // calls Gadget::Gadget(initializer_list)
                  // or calls default constructor if exists
```

```{important}
Reguła wywołań konstruktorów:

- () - inicjalizacja wywołuje normalne konstruktory
- {} - inicjalizacja wywołuje również konstruktory z listą inicjalizacyjną
   
  - konstruktory z `std::initializer_list` maję wyższy priorytet
```

### Listy inicjalizacyjne a dedukcja typów

`std::initializer_list` jest jedynym wyjątkiem różniącym dedukcję typów `auto` od dedukcji typów w szablonach:

```c++
auto items = { 1, 2, 4, 5 }; // items is std::initializer_list<int>

template <typename T>
void foo(T param)
{
    // ...
}

foo({ 1, 2, 3, 4, 5 }); // error - deduction failed
```

## Słowo kluczowe - decltype

Słowo kluczowe `decltype` umożliwia kompilatorowi określenie zadeklarowanego typu dla podanego jako argument obiektu lub wyrażenia.

```c++
std::map<std::string, float> coll;

decltype(coll) coll2; // coll2 has type of coll
using EntryT = decltype(coll)::value_type; // EntryT is std::pair<const std::string, float>
```

Jeżeli podajemy jako argument wywołania `decltype()` wyrażenie, to nie jest ono ewaluowane.

```c++
map<int, std::string> coll;

cout << "sizeof: " << sizeof(decltype(coll[0])) << "\n"; // prints 8

static_assert(std::is_same_v<decltype(coll[0]), string&>)

assert(coll.size() == 0);
```

## Nowa składnia deklaracji funkcji

Nowa alternatywna składnia deklaracji funkcji pozwala deklarować typ zwracany po liście parametrów funkcji.

Pozwala to na specyfikację zwracanego typu wewnątrz funkcji oraz z użyciem argumentów funkcji.

```c++
int multiply(int a, int b);

auto multiply(int a, int b) -> int;
```

W połączeniu z `decltype` umożliwia specyfikację typu na podstawie wyrażenia wykorzystującego argumenty funkcji:

```c++
template <typename T1, typename T2>
auto multiply(T1 a, T2 b) -> decltype(a * b)
{
    return a * b;
}
```

## Automatyczna dedukcja typu zwracanego z funkcji (C++14)

### Dedukcja z auto

W C++14 typ zwracany z funkcji może być automatycznie dedukowany z implementacji funkcji. Mechanizm dedukcji jest taki sam jak mechanizm automatycznej dedukcji typów zmiennych.

```c++
auto multiply(int x, int y)
{
    return x * y;
}
```

Jeśli w funkcji występuje wiele instrukcji `return` muszą one wszystkie zwracać wartości tego samego typu.

```c++
auto get_name(int id)
{
    if (id == 1)
        return "Gadget"s;
    else if (id == 2)
        return "SuperGadget"s;
    return string("Unknown");
}
```

Rekurencja dla funkcji z `auto` jest możliwa, o ile rekurencyjne wywołanie następuje po przynajmniej jednym wywołaniu `return` zwracającego wartość nierekurencyjną.

```c++
auto factorial(int n)
{
    if (n == 1)
        return 1;
    return factorial(n-1) * n;
}
```

### Dedukcja z decltype(auto)

Deklaracja `decltype(auto)` jako typu zwracanego z funkcji powoduje zastosowanie do dedukcji typu mechanizmu `decltype` (zachowującego referencje i modyfikatory `const` oraz `volatile`) zamiast mechanizmu `auto`.

```c++
template<class Fun, class... Args>
decltype(auto) call_wrapper(Fun fun, Args&&... args) 
{ 
    return fun(std::forward<Args>(args)...); 
}
```

Mechanizm `decltype` może być również używany do deklaracji zmiennych:

```c++
int i;
int&& f();
auto x3a = i;              // decltype(x3a) is int
decltype(i) x3d = i;       // decltype(x3d) is int
auto x4a = (i);            // decltype(x4a) is int
decltype((i)) x4d = (i);   // decltype(x4d) is int&
auto x5a = f();            // decltype(x5a) is int
decltype(f()) x5d = f();   // decltype(x5d) is int&&

Gadget g;

const Gadget& cg = g;

auto my_auto1 = cg;             // auto type deduction: my_auto1's type is Gadget

decltype(auto) my_auto2 = cg;   // decltype type deduction: 
                                // my_auto2's type is const Widget&
```

## Structured bindings (C++17)

W C++17 można zdefiniować i jednocześnie zainicjować wiele zmiennych przy pomocy tzw. *structured binding*.

Typy zmiennych są dedukowane za pomocą mechanizmu `auto`.

### Typy wiązań

Do realizacji wiązania mogą być użyte:

1. Wszystkie elementy tablicy

    ```c++
    auto foo() -> int(&)[2]

    auto [first, second] = foo();
    ```

2. Wszystkie elementy krotki lub obiektu kompatybilnego typu (np. std::pair, std::array)

    - std::tuple

        ```c++
        tuple<int, std::string, double> tpl(1, "text"s, 2.3);

        auto [first, second, third] = tpl;

        std::cout << first << " " << second << " " << third << '\n';
        ```

    - std::pair

        ```c++
        set<int> unique_numbers;

        if (auto [where, is_inserted] = unique_numbers.insert(1), is_inserted)
            cout << (*where) << " has been inserted" << endl;;        
        ```

    - std::array

        ```c++
        std::array<int, 4> get_data();

        auto [i, j, k, l] = get_data();

        auto [i, j, k] = get_data(); // ERROR - number of items doesn't fit
        ```

    Takie wiązanie jest realizowane tylko jeśli `std::tuple_size<E>` jest typem kompletnym (`E` jest typem krotki lub kompatybilnego obiektu)

3. Wszystkie niestatyczne składowe obiektu klasy/struktury/unii

    - wszystkie składowe muszą być publiczne i być bezpośrednio zdefiniowane w klasie/strukturze wiązanego obiektu lub w jego klasie bazowej
    - anonimowe unie nie są dozwolone

    ```c++
    struct Data
    {
        int n;
        char c;
        double d;
    };

    //...

    Data data1 { 1, 'A', 3.14 };

    auto [member1, member2, member3] = data1;

    std::cout << member1 << " " << member2 << " " << member3 << '\n';
    ```

Jeśli liczba zmiennych umieszczonych w nawiasach `[]` nie zgadza się z liczbą składowych obiektu zwróconego, kompilator zgłasza błąd.

### Mechanizm wiązania structured binding

Mechanizm działania wiązania *structured binding* wykorzystuje nową (anonimową) zmienną, a nowe identyfikatory wprowadzone w wiązaniu odwołują się do pól tej anonimowej zmiennej.

Kod wiązania:

```c++
struct Timestamp
{
    int hours, minutes, seconds;
};

Timestamp timestamp{12, 0, 30};

auto [h, m, s] = timestamp;
```

Odpowiada koncepcyjnie:

```c++
auto e = timestamp;
auto& h = e.hours;
auto& m = e.minutes;
auto& s = e.seconds;
```

Obiekt `e` istnieje tak długo jak istnieją zdefiniowane do niego wiązania.

### Kwalifikatory dla wiązań

Deklaracje *structured bindings* mogą być dekorowane kwalifikatorami w postaci referencji, modyfikatorów `const` oraz `volatile`, `alignas`, przy czym dekoracja taka dotyczy całego anonimowego obiektu:

```c++
int a[] = { 42, 13 };

auto [x, y] = a;

auto& [rx, ry] = a; // rx and ry refer to the elements in a

const auto [v, w] = a; // v and w have type const int, initialized by the elements of a

alignas(16) auto[i, d] = foo(); // i and d refers to implicit entity, which is 16-byte aligned
```

### Praktyczne wykorzystanie structured bindings

1. *Structured bindings* umożliwiają wygodną iterację po mapach w C++17:

    ```c++
    std::map<std::string, double> data = { { "pi", 3.14, "e }, { "e", 2.71 } };

    for (const auto& [key, value] : data)
        std::cout << key << " - " << value << "\n";
    ```

2. Inicjalizacja wielu wartości na raz w instrukcji `for`:

    ```c++
    std::vector vec = { 1, 2, 3 };

    for (auto [i, it] = tuple{ 0, begin(vec) } ; i < size(vec); ++i, ++it)
    {
        std::cout << i << " - " << *it << "\n";
    }
    ```

## Instrukcje if oraz switch z sekcją inicjującą (C++17)

W C++17 wprowadzono dodatkową składnię dla instrukcji `if` oraz `switch` umożliwiającą zgrupowanie instrukcji inicjującej oraz sprawdzającej warunek.

Nowa (dodatkowa) składnia:

```c++
if (init; condition) 
{}

switch(init; condition)
{}
```

W efekcie kod, który w C++98 wyglądał tak:

```c++
Status status = g.status();

if (status == Status::bad)
{
    std::cerr << "Gadget is broken(status=" << static_cast<int>(status) << std::endl;        
}
```

możemy zastąpić bardziej zwięzłym kodem:

```c++
if (Status status = g.status(); status == Status::bad)
{
    std::cerr << "Gadget is broken(status=" << static_cast<int>(status) << std::endl;        
}
```

Przykład wykorzystania nowej wersji instrukcji `if` w pracy z muteksami:

```c++
if (std::lock_guard<std::mutex> lk{mtx}; !q.empty())
{
    std::cout << q.front() << std::endl;
}
```

Instrukcja `switch` z nową składnią:

```c++
switch (Gadget g{2}; auto s = g.status())
{
case Status::on:
    cout << "Gadget is on" << endl;
    break;
case Status::off:
    cout << "Gadget is off" << endl;
    break;
case Status::bad:
    cout << "Gadget is broken" << endl;
    break;
}
```

### Obiekty tymczasowe w sekcji inicjującej

Obiekt tymczasowy utworzony na potrzeby inicjalizacji istnieje tylko w obrębie sekcji inicjującej (tak jak w pętli `for`).

Przykład z *bugiem*:

```c++
if (std::lock_guard<std::mutex>(mtx); !q.empty()) // ERROR - locks ends before ;
{
    std::cout << q.front() << std::endl;
}
```

Poprawiony kod:

```c++
if (std::lock_guard<std::mutex> _(mtx); !q.empty()) // OK - lock has name
{
    std::cout << q.front() << std::endl;
}
```

lub

```c++
if (std::lock_guard lk(mtx); !q.empty())
{
    std::cout << q.front() << std::endl;
}
```

### Structured bindings i if z sekcją inicjującą

Instrukcja `if` z sekcją inicjującą może być połączona z przypisaniem wielu wartości do zmiennych za pomocą *structured bindings*:

```c++
map<int, string> dictionary;

if (auto [pos, is_inserted] = dictionary.insert(std::pair(42, "fourty two"s); !is_inserted)
{
    const auto& [key, value] = *pos;

    cout << key << " is already in a dictionary" << endl;
}
```
