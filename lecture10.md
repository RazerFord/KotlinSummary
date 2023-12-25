### Async/Await

```kotlin
fun main() {
    runBlocking(Dispatchers.Default) {
        launch {
            // выполнятся параллельно
            val result = async {
                delay(Random.nextLong(3000))
                123
            }
            val result1 = async {
                delay(Random.nextLong(3000))
                123
            }
            result.await() // дождаться результат
        }
    }
}
```

- `suspend`-функции выполняются по очереди. Пусть `suspend`-функция что-то скачивает. И мы хотим скачать что-то из двух мест и сагреггировать. Два вызова `suspend`-функции внутри корутины - два последовательных скачивания. А две `async`-корутины - это два параллельных скачивания

```kotlin
fun main() {
    runBlocking(Dispatchers.Default) {
        launch {
            // следующие две `suspend` функции 
            // выполнятся последовательно
            test()
            test()
        }
    }
}
```

### `awaitAll`

Есть `awaitAll` для списка `Defered`. Но он привередлив в плане исключений. Одно падение - падение всего сразу. Если это не устраивает - надо отдельно обрабатывать

### Пример с обработкой исключений

```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
fun main() {
    val data = listOf(1, 2, 5, 0)
    runBlocking(Dispatchers.Default) {
        fun sleepAndDiv(v: Int) =
            async(SupervisorJob()) {
                println("start $v")
                delay(Random.nextLong(5000))
                println("after delay $v")
                5 / v
            }
        launch {
            val result = data.map {
                sleepAndDiv(it)
            }.map {
                // исключение перехватываем здесь
                try {
                    it.await()
                } catch (exc: ArithmeticException) {
                    println(
                        it.getCompletionExceptionOrNull()
                    )
                    null
                }
            }
            println("result: $result")
        }
    }
}
```

```kotlin
fun main() {
    val data = listOf(1, 2, 5, 0)
    runBlocking(Dispatchers.Default) {
        fun sleepAndDiv(v: Int) =
            async(SupervisorJob()) {
                try {
                    println("start $v")
                    delay(Random.nextLong(5000))
                    println("after delay $v")
                    5 / v
                } catch (exc: ArithmeticException) {
                    null
                }
            }
        launch {
            val result = data.map {
                sleepAndDiv(it)
            }.map {
                it.await()
            }
            println("result: $result")
        }
    }
}
```

### `Flow` - итераторы `Python`

`flow` - конструктор. Возвращает реализацию интерфейса `Flow` - с `suspend`-методом `collect`
- Параметр билдера - блок, исполняющийся для извлечения элемента
- `emit` - `suspend`-метод. Можно увидеть, что в примере `emit` и `collect` работают в одной корутине

```kotlin
fun main() {
    runBlocking {
        launch {
            val flow = randomData(10, 200)
            println("before collect")
            // пока не вызовется этот метод flow не выполнится
            // тело flow и эта корутина работают в одной Job
            flow.collect { println(it) }

            // вернет первый элемент
            flow.first()

            // снова создаст `flow` и вернет первый элемент
            flow.first()
        }
    }
}

fun randomData(count: Int, delayMs: Long) = flow {
    println("start flow")
    repeat(count) {
        delay(delayMs)
        // кладем данные с помощью emit
        emit(Random.nextInt(100))
    }
}
```

Когда вызывается `flow`, создается `Continuation`, который способен останавливаться в этом блоке кода. Когда вызываем `collect()`, то вызывается `suspend`, передается управление `Continuation`, который есть `Flow`. После доходит до `emit`, после его вызова контекст переключится обратно. `Flow` ленивый. При создании `Flow` объект не создается. При вызове `collect`, создается объект, по которому итерируемся. `collect()` будет работать пока `count` не закончится, если снова вызвать `collect()`, то он начнет работу сначала.

- `Flow` - не корутина. Это реактивная структура
- Она умеет создавать `Continuation` над переданным блоком кода, и передавать управление на него из разных корутин

- это похоже на `Python`-генератор
- только более ленивый и более ограниченный в логике итерирования
- `Python`-генератор сразу создает объект, а потом умеет по запросу идти до следующего `yield`
- `Flow` оттягивает момент создания объекта до
начала итерирования
- И то, что возвращает `flow-builder` - это нечто,
умеющее порождать итератор. Реально итератор создается при вызове `collect` или одного из производных методов. В случае `collect` итератор работает, пока код, переданный в конструктор `Flow`, не завершится
- Есть метод `first`, возвращает то, что породил первый `emit`, но следующий вызов `first` снова создаст новый итератор. Тут `Flow` похож на `Java-stream` с терминальными методами

### Нетерминальные методы. Их вызовы не создают `Flow`

- Стандартный набор: `map`, `filter`, `flatMap`, `take`, `dropWhile` и т.п.
- Реактивно-специфичные: `debounce`, `sample`
- `debounce` получает параметром временной интервал, если после `emit`-а времени прошло
больше интервала, то значение запоминается
- `sample` делит время на интервалы, если в интервал попало больше одного, то в
поток попадет последнее
- `timeout` - бросает исключение, если ничего не
породилось в течение заданного таймаута (если работа завершена - то для timeout это `ok`)

```kotlin
suspend fun main() = runBlocking {
    flow {
        repeat(10) {
            emit(it)
            delay(160)
        }
    }.sample(200).collect(::println)
}
```

