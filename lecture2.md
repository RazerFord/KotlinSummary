### Строки

Определение строковых литералов:
```kotlin
val str1 = "hello world"
val str2 = """
    hello world
    """ // в строке будут содержаться все символы между `"""` включая спец символы и другие
```
Различные методы над строковыми литералами:
```kotlin
val str2 = """
    hello world
    """.trimIdent()
```

`.trimIdent()` - обнаруживает общий минимальный отступ для всех строк ввода, удаляет его из каждой строки, а также удаляет первую и последнюю строки, если они пустые

```kotlin
val str2 = """
    hello world
    """.trimIdent()
```
`.trimMargin(prefix="|")` - из каждой строки исходной строки удаляет префикс до символа `prefix` включительно, и удаляет первую и последнюю строки, если они пустые
```kotlin
val str2 = """
    |hello world
    """.trimMargin() // "hello world" 
```

```kotlin
"a" in str // аналогично "a".contains(str)
"a" !in str // аналогично !("a".contains(str))
```

```kotlin
"abc".zipWithNext() // [(a, b), (b, c)]
```
`.zipWithNext()` - возвращает список пар каждых двух соседних элементов в этой коллекции.

Вложенные интерполяции:
```kotlin
val a = "hello"
val d = "world"
println("${a"${d}"}")
```

`it` - синтаксический сахар для `e -> e` в `lambda`
```kotlin
"abc".forEach {
    it // e -> e
}
```

`StringBuilder` - несинхронизированный
`StringBuffer` - синхронизированный

### Функции

Функция может определяться: внутри класса (`kotlin`-метод), вне класса, внутри блока кода

```kotlin
fun main() {
    val func: () -> Unit = fun() {
        println("hello world")
    }
    func()
    fun proc() {}
}
```
Если функция не возвращает никакого полезного значения, её возвращаемый тип - `Unit`. `Unit` - тип только с одним значением - `Unit`. `Unit` аналог `void` в `Java`. В `Kotlin` тип возвращаемого значения в качестве `Unit` указывать не принято

### Модификаторы доступа

`private` — если это метод класса, то он доступен в пределах одного класса. Если это глобальная функция, то доступна внутри одного файла.

`protected` — если это метод класса, то он доступен в пределах класса и его наследников.

`internal` — доступ к членам модуля (module). Если это глобальная функция, то доступна внутри одного модулю.

`public` — доступно везде. 

Если явно не использовать никакого модификатора, то по умолчанию применяется `public`. Функции объявленные внутри блока только `public`

Также по умолчанию функции являются финальными

### Традиционный способ определения функций
```kotlin
fun test(arg1: String, arg2: Int[, name: Type]): Int { // ключевое слово `fun`, список аргументов, возвращаемое значение после `:`
    ...
}
```
### Функции с одним выражением
После сигнатуры функции пишется `=` и одно выражение. Тип функции может автоматически выведен, если нет рекурсии

```kotlin
fun test(arg1: String, arg2: Int[, name: Type]): Int = ...
```

### Анонимные функции

- Могут быть без параметров, тогда этом просто фигурные скобки с предложениями.
```kotlin
val sum: () -> Int = { a + b }
```
- Тип последнего выражения в теле анонимной функции будет типом самой анонимной функции
```kotlin
val test: () -> String = { 
    a + b 
    "lol"
}
```
- Если в конце не выражение, то анонимная функция возвращает Unit
```kotlin
val test: () -> Unit = {}
```

```kotlin
val f6: () -> String = ::produceString // если produceString определена как "не-метод". Ищет ближайшую по scope функцию
```

Вызов анонимной функции:

```kotlin
f2(1, 2)
f2.invoke(1, 2)
```
Под капотом каждая анонимная функция - это анонимный класс с методом `invoke`

### Коварный случай

```kotlin
fun test() = { 10 }
test() // lambda
test()() // 10
```

### Анонимные функции с одним параметром

Это функции вида

```kotlin
val incr: (Int) -> Int = { v -> v + 1}
val incr1 = { v: Int -> v + 1} // компилятор выводит тип
```

Если функция принимает один аргумент, то существует синтаксический сахар в виде `it`. Если есть `e -> e`, то можем заменить на `it`

```kotlin
val incr2: (Int) -> Int = { it + 1}
```

### Классы

Можно определять следующим образом
```kotlin
class C
class C {}
class C() {}
class C(val x: Int, val y: Int) {} // создает класс, у которого будут свойства `x` и `y`. По умолчанию публичные
```
Указание конструктора после имени класса называется первичный конструктор. Он присваивает значения которые передали в конструктор свойствам, которые находятся на позиции `val`. Тело первичного конструктора можно написать в блоке инициализации

Сразу описываем публичный конструктор и сразу создаем поле с именем и типом, равным параметру конструктора, `Kotlin` порождает конструктор, который копирует параметр в поле, и порождает `get`-метод, а поле будет `final` в `JVM`

Логика порожденного конструктора формируется из кусочков, сначала - инициализация полей, отмеченных как val в списке параметров, а дальше идут кусочки, соответствующие `val` объявлениям из тела класса, и `init`-блокам в том порядке, как они идут

Если в первичном конструкторе поле не объявлено как `var` или `val`, то он не создастся в классе. 

```kotlin
class Test constructor(val x: Int) {
    val y: Int
    init {
        y = x + 10
    }

    constructor() : this(1) {...} /// Вторичный конструктор. Должен вызывать первичный при его наличии
}
Test(10).x // будет вызов геттер метода для `x`
```

Если метод не берет параметры, то лучше его сделать `property` c `getter`
