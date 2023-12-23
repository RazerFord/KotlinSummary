### Статика

Конструкция `object`. Ставится там же, где класс. Круглых скобок нет. Одно пространство имен с классами. Под капотом есть поле `INSTANCE`. Инициализируется в статическом инициализаторе и потокобезопасности. С ленивостью посложнее. Но может использоваться как `namespace` для
статических конструкций

```kotlin
object Singleton {
    val VALUE = 12345

    init { // блок статической инициализации
        println("I'm singleton")
    }
}
object Other {
    val ownValue = 23456;
    val value = Singleton.VALUE
    init {
        println("I'm other")
    }
}
fun main() {
    println(Other.ownValue)
}
// hello
// I'm singleton
// I'm other
// 23456
```
Начали работу. Класс `Other` нам известен, но не инициализирован. Только когда `Other.ownValue` стал нужен, начинаем инициализировать `Other`. И нам становится нужен `Singleton.VALUE`. Тут инициализируем `Singleton`. Печатаем "I'm singleton". Доходим до печати "I'm other". Инициализировали `Singleton` - хотя в `main` им не воспользовались

### Примеры использования

- Собираем данные из конфиг файлов в синглтон `Config`
- Создаем сложный компаратор (объект без состояния)
- Формируем `namespace` для констант/свойств/функций

### Компаньоны

Объединение методов и свойств, не привязанных к одному объекту, но связанных с классом. Способ добавить статику в класс

Но также можно обычный объект поместить внутрь класса. Только надо явно указать его имя

```kotlin
class Test(private val x: Int) {
    object Static {
        fun test(x: Test) {
            println(x.x)
        }
    }
    companion object {
        fun test(x: Test) {
            println(x.x)
        }
    }
}
```

Под капотом для `companion object` вместе с классом `Companion` создается статическое поле `companion: Companion` в классе `Test`. Для `object` создается класс внутри `Test` и поле `INSTANCE` внутри `object Static` 

Элементы компаньона "напрямую" доступны из других методов класса, даже если методы компаньона приватные. Приватные же методы `object` не доступны из методов внешнего класса. Но все приватные методы и свойства доступны в компаньоне и в `object`.

```kotlin
class Test(private val x: Int) {
    fun test() {
        Static.test() // нет доступа
    }

    object Static {
        private fun test(x: Test) { // приватный метод
            println(x.x)
        }
    }
}
```

### Расширения

Имеется класс `Point`. Хочется реализовать функцию, что будет создавать новый объект, и будет это делать в ООП стиле. Можно использовать расширения `Kotlin`. Для этого к имени функции нужно приписать имя класса - слева через точку, и потом вызвать этот дополнительный метод/

Если в классе есть и функция-член, и функция-расширение с тем же возвращаемым типом, таким же именем и применяется с такими же аргументами, то функция-член имеет более высокий приоритет

```kotlin
class Example {
    fun printFunctionType() { println("Class method") }
}

fun Example.printFunctionType() { println("Extension function") }

Example().printFunctionType() // Class method
```

```kotlin
open class C1
class C2() : C1()
fun C1.m() = println("C1.m")
fun C2.m() {
    println("C2.m")
}
fun f(c1: C1) = c1.m()
fun main() {
    f(C1()) // C1.m
    f(C2()) // C1.m
}
```

Какая функция-расширение будет вызвана определяется на этапе компиляции

### `infix`

Ключевое слово `infix` перед определением функции или расширения делает возможным вызов функции в инфиксном стиле. Работает с рашши

```kotlin
class Test {
    infix fun test(x: Int): Int { // либо так создается `infix` fun
        return x
    }
}
infix fun Test.test(x: Int): Int { // либо так создается `infix` fun
    return x
}
fun main() {
    println(Test() test 42)
}

```
