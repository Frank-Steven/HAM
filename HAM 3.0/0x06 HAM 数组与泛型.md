# 0x06 HAM 数组与泛型

数组是内置函数，有大量的语法糖。

## 数组的声明

数组以 `Array` 函数创建：

```HAM
arr1 = Array(),
arr2 = Array(1, 2, 3)
```

可以用中括号来简化，上面的代码等价于：

```HAM
arr1 = [],
arr2 = [1, 2, 3]
```

## 数组的类型集合

在对应类型后加 `[]` 表示该类型的数组集合，这是**数组集合表达式**：

```HAM
arr3 = [2, 3, 4] as Int[]  // [2, 3, 4]
arr4 = [1]       as Int[5] // [1, {}, {}, {}, {}]
arr5 = [1, 2, 3] as Int[2] // {}
arr6 = [1, 'h']  as Int[2] // {}
```

当然也可以表示高维数组：

```HAM
arr6 = [[1], [2, 3]] as Int[] // 犬牙交错数组，每一项的长度可以不同
```

> 提示：`Int[][]` 其实就是 `(Int[])[]`

数组长度与标注不一致时，若长度不足用 `{}` 补齐，长度超出则视为与标注不匹配，结果为 `{}`。

注意数组字面量的类型也满足格言：`typeof([1, 2, 3])` 就是 `{ [1, 2, 3] }`。

## 数组的操作

### 引用元素

可以用 `Array.at` 函数来获取数组对应下标的元素：

```HAM
Array.at(arr2, 0) // 1
```

可以用中括号来简化，上面的代码等价于：

```HAM
arr2[0] // 1
```

或使用 `|>` 运算符：

```HAM
arr2 |> Array.at(_, 0) // 1
```

### 修改元素并覆写

可以用 `Array.modify` 函数来修改并覆写数组的元素：

```HAM
arr3 = Array.modify(arr3, 1, 0) // arr3 变为 [2, 0, 4]
```

也可以覆写由中括号拿到的引用：

```HAM
arr3[1] = 2 // arr3 变为 [2, 2, 4]
```

### 过滤器

可以用 `Array.filter` 函数来过滤数组中符合条件的元素：

```HAM
Array.filter(arr, x => x > 1) // [2, 3]
```

HAM 默认为数组重载了 `|` 运算符，用于简化代码：

```HAM
arr2 | x => x > 1 // [2, 3]
```

也可以用 `_` 语法糖来简化单参函数：

```HAM
arr2 | _ > 1 // [2, 3]
```

还可以用 `|>` 运算符把管道式调用串起来，例如先过滤再映射：

```HAM
arr2 |> filter(_, x => x > 1) |> map(_, x => x * 2) where Array // [4, 6]
```

### 长度

可以用 `Array.length` 函数获取数组的长度：

```HAM
Array.length(arr1) // 0
Array.length(arr2) // 3
```

### 拼接

HAM 默认为数组重载了 `+` 运算符，用于数组的拼接：

```HAM
arr2 + [4, 5] // [1, 2, 3, 4, 5]
```

当然也可以用 `Array.concat` 函数来拼接两个数组：

```HAM
Array.concat(arr2, [4, 5]) // [1, 2, 3, 4, 5]
```

### 映射

可以用 `Array.map` 函数对数组中的每个元素应用一个函数，返回映射后的新数组：

```HAM
Array.map(arr2, x => x * 2) // [2, 4, 6]
```

如果 `Array.map` 的函数返回数组，可以用 `Array.flatMap` 把结果压平一层，相当于 `Array.map` 后再 `Array.flatten`：

```HAM
Array.flatMap(arr2, [_, -_]) // [1, -1, 2, -2, 3, -3]
```

`Array.flatten` 可以把数组的数组压平一层：

```HAM
Array.flatten(arr6) // [1, 2, 3]
```

### 折叠

可以用 `Array.fold` 函数把数组归约为一个值，需要给定初始值和一个二元函数，从左到右依次把元素合入累加器：

```HAM
Array.fold(arr2, 0, (acc, x) => acc + x) // 6
```

### 切片

