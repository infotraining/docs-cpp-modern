# Dedukcja argumentów dla szablonów klas - CTAD (C++17)

C++17 wprowadza mechanizm dedukcji argumentów szablonu klasy (*Class Template Argument Deduction*). Typy parametrów szablonu klasy mogą być dedukowane na podstawie argumentów przekazanych do konstruktora tworzonego obiektu.

```c++
template <typename T>
class complex
{
    T re_, img_;
public:
    complex(T re, T img) : re_{re}, img_{img}
    {}
};

auto c1 = complex<int>{5, 2}; // OK - all versions of C++ standard

auto c2 = complex{5, 3}; // OK since C++17 - compiler deduces complex<int>

auto c3 = complex(5.1, 6.5); // OK in C++17 - compiler deduces complex<double>

auto c4 = complex{5, 4.1}; // ERROR - args don't have the same type
auto c5 = complex(5, 4.1); // ERROR - args don't have the same type
```

```{warning}
Nie można częściowo dedukować argumentów szablonu klasy. Należy wyspecyfikować lub wydedukować wszystkie parametry z wyjątkiem parametrów domyślnych.
```

Praktyczny przykład dedukcji argumentów szablonu klasy:

```c++
std::mutex mtx;

std::lock_guard lk{mtx}; // deduces std::lock_guard<std::mutex>
```

## Podpowiedzi dedukcyjne (*deduction guides*)

C++17 umożliwia tworzenie podpowiedzi dla kompilatora, jak powinny być dedukowane typy parametrów szablonu klasy na podstawie wywołania odpowiedniego konstruktora.

Daje to możliwość poprawy/modyfikacji domyślnego procesu dedukcji.

Dla szablonu:

```c++
template <typename T>
class S
{
private:
    T value;
public:
    S(T v) : value(v)
    {}
};
```

Podpowiedź dedukcyjna musi zostać umieszczona w tym samym zakresie (przestrzeni nazw) i może mieć postać:

```c++
template <typename T> S(T) -> S<T>; // deduction guide
```

gdzie:

- `S<T>` to tzw. typ zalecany (*guided type*)
- nazwa podpowiedzi dedukcyjnej musi być niekwalifikowaną nazwą klasy szablonowej zadeklarowanej wcześniej w tym samym zakresie
- typ zalecany podpowiedzi musi odwoływać się do identyfikatora szablonu (*template-id*), do którego odnosi się podpowiedź

Użycie podpowiedzi:

```c++
S x{12}; // OK -> S<int> x{12};
S y(12); // OK -> S<int> y(12);
auto z = S{12}; // OK -> auto z = S<int>{12};
S s1(1), s2{2}; // OK -> S<int> s1(1), s2{2};
S s3(42), s4{3.14}; // ERROR
```

W deklaracji `S x{12};` specyfikator `S` jest nazywany symbolem zastępczym dla klasy (*placeholder class type*).

W przypadku użycia symbolu zastępczego dla klasy, nazwa zmiennej musi zostać podana jako następny element składni. W rezultacie poniższa deklaracja jest błędem składniowym:

```c++
S* p = &x; // ERROR - syntax not permitted
```

Dany szablon klasy może mieć wiele konstruktorów oraz wiele podpowiedzi dedukcyjnych:

```c++
template <typename T>
struct Data
{
    T value;

    using type1 = T;

    Data(const T& v)
        : value(v)
    {
    }

    template <typename ItemType>
    Data(initializer_list<ItemType> il)
        : value(il)
    {
    }
};

template <typename T>
Data(T)->Data<T>;

template <typename T>
Data(initializer_list<T>)->Data<vector<T>>;

Data(const char*) -> Data<std::string>;

//...

Data d1("hello"); // OK -> Data<string>

const int tab[10] = {1, 2, 3, 4};
Data d2(tab); // OK -> Data<const int*>

Data d3 = 3; // OK -> Data<int>

Data d4{1, 2, 3, 4}; // OK -> Data<vector<int>>

Data d5 = {1, 2, 3, 4}; // OK -> Data<vector<int>>

Data d6 = {1}; // OK -> Data<vector<int>>

Data d7(d6); // OK - copy by default rule -> Data<vector<int>>

Data d8{d6, d7}; // OK -> Data<vector<Data<vector<int>>>>
```

