---
# some information about your slides (markdown enabled)
title: Эффективная реализация лямбда-функций в объектно-ориентированных языках программирования

# apply unocss classes to the current slide
class: text-center

# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
---

# Эффективная реализация лямбда-функций в объектно-ориентированных языках программирования

### Болохононов Артем
### Научный руководитель: Трепаков Иван Сергеевич

---
# some information about your slides (markdown enabled)
title: Эффективная реализация
---

# Избыточная аллокация
## Мотивация

Распространенный сценарий использования лямбда-функций:

<div class="grid grid-cols-2 gap-4">

<div>

<v-click>

````md magic-move
```go{*}
val bottom = 5
val l = { x => x % 10 >= bottom }
// Print filtered values
pfv(collection, l)

def pfv(c: Array[Int], pred: (Int) -> Bool) {
  for (e <- c) {
    if (pred(e)) {
      println(e)
    }
  }
}
```
```go{*|*}
val bottom = 5
val l = { x => x % 10 >= bottom } /* on heap */
// Print filtered values
pfv(collection, l)

def pfv(c: Array[Int], pred: (Int) -> Bool) {
  for (e <- c) {
    if (pred(e)) {
      println(e)
    }
  }
}
```
```go
val bottom = 5
// val l = { x => x % 10 >= bottom } /* exploded */
// Print filtered values
for (e <- collection) {
  if (e % 10 >= bottom) {
    println(e)
  }
}
```
```go{*}
val bottom = 5
val l = { x => x % 10 >= bottom }
// Print filtered values
pfv(collection, l)

// Cannot be inlined
def pfv(c: Array[Int], pred: (Int) -> Bool) {
  for (e <- c) {
    if (pred(e)) {
      println(e)
    }
  }
}
```
```go
val bottom = 5
val l = { x => x % 10 >= bottom } /* stack-allocated */
// Print filtered values
pfv(collection, l)

// Cannot be inlined   /* stack-allocated */
def pfv(c: Array[Int], pred: (Int) -> Bool) {
  for (e <- c) {
    if (pred(e)) {
      println(e)
    }
  }
}
```
```go
val bottom = 5
val l = { x => x % 10 >= bottom } /* on heap */
// Print filtered values
pfv(collection, l)
if (cond) array.append(l)

// Cannot be inlined
def pfv(c: Array[Int], pred: (Int) -> Bool) {
  for (e <- c) {
    if (pred(e)) {
      println(e)
    }
  }
}
```
````

</v-click>

</div>
<div>

<div v-click="2" style="margin-bottom: 13px;">

  ## Проблема

  Произошла аллокация, хотя её можно избежать так как часто лямбда-функции не утекают. 

</div>

<v-click at="3">
  
  ## Потенциальные оптимизации

</v-click>

<v-click at="4">

  - Скаляризация:
    * Может помочь только в случае, когда функция может заинлайнится,

</v-click>

<v-click at="6">

  - Стековая аллокация:
    * Может помочь только в случае, когда нет утеканий.

</v-click>

</div>
</div>

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---
# some information about your slides (markdown enabled)
title: Эффективная реализация
---

# Singleton lambda
## Описание

<v-click> Создание экземпляров лямбда-функций без замыканий создает один и тот же объект (в контексте одного класса). </v-click>

<v-click> Поэтому безконтекстовые лямбда-функции можно создавать лишь единожды. </v-click>

<!-- <v-click> Во многих языках сравнение лямбда-функций на равенство  </v-click> -->

<style>
.centerrr {
  margin: auto;
  max-width: 70%;
  justify-content: center;
  width: 75%;
}
</style>


<img v-click="3" src="/pics/singleton.svg" class="centerrr"/>

<v-click at="4"> Такой подход можно использовать только с лямбда-функциями без замыканий. Но как оптимизировать другие случаи? </v-click>

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---
# some information about your slides (markdown enabled)
title: Эффективная реализация

# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
---

# Эвакуация
## Идея


<p style="opacity: 0"> Gap  </p>

<div class="grid grid-cols-2 gap-4">
<div>

