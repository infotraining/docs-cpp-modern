# Wyrażenia constexpr

C++11 wprowadza dwa znaczenia dla "stałej":

- `constexpr` - stała ewaluowana na etapie kompilacji
- `const` - stała, której wartość nie może ulec zmianie

Stałe wyrażenie (**constant expression**) jest wyrażeniem ewaluowanym przez kompilator na etapie kompilacji. Nie może zawierać wartości, które nie są znane na etapie kompilacji i nie może mieć efektów ubocznych.

Jeśli wyrażenie inicjalizujące dla `constexpr` nie będzie mogło być wyliczone na etapie kompilacji kompilator zgłosi błąd:

```c++
int x1 = 7;  // variable
constexpr int x2 = 7;  // constant at compile-time

constexpr int x3 = x1; // error: initializer is not a contant expression

constexpr auto x4 = x2; // Ok
```

W wyrażeniu `contexpr` można użyć:

- Wartości typów całkowitych, zmiennoprzecinkowych oraz wyliczeniowych
- Operatorów nie modyfikujących stanu (np. +, ? i [] ale nie = lub ++)
- Funkcji `constexpr`
- Typów literalnych
- Stałych `const` zainicjowanych stałym wyrażeniem

## Stałe wartości constexpr

W C++11 `constexpr` przed definicją zmiennej definiuje ją jako stałą, która musi zostać zainicjowana wyrażeniem stałym.

Stała `const` w odróżnieniu od stałej `constexpr` nie musi być zainicjowana wyrażeniem stałym.

```c++
constexpr int x = 7;

constexpr auto prefix = "Data";

constexpr double pi = 3.1415;

constexpr double pi_2 = pi / 2;
```

## Funkcje constexpr

W C++11 funkcje mogą zostać zadeklarowane jako `constexpr` jeśli spełniają dwa wymagania:

- Ciało funkcji zawiera tylko jedną instrukcję `return` zwracającą wartość, która nie jest typu `void`
- Typ wartości zwracanej oraz typy parametrów powinny być typami dozwolonymi dla wyrażeń `constexpr`

C++14 znacznie redukuje ograniczenia odnoszące się do implementacji funkcji `constexpr`. Funkcją `constexpr` może zostać dowolna funkcja o ile:

- nie jest wirtualna
- typ wartości zwracanej oraz typy parametrów są typami literalnymi
- zmienne użyte wewnątrz funkcji są zmiennymi typów literalnych
- nie zawiera instrukcji `asm`, `goto`, etykiet oraz bloków `try-catch`
- zmienne użyte wewnątrz funkcji nie są statyczne oraz nie są `thread-local`
- zmienne użyte wewnątrz funkcji są zainicjowane

```{important}
Funkcje `constexpr` nie mogą mieć żadnych efektów ubocznych. Zapisywanie stanu do nielokalnych zmiennych jest błędem kompilacji.
```

Przykład rekurencyjnej funkcji `constexpr`:

```c++
constexpr int factorial(int n)
{
    return (n == 0) ? 1 : n * factorial(n-1);
}
```

Funkcja `constexpr` może zostać użyta w kontekście, w którym wymagana jest stała ewaluowana na etapie kompilacji (np. rozmiar tablicy natywnej lub stała będąca parametrem szablonu):

```c++
#include <array>

const int size = 2;

int arr1[factorial(1)];
int arr2[factorial(size)];
std::array<int, factorial(3)> arr3;
```

```c++
template <typename T, size_t N>
constexpr size_t size_of_array(T(&)[N])
{
   return N;
}

int arr4[factorial(size_of_array(arr2))] = {};
```

### Instrukcje warunkowe w funkcjach constexpr

Pominięty blok kodu w instrukcji warunkowej nie jest ewaluowany na etapie kompilacji.

```c++
constexpr int low = 0;
constexpr int high = 99;
```

```c++
#include <stdexcept>

constexpr int check(int i)
{
    return (low <= i && i < high) ? i : throw std::out_of_range("range error");
}
```

```c++
constexpr int val0 = check(50);  // OK

constexpr int val2 = check(200);  // Error
```

## Typy literalne

C++11 wprowadza pojęcie **typu literalnego** (*literal type*), który może być użyty w stałym wyrażeniu `constexpr`:

Typem literalnym jest:

- Typ arytmetyczny (całkowity, zmiennoprzecinkowy, znakowy lub logiczny)
- Typ referencyjny do typu literalnego (np: `int&`, `double&`)
- Tablica typów literalnych
- Klasa, która:
  - ma trywialny destruktor (może być `default`)
  - wszystkie niestatyczne składowe i typy bazowe są typami literalnymi
  - jest agregatem lub ma przynajmniej jeden konstruktor `contexpr`, który nie jest konstruktorem kopiującym lub przenoszącym (konstruktor musi mieć pustą implementację, ale umożliwia inicjalizację składowych na liście inicjalizującej)

```c++
class Complex
{
    double real_, imaginary_;
public:
    constexpr Complex(const double& real, const double& imaginary)
        : real_ {real}, imaginary_ {imaginary}
    {}

    constexpr double real() const { return real_; };
    constexpr double imaginary() const { return imaginary_; }
};
```

