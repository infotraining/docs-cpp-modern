# Szablony

## Aliasy

### Aliasy typów

W C++11 deklaracja `using` może zostać użyta do tworzenia bardziej czytelnych aliasów dla typów - zamiennik dla `typedef`.

```c++
using CarID = int;

using Func = int(*)(double, double);

using DictionaryDesc = std::map<std::string, std::string, std::greater<std::string>>;
```

### Aliasy szablonów

Aliasy typów mogą być parametryzowane typem. Można je wykorzystać do tworzenia częściowo związanych typów szablonowych.

```c++
template <typename T>
using StrKeyMap = std::map<std::string, T>;

StrKeyMap<int> my_map; // std::map<std::string, int>
```

Parametrem aliasu typu szablonowego może być także stała znana w czasie kompilacji:

```c++
template <std::size_t N>
using StringArray = std::array<std::string, N>;

StringArray<255> arr1;
```

Aliasy szablonów nie mogą być specjalizowane.

```c++
template <typename T>
using MyAllocList = std::list<T, MyAllocator>;

template <typename T>
using MyAllocList = std::list<T*, MyAllocator>; // error
```

Przykład utworzenia aliasu dla szablonu klasy *smart pointer'a*:

```c++
template <typename Stream>
struct StreamDeleter
{
    void operator()(Stream* os)
    {
        os->close();
        delete os;
    }
}

template <typename Stream>
using StreamPtr = std::unique_ptr<Stream, StreamDeleter<Stream>>;

// ...
{
    StreamPtr<ofstream> p_log(new ofstream("log.log"));
    *p_log << "Log statement";        
}
```

#### Aliasy i typename

Od C++14 biblioteka standardowa używa aliasów dla wszystkich cech typów, które zwracają typ.

```c++
template <typename T>
using is_void_t = typename is_void<T>::type;
```

W rezultacie kod odwołujący się do cechy:

```c++
typename is_void<T>::type
```

możemy uprościć do:

```c++
is_void_t<T>;
```

## Szablony zmiennych

W C++14 zmienne mogą być parametryzowane przy pomocy typu. Takie zmienne nazywamy **zmiennymi szablonowymi** (*variable templates*).

```c++
template<typename T>
constexpr T pi{3.1415926535897932385};
```

Aby użyć zmiennej szablonowej, należy podać jej typ:

```c++
std::cout << pi<double> << '\n';
std::cout << pi<float> << '\n';
```

Parametrami zmiennych szablonowych mogą być stałe znane na etapie kompilacji:

```c++
template<int N>
std::array<int, N> arr{};    

int main()
{
    arr<10>[0] = 42; // sets first element of global arr

    for (std::size_t i = 0; i < arr<10>.size(); ++i) 
    {   
        // uses values set in arr
        std::cout << arr<10>[i] << '\n';    
    }
}
```

### Zmienne szablonowe a jednostki translacji

Deklaracja zmiennych szablonowych może być używana w innych jednostkach translacji.

- plik - `header.hpp`

    ```c++
    template<typename T> T val{};     // zero initialized value
    ```

- plik - "unit1.cpp"

    ```c++
    #include "header.hpp"

    int main()
    {
        val<long> = 42;
        print();
    } 
    ```

- plik - "unit2.cpp"

    ```c++
    void print()
    {
        std::cout << val<long> << '\n'; // OK: prints 42
    }
    ```

## Variadic templates

W C++11 szablony mogą akceptować dowolną ilość (również zero) parametrów. Jest to możliwe dzięki użyciu specjalnego grupowego parametru szablonu tzw. *parameter pack*, który reprezentuje wiele lub zero parametrów szablonu.

### Parameter pack

*Parameter pack* może być

- grupą parametrów szablonu

    ```c++
    template<typename... Ts> // template parameter pack
    class tuple
    {
        //...
    };

    tuple<int, double, string&> t1; // 3 arguments: int, double, string&
    tuple<> empty_tuple; // 0 arguments
    ```

