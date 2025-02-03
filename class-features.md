# Klasy w C++11

Nowe elementy klas w C++11 umożliwiają:

- Jawne definiowanie operacji specjalnych jako domyślne
- Jawne blokowanie domyślnych operacji specjalnych
- Blokowanie dziedziczenia po danej klasie
- Blokowanie nadpisywania implementacji metod w klasach pochodnych
- Upewnienie się na etapie kompilacji, że nadpisywana metoda istnieje w klasie bazowej
- Użycie przez konstruktor innych konstruktorów klasy
- Inicjalizowanie składowych klasy w miejscu deklaracji

## Specjalne funkcje składowe klas

Specjalne funkcje składowe klas w C++11:

- Konstruktor domyślny
- Destruktor
- Operacje kopiowania - konstruktor kopiujący i kopiujący `operator=`
- Operacje przenoszenia - konstruktor przenoszący i przenoszący `operator=`

Wszystkie specjalne funkcje składowe jeżeli są domyślnie generowane przez kompilator, to posiadają następujące cechy:

- są publiczne
- są inline
- są non-explicit

C++11 daje możliwość jawnego zadeklarowania funkcji specjalnych jako domyślnych lub usunięcia ich z interfejsu klasy.

### Domyślne specjalne funkcje składowe - default

Deklaracja `default` - wymusza na kompilatorze generację domyślnej implementacji dla deklaracji specyfikowanej przez użytkownika (np. generacja domyślnego konstruktora w przypadku, gdy istnieją inne konstruktory przyjmujące parametry)

```c++
class Gadget
{
public:
    Gadget(const Gadget&); // copy constructor will prevent 
                           // generating implicitly declared 
                           // default ctor and move operations

    Gadget() = default;
    Gadget(Gadget&&) noexcept = default;
};
```

Operacje zadeklarowane jako `default` są traktowane jako *user-declared*.

W efekcie klasa:

```c++
class Any // default copy semantics enabled
{
    std::string name;
};
```

nie jest taka sama jak klasa zaimplementowana w poniższy sposób:

```c++
class Any  // default copy semantics deprecated in C++14 (and later probably disabled)
{
    std::string name;

    ~Any = default;
};
```

### Usunięte funkcje składowe - delete

Deklaracja `delete` - usuwa wskazaną funkcję lub funkcję składową z interfejsu klasy. Nie jest generowany kod takiej funkcji, a wywołanie jej, pobranie adresu lub użycie w wyrażeniu z `sizeof` jest błędem kompilacji.

Aby zablokować kopiowanie obiektów danego typu w C++11 można wykorzystać następujący idiom:

```c++
// prevents object from making copies and from move operations
class NoCopyable
{
protected:
    NoCopyable() = default;

public:
    NoCopyable(const NoCopyable&) = delete;
    NoCopyable& operator=(const NoCopyable&) = delete;
};
```

Usuwanie funkcji z interfejsu przy pomocy słowa `delete` nie jest ograniczone tylko do funkcji specjalnych klas. Zastosowanie `delete` dla funkcji wolnej umożliwia uniknięcie niejawnej konwersji argumentów wywołania funkcji:

```c++
void integral_only(int a)
{
    cout << "integral_only: " << a << endl;
}

void integral_only(double d) = delete;

// ...

integral_only(10); // OK

short s = 3;
integral_only(s); // OK - implicit conversion to short

integral_only(3.0); // error - use of deleted function
```

## Domyślna inicjalizacja nie-statycznych składowych klasy

W C++11 można inicjować nie-statyczne składowe klasy bezpośrednio w miejscu ich deklaracji.

Wartość użyta do inicjalizacji będzie przypisana składowej, jeśli wywoływany konstruktor nie nadpisze jej inną wartością.

Operacje kopiowania lub przenoszenia ignorują wartości domyślne

```c++
int default_id()
{
    static int id = 0;
    return ++id;
}

class Gadget
{
private:
    int id_ = default_id();
    double price_ = 0.99;
    std::string name_{"unknown"};
public:
    Gadget() = default;

    Gadget(int id) : id_ {id} {}

    Gadget(int id, double price) : id_{id}, price_ {price} {}

    Gadget(int id, double price, const std::string& name) 
        : id_ {id}, price_ {price}, name_ {name} 
    {}
};

// ...

Gadget a;            // a.id_ = 1, a.price_ = 0.99, a.name_ = "unknown"

Gadget b = 5;        // a.id_ = 5, a.price_ = 0.99, a.name_ = "unknown"

Gadget c {7, 2.99};  // a.id_ = 7, a.price_ = 2.99, a.name_ = "unknown"

Gadget d {9, 2.99, "item#1"};  // a.id_ = 9, a.price_ = 2.99, a.name_ = "item#1"
```

```{warning}
Użycie domyślnej inicjalizacji wewnątrz struktury powoduje, że przestaje ona być agregatem.
```

## Delegowanie konstruktorów