- *Терминальные* `fold/reduce`
- Накопительные `runningFold/runningReduce`. Получаем поток промежуточных значений. Может пригодиться в мотивирующем примере. Сделать накопительную свертку с количеством по ключевым словам и над ней вызвать `firstOrNull`

### Терминальные методы

- `collect`
- производный: `single` - первый элемент с проверкой, что нет других. Бросает исключение
- производный `first` - первый, возможножно,
что из многих
- `singleOrNull`, `firstOrNull`
- `toList`, `toSet` - могут зависнуть на бесконечном `Flow`

### Callback - стиль

- `onEach` - нетерминальный метод, описывающий действия над элементом
- `onCompletion` - действия по завершении `onStart`, `onEmpty` - понятно
В `on`-методах можно делать `emit`

```kotlin
suspend fun main() = runBlocking {
    val x = flow {
        repeat(10) {
            emit(it)
            delay(160)
        }
    }.sample(200).onStart {
        println("start 1")
    }.onStart {
        println("start 2")
    }.onStart {
        println("start 3")
    }.collect(::println)
}
```

### Один Flow в разных корутинах

- Никаких проблем - можно из разных корутин вызывать методы одного экземпляра одного `Flow`. Когда дойдет до терминального - для каждой будет создан свой итератор. И в каждом будет свое состояние. А после терминального следующий терминальный породит новый итератор и новое состояние

```kotlin
val random = Random(42)
fun main() {
    runBlocking(Dispatchers.Default) {
        launch {
            val flow = randomData(5, 200)
            repeat(100) {
                launch { println(flow.toList()) }
            }
        }
    }
}
```

Может быть удобным запустить Flow в своей корутине. Хотя бы для того, чтобы остановить итератор по cancel. Например, по таймауту и по сложному критерию, который через цепочку нетерминальных сложно выразить

### `launchIn`

- Есть отдельный терминальный метод `launchIn`
- Принимает параметром контекст. В этом контексте запускает корутину. В теле корутины запускается `collect`
- `launchIn` возвращает `Job`

```kotlin
suspend fun main() = runBlocking {
    val job = flow {
        repeat(10) {
            emit(it)
            delay(160)
        }
    }.sample(200).onStart {
        println("start 1")
    }.onStart {
        println("start 2")
    }.onStart {
        println("start 3")
    }.launchIn(this)
    job.join()
}
```

### `withTimeoutOrNull`

- Есть универсальный способ вызвать `suspend`-функцию с таймаутом
- Функция `withTimeoutOrNull`
- Внутри себя создает корутину, в ней вызывает функцию
- По истечении таймаута вызывает `cancel`

```kotlin
suspend fun main() = runBlocking {
    repeat(10) {
        // по таймауту вырубает корутину
        withTimeoutOrNull(10000) {
            randomData(10, 1000).collect(::println)
        }
    }
}


fun randomData(count: Int, delayMs: Long) = flow {
    println("start flow: ${currentCoroutineContext().job}")
    println("start flow")
    repeat(count) {
        delay(delayMs)
        emit(Random.nextInt(100))
    }
}
```

### Canceling flow

- Чтобы пользовательский код `Flow` смог услышать призыв остановить работу
- В типовых `Flow` такой проблемы не будет. Но бывают специальные `Flow` - получающиеся из обычных коллекций. На них могут работать вычислительные задачи. И они не услышат `cancel`. Можно вызвать `cancelable()` над `Flow`. И тогда проверка выполнится при заборе одного из значений

```kotlin
fun createFlow(): Flow<Int> = flow {
    for (i in 1..10) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking(Dispatchers.Default) {
    val flow = createFlow()
    supervisorScope {
        launch {
            flow.onEach { value ->
                if (value == 3) cancel() // работает
                println(value)
            }.launchIn(this)
        }
    }
    println(1111)
    delay(1000)
}
```

```kotlin
fun createFlow(): Flow<Int> = (1..10).asFlow()

fun main() = runBlocking(Dispatchers.Default) {
    val flow = createFlow()
    supervisorScope {
        launch {
            flow.onEach { value ->
                if (value == 3) cancel() // не работает
                println(value)
            }.launchIn(this)
        }
    }
    println(1111)
    delay(1000)
}
```

```kotlin
fun createFlow(): Flow<Int> = (1..10).asFlow()

fun main() = runBlocking(Dispatchers.Default) {
    val flow = createFlow().cancelable()
    supervisorScope {
        launch {
            flow.onEach { value ->
                if (value == 3) cancel() // снова работает
                println(value)
            }.launchIn(this)
        }
    }
    println(1111)
    delay(1000)
}
```

### Channel

- В первом приближении - аналог блокирующей очереди
- Нет класса `Channel`
- Есть интерфейсы `SendChannel<T>` и `ReceiveChannel<T>`
- Есть наследующий обоих интерфейс `Channel<T>`
- Есть `factory`-метод `Channel<T>` создающий нужную реализацию

```kotlin
val channel = Channel<Int>()
fun main() = runBlocking(Dispatchers.Default) {
    launch {
        for (x in 1..5) {
            println("before send")
            channel.send(x * x)
            println("after send")
        }
    }
    repeat(5) {
        delay(1000)
        println(channel.receive())
    }
}
```

- По умолчанию используется режим рандеву
- При посылке дожидаемся того, что будет читать
- Передаем из рук в руки
- Возможны варианты: конечный буфер, бесконечный буфер
- Поведение при записи в переполненный: suspend, перезапись старых, исключение
- Поведение при чтении из пустого : suspend, исключение
