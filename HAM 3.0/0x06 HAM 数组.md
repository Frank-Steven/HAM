# 0x06 HAM 数组

数组是内置函数，有大量的语法糖

### 声明

```ham
arr1 = [],
arr2 = [1, 2, 3]
```

```ham
arr1 = Array(),
arr2 = Array(1, 2, 3)
```

### 引用元素

```ham
arr2[0]
```

```ham
arr2.at(0)
```

### 修改元素并覆写

```ham
arr2[1] = 0
```

```ham
arr2 = arr2.modify(1, 0)
```

### 过滤器

```ham
arr2 | _ > 1
```

```ham
arr2 | x => x > 1
```

```ham
arr2.filter(x => x > 1)
```

### 数组有一套不可变 API

```ham
// 数组不可变 API
```