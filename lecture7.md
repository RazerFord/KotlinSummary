### Замыкания

Изменять состояние переменной не можем, потому что значение неизменяемой переменной примитивного типа можно добавить в замыкание. С адресом - сложнее, он на стеке. Объект - можно, так как он в куче

Обход ограничений с помощью `wrapper`

```Java
public static Pair<IntConsumer, IntSupplier> create() {
    AtomicInteger v = new AtomicInteger();
    return new kotlin.Pair<>(n ->
    v.addAndGet(n), () -> v.get()
    );
}
```

Необходимо, чтобы переменная была `effectively final`

Так нельзя

```Java
public static Pair<IntFunction, IntFunction>
createPair(int delta) {
    IntFunction f1 = n -> n + delta; // ошибка компиляции
    delta *= 2;
    IntFunction f2 = n -> n + delta; // ошибка компиляции
    return new Pair<>(f1, f2);
}
```

Так можно

```Java
public static Pair<IntFunction, IntFunction>
createPair(int delta) {
    IntFunction f1 = n -> n + delta;
    delta2 = 2 * delta;
    IntFunction f2 = n -> n + delta;
    return new Pair<>(f1, f2);
}
```

В `Kotlin` почти также. Только функциональные типы унифицированы. И можно работать с переменной "напрямую"

```kotlin
fun create(): Pair<(Int) -> Unit, () -> Int> {
    var v = 0
    return Pair({ n -> v += n}, { v })
}
```

Под капотом - примерно то, что сделано в Java - примере. Для `v` создается объект класса `IntRef`, и у него есть поле `v.element = 0`. `IntRef` уже и передается в лямбды

```kotlin
fun createPair(delta: Int):
Pair<(Int) -> Int, (Int) -> Int> {
    val f1 = { n: Int -> n + delta }
    val d = delta * 2
    val f2 = { n: Int -> n + d }
    return Pair(f1, f2)
}
```

### Делегирование

Допустим есть некоторый класс `A`, который реализует какой-то интерфейс. Необходимо определить еще один класс `B`, который реализует этот интерфейс, и внутри себя будет дергать методы класса `A`, чтобы не повторять реализацию некоторых методов интерфейса. В `Kotlin` есть синтаксический сахар для этого

Шаблон делегирования является хорошей альтернативой наследованию, и `Kotlin` поддерживает его нативно, освобождая вас от необходимости написания шаблонного кода

Класс `Derived` может реализовать интерфейс `Base`, делегируя все свои `public` члены указанному объекту

```kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() {
        println(x)
    }
}

class Derived(b: Base) : Base by b

fun main() {
    Derived(BaseImpl(12)).print()
}
```

Ключевое слово `by` в оглавлении `Derived`, указывает, что `b` будет храниться внутри экземпляра `Derived`, и компилятор сгенерирует все методы из `Base`, которые при вызове будут переданы объекту `b`

Делегированные методы перегружаются обычным способом

При наследовании метод суперкласса может реализовать крупную схему поведения. При делегировании это в лучшем случае усложняется

### Nullable

В `JVM` ссылки могут быть `null`, в результате можем получить ошибку в `runtime`

Возможное решение: вставлять проверки, использовать `option` с двумя вариантами `Some(value)` и `None`

Путь `Kotlin` - для каждого типа есть `nullable`-модификация, обозначается вопросиком на конце, например `String?`

Смысл в том, чтобы изолировать преобразования типов данных. От тех точек, в которых может быть получен `null`

Например, при получении значения по ключу, если ключа нет, то может вернуться `null`

При явной проверке на `null` в ветке, где значение гарантированно не `null` - оно рассматривается как значение обычного типа. С теми же оговорками, как для умного приведения

```kotlin
fun main() {
    val name: String? = getName()
    if (name != null) {
        name.length // работает как со String
    }
//    name.length ошибка компиляции
}
```

`?.` (`safe call`) - обращение к методу или свойству если не `null`, иначе вернет `null`

```kotlin
println(order?.date?.month)
```

### let

`let` - стандартное расширение. Применение блока к данному объекту. Результат блока - результат `let`. Идиоматично применяется в связке с `safe call`

```kotlin
val name: String? = getName()

println(name.let { it ?: "" }.length)

null.let {
    println(it)
}
```

```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this)
}
```

### `run`

То же самое, что и `let`

### `also`

То же самое, что и `let`, но без возвращаемого значения

```kotlin
public inline fun <T> T.also(block: (T) -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block(this)
    return this
}
```

### `apply`

`apply` - примерно как `also`. Только не-`null` значение предстает как `this`

### `!!`

Принудительно приводит `nullable` в обычный тип. Если переменная `null`, то бросает `NPE`

### `?:`

`?:` - элвис оператор. Если переменная `null`, то возвращает второй операнд, иначе первый

```kotlin
println(name ?: "default")
```

### Задача

Есть список ключей. Хотим для каждого ключа обратиться в словарь. Получится список элементов типа Person? Хотим сделать map для непустых.

Можно использовать - `filterNotNull` и `mapNotNull`