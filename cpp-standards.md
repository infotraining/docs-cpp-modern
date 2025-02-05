# Standardy ISO C++

Standardy ISO C++ definiują specyfikacje dla języka programowania C++. Te standardy są utrzymywane przez Międzynarodową Organizację Normalizacyjną (ISO) poprzez ISO/IEC JTC1/SC22/WG21 (Grupa Robocza C++). Standardy zapewniają, że C++ pozostaje spójne, a kod napisany zgodnie z jednym standardem może być kompilowany i uruchamiany poprawnie na różnych kompilatorach i systemach obsługujących ten standard.

Oto kluczowe standardy C++:

* **C++98** (ISO/IEC 14882:1998): Pierwsza znormalizowana wersja C++. Wprowadziła kilka funkcjonalności, takich jak szablony, wyjątki, przestrzenie nazw i Standard Template Library (STL).

* **C++03** (ISO/IEC 14882:2003): Wydanie naprawiające błędy w C++98. Dokonano drobnych dostosowań i ulepszeń języka na podstawie opinii z pierwotnego standardu C++98.

* **C++11** (ISO/IEC 14882:2011): Znacząca aktualizacja, która wprowadziła wiele nowych funkcjonalności, w mechanizm dedukcji `auto` i `decltype`, pętle for dla zakresów, wyrażenia lambda, inteligentne wskaźniki i wsparcie dla wielowątkowości.

* **C++14** (ISO/IEC 14882:2014): Aktualizacja przyrostowa do C++11, wprowadziła drobne ulepszenia i poprawki błędów, w tym lambdy generyczne i dedukcję typów zwracanych z funkcji.

* **C++17** (ISO/IEC 14882:2017): Dodano funkcje takie jak *structured bindings*, `if constexpr`, *wyrażenia fold* oraz Parallel-STL.

* **C++20** (ISO/IEC 14882:2020): Znacząca aktualizacja z wieloma nowymi funkcjonalnościami, takimi jak koncepty, biblioteka Ranges, korutyny oraz moduły.

* **C++23** (ISO/IEC 14882:2023): Kilka nowości w rdzeniu języka (this jako parametr metody, wielowymiarowy operator indeksowania, statyczny operator `[]` oraz `()`) oraz w bibliotece standardowej (`std::mdspan`, `std::generator`, `std::print`).

Każdy z tych standardów ma na celu poprawę języka, uczynienie go bardziej potężnym, wydajnym i łatwiejszym w użyciu, przy jednoczesnym zachowaniu kompatybilności wstecznej.

## Dokument standardu C++

Darmowa wersja bliska wersji finalnej standardu C++17:
  - [Working Draft, Standard for Programming Language C++](https://wg21.link/n4659)

## Stała __cplusplus

Stała preprocesora `__cplusplus` definiuje, która wersja standardu jest dostępna w trakcie kompilacji. Wartości, jakie może przyjmować, to:

| Wersja standardu  | Stała     |
| ----------------- | --------- |
| dla C++98 i C++03 | `199711L` |
| dla C++11         | `201103L` |
| dla C++14         | `201402L` |
| dla C++17         | `201703L` |
| dla C++20         | `202002L` |
| dla C++23         | `>20202L` |

Najczęściej używamy tej stałej do warunkowego kompilowania kodu w zależności od wersji standardu C++:

```cpp
#include <iostream>

int main() {
    // Check the value of __cplusplus to determine the C++ standard version
    #if __cplusplus == 199711L
        std::cout << "C++98 or C++03" << std::endl;
    #elif __cplusplus == 201103L
        std::cout << "C++11" << std::endl;
    #elif __cplusplus == 201402L
        std::cout << "C++14" << std::endl;
    #elif __cplusplus == 201703L
        std::cout << "C++17" << std::endl;
    #elif __cplusplus == 202002L
        std::cout << "C++20" << std::endl;
    #elif __cplusplus > 202002L
        std::cout << "C++23 or later" << std::endl;
    #else
        std::cout << "Unknown C++ version" << std::endl;
    #endif

    return 0;
}
```