Konstruktor może wywoływać inne konstruktory tej samej klasy.

```c++
class Item
{
private:
    int id_;
public:
    Item(int id) : id_ {id} // non-delegating constructor
    {
        // a
    } 

    Item() : Item {-1} // delegating constructor
    {
        // b (called after a)
    }

    Item(const std::string& id) : Item {std::stoi(id)} // delegating constructor
    {
        // c (called after a)
    }

};
```

```{important}
Obiekt jest uznany za poprawnie skonstruowany, gdy pierwszy konstruktor wywołany zakończy się bez wyjątku (=\> wywołany będzie destruktor obiektu).
```

## Dziedziczenie konstruktorów

Deklaracja `using` może być użyta w połączeniu z konstruktorami klasy bazowej. Powoduje to niejawne deklaracje konstruktorów klasy pochodnej, które przyjmują takie same listy paramtrów co konstruktory klasy bazowej. Ich implementacja polegająca na wywołaniu wersji z klasy bazowej jest generowana tylko wtedy, gdy są one rzeczywiście użyte.

```c++
class Base
{
public:
    explicit Base(int);
    void do_something(int);
}; 

class Derived : public Base
{
public:
    using Base::Base; // OK in C++11
                      // implicit declaration of Derived::Derived(int)

    Derived(int, int); // overloaded inherited Base ctor
};
```

Użycie dziedziczenia konstruktorów w klasach pochodnych, które dodają nowe pola może być ryzykowne:

```c++
class Augmented : public Derived
{
public:
    using Derived::Derived;

private:
    std::string name_;
    int value_;
};

Augmented a {10}; // a.name_ == "" - default init 
                  // and a.value_ is uninitialized
```

## Kontrola nadpisywania metod wirtualnych - override

Słowo `override` ma specjalne znaczenie w deklaracji klas i powoduje sprawdzenie na etapie kompilacji, czy nadpisywana metoda jest zadeklarowana w taki sam sposób w klasie bazowej.

```c++
class Base
{
public:
    virtual void f();
    virtual void g() const;
    virtual void h(char);
    void k();
};

class Derived1 : public Base
{
public:
    void f(); // overrides Base::f()
    void g(); // doesn't override B::g() const
    virtual void h(char); // overrides B::h(char)
    void k(); // doesn't override
};

class Derived2: public Base
{
public:
    void f() override; // OK - overrides Base::f()
    void g() override; // error - doesn't override B::g() const
    virtual void h(char) override; // OK - overrides B::h(char)
    void k() override; // error - B::k() is not virtual
};
```

## Blokowanie dziedziczenia lub nadpisywania metod - final

Od C++11 możemy zablokować dziedziczenie po klasie słowem `final`:

```c++
class NoInheritable final
{
    // ...
};

class Derived : public NoInheritable // error - base marked as final
{   
    // ... 
};
```

Można też określić, że implementacja metody wirtualnej jest ostateczna i nie powinna być nadpisywana:

```c++
class Base
{
public:
    virtual void f() const;
};

class Derived1 : public Base
{
public:
    void f() const override final; // enables additional optimization 
};

class Derived2 : public Derived1
{
public:
    void f() const override; // error - f() attempts to override Derived1::f() 
                             // marked as final 
};
```

## Statyczne składowe inline

W C++17 statyczne zmienne oznaczone jako `inline` są uznawane jako definicja takiej zmiennej w programie.

- gwarantowana jest jednokrotna definicja zmiennej nawet wtedy, gdy nagłówek z definicją jest włączany w wielu jednostkach translacji
- nie musimy tworzyć pliku *cpp* tylko na potrzeby definicji zmiennych globalnych/statycznych

### Definicja składowej statycznej inline

- Plik `gadget.hpp`

```c++
class Gadget
{
public:
    static size_t count() 
    {
        return counter_;
    }
private:
    Gadget() 
    {
        ++counter_;
    }

    Gadget(const Gadget&) = delete;
    Gadget& operator=(const Gadget&) = delete;

    ~Gadget()
    {
        --counter_;
    }

    static inline size_t counter_ = 0;
    static inline const std::string class_id = "Gadget";
};
```

- Plik `a.cpp`

```c++
#include "gadget.hpp"
#include <iostream>

int main()
{
    std::cout << "No of gadgets: " << Gadget::count() << "\n";
}
```

- Plik `b.cpp`

```c++
#include "gadget.hpp"

void bootstrap(GadgetFactory& gf)
{
    gf.register(Gadget::class_id, &make_unique<Gadget>);
}
```

Zmienne statyczne `inline` mogą być:

- inicjalizowane przed funkcją `main()` lub przed pierwszym użyciem
- mogą być `thread_local`
- modyfikator `constexpr` implikuje, że zmienna statyczna jest `inline`

Przykład (plik `monitor.hpp`):

```c++
class Monitor
{
public:
    Monitor() { /* ... */ };

    void log(const std::string& msg);

    inline static thread_local Monitor global_monitor;
};
```
