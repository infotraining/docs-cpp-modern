# Variadic templates

W C++11 szablony mogą akceptować dowolną ilość (również zero) parametrów. Jest to możliwe dzięki użyciu specjalnego grupowego parametru szablonu tzw. *parameter pack*, który reprezentuje wiele lub zero parametrów szablonu.

### Parameter pack

*Parameter pack* może być

- grupą parametrów szablonu

    ```cpp
    template<typename... Ts> // template parameter pack
    class tuple
    {
        //...
    };

    tuple<int, double, string&> t1; // 3 arguments: int, double, string&
    tuple<> empty_tuple; // 0 arguments
    ```

- grupą argumentów funkcji szablonowej

    ```cpp
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

```cpp
template <typaname... Ts>  // template  parameter pack
struct X
{
    tuple<Ts...> data;  // pack expansion
};
```

Rozpakowanie paczki parametrów (*pack expansion*) może zostać zrealizowane przy pomocy wzorca zakończonego elipsą `...`:

```cpp
template <typaname... Ts>  // template  parameter pack
struct XPtrs
{
    tuple<Ts const*...> ptrs;  // pack expansion
};

XPtrs<int, string, double> ptrs; // contains tuple<int const*, string const*, double const*>
```

Najczęstszym przypadkiem użycia wzorca przy rozpakowaniu paczki parametrów jest implementacja **perfect forwarding'u**:

```cpp
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

```cpp
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

```cpp
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

```cpp
template <typename... Types>
struct VerifyCount
{
    static_assert(Count<Types...>::value == sizeof...(Types),
                    "Error in counting number of parameters");
};
```

### Forwardowanie wywołań funkcji

**Variadic templates** są niezwykle przydatne do forwardowania wywołań funkcji.

```cpp
template <typename T, typename... Args>
std::unique_ptr<T> make_unique(Args&&... params)
{
    return std::unique_ptr<T>(new T(std::forward<Args>(params)...);
}
```

### Ograniczenia paczek parametrów

Klasa szablonowa może mieć tylko jedną paczkę parametrów i musi ona zostać umieszczona na końcu listy parametrów szablonu:

```cpp
template <size_t... Indexes, typename... Ts>  // error
class Error;
```

Można obejść to ograniczenie w następujący sposób:

```cpp
template <size_t... Indexes> struct IndexSequence {};

template <typename Indexes, typename Ts...>
class Ok;

Ok<IndexSequence<1, 2, 3>, int, char, double> ok;
```

Funkcje szablonowe mogą mieć więcej paczek parametrów:

```cpp
template <int... Factors, typename... Ts>
void scale_and_print(const Ts&... args)
{
    print(ints * args...);
}

scale_and_print<1, 2, 3>(3.14, 2, 3.0f);  // calls print(1 * 3.14, 2 * 2, 3 * 3.0)
```

**Uwaga!** Wszystkie paczki w tym samym wyrażeniu rozpakowującym muszą mieć taki sam rozmiar.

```cpp
scale_and_print<1, 2>(3.14, 2, 3.0f);  // error
```

### "Nietypowe" paczki parametrów

- Podobnie jak w przypadku innych parametrów szablonów, paczka parametrów nie musi być paczką typów, lecz może być paczką stałych znanych na etapie kompilacji

```cpp
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

```cpp
static_assert(MaxValue<1, 5345, 3, 453, 645, 13>::value == 5345, "Error");
```
