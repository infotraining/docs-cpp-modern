# Aliasy

### Aliasy typów

W C++11 deklaracja `using` może zostać użyta do tworzenia bardziej czytelnych aliasów dla typów - zamiennik dla `typedef`.

```cpp
using CarID = int;

using Func = int(*)(double, double);

using DictionaryDesc = std::map<std::string, std::string, std::greater<std::string>>;
```

### Aliasy szablonów

Aliasy typów mogą być parametryzowane typem. Można je wykorzystać do tworzenia częściowo związanych typów szablonowych.

```cpp
template <typename T>
using StrKeyMap = std::map<std::string, T>;

StrKeyMap<int> my_map; // std::map<std::string, int>
```

Parametrem aliasu typu szablonowego może być także stała znana w czasie kompilacji:

```cpp
template <std::size_t N>
using StringArray = std::array<std::string, N>;

StringArray<255> arr1;
```

Aliasy szablonów nie mogą być specjalizowane.

```cpp
template <typename T>
using MyAllocList = std::list<T, MyAllocator>;

template <typename T>
using MyAllocList = std::list<T*, MyAllocator>; // error
```

Przykład utworzenia aliasu dla szablonu klasy *smart pointer'a*:

```cpp
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

```cpp
template <typename T>
using is_void_t = typename is_void<T>::type;
```

W rezultacie kod odwołujący się do cechy:

```cpp
typename is_void<T>::type
```

możemy uprościć do:

```cpp
is_void_t<T>;
```
