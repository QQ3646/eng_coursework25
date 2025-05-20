---
# some information about your slides (markdown enabled)
title: Эффективная реализация лямбда-функций в объектно-ориентированных языках
  программирования

# apply unocss classes to the current slide
class: text-center

# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
---

# Efficient implementation of lambda functions in object-oriented programming languages

### Bolokhonov Artem
### Academic advisor: Ivan Sergeevich Trepakov

---
# some information about your slides (markdown enabled)
title: Эффективная реализация
---

# Excessive allocation
## Motivation

A common scenario for the use of lambda functions:

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

  ## Issue

  Allocation has occurred, although it is avoidable since lambda functions often do not escape. 

</div>

<v-click at="3">
  
  ## Potential optimizations

</v-click>

<v-click at="4">

  - Scalarization:
    * Can only help if the function can be inlined,

</v-click>

<v-click at="6">

  - Stack allocation:
    * Can only help if there are no escapes.

</v-click>

</div>
</div>

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---
# some information about your slides (markdown enabled)
title: Эффективная реализация
---

# Singleton lambda
## Description

<v-click> Creating instances of lambda functions without closures creates the same object (in the context of the same class). </v-click>

<v-click> Therefore, context-free lambda functions can only be created once. </v-click>

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

<v-click at="4"> This approach can only be used with lambda functions without closures. But how to optimize other cases? </v-click>

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---
# some information about your slides (markdown enabled)
title: Эффективная реализация

# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
---

# Evacuate analysis
## Concept


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

  #### Let's relax the condition for adding to the array 

</div>

<v-click at="2">
  
  #### But what if control comes to `array.append(...)`?

</v-click>

<v-click at="3">
    <ul> 
      <li style="list-style-type: none;"> 
        <ul> 
          <li style="list-style-type: '– ';"> 
            Such behavior would be incorrect!
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
            In such situations, we will copy the object from the stack to the heap and pass the copy.
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
            We do the same for the arguments of the function.
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

# Evacuate analysis
## Algorithm


Escaped uses will be considered:

```go
// Assignment to {field of another object, global variable, array}
val l = { ... }
staticField = l
array.append(l)
object.field = l
```

```go
// Return from the function if the object was created inside the function body
val l = { ... }
return l
```

```go
// Return from function if the object is returned as a non-evacuated parent class
def function(l: Lambda): AnyRef = return l
```

```go
// Passing with a parameter if the object is passed as a non-evacuated parent class
def function(l: AnyRef)

val l: Lambda = { ... }
function(l)
```

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---

# Evacuate analysis
## Algorithm

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

### Local part:
<ol> 
  <li v-click="1"> 
    Looking for all sources of lambda functions.  </li>
  <li v-click="2"> 
    Looking for all uses of sources. 
 </li>
  <li v-click="3"> 
    Replace object creation with stack allocations (if there is at least one non-evacuated use).
  </li>
  <li v-click="4"> 
    Add an evacuation before each leaked use.
  </li>
  <li v-click="6"> 
    <i>Combine all evacuations in a basic block.</i>
  </li>
</ol>


</div>
</div>

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---

# Evacuate analysis
## Algorithm

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

### Interprocedural part:
<ol> 
  <li v-click="1"> 
    Before arranging evacuations, we should consider methods with parameters of functional types.
  </li>
  <li v-click="2"> 
    For each such parameter, we check whether there are leaking uses that dominate the return point.
  </li>
  <li v-click="3"> 
    If there is such a use, mark the <b>argument</b> as <i>escaping but not requiring evacuation</i>
  </li>
  <li v-click="4"> 
    We take the obtained information into account in the local analysis.
  </li>
</ol>


</div>
</div>

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---

# Test stand
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

# Evacuate analysis
## Results (GEOMEAN, ns/op, lower --- better)

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


Measurements were performed on a specialized set of micro-benchmarks.

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

# Conclusion

1. Singleton Lambda optimization was implemented,
1. The local part of the evacuate analysis was developed and implemented,
2. The interprocedural part of the evacuate analysis was developed and implemented,
3. The solution has been tested on the Huawei VM.

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---
# some information about your slides (markdown enabled)
title: Эффективная реализация

# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
---

# Conclusion

1. Singleton Lambda optimization was implemented,
1. The local part of the evacuate analysis was developed and implemented,
2. The interprocedural part of the evacuate analysis was developed and implemented,
3. The solution has been tested on the Huawei VM.

# Further areas of work:
1. Improvement of the evacuation placement algorithm,
2. Introduction of specialized evacuations to reduce time costs,
3. Testing optimizations on real projects / general purpose benchmarks.

<SlideCurrentNo class="absolute right-40px bottom-30px"/>

---
class: text-center
layout: cover
---

# Thank you for your attention
