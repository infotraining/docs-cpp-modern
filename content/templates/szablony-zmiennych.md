# Szablony zmiennych

W C++14 zmienne mogą być parametryzowane przy pomocy typu. Takie zmienne nazywamy **zmiennymi szablonowymi** (*variable templates*).

```cpp
template<typename T>
constexpr T pi{3.1415926535897932385};
```

Aby użyć zmiennej szablonowej, należy podać jej typ:

```cpp
std::cout << pi<double> << '\n';
std::cout << pi<float> << '\n';
```

Parametrami zmiennych szablonowych mogą być stałe znane na etapie kompilacji:

```cpp
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

    ```cpp
    template<typename T> T val{};     // zero initialized value
    ```

- plik - "unit1.cpp"

    ```cpp
    #include "header.hpp"

    int main()
    {
        val<long> = 42;
        print();
    } 
    ```

- plik - "unit2.cpp"

    ```cpp
    void print()
    {
        std::cout << val<long> << '\n'; // OK: prints 42
    }
    ```
