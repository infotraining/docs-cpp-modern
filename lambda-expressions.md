# Wyrażenia lambda

Jedną z najciekawszych nowości w standardzie C++11 jest możliwość tworzenia wyrażeń lambda.

**Wyrażenie lambda** jest definiowane najczęściej bezpośrednio "w miejscu" jego użycia (*in-place*). Zwykle jest użyte jako parametr innej funkcji, oczekującej wskaźnika do funkcji lub funktora - w ogólności obiektu wywoływalnego (*callable object*).

Każde wyrażenie lambda powoduje utworzenie przez kompilator unikalnej **klasy domknięcia** (*closure class*), która implementuje operator wywołania funkcji posiadający implementację użytą w wyrażeniu.

**Domknięciem** (*closure*) nazywana jest instancja klasy domknięcia. W zależności od sposobu przechwycenia zmiennych lokalnych obiekt ten przechowuje kopie lub referencje do przechwyconych zmiennych.

## Definiowanie wyrażeń lambda

Minimalne wyrażenie lambda:

```c++
[] { std::cout << "Simple lambda expression\n"; }
```

Lambdy mogą przyjmować parametry, a także zwracać wartość:

```c++
auto l = [] (int x, int y) { return x + y; };

auto result = l(2, 3); // result == 5
```

Jeśli implementacja lambdy nie zawiera instrukcji `return` typem zwracanym lambdy jest `void`.

Jeśli implementacja lambdy zawiera tylko instrukcję `return` typem zwracanym lambdy jest typ użytego wyrażenia

W każdym innym przypadku (w C++11) należy zadeklarować typ zwracany:

```c++
[](bool condition) -> int { 
    if (condition) 
        return 1;
    else
        return 2;
}
```

Od C++14 typ zwracany dla lambdy jest automatycznie dedukowany.

Wygodnie jest użyć lambd do tworzenia predykatów lub funktorów wymaganych przez algorytmy standardowe (na przykład w funkcji `std::sort`):

```c++
std::array<double, 6> values = { 5.0, 4.0, -1.4, 7.9, -8.22, 0.4 };

// sortowanie wg wartości bezwzględnych
std::sort(begin(values), end(values),
          // lambda expression as comparer
          [](double a, double b) { return std::abs(a) < std::abs(b); });

// znalezienie wartości mniejszej od zera
auto first_less_than_zero 
    = std::find_if(begin(values), end(values), [](double v) { return v < 0.0; });
```

## Domknięcia zmiennych

W implementacji wyrażenia lambda można odwoływać się do nazw lokalnych zmiennych oraz nazw składowych obiektu (pól `*this`) tylko jeśli zostaną one przechwycone.

Do przechwytywania służy para nawiasów kwadratowych `[]`, w których możemy wymieniać nazwy przechwytywanych zmiennych oraz określać sposób w jaki zostaną one przechwycone.

- `[]` puste nawiasy oznaczają, że wewnątrz lambdy nie można użyć jakiejkolwiek nazwy z zewnętrznego zakresu
- `[&]` niejawne przechwycenie przez referencję. Lambda ma dostęp do odczytu i zapisu zmiennych z zakresu w którym została utworzona. Obiekt domknięcia przechowuje referencje do zewnętrznych zmiennych.

    ```c++
    std::vector<int> vec;

    auto pusher = [&] (int x) { vec.push_back(x); };
    pusher(1);
    pusher(2);

    assert(vec == std::vector{1, 2});
    ```

- `[=]` niejawne przechwycenie przez wartość. Mogą być użyte wszystkie nazwy z zewnętrznego zakresu. Nazwy te odnoszą się do kopii lokalnych zmiennych. Ich wartość jest taka, jaka była w momencie tworzenia obiektu domknięcia.

    ```c++
    int factor = 5;

    auto multiply_by_factor = [=](int x) { return x * factor; };

    factor = 10;

    assert(multiply_by_factor(3) == 15);
    ```

- `[capture-list]` jawne przechwycenie zmiennych wynienionych na liście. Domyślnie wymienione zmienne są przechwytywane przez wartość. Jeśli nazwy zmiennej jest poprzedzona przez `&` oznacza to przechwycenie przez referencję (np. `[x, y, &z]`).

    ```c++
    int counter{};
    auto increment = [&counter] { ++counter; }

    increment();
    assert(counter == 1);
    ```

    ```c++
    std::vector<int> v = { 1, 2, 3, 4, 5 };

    int even_count = 0;
    int odd_count = 0;

    std::for_each(v.begin(), v.end(),
             [&even_count, &odd_count] (int n) {                  
                 if (n % 2 == 0)
                     ++even_count;
                 else
                     ++odd_count;
             });

    assert(even_count == 2);
    assert(odd_count == 3);
    ```

- `[&, capture-list]` niejawne przechwycenie przez referencję wszystkich zmiennych poza tymi, które są wymienione na liście (te są przechwytywane przez wartość). Lista może zawierać `this`.
- `[=, capture-list]` niejawne przechwycenie przez wartość wszystkich zmiennych poza tymi, które są wymienione na liście i poprzedzone `&` są przechwytywane przez referencję. Lista nie może zawierać `this`.

## Modyfikowalne obiekty domknięć - mutable

Domyślnie obiekty domknięć są *immutable*. Domyślnie przechwycone przez wyrażenie lambda wartości oraz operator `()` są wewnątrz klasy domknięcia zadeklarowane jako składowe `const`.

Jeśli lambda zostanie oznaczona jako `mutable`, to może ona modyfikować przechwycone przez wartość zmienne. Tym samym można modyfikować stan obiektu domknięcia.

```c++
auto create_generator(int seed)
{
    return [seed]() mutable {
        return seed++;
    };
}

std::vector<int> vec(100);
std::generate(begin(vec), end(vec), create_generator(1));
```

## Typ lambdy

Standard nie definiuje w jaki sposób wyrażenia lambda zostaną zaimplementowane. W praktyce każde wyrażenie lambda może generować osobną klasę domknięcia. Tym samym dwa wyrażenia lambda posiadające identyczną implementację mają różne typy.

Aby określić typ klasy domknięcia należy użyć operatora `decltype()`:

```c++
auto compare = [](const std::unique_ptr<int>&a , const std::unique_ptr<int>& b) { return *a < *b; };

std::set<std::unique_ptr<int>, decltype(compare)> numbers(compare);

numbers.emplace(std::make_unique<int>(10));
numbers.emplace(std::make_unique<int>(1));
numbers.emplace(std::make_unique<int>(80));
numbers.emplace(std::make_unique<int>(30));
```

## Przechowywanie obiektów domknięć

### auto

Jeśli chcemy przechować obiekt domknięcia w zmiennej lokalnej najlepiej wykorzystać mechanizm dedukcji typu `auto`. Typ lambdy jest wtedy automatycznie dedukowany przez kompilator.

```c++
int threshold = 42;
auto less_than_comp = [threshold](int x) { return x < threshold; };

std::vector<int> vec = { 665, 534,12, 432, 534 };
assert(std::any_of(begin(vec), end(vec), less_than_comp); // passing stored closure as arg
```

### Wskaźnik do funkcji

Obiekty domknięć, które nie przechwytują niczego (mają puste nawiasy []) mogą być przypisywane do wskaźników do funkcji:

```c++
using callback_t = void(*)(const std::string& msg);

callback_t call_me = [](const std::string& msg) { std::cout << "Print: " << msg << "\n"; };

call_me();
```

### std::function

Innym mechanizmem przechowania lub przekazania lambdy jako parametr jest użycie `std::function` - nowego wrappera pozwalającego przechowywać obiekty wywoływalne (*callable*) - lambdy, funktory oraz wskaźniki do funkcji.

```c++
Logger logger;

queue<std::function<void()>> work_queue;

work_queue.push([] { std::cout << "Start" << "\n"; });
work_queue.push([&logger] { logger.log("Running"); });
work_queue.push([] { std::cout << "Stop" << "\n"; });

//...

while(!work_queue.empty())
{
    auto work_to_do = work_queue.front();
    work_to_do();

    work_queue.pop();
}
```