````md magic-move
```go{*}
val bottom = 5
val l = { x => x % 10 >= bottom } /* on heap */
// Print filtered values
pfv(collection, l)
if (cond) array.append(l)

// Cannot be inlined
def pfv(c: Array[Int], pred: (Int) -> Bool) {
  for (e <- c) {
    if (pred(e)) {
      println(e)
    }
  }
}
```
```go{*|*}
val bottom = 5
val l = { x => x % 10 >= bottom } /* stack-allocated */
// Print filtered values
pfv(collection, l)
if (cond) array.append(l)

// Cannot be inlined   /* stack-allocated */
def pfv(c: Array[Int], pred: (Int) -> Bool) {
  for (e <- c) {
    if (pred(e)) {
      println(e)
    }
  }
}
```
```go
val bottom = 5
val l = { x => x % 10 >= bottom } /* stack-allocated */
// Print filtered values
pfv(collection, l)
if (cond) array.append(l) /* incorrect */

// Cannot be inlined   /* stack-allocated */
def pfv(c: Array[Int], pred: (Int) -> Bool) {
  for (e <- c) {
    if (pred(e)) {
      println(e)
    }
  }
}
```
```go
val bottom = 5
val l = { x => x % 10 >= bottom } /* stack-allocated */
// Print filtered values
pfv(collection, l)

if (cond) {
  val heap_l = Evacuation(l)
  array.append(heap_l) /* correct */
}

// Cannot be inlined   /* stack-allocated */
def pfv(c: Array[Int], pred: (Int) -> Bool) {
  for (e <- c) {
    if (pred(e)) {
      println(e)
    }
  }
}
```
```go
val bottom = 5
val l = { x => x % 10 >= bottom } /* stack-allocated */
// Print filtered values
pfv(collection, l)

if (cond) {
  val heap_l = Evacuation(l)
  array.append(heap_l) /* correct */
}

// Cannot be inlined   /* stack-allocated */
def pfv(c: Array[Int], pred: (Int) -> Bool) {
  for (e <- c) {
    if (pred(e)) {
      println(e)
    }
  }

  val heap_pred = Evacuation(l)
  array.append(heap_pred) /* correct */
}
```
````

</div>
<div>

<div v-click="1" style="margin-bottom: 13px;">

  #### Ослабим условие на добавление в массив 

</div>

<v-click at="2">
  
  #### Но что делать, если управление дойдет до `array.append(...)`?

</v-click>

<v-click at="3">
    <ul> 
      <li style="list-style-type: none;"> 
        <ul> 
          <li style="list-style-type: '– ';"> 
            Подобное поведение будет некорректным!
          </li>
        </ul> 
      </li> 
    </ul>
</v-click>

<v-click at="4">
    <ul> 
      <li style="list-style-type: none;"> 
        <ul> 
          <li style="list-style-type: '– ';"> 
            В таких ситуациях будем копировать объект со стека на кучу и передавать уже её.
          </li>
        </ul> 
      </li> 
    </ul>
</v-click>

<v-click at="5">
    <ul> 
      <li style="list-style-type: none;"> 
        <ul> 
          <li style="list-style-type: '– ';"> 
            Аналогично поступаем и для аргументов функции.
          </li>
        </ul> 
      </li> 
    </ul>
</v-click>

</div>
</div>

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---
# some information about your slides (markdown enabled)
title: Эффективная реализация

# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
---

# Эвакуация
## Алгоритм


Утекающими использованиями будем считать:

```go
// Присваивание в {поле другого объекта, глобальную переменную, массив}.
val l = { ... }
staticField = l
array.append(l)
object.field = l
```

```go
// Возврат из функции, если объект был создан внутри тела функции
val l = { ... }
return l
```

```go
// Возврат из функции, если объект возвращается как неэвакуируемый родительский класс
def function(l: Lambda): AnyRef = return l
```

```go
// Передача параметром, если объект передается как неэвакуируемый родительский класс
def function(l: AnyRef)

val l: Lambda = { ... }
function(l)
```

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---

# Эвакуация
## Алгоритм

<div class="grid grid-cols-2 gap-4">
<div>

