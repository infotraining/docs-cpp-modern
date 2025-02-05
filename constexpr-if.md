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

````{note} Mechanizm kompilacji szablonów

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
````