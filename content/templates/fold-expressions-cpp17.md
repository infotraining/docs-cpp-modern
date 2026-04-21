# Fold expressions (C++17)

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

```cpp
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

```cpp
template <typename... Args>
auto sum(Args&&... args)
{
    return (... + args);
}
```

```cpp
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

```cpp
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


##### all_true

* Wariadyczna funkcja przyjmująca dowolną liczbę argumentów konwertowalnych do wartości logicznych i zwracająca ich iloczyn logiczny (`operator &&`):

```cpp
template <typename... Args>
bool all_true(Args... args)
{
    return (... && args);
}
```

```cpp
bool result = all_true(true, true, false, true);
```

##### print()

* Funkcja `print()` wypisująca przekazane argumenty. Implementacja wykorzystuje wyrażenie *binary left fold* dla operatora `<<`:

```cpp
#include <iostream>

template <typename... Args>
void print(Args&&... args)
{
    (std::cout << ... << args) << "\n";
}
```

```cpp
print(1, 2, 3, 4);
```

##### invoke() & call_foreach()

Implementacja wariadycznej wersji algorytmu `foreach()` z wykorzystaniem funkcji `std::invoke()`:

```cpp
#include <iostream>

template <typename F, typename... Args>
decltype(auto) invoke(F&& f, Args&&... args)
{
    return std::forward<F>(f)(std::forward<Args>(args)...);
}

auto call_foreach = [](auto&& fun, auto&&... args) {
    (..., invoke(fun, std::forward<decltype(args)>(args));
};
```

```cpp
#include <string>    

using namespace std::literals;

struct Printer
{
    template <typename T>
    void operator()(T&& arg) const { std::cout << arg; }
};

call_foreach(Printer{}, 1, " - one; ", 3.14, " - pi;"s);
```
