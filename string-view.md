# Klasa std::string_view

Typ `std::string_view` jest lekkim widokiem *read-only* dla sekwencji znaków. Jest on zaprojektowany do wydajnego odwoływania się do stringów (tablic znaków) lub ich fragmentów bez konieczności alokacji pamięci.

Typ `std::string_view` jest zdefiniowany w nagłówku `<string_view>`.

```c++
#include <string_view>

const char* txt_c = "text";
std::string_view txt_sv = c_txt; // txt_sv refers to txt_c c-string
```

```c++
std::string txt_str = "text";
std::string_view txt_sv = txt_str; // txt_sv refers to txt_str string
```

```c++
void print_text(std::string_view txt)
{
    std::cout << txt << std::endl;
}

print_text("text"); // prints "text"
print_text(std::string("Text")); // prints "text"

const char* txt = "Hello World!!!";
print_text(std::string_view{txt, 6}); // prints "Hello"
```

## Najważniejsze cechy std::string_view

* Czas życia danych (bufora znaków) nie jest kontrolowany przez obiekt typu `string_view`

* Brak wsparcia dla alokatorów - nie są potrzebne

* Przekazywanie przez wartość do funkcji jest efektywne

  * typowa implementacja: wskaźnik na stały znak (`const char*`) i rozmiar

* Zdefiniowane są odpowiedniki dla innych typów znakowych niż `char`:

  * `std::wstring_view` - dla typu `wchar_t`
  * `std::u16string_view` - dla typu `char16_t`
  * `std::u32string_view` - dla typu `char32_t`

* Literał: `sv`

  * zdefiniowany w nagłówku `<literals>`
  * zdefiniowany jako `constexpr`

* Obiekt `string_view` zapewnia podobną funkcjonalność jak `std::string`:

  * `operator[]`
  * `at()`
  * `data()`
  * `size()`
  * `length()`
  * `find()`
  * `find_first_of()`
  * `find_last_of()`

* Zapewnia operatory porównania i wyliczania skrótu (`std::hash<std::string_view>`)

## Różnice między string_view a string

* Wartość po konstrukcji domyślnej dla wewnętrznego wskaźnika to `nullptr`

  * `string::data` nie może zwrócić `nullptr`

    ```{code-block} cpp
    string_view txt;

    assert(txt.data() == nullptr);
    assert(txt.size() == 0);
    ```

* Typ `string_view` nie ma gwarancji, że bufor znaków jest zakończony zerem (*null terminated string*)

  ```{code-block} cpp
  char txt[3] = { 't', 'x', 't' };

  string_view txt_v(txt, sizeof(txt)); // this view is not null terminated
  ```

```{warning}
Dla `string_view` zawsze należy sprawdzić rozmiar operacją `size()` zanim użyty zostanie `operator[]` lub wywołana zostanie metoda `data()`
```

## Konwersje string `<->` string_view

* Konwersja `string -> string_view` jest szybka

  - dozwolona niejawna konwersja przy pomocy `std::string::operator string_view()`

* Konwersja `string_view -> string` jest kosztowna

  - wymagana jawna konwersja - `explicit std::string::string(string_view sv)`

## Użycie string_view w API funkcji

* Obiekty `string_view` przekazywane jako argumenty wywołania funkcji powinny być przekazywane przez wartość.

* `std::string_view` powinno być stosowane zamiast `string` jeśli:

  * API nie wymaga, aby tekst był zakończony zerem
    * nie można przekazywać `std::string_view` do funkcji języka C
  
  * odbiorca respektuje czas życia obiektu
  
  * dostęp do danych przy pomocy metody `data()` uwzględnia potencjalny pusty wskaźnik (`nullptr`)

* Należy unikać zwracania `string_view`, chyba że jest to świadomy wybór programisty
  
  * zwrócenie `string_view` może być niebezpieczne - należy pamiętać o tym, że `string_view` jest **non-owning view**

    ````{warning}
    ```{code-block} cpp
    std::string_view get_first_token(std::string_view text)
    {
        return text.substr(0, text.find(' '));
    }
    ```

    Jeśli wywołamy funkcję `get_first_token()` w następujący sposób:

    ```{code-block} cpp
    std::string_view token = get_first_token("Hello World!!!");
    ```

    Dostaniemy instancję `std::string_view` z wiszącym wskaźnikiem odnoszącym się do nieaktualnej już tablicy znaków, która 
    została zwolniona w momencie wyjścia z funkcji.
    ````

* Dostarczanie obydwu wersji funkcji jako przeciążeń może powodować dwuznaczności:

  ```{code-block} cpp
  void foo(const string& s);

  void foo(string_view sv);

  foo("ambigous"); // ERROR - ambigous call
  ```