- grupą argumentów funkcji szablonowej

    ```c++
    template <typename T, typename... Args>
    shared_ptr<T> make_shared(Args&&... params)
    {
        //...
    }

    auto sptr = make_shared<Gadget>(10); // Args as template param: int
                                         // Args as function param: int&&
    ```

### Rozpakowanie paczki parametrów

Podstawową operacją wykonywaną na grupie parametrów szablonu jest rozpakowanie jej za pomocą operatora `...` (tzw. *pack expansion*).

```c++
template <typaname... Ts>  // template  parameter pack
struct X
{
    tuple<Ts...> data;  // pack expansion
};
```

Rozpakowanie paczki parametrów (*pack expansion*) może zostać zrealizowane przy pomocy wzorca zakończonego elipsą `...`:

```c++
template <typaname... Ts>  // template  parameter pack
struct XPtrs
{
    tuple<Ts const*...> ptrs;  // pack expansion
};

XPtrs<int, string, double> ptrs; // contains tuple<int const*, string const*, double const*>
```

Najczęstszym przypadkiem użycia wzorca przy rozpakowaniu paczki parametrów jest implementacja **perfect forwarding'u**:

```c++
template <typename... Args>
void make_call(Args&&... params)
{
    callable(forward<Args>(params)...);
}
```

### Idiom Head/Tail

Praktyczne użycie **variadic templates** wykorzystuje często idiom **Head/Tail** (znany również **First/Rest**).

Idiom ten polega na zdefiniowaniu wersji szablonu akceptującego dwa parametry: - pierwszego `Head` - drugiego `Tail` w postaci paczki parametrów

W implementacji wykorzystany jest parametr (lub argument) typu `Head`, po czym rekursywnie wywołana jest implementacja dla rozpakowanej paczki parametrów typu `Tail`.

Dla szablonów klas idiom wykorzystuje specjalizację częściową i szczegółową (do przerwania rekurencji):

```c++
template <typename... Types>
struct Count;

template <typename Head, typename... Tail>
struct Count<Head, Tail...>
{
    constexpr static int value = 1 + Count<Tail...>::value; // expansion pack
};

template <>
struct Count<>
{
    constexpr static int value = 0;
};

//...
static_assert(Count<int, double, string&>::value == 3, "must be 3");
```

W przypadku szablonów funkcji rekurencja może być przerwana przez dostarczenie odpowiednio przeciążonej funkcji. Zostanie ona w odpowiednim momencie rozwijania rekurencji wywołana.

```c++
void print()
{}

template <typename T, typename... Tail>
void print(const T& arg1, const Tail&... params)
{
    cout << arg1 << endl;
    print(params...); // function parameter pack expansion
}
```

### Operator sizeof

Operator `sizeof...` umożliwia odczytanie na etapie kompilacji ilości parametrów w grupie.

```c++
template <typename... Types>
struct VerifyCount
{
    static_assert(Count<Types...>::value == sizeof...(Types),
                    "Error in counting number of parameters");
};
```

### Forwardowanie wywołań funkcji

**Variadic templates** są niezwykle przydatne do forwardowania wywołań funkcji.

```c++
template <typename T, typename... Args>
std::unique_ptr<T> make_unique(Args&&... params)
{
    return std::unique_ptr<T>(new T(std::forward<Args>(params)...);
}
```

### Ograniczenia paczek parametrów

Klasa szablonowa może mieć tylko jedną paczkę parametrów i musi ona zostać umieszczona na końcu listy parametrów szablonu:

```c++
template <size_t... Indexes, typename... Ts>  // error
class Error;
```

Można obejść to ograniczenie w następujący sposób:

```c++
template <size_t... Indexes> struct IndexSequence {};

template <typename Indexes, typename Ts...>
class Ok;

Ok<IndexSequence<1, 2, 3>, int, char, double> ok;
```

Funkcje szablonowe mogą mieć więcej paczek parametrów:

```c++
template <int... Factors, typename... Ts>
void scale_and_print(Ts const&... args)
{
    print(ints * args...);
}

scale_and_print<1, 2, 3>(3.14, 2, 3.0f);  // calls print(1 * 3.14, 2 * 2, 3 * 3.0)
```

**Uwaga!** Wszystkie paczki w tym samym wyrażeniu rozpakowującym muszą mieć taki sam rozmiar.

```c++
scale_and_print<1, 2>(3.14, 2, 3.0f);  // error
```

### "Nietypowe" paczki parametrów

- Podobnie jak w przypadku innych parametrów szablonów, paczka parametrów nie musi być paczką typów, lecz może być paczką stałych znanych na etapie kompilacji

```c++
template <size_t... Values>
struct MaxValue;  // primary template declaration

template <size_t First, size_t... Rest>
struct MaxValue<First, Rest...>
{
    static constexpr size_t rvalue = MaxValue<Rest...>::value;
    static constexpr size_t value = (First < rvalue) ? rvalue : First;    
};

template <size_t Last>
struct MaxValue<Last>
{
    static constexpr size_t value = Last;  // termination of recursive expansion
};
```

```c++
static_assert(MaxValue<1, 5345, 3, 453, 645, 13>::value == 5345, "Error");
```

### Variadic Mixins

**Variadic templates** mogą być skutecznie wykorzystane do implementacji klas **mixin**

```c++
#include <vector>
#include <string>

template <typename... Mixins>
class X : public Mixins...
{
public:
    X(Mixins&&... mixins) : Mixins(mixins)... 
    {}
};
```

Klasa taka może zostać wykorzystana później w następujący sposób:

```c++
X<std::vector<int>, std::string> x({ 1, 2, 3 }, "text");

x.std::string::size();
```

```c++
x.std::vector<int>::size();
```

## Fold expressions (C++17)

### Wyrażenia *fold* w językach funkcjonalnych

Koncept *redukcji* jest jednym z podstawowych pojęć w językach funkcjonalnych.

**Fold** w językach funkcjonalnych to rodzina funkcji wyższego rzędu zwana również **reduce**, **accumulate**, **compress** lub **inject**. Funkcje **fold** przetwarzają rekursywnie uporządkowane kolekcje danych (listy) w celu zbudowania końcowego wyniku przy pomocy funkcji (operatora) łączącej elementy.

Dwie najbardziej popularne funkcje z tej rodziny to fold (fold left) i foldr (fold right).

Przykład:

Redukcja listy [1, 2, 3, 4, 5] z użyciem operatora (+):

- użycie funkcji `fold` - redukcja od lewej do prawej

    ```haskell
    fold (+) 0 [1..5]
    ```

    ```bash
    (((((0 + 1) + 2) + 3) + 4) + 5)
    ```

- użycie funkcji `foldr` - redukcja od prawej do lewej

    ```haskell
    foldr (+) 0 [1..5]
    ```

    ```bash
    (1 + (2 + (3 + (4 + (5 + 0)))))
    ```

#### Redukcja w C++98 - std::accumulate

W C++ redukcja jest obecna poprzez implementację algorytmu `std::accumulate`.

```c++
#include <vector>
#include <numeric>
#include <string>

using namespace std::string_literals;

std::vector<int> vec = {1, 2, 3, 4, 5};

std::accumulate(std::begin(vec), std::end(vec), "0"s, 
                   [](const std::string& reduction, int item) { 
                       return "("s + reduction +  " + "s + std::to_string(item) + ")"s; });
```

### Wyrażenia fold

Wyrażenia typu *fold* umożliwiają uproszczenie rekurencyjnych implementacji dla zmiennej liczby argumentów szablonu.

Przykład z wariadyczną funkcją `sum(1, 2, 3, 4, 5)` z wykorzystaniem *fold expressions* może być w C++17 zaimplementowany następująco:

```c++
template <typename... Args>
auto sum(Args&&... args)
{
    return (... + args);
}
```

```c++
sum(1, 2, 3, 4, 5);
```

#### Składnia wyrażeń fold

Niech  $e = e_1, e_2, \dotso, e_n$ będzie wyrażeniem, które zawiera nierozpakowany *parameter pack* i $\otimes$ jest *operatorem fold*, wówczas **wyrażenie fold** ma postać:

