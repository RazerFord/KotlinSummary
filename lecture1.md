## Базовая структура `Kotlin`-программы

В `Kotlin` бывает код вне класса. Это достигается за счет того, что компилятор смотрит в каком файле находится класс и генерирует класс с именем `НазваниеФайлаKt.class` внутрь которого ложит функцию.

### Пример 1

```kotlin
fun main() {
    println("Hello, World!")
}
```

Для функции `main()` будет сгенерирован класс, внутри которого будет статическая функция, вызывающая `main()`, описанный выше:

```Java
public static void main(String[] var0) {
    main();
}
```

### Пример 2

```kotlin
fun main(arg: Array<String>) {
    println("Hello, World!")
}
```

Для функции `main(Array<String>)` будет сгенерирован класс, внутри которого будет тело описанной выше функции:

```Java
public static void main(String[] var0) {
    System.out.println("Hello, World!");
}
```

### Другие сведения

- Точки с запятой в конце не нужны.
- Массивы это класс `Array` с дженерик параметром. При компиляции `Array<T>` превращается в обычный `Java`-массив:
```kotlin
val arr: Array<String>
```
- То что находится в `System.out` или `java.lang` можно вызывать как обычную функцию
```kotlin
println("Hello, World!") // System.out.println("Hello, World!")
```
- `Kotlin` - компилятор сам выводит тип
- `Kotlin` - статический язык со строгой типизацией
- `val` не изменяемые переменные. В `Java` это `final var`
- `var` изменяемые переменные
- Все исключения, как `RuntimeException` в `Java`
- Если функция принимает в качестве последнего параметра лямбду, то чтобы ее передать, можно после вызова функции написать лямбду `{ ... }`:
```kotlin
fun test(f: (Int) -> Int){
    f(10)
}

fun main() {
    test {
        it
    }
}

```

### Примитивные типы

- Int, Long, Byte, Char, Float, Double, Boolean - представляют собой классы. Компилятор старается подставить примитивные типы вместо классов

### Строковая интерполяция

```kotlin
val str = "name"
val compStr = "Hello, $str" // Hello, name
```

### Свойства

```kotlin
class Address {
    val name: String = "Holmes" // неизменяемые
    var city: String = "Baker" // изменяемые
}
```

Для того, чтобы воспользоваться достаточно просто обратиться к нему

```kotlin
val address = Address()
println(address.name)
```

Для свойств можно задать `getter` и `setter`:
```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```
```kotlin
class Rectangle(val width: Int, val height: Int) {
    val area: Int
        get() = this.width * this.height // тип свойства необязателен, поскольку он может быть выведен из возвращаемого типа геттера
        set() = ...
}
```

### Range

```kotlin
1..10 // [1, 10]
1..<10 // [1, 10)
1 until 10 // [1, 10)
10 downTo 1 // [10, 1]

1..10 step 3 // [1, 10]
1..<10 step 3 // [1, 10)
1 until 10 step 3 // [1, 10)
10 downTo 1 step 3 // [10, 1]
```

### Побитовые операции

```kotlin
shl(bits) – сдвиг влево с учётом знака (<< в Java)
shr(bits) – сдвиг вправо с учётом знака (>> в Java)
ushr(bits) – сдвиг вправо без учёта знака (>>> в Java)
and(bits) – побитовое И
or(bits) – побитовое ИЛИ
xor(bits) – побитовое исключающее ИЛИ
inv() – побитовое отрицание
```

```kotlin
1 shl 10
32 shr 2
32 ushr 2
1 and 12
6 or 1
6 xor 7
5 inv 4
```