Podpowiedzi dedukcyjne nie są szablonami funkcji - służą jedynie dedukowaniu argumentów szablonu i nie są wywoływane. W rezultacie nie ma znaczenia czy argumenty w deklaracjach dedukcyjnych są przekazywane przez referencje, czy nie.

```c++
template <typename T> 
struct X
{
    //...
};

template <typename T>
struct Y
{
    Y(const X<T>&);
    Y(X<T>&&);
};

template <typename T> Y(X<T>) -> Y<T>; // deduction guide without references
```

W powyższym przykładzie podpowiedź dedukcyjna nie odpowiada dokładnie sygnaturom konstruktorów przeciążonych. Nie ma to znaczenia, ponieważ jedynym celem podpowiedzi jest umożliwienie dedukcji typu, który jest parametrem szablonu. Dopasowanie wywołania przeciążonego konstruktora odbywa się później.

### Niejawne podpowiedzi dedukcyjne

Ponieważ często podpowiedź dedukcyjna jest potrzebna dla każdego konstruktora klasy, standard C++17 wprowadza mechanizm **niejawnych podpowiedzi dedukcyjnych** (*implicit deduction guides*). Działa on w następujący sposób:

- Lista parametrów szablonu dla podpowiedzi zawiera listę parametrów z szablonu klasy
  - w przypadku szablonowego konstruktora klasy kolejnym elementem jest lista parametrów szablonu konstruktora klasy
- Parametry "funkcyjne" podpowiedzi są kopiowane z konstruktora lub konstruktora szablonowego
- Zalecany typ w podpowiedzi jest nazwą szablonu z argumentami, które są parametrami szablonu wziętymi z klasy szablonowej

Dla klasy szablonowej rozważanej powyżej:

```c++
template <typename T>
class S
{
private:
    T value;
public:
    S(T v) : value(v)
    {}
};
```

niejawna podpowiedź dedukcyjna będzie wyglądać następująco:

```c++
template <typename T> S(T) -> S<T>; // implicit deduction guide
```

W rezultacie programista nie musi implementować jej jawnie.

## Specjalny przypadek dedukcji argumentów klasy szablonowej

Rozważmy następujący przypadek dedukcji:

```c++
S x{42}; // x has type S<int>

S y{x};
S z(x);
```

W obu przypadkach dedukowany typ zmiennych `y` i `z` to `S<int>`. Mechanizm dedukcji argumentów klasy szablonowej dedukuje typ taki sam jak typ oryginalnego obiektu a następnie wywoływany jest konstruktor kopiujący.

- dla deklaracji `S<T> x;` `S{x}` dedukuje typ: `S<T>{x}` zamiast `S<S<T>>{x}`

W niektórych przypadkach może być to zaskakujące i kontrowersyjne:

```c++
std::vector v{1, 2, 3}; // vector<int>
std::vector data1{v, v}; // vector<vector<int>>
std::vector data2{v}; // vector<int>!
```

W powyższym kodzie dedukcja argumentów szablonu `vector` zależy od ilości argumentów przekazanych do konstruktora!

## Agregaty a dedukcja argumentów

Jeśli szablon klasy jest agregatem, to mechanizm automatycznej dedukcji argumentów szablonu wymaga napisania jawnej podpowiedzi dedukcyjnej.

Bez podpowiedzi dedukcyjnej dedukcja dla agregatów nie działa:

```c++
template <typename T>
struct Aggregate1
{
    T value;
};

Aggregate1 agg1{8}; // ERROR
Aggregate1 agg2{"eight"}; // ERROR
Aggregate1 agg3 = 3.14; // ERROR
```