- Unary **left fold**

  $(\dotso\; \otimes\; e)$

który jest rozwijany do postaci $(((e_1 \otimes e_2) \dotso ) \otimes e_n)$

- Unary **right fold**

  $(e\; \otimes\; \dotso)$

który jest rozwijany do postaci $(e_1 \otimes ( \dotso (e_{n-1} \otimes e_n)))$

Jeśli dodamy argument nie będący paczką parametrów do operatora `...`, dostaniemy dwuargumentową wersję **wyrażenia fold**. W zależności od tego po której stronie operatora `...` dodamy dodatkowy argument otrzymamy:

- Binary **left fold**

  $(a \otimes\; \dotso\; \otimes\; e)$

który jest rozwijany do postaci $(((a \otimes e_1) \dotso ) \otimes e_n)$

- Binary **right fold**

    $(e\; \otimes\; \dotso\; \otimes\; a)$

który jest rozwijany do postaci $(e_1 \otimes ( \dotso (e_n \otimes a)))$

Operatorem $\otimes$ może być jeden z poniższych operatorów C++:

```c++
+  -  *  /  %  ^  &  |  ~  =  <  >  <<  >>
+=  -=  *=  /=  %=  ^=  &=  |=  <<=  >>=
==  !=  <=  >=  &&  ||  ,  .*  ->*
```

#### Elementy identycznościowe

Operacja fold dla pustej paczki parametrów (*parameter pack*) jest ewaluowana do określonej wartości zależnej od rodzaju zastosowanego operatora. Zbiór operatorów i ich rozwinięć dla pustej listy parametrów prezentuje tabela:

| Operator | Wartość zwracana jako element identycznościowy |
|:--------:|:----------------------------------------------:|
| `&&`     | `true`                                         |
| `||`     | `false`                                        |
| `,`      | `void()`                                       |

Jeśli operacja fold jest ewaluowana dla pustej paczki parametrów dla innego operatora, program jest nieprawidłowo skonstruowany (*ill-formed*).

#### Przykłady zastosowań wyrażeń fold

Wariadyczna funkcja przyjmująca dowolną liczbę argumentów konwertowalnych do wartości logicznych i zwracająca ich iloczyn logiczny (`operator &&`):

```c++
template <typename... Args>
bool all_true(Args... args)
{
    return (... && args);
}
```

```c++
bool result = all_true(true, true, false, true);
```

* * * * *

Funkcja `print()` wypisująca przekazane argumenty. Implementacja wykorzystuje wyrażenie *binary left fold* dla operatora `<<`:

```c++
#include <iostream>

template <typename... Args>
void print(Args&&... args)
{
    (std::cout << ... << args) << "\n";
}
```

```c++
print(1, 2, 3, 4);
```

* * * * *

Iteracja po elementach różnych typów:

```c++
#include <iostream>

struct Window {
    void show() const { std::cout << "showing Window\n"; }
};

struct Widget {
    void show() const { std::cout << "showing Widget\n"; }
};

struct Toolbar {
    void show() const { std::cout << "showing Toolbar\n"; }
};
```

```c++
Window wnd;
Widget widget;
Toolbar toolbar;        

auto printer = [](const auto&... args) { args.show(), ...); };

printer(wnd, widget, toolbar);
```

* * * * *

Implementacja wariadycznej wersji algorytmu `foreach()` z wykorzystaniem funkcji `std::invoke()`:

```c++
#include <iostream>

template <typename F, typename... Args>
auto invoke(F&& f, Args&&... args)
{
    return std::forward<F>(f)(std::forward<Args>(args)...);
}

struct Printer
{
    template <typename T>
    void operator()(T&& arg) const { std::cout << arg; }
};
```

```c++
#include <string>    

using namespace std::literals;

auto foreach = [](auto&& fun, auto&&... args) {
    (..., invoke(fun, std::forward<decltype(args)>(args));
};

foreach(Printer{}, 1, " - one; ", 3.14, " - pi;"s);
```