````md magic-move
```go{*|3,14|3,5,8,9,14,15,16}
def makeLambda() {
  val bottom = 5
  val l = { x => x % 10 >= bottom } /* on heap */
  // Print filtered values
  foo(42, l)

  if (cond) {
    array.append(l)
    anotherarray.append(l)
  }
}

// Cannot be inlined
def foo(value: Int, pred: (Int) -> Int) {
  print("Result = %d", pred(value))
  staticLambda = pred
}
```
```go{3,5,8,9,14,15,16}
def makeLambda() {
  val bottom = 5
  val l = { x => x % 10 >= bottom } /* stack-allocated */
  // Print filtered values
  foo(42, l)

  if (cond) {
    array.append(l)
    anotherarray.append(l)
  }
}

// Cannot be inlined
def foo(value: Int, pred: (Int) -> Int) {
  print("Result = %d", pred(value))
  staticLambda = pred
}
```
```go{3,8,10,16,19}
def makeLambda() {
  val bottom = 5
  val l = { x => x % 10 >= bottom } /* stack-allocated */
  // Print filtered values
  foo(42, l)

  if (cond) {
    val evacuated1 = Evacuated(l)
    array.append(l)
    val evacuated2 = Evacuated(l)
    anotherarray.append(l)
  }
}

// Cannot be inlined
def foo(value: Int, pred: (Int) -> Int) {
  print("Result = %d", pred(value))

  val evacuated3 = Evacuated(l)
  staticLambda = pred
}
```
```go{3,9,11,16,20}
def makeLambda() {
  val bottom = 5
  val l = { x => x % 10 >= bottom } /* stack-allocated */
  // Print filtered values
  foo(42, l)

  if (cond) {
    val evacuated1 = Evacuated(l)
    array.append(evacuated1)
    val evacuated2 = Evacuated(l)
    anotherarray.append(evacuated2)
  }
}

// Cannot be inlined
def foo(value: Int, pred: (Int) -> Int) {
  print("Result = %d", pred(value))

  val evacuated3 = Evacuated(l)
  staticLambda = evacuated3
}
```
```go{8,9,10|*}
def makeLambda() {
  val bottom = 5
  val l = { x => x % 10 >= bottom } /* stack-allocated */
  // Print filtered values
  foo(42, l)

  if (cond) {
    val evacuated1 = Evacuated(l)
    array.append(evacuated1)
    anotherarray.append(evacuated1)
  }
}

// Cannot be inlined
def foo(value: Int, pred: (Int) -> Int) {
  print("Result = %d", pred(value))

  val evacuated3 = Evacuated(l)
  staticLambda = evacuated3
}
```
````
</div>
<div>

### Локальная часть:
<ol> 
  <li v-click="1"> 
    Ищем все источники лямбда-функций.
  </li>
  <li v-click="2"> 
    Ищем все использования источников.
  </li>
  <li v-click="3"> 
    Заменяем создание объекта на стековую аллокацию (если есть хотя бы одно неутекаемое использование).
  </li>
  <li v-click="4"> 
    Перед каждым утекаемым использованием ставим эвакуацию.
  </li>
  <li v-click="6"> 
    <i>Объеденяем все эвакуации на линейном участке.</i>
  </li>
</ol>


</div>
</div>

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---

# Эвакуация
## Алгоритм

<div class="grid grid-cols-2 gap-4">
<div>

````md magic-move
```go
def makeLambda() {
  val bottom = 5
  val l = { x => x % 10 >= bottom } /* stack-allocated */
  // Print filtered values
  foo(42, l)

  if (cond) {
    val evacuated1 = Evacuated(l)
    array.append(evacuated1)
    anotherarray.append(evacuated1)
  }
}

// Cannot be inlined
def foo(value: Int, pred: (Int) -> Int) {
  print("Result = %d", pred(value))

  val evacuated3 = Evacuated(l)
  staticLambda = evacuated3
}
```
```go{*|15-16}
def makeLambda() {
  val bottom = 5
  val l = { x => x % 10 >= bottom } /* on heap */
  // Print filtered values
  foo(42, l)

  if (cond) {
    array.append(l)
    anotherarray.append(l)
  }
}

// Cannot be inlined
def foo(value: Int, pred: (Int) -> Int) {
  print("Result = %d", pred(value))
  staticLambda = pred
}
```

