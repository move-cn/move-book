# 编码规范

## 命名

### 模块

1. 模块名称应使用 `snake_case`。
2. 模块名称应具有描述性，不应过长。

```move
module book::conventions { /* ... */ }
module book::common_practices { /* ... */ }
```

### 常量

1. 常量名称应使用 `SCREAMING_SNAKE_CASE`。
2. 错误常量应使用 `EPascalCase`。

```move
const MAX_PRICE: u64 = 1000;
const EInvalidInput: u64 = 0;
```

### 函数

1. 函数名称应使用 `snake_case`。
2. 函数名称应具有描述性。

```move
public fun add(a: u64, b: u64): u64 { a + b }
public fun create_if_not_exists() { /* ... */ }
```

### 结构体

1. 结构体名称应使用 `PascalCase`。
2. 结构体字段应使用 `snake_case`。
3. 能力结构体名称应以 `Cap` 结尾。

```move
public struct Hero has key {
    id: UID,
    value: u64,
    another_value: u64,
}

public struct AdminCap has key { id: UID }
```

### 结构体方法

1. 结构体方法应使用 `snake_case`。
2. 如果多个结构体具有相同的方法名称，方法名称应以结构体名称为前缀。在这种情况下，可以使用 `use fun` 为方法添加别名。

```move
public fun value(h: &Hero): u64 { h.value }

public use fun hero_health as Hero.health;
public fun hero_health(h: &Hero): u64 { h.another_value }

public use fun boar_health as Boar.health;
public fun boar_health(b: &Boar): u64 { b.another_value }
```