```c++
constexpr Complex c1 {1, 2};
```

## Przykłady zastosowań wyrażeń i funkcji stałych constexpr

### Operacje na polach bitowych

Interesującym zastosowaniem funkcji `constexpr` jest implementacja operatorów bitowych dla wyliczeń.

```c++
namespace Constexpr
{
    enum class Bitmask { b0 = 0x1, b1 = 0x2, b2 = 0x4 };

    constexpr Bitmask operator|(Bitmask left, Bitmask right)
    {
        return Bitmask( static_cast<int>(left) | static_cast<int>(right) ); 
    }
}
```

Umożliwia to, zastosowanie czytelnych wyrażeń bitowych np. w etykietach instrukcji `switch`:

```c++
#include <iostream>

using namespace std;
using namespace Constexpr;

Bitmask b = Bitmask::b0 | Bitmask::b1;

switch (b)
{
    case Bitmask::b0 | Bitmask::b1:
        cout << "b0 | b1 - " << static_cast<int>(b) << endl;
        break;
    default:
        cout << "Other value...";
}
```

## Lambdy constexpr (C++17)

Od C++17 wyrażenia lambda są traktowane domyślnie jako wyrażenia `constexpr` (jeśli jest to możliwe). Można explicite zastosować również słowo kluczowe `constexpr` w definicji lambdy.

```c++
auto squared = [](auto x) { return x * x; } // implicitly consexpr

std::array<int, squared(8)> arr1; // OK - array<int, 64>

auto squared = [](auto x) constexpr { // OK - since C++17
    return x * x;
};
```

Jeśli w definicji wyrażenia lambda nie są spełnione wymagania dla wyrażeń `constexpr`, to kompilator:

- domyślnie przyjmie, że definicja lambdy nie jest `constexpr`

    ```c++
    auto is_even = [](int x) { 
        static size_t counter = 0;
        counter++;
        //...
        return x % 2 == 0;
    }; // OK - but not constexpr
    ```

- lub w przypadku jawnej deklaracji `constexpr` zgłosi błąd kompilacji

    ```c++
    auto is_even = [](int x) constexpr { 
        static size_t counter = 0;
        counter++;
        //...
        return x % 2 == 0;
    }; // ERROR - lambda expression is not constexpr
    ```

## constexpr if (C++17)

C++17 wprowadza do standardu C++ nową postać instrukcji warunkowej `if`, która działa na etapie kompilacji - tzw. `constexpr if`.

Działanie `constexpr if` polega na wyborze podczas kompilacji bloku instrukcji `then`/`else` w zależności od warunku, który jest wyrażeniem `constexpr`.

Składnia:

```c++
if constexpr(condition)
{
   // ...
}
else
{
   // ...
}
```

`constexpr if` umożliwia znaczne uproszczenie kodu szablonowego, który bardzo często w C++11 był mocno skomplikowany.

Przykład w C++11:

```c++
template<class T>
auto compute(T x) -> enable_if_t<supportsAPI<T>::value, int>
{
    return optimized_computation(x);
}

template<class T>
auto compute(T x) -> enable_if_t<!supportsAPI<T>::value, int>
{
    return generic_computation(x);
}
```

Powyższy kod może być dużo prościej wyrażony w C++17 za pomocą `constexpr if`:

```c++
template<class T>
auto compute(T x) 
{
    if constexpr(supportsAPI<T>::value)
    {
        return optimized_computation(x);
    }
    else
    {
        return generic_computation(x);
    }
}
```

### Discarded statements

Kod (grupa instrukcji), który jest ominięty przy kompilacji (tzw. *discarded statement*), nie jest instancjonowany, ale musi być poprawny składniowo. Mechanizm *constexpr if* zasadniczo odpowiada pierwszemu etapowi przetwarzania szablonów przez kompilator (faza definicji).

```c++
template <typename T>
void foo(T obj);

void f_with_discarded_statements()
{
    if constexpr(std::numeric_limits<char>::is_signed) 
    {
        foo(42);
        static_assert(std::numeric_limits<char>::is_signed, "char is unsigned"); // always fails if char is unsigned
    }   
    else
    {
        undeclared(42);  // always error if undeclared() not declared
        static_assert(!std::numeric_limits<char>::is_signed, "char is signed"); // always fails if char is signed
    }
}
```

Powyższy kod nigdy się nie skompiluje.

### Mechanizm kompilacji szablonów

1. Faza pierwsza:
    - wykrywane są błędy składniowe
    - użycie nieznanych typów, funkcji, itp. generuje błąd kompilacji
    - sprawdzane są statyczne asercje

2. Faza druga:
    - kod zależny od parametru szablonu jest podwójnie sprawdzany

```c++
template <typename T>
void foo(T t)
{
    undeclared();
    undeclared(t);

    static_assert(sizeof(int) > 4, "small int"); // 1st phase error if sizeof(int) <= 4
    static_assert(sizeof(T) > 4, "small T"); // 2nd phase error if sizeof(T) <= 4
    static_assert(false, "Error"); // always fails when template is compiled (even if not called)
}
```