```{warning}
Mechanizm używany przez `std::function` to *type-erasure*. W rezultacie wywołanie pośrednie funkcji lub lambdy może odbyć się za pośrednictwem funkcji wirtualnej.  

Zalecanym mechanizmem typowania lambd jest zatem `auto`.
```

## Przekazywanie obiektów domknięć do funkcji

Jeśli chcemy przekazać obiekt domknięcia jako parameter funkcji, to najlepszym sposobem jest wykorzystanie szablonu funkcji:

```c++
template <typename F>
void caller(F f)
{
    f("calling Elvis");
}

caller([](const std::string& msg) { std::cout << msg << "\n"; });
```

## Zagnieżdżone funkcje lambda

Funkcje lambda można zagnieżdżać.

```c++
auto timestwoplusthree = [](int x) { return [](int y) { return y * 2; }(x) + 3; };

assert(timestwoplusthree(5) == 13);
```

## Funkcje lambda w metodach klas

Definiując wyrażenie lambda wewnątrz metody zwykle chcemy przechwycić składowe klasy. Należy w tym celu użyć składni `[this]`, która powoduje przechwycenie wskaźnika `this` obiektu:

```c++
class Scaler
{
public:
    explicit Scaler(int scale) : scale_{scale} {}

    void apply_scale(std::vector<int>& v) const
    {
        std::transform(v.begin(), v.end(), v.begin(), [this](int n) { return n * scale_; });
    }
private:
    int scale_;
};

int main()
{
    std::vector<int> values = { 1, 2, 3, 4 };

    Scaler s{3};
    s.apply_scale(values);
}
```

## Lambdy w C++14

### Generyczne wyrażenia lambda

W C++11 parametry wyrażeń lambda musiały być zadeklarowane z użyciem konkretnego typu.

C++14 daje możliwość zadeklarowania typu parametru jako `auto` (*generic lambda*).

```c++
auto lambda = [](const auto& x, const auto& y) { return x + y; }
```

Powoduje to dedukcję typu parametru lambdy w ten sam sposób w jaki dedukowane są typy argumentów szablonu. W rezultacie kompilator generuje kod równoważny poniższej klasie domknięcia:

```c++
struct UnnamedClosureClass
{
    template <typename T1, typename T2>
    auto operator()(const T1& x, const T2& y) const 
    {
        return x + y;
    }
};

auto lambda = UnnamedClosureClass();
```

Upraszcza to implementację wielu wyrażeń lambda:

```c++
std::vector<std::shared_ptr<Gadget>> gadgets;

//...

std::sort(gadgets.begin(), gadgets.end(), 
     [](const auto& g1, const auto& g2) { return g1->price() < g2->price(); });
```

### Wyrażenia przechwytujące

C++14 umożliwia zainicjowanie przechwyconej zmiennej dowolnym wyrażeniem.

Umożliwia to przechwycenie zewnętrznej zmiennej, która nie jest kopiowalna, ale jest transferowalna (*move only*).

```c++
std::unique_ptr<Gadget> g = std::make_unique<Gadget>("mp3 player");

auto lambda = [gadget = std::move(g)] { std::cout << gadget->id() << std::endl; };
```

### Automatyczna dedukcja zwracanego typu

W C++14 reguły dotyczące dedukcji zwracanego typu z lambdy zostały znacznie poluzowane. Automatyczna dedukcja jest realizowana również w sytuacji, gdy implementacja zawiera wiele instrukcji `return` o ile zwracają one dane tego samego typu.

```c++
auto it = partition(cont.begin(), cont.end(), 
                    [](const auto& value) {
                        if (value % 5 == 0) return true;
                        if (value % 10 == 0) return false;
                        return false;
                    });
```

## Funkcje lambda wyższego rzędu

Lambdy mogą przyjmować inne lambdy jako parametry, tworząc funkcje wyższego rzędu:

```c++
auto add = [](int x) {
    return [x](int y) { return x + y; };
};

auto higher_order = [](auto f, int z) {
    return f(z) * 2;
};

auto answer = higher_order(add(7), 8);

// Print the result, which is (7+8)*2.
std::cout << answer << "\n";
```
