[[Start Page|На главную]]

---
# Стирание типов
Стирание типов: (void*) reinterpret_cast<void*>(bar);
Оно так же присутствует в std::functions, std::any, std::variants, std::string_view и там где используется полиморфизм. Стирание типов ещё называется type erasure

# lvalue, rvalue...
lvalue и rvalue - это категории выражений, влияющих на семантику работы с объектами. rvalue - это временное значение в памяти (литерал). lvalue постоянное именованное значение в памяти (переменная). xvalue (expiring value) - это значение gvalue, которое обозначает объект, ресусры которого можно повторно использовать. Например std::move(x). gvalue (generalized value) - это выражение, оценка которого определяет идентичность объекта или функции.

# declval, decltype
`declval<T>()` возвращает rvalue ссылку на объект класса, не вызывая конструктор. Таким образом можно извлекать тип членов класса без конструирования объекта посредством decltype(). Пример:

```cpp
#include <utility>

class A {
public:
	A() = delete;
	int foo() {
		return 7;
	}
};

int main(void) {
	decltype(std::declval<A>().foo()) variable = 7; // int
	return 0;
}
```
# Typename
typename позволяет явно указать компилятору, что мы имеем дело с типом, а не с полем класса (например когда мы работаем с шаблонами). Пример

```cpp
template<class T>
void foo() {
    typename T::boo variable = 7;
}

class A {
public:
    typedef unsigned long long boo;
    boo variable;
};  

class B {
public:
    double boo;
};

int main(void) {
    foo<A>(); // OK
    // foo<B>(); // Error: B class doesn't have boo type
    return 0;
}
```