Gdy napiszemy dla agregatu podpowiedź, to możemy zacząć korzystać z mechanizmu dedukcji:

```c++
template <typename T>
struct Aggregate2
{
    T value;
};

template <typename T>
Aggregate2(T) -> Aggregate2<T>;

Aggregate2 agg1{8}; // OK -> Aggregate2<int>
Aggregate2 agg2{"eight"}; // OK -> Aggregate2<const char*>
Aggregate2 agg3 = { 3.14 }; // OK -> Aggregate2<double>
```

## Podpowiedzi dedukcyjne w bibliotece standardowej

Dla wielu klas szablonowych z biblioteki standardowej dodano podpowiedzi dedukcyjne w celu ułatwienia tworzenia instancji tych klas.

### std::pair\<T\>

Dla pary STL dodana w standardzie podpowiedź to:

```c++
template<class T1, class T2>
pair(T1, T2) -> pair<T1, T2>;

pair p1(1, 3.14); // -> pair<int, double>

pair p2{3.14f, "text"s}; // -> pair<float, string>

pair p3{3.14f, "text"}; // -> pair<float, const char*>

int tab[3] = { 1, 2, 3 };
pair p4{1, tab}; // -> pair<int, int*>
```

### std::tuple\<T...\>

Szablon `std::tuple` jest traktowany podobnie jak `std::pair`:

```c++
template<class... UTypes>
tuple(UTypes...) -> tuple<UTypes...>;

template<class T1, class T2>
tuple(pair<T1, T2>) -> tuple<T1, T2>;

//... other deduction guides working with allocators

int x = 10;
const int& cref_x = x;

tuple t1{x, &x, cref_x, "hello", "world"s}; -> tuple<int, int*, int, const char*, string>
```

### std::optional\<T\>

Klasa `std::optional` jest traktowana podobnie do pary i krotki.

```c++
template<class T> optional(T) -> optional<T>;

optional o1(3); // -> optional<int>
optional o2 = o1; // -> optional<int>
```

### Inteligentne wskaźniki

Dedukcja dla argumentów konstruktora będących wskaźnikami jest zablokowana:

```c++
int* ptr = new int{5};
unique_ptr uptr{ip}; // ERROR - ill-formed (due to array type clash)
```

Wspierana jest dedukcja przy konwersjach:

- z `weak_ptr`/`unique_ptr` do `shared_ptr`:

    ```c++
    template <class T> shared_ptr(weak_ptr<T>) ->  shared_ptr<T>;
    template <class T, class D> shared_ptr(unique_ptr<T, D>) ->  shared_ptr<T>;
    ```

- z `shared_ptr` do `weak_ptr`

    ```c++
    template<class T> weak_ptr(shared_ptr<T>) -> weak_ptr<T>;
    ```

```c++
unique_ptr<int> uptr = make_unique<int>(3);

shared_ptr sptr = move(uptr); -> shared_ptr<int>

weak_ptr wptr = sptr; // -> weak_prt<int>

shared_ptr sptr2{wptr}; // -> shared_ptr<int>
```

### std::function

Dozwolone jest dedukowanie sygnatur funkcji dla `std::function`:

```c++
int add(int x, int y)
{
    return x + y;
}

function f1 = &add;
assert(f1(4, 5) == 9);

function f2 = [](const string& txt) { cout << txt << " from lambda!" << endl; };
f2("Hello");
```

### Kontenery i sekwencje

Dla kontenerów standardowych dozwolona jest dedukcja typu kontenera dla konstruktora akceptującego parę iteratorów:

```c++
vector<int> vec{ 1, 2, 3 };
list lst(vec.begin(), vec.end()); // -> list<int>
```

Dla `std::array` dozwolona jest dedukcja z sekwencji:

```c++
std::array arr1{ 1, 2, 3 }; // -> std::array<int, 3>
```