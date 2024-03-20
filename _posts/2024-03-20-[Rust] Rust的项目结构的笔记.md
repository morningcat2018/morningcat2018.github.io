---
layout:     post
title:      关于Rust的项目结构的笔记
subtitle:   及 crate 的理解与使用
date:       2024-03-20
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - Rust
---

层级
- Package
- Crate
- Module
- Path

# Package
cargo的特性, 构建、测试、共享Crate

组成:
- 一个  Cargo.toml 文件, 描述了如何构建这些 Crates
- 至少包含一个 crate
- 最多只能包含一个 library crate
- 可以包含任意个 binary crate

> cargo new demo-pro

会产生一个名为 demo-pro 的 Package

目录结构如下:
![在这里插入图片描述](/img/rust/rust001.png)
---

cargo 的惯例:

1. src/main.rs 是 binary crate 的 crate root
且此crate的名称与package名一致

2. src/lib.rs 是 library crate 的 crate root (此文件需要自己创建)
且此crate的名称与package名一致

cargo 会把 crate root 文件交给 rustic 来构建 library 或 binary 

3. 一个package 可以有多个 binary crate
![在这里插入图片描述](/img/rust/rust002.png)




# Crate
一个模块树, 可以产生一个 library 或 可执行文件

---

Crate 类型:
- binary 二进制文件
- library 库文件

---

Crate Root:
- 是源代码文件
- rustc 从这里开始组成项目的根Module



# Module
控制代码的组织、作用域、私有路径

---

- 在一个 crate 内可以将代码进行分组
- 易于复用
- 控制项目私有性
- 使用 mod 关键字进行创建
- 可以嵌套
- 可以包含其他项的定义(struct , enum, trait, fn)

### 案例1:
src/lib.rs
```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}
        fn serve_order() {}
        fn take_payment() {}
    }
}
```
![在这里插入图片描述](/img/rust/rust003.png)

### 案例2:
src/lib.rs
```rust
pub mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
        pub fn seat_at_table() {}
    }

    pub mod serving {
        pub fn take_order() {}
        pub fn serve_order() {}
        pub fn take_payment() {}
    }
}

pub fn eat_at(){
    crate::front_of_house::hosting::add_to_waitlist();
}
```

src/main.rs
```rust
fn eat_at2(){
    demo_pro::front_of_house::hosting::add_to_waitlist();
}

fn main() {
    eat_at2();
}
```

### 案例3: super
src/lib.rs
```rust
pub mod front_of_house {

    fn method1() {}

    pub mod hosting {
        fn method2() {}

        pub fn method3() {
            method2();
            super::method1();
            // or
            crate::front_of_house::method1();
        }
    }
}
```

### 案例4 use 关键字
src/lib.rs
```rust
pub mod front_of_house {

    fn method1() {}

    pub mod hosting {
        fn method2() {}

        pub fn method3() {
            method2();
            super::method1();
            // or
            crate::front_of_house::method1();
        }
    }
}

use crate::front_of_house::hosting;

pub fn eat_at3(){
    hosting::method3();
}
```


src/main.rs
```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}

```

### 案例5 as 关键字


src/main.rs
```rust
use std::collections::HashMap as MyMap;

fn main() {
    let mut map = MyMap::new();
    map.insert(1, 2);
}
```

### 案例6 pub use

重新导出

### 案例7 将模块重新拆分为不同文件

理论:
![在这里插入图片描述](/img/rust/rust004.png)


1. 一级拆分 (等同案例4的效果)

![在这里插入图片描述](/img/rust/rust005.png)

src/lib.rs
```rust
pub mod front_of_house;

use crate::front_of_house::hosting;

pub fn eat_at3(){
    hosting::method3();
}
```

src/front_of_house.rs
```rust
fn method1() {}

pub mod hosting {
    fn method2() {}

    pub fn method3() {
        method2();
        super::method1();
        // or
        crate::front_of_house::method1();
    }
}
```

2. 二级拆分 (等同案例2的效果)

![在这里插入图片描述](/img/rust/rust006.png)

src/lib.rs
```rust
pub mod front_of_house;

pub fn eat_at(){
    crate::front_of_house::hosting::add_to_waitlist();
}
```

src/front_of_house.rs

```rust
pub mod hosting;

pub mod serving;
```

src/front_of_house/hosting.rs
```rust
pub fn add_to_waitlist() {}
pub fn seat_at_table() {}
```

src/front_of_house/serving.rs
```rust
pub fn take_order() {}
pub fn serve_order() {}
pub fn take_payment() {}
```