### Циклы

Классические `while` и `do-while` циклы. Выражениями не являются, но часто ведут себя как выражения с типом `Unit`. В цикле `do-while` можно использовать локальную переменную в условии.

```kotlin
fun value() = 5
val f3: () -> Unit = {
    value()
    while (Math.random() < 0.5) println()
}
```

Классический `for` делается через `range`, также используют через итератор
```kotlin
for (v in data) { println(v) }
for (i in 0..10) { println(i) }
```

Существуют метки для циклов. С помощью меток можно выйти через несколько циклов.
```kotlin
test@while(true) {
    var i = 0
    while (true) {
        while (true) {
            i++
            if (i == 10) {
                break@test // continue or break
            }
        }
    }
}
```

### Условные операторы

`if` - выражением не является. Но `if () {...} else { ... }` - является выражением. Значением ветки является значение последнего выражения. Или Unit, если последнее предложение - не выражение. Типом будет наиболее точный супертип типов результатов веток.

```kotlin
val result = if (b) 10 else 42
```

`when` аналог `switch-case`. `when` не требует `break`. Можно указывать варианты выражения, а можно нет:

```kotlin
when(cond) {
    2 -> 28
    1, 2, 3, -> 42
    is String -> 58
    else -> 97
}

when {
    cond == 2 -> 28
    cond is String -> 58
    else -> 97
}
```

Аналогично в `when` можно использовать `in`, `!in`, `is`, `!is`. Если `when` обрабатывает все случаи, то он может использоваться, как выражение, которое возвращает наиболее общий супертип. Последнее выражение любой ветки `when` - возвращаемое значение. Позволяет использовать со `smart cast`

```kotlin
val x = when(p) {
    is String -> 10
    else -> 42
}
```

### Smart cast
Не работает в следующих случаях:
- если есть противоречие в возможных вариантах
- разрешение типа не по силам компилятору
- c нелокальными var-ами
- со свойствами со своим get()
- везде, где нет гарантий "атомарности"

```kotlin
if (i is String || i is Int) { // не Smart cast, так как есть противоречия

}
```
Сложный `smart cast` не работает

```kotlin
    if (x is Int || x is String) {
        if (x is Int) {
            return
        }
        // x is String? не работает
    }
```

### Data классы

Нередко мы создаём классы, единственным назначением которых является хранение данных. Функционал и некоторые служебные функции таких классов зависят от самих данных, которые в них хранятся. В `Kotlin` они называются классами данных и помечены `data`

Компилятор автоматически формирует следующие члены данного класса из свойств, объявленных в основном конструкторе:

- пару функций `equals()/hashCode()`,
- функцию `toString()`
- компонентные функции `componentN()`, которые соответствуют свойствам, в соответствии с порядком их объявления,
- функцию `copy()`

```kotlin
data class PersonData(val name: String, val age: Int)
val pd2 = PersonData("vasya", 23)
val pd3 = pd2.copy(name="petya")
```

### Статика

В класса нет никакой статики: полей, функций, классов

Статика на уровне файла:
- `val/var`. Это аналог статических полей. Можно реализовать `get/set`. И это может быть свойство с полем и без поля
- Перед `val` (без `const` создастся `getter` для приватного поля) можно указать `const` - аналог `static final`. Применимо в статическом контексте. Никаких своих `get`

### Инициализация `const val` - константа времени компиляции

`const val` - константа времени компиляции

Инициализируется только статически известными значениями. Можно позволить себе выражения, даже конкатенацию. Аргументами могут быть другие `const val`

### Пакеты

- В начале файла указывается имя пакета после `package`
- В разных файлах можно указать один `package`
- Все, что определяется в разных файлах одного `package`-а, "видит" друг друга
- Если пакет не указан, то это специальный пакет "умолчанию"
- Из других пакетов его элементы должны быть явно импортированы

```kotlin
import abraabra // лежит в пакете с именем по

fun main(arg: Array<String>) {
    abraabra()
}
```