可以用 `Array.take`、`Array.drop`、`Array.slice` 获取数组的一部分，都返回新数组：

```HAM
Array.take(arr2, 2)     // [1, 2]
Array.drop(arr2, 2)     // [3]
Array.slice(arr2, 1, 3) // [2, 3]
```

`Array.slice(from, to)` 是 `[from, to)` 半开区间；`Array.take`、`Array.drop` 超出长度时截到末尾，`Array.slice` 超出范围返回 `{}`。

### 反转

可以用 `Array.reverse` 函数反转数组，返回新数组：

```HAM
Array.reverse(arr2) // [3, 2, 1]
```

### 查找

可以用 `Array.indexOf` 获取元素第一次出现的下标，用 `Array.contains` 判断数组是否包含元素：

```HAM
Array.indexOf(arr2, 2) // 1
Array.indexOf(arr2, 9) // {}
Array.contains(arr2, 2) // true
```

`Array.indexOf` 找不到时返回 `{}`。

## 泛型

泛型在 HAM 中有两种实现方式。

第一种是用函数的参数来表示类型，使用时，需先传入一个参数：

```HAM
identity = (T: Set) => (x: T) => x

identity({ 1 })(1)    // 1 注意类型仍为 { 1 }
identity(Int)(1)      // 1 注意类型变成了 Int
identity(Float)(3.14) // 3.14
identity(Int)(3.14)   // {} 因为 3.14 isnt Int
```

第二种泛型是一个新语法，它更加智能，使用时无需（也不可以）传入类型参数，HAM 会根据传入的参数自动推断类型。这种泛型更像是一种类型集合的限制，用来限制类型退化的范围。

在定义时，使用尖括号 `<>` 来标记类型参数，用冒号 `:` 来标记该类型的范围。如 `<T: U>` 表示类型 `T` 满足 `T subseteq U`：

```HAM
muladd = <T>(mul: T, x: T, y: T) -> T => (x + y) * mul,
typeof(muladd)        // <T>(T, T, T) -> T
muladd(2, 3, 4)       // 14   类型为 Int，因为 `+ 和 `* 会使用 Int 的运算，T 退化为 Int
muladd(2.0, 3.0, 4.0) // 14.0 类型为 Float，因为 `+ 和 `* 会使用 Float 的运算，T 退化为 Float
muladd(2, 3.0, 4)     // 14.0 类型为 Float，因为 `+ 和 `* 会使用 Float 的运算，T 退化为 Float
```

我们也可以用泛型写出一些数组操作的类型集合：

```HAM
typeof(Array.map)     // <T, U>(T[], (T) -> U) -> U[]
typeof(Array.filter)  // <T>(T[], (T) -> Bool) -> T[]
typeof(Array.flatten) // <T>(T[][]) -> T[]
```

在不标记参数类型的时候，HAM 会自动生成一个泛型，以供类型推导：

```HAM
square = (x) => x * x,
typeof(square) // <A, B>(A) -> B
```

## 附录

### 不可变 API 一览表

| API                                 | 说明                     |
| ----------------------------------- | ------------------------ |
| `Array.at(array, index)`            | 获取数组对应下标的元素   |
| `Array.modify(array, index, value)` | 修改并覆写数组的元素     |
| `Array.filter(array, predicate)`    | 过滤数组中符合条件的元素 |
| `Array.length(array)`               | 获取数组的长度           |
| `Array.concat(...arrays)`           | 拼接多个数组             |
| `Array.map(array, f)`               | 每个元素经 `f` 映射      |
| `Array.flatMap(array, f)`           | 映射后压平一层           |
| `Array.flatten(array)`              | 把数组的数组压平一层     |
| `Array.fold(array, init, f)`        | 从左到右折叠             |
| `Array.take(array, n)`              | 取前 `n` 个元素          |
| `Array.drop(array, n)`              | 去掉前 `n` 个元素        |
| `Array.slice(array, from, to)`      | 切片                     |
| `Array.reverse(array)`              | 反转                     |
| `Array.indexOf(array, x)`           | 第一个 `x` 的下标        |
| `Array.contains(array, x)`          | 是否包含 `x`             |