```go{14}
def makeLambda() {
  val bottom = 5
  val l = { x => x % 10 >= bottom } /* on heap */
  // Print filtered values
  foo(42, l)

  if (cond) {
    array.append(l)
    anotherarray.append(l)
  }
}

// Cannot be inlined
def foo(value: Int, /* Evacuated */ pred: (Int) -> Int) {
  print("Result = %d", pred(value))
  staticLambda = pred
}
```
```go{3,5,8,9}
def makeLambda() {
  val bottom = 5
  val l = { x => x % 10 >= bottom } /* on heap */
  // Print filtered values
  foo(42, l)

  if (cond) {
    array.append(l)
    anotherarray.append(l)
  }
}

// Cannot be inlined
def foo(value: Int, /* Evacuated */ pred: (Int) -> Int) {
  print("Result = %d", pred(value))
  staticLambda = pred
}
```
```go{3,5,8,9}
def makeLambda() {
  val bottom = 5
  val l = { x => x % 10 >= bottom } /* on heap */
  // Print filtered values
  foo(42, l) /* Evacuated arg */

  if (cond) {
    array.append(l)
    anotherarray.append(l)
  }
}

// Cannot be inlined
def foo(value: Int, /* Evacuated */ pred: (Int) -> Int) {
  print("Result = %d", pred(value))
  staticLambda = pred
}
```
```go{3}
def makeLambda() {
  val bottom = 5
  val l = { x => x % 10 >= bottom } /* still on heap */
  // Print filtered values
  foo(42, l) /* Evacuated arg */

  if (cond) {
    array.append(l)
    anotherarray.append(l)
  }
}

// Cannot be inlined
def foo(value: Int, /* Evacuated */ pred: (Int) -> Int) {
  print("Result = %d", pred(value))
  staticLambda = pred
}
```
````
</div>
<div>

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

### Межпроцедурная часть:
<ol> 
  <li v-click="1"> 
    Перед расстановкой эвакуаций нужно рассматреть методы с параметрами функциональных типов.
  </li>
  <li v-click="2"> 
    Для каждого такого параметра проверяем существуют ли утекающие использования, которые доминируют выход из функции.
  </li>
  <li v-click="3"> 
    В случае, если такое использование есть, помечаем <b>аргумент</b> как <i>утекающий, но при этом не требующий эвакуации</i>.
  </li>
  <li v-click="4"> 
    Учитываем полученную информацию при локальном анализе.
  </li>
</ol>


</div>
</div>

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---

# Тестовый стенд
## Huawei VM

<style>
.aaa {
    width: 300px;
    height: 300px;
    object-fit: contain;

}
</style>


<div class="absolute right-35% bottom-110px">
<img src="/pics/Huawei.svg" class="aaa"/>
</div>

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---
# some information about your slides (markdown enabled)
title: Эффективная реализация

# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
---

# Эвакуация
## Результаты (GEOMEAN, ns/op, lower --- better)

<style>
.red {
  color: red;
  text-align: center;
}
</style>

<style>
.greeen {
  color: green;
  text-align: center;
}
</style>

<style>
.ctr {
  text-align: center;
}
</style>


Замеры проводились на специализированном наборе микро-бенчмарков.

| Bench name | <div class="ctr"> HVM </div> | <div class="ctr"> Singleton </div> | <div class="ctr"> Evacuation </div> |
| --- | --- | --- | --- |
| LambdaWithoutContext | <div class="ctr">  1772.61 </div> | <div class="greeen">  1343.86 (-24%) </div> | <div class="ctr"> ~ </div> |
| CEscape.LambdaWithContext | <div class="ctr">  8.44 </div> | <div class="ctr"> ~ </div> | <div class="greeen"> 6.59 (-21%) </div> |
| Escape.LambdaWithContext | <div class="ctr">  8.79 </div> | <div class="ctr">  ~ </div> | <div class="red"> 9.59 (+9%) </div> |

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---
# some information about your slides (markdown enabled)
title: Эффективная реализация

# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
---
# Заключение

1. Была реализована оптимизация Singleton Lambda,
1. Был разработан и реализован локальный анализ утеканий,
2. Была разработана и реализована межпроцедурная часть анализа утеканий,
3. Решение было протестировано на Huawei VM.

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---
# some information about your slides (markdown enabled)
title: Эффективная реализация

# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
---

# Заключение

1. Была реализована оптимизация Singleton Lambda,
1. Был разработан и реализован локальный анализ утеканий,
2. Была разработана и реализована межпроцедурная часть анализа утеканий,
3. Решение было протестировано на Huawei VM.

# Дальнейшие направления работы:
1. Улучшение алгоритма расстановки эвакуаций,
2. Введение специализированных эвакуаций для сокращения временных издержек,
3. Тестирование оптимизаций на настоящих проектах / бенчмарках общего назначения.

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---
class: text-center
layout: cover

---

# Спасибо за внимание