layout: true
class: center middle

---

## Exotically Sized Types

###   Michael Jackson
### Sheely   Warley

<br>
December 6, 2016
<br>

Harvey Mudd College: Memory Safety in Rust

---

layout: false
class: left

## Static Size and the Stack

![stack1](unsized-types-rust-presentation/docs/_images/stack1.png)
---

## Static Size and the Stack

![stack2](_images/stack2.PNG)

Easy for compiler to do arithmatic before hand and know when and how to destroy things.

---

## Zero Sized Types

What does the compiler do when you write this?

---

layout: false
class: left

## Zero Sized Types

What does the compiler do when you write this?

```rust
struct Empty;

struct Composite {
  x: Empty,
  y: (),
  z: [u8; 0],
}
```
---

## Zero Sized Types

<br>
Say we have a `Map<Key, Value>` already implemented.

<br>
And you want to implement a `Set<Key>`.

---

## Zero Sized Types

<br>
Say we have a `Map<Key, Value>` already implemented.

<br>
And you want to implement a `Set<Key>`.

<br>
As a `Map<Key, SmallUselessValue>`?

---

## Zero Sized Types

<br>
Say we have a `Map<Key, Value>` already implemented.

<br>
And you want to implement a `Set<Key>`.

<br>
As a `Map<Key, SmallUselessValue>`?

<br>
Instead, consider `Map<Key, ()>`!

---

## Dynamically Sized Types

<br>
Some questions to ponder.  In Rust:

<br>
 * What's the size of a `Vec<T>`?
---

## Dynamically Sized Types

<br>
Some questions to ponder.  In Rust:

<br>
 * What's the size of a `Vec<T>`?
<br><br>
 * What's the size of `&[T]`?

---

## Dynamically Sized Types

<br>
Some questions to ponder.  In Rust:

<br>
 * What's the size of a `Vec<T>`?
 <br><br>
 * What's the size of `&[T]`?
 <br><br>
 * What's the size of `&[u8; 3]`?

---

## Dynamically Sized Types

<br>
 Thin Pointer: A one word pointer which contains a memory address.

 Fat Pointer: A multiword structure which *also* contains extra data about the instance (e.g. size).

---

## Dynamically Sized Types

<br>
 Thin Pointer: A one word pointer which contains a memory address.

 Fat Pointer: A multiword structure which *also* contains extra data about the instance (e.g. size).

<br>
 * Can't manipulate an unsized type, need to access them behind a pointer.

 * Any interactions with unsized types will occur through a Fat Pointer.

 * How do we interact with these DSTs?

---


## Dynamically Sized Types

Interacting with Thin/Fat pointers and arrays.

```rust
struct Rc<T: ?Sized> {
    ptr: *mut RcData<T>,
}
```

---

## Dynamically Sized Types

Interacting with Thin/Fat pointers and arrays.

```rust
struct Rc<T: ?Sized> {
    ptr: *mut RcData<T>,
}

struct RcData<T: ?Sized> {
    ref_count: usize,
    data: T,
}
```

---

## Dynamically Sized Types

Interacting with Thin/Fat pointers and arrays.

```rust
struct Rc<T: ?Sized> {
    ptr: *mut RcData<T>,
}

struct RcData<T: ?Sized> {
    ref_count: usize,
    data: T,
}

#[allow(unused_variables)]
fn main() {
    let mut rc_data = RcData{ ref_count: 1, data: [1,2,3] };
    let rc1: Rc<[u8; 3]> = Rc{ ptr: &mut rc_data};
    let rc2: Rc<[u8]> = {
        let Rc { ptr: thin } = rc1;
        let fat = thin as *mut RcData<[u8]>;
        Rc { ptr: fat }
    };
}
```

---

## Dynamically Sized Types

### Polymorphism and Traits

<br>
Rust also makes use of unsized types to represent "Trait Objcects".

---

## Dynamically Sized Types

### Polymorphism and Traits

<br>
Rust also makes use of unsized types to represent "Trait Objcects".

<br>
We can refer to `T: Foo`, without the compiler knowing the type of `T`!

---

## Dynamically Sized Types

### Trait Objects

```rust
trait Foo { fn method(&self) -> String; }
```
---
## Dynamically Sized Types

### Trait Objects

```rust
trait Foo { fn method(&self) -> String; }

impl Foo for u8 {
    fn method(&self) -> String { format!("u8: {}", *self) }
}
impl Foo for String {
    fn method(&self) -> String { format!("String: {}", *self) }
}
```
---

## Dynamically Sized Types

### Trait Objects

```rust
trait Foo { fn method(&self) -> String; }

impl Foo for u8 {
    fn method(&self) -> String { format!("u8: {}", *self) }
}
impl Foo for String {
    fn method(&self) -> String { format!("String: {}", *self) }
}

fn static_dispatch<T: Foo> (arg: T) {
    println!("{}", arg.method())
}

fn main() {
    let s: String = String::from("a string");
    let n: u8 = 0;
    static_dispatch(s);
    static_dispatch(n);
}
```
---

## Dynamically Sized Types

### Trait Objects

```rust
trait Foo { fn method(&self) -> String; }

impl Foo for u8 {
    fn method(&self) -> String { format!("u8: {}", *self) }
}
impl Foo for String {
    fn method(&self) -> String { format!("String: {}", *self) }
}

fn dynamic_dispatch(arg: &Foo) {
    println!("{}", arg.method())
}

fn main() {
    let s: String = String::from("a string");
    let n: u8 = 0;
    dynamic_dispatch(&s);
    dynamic_dispatch(&n);
}
```
---

## Dynamically Sized Types

### Trait Objects

The fat pointer:
```rust
pub struct TraitObject {
    pub data: *mut (),
    pub vtable: *mut (),
}
```
And the vtable itself:
```rust
struct FooVtable {
    destructor: fn(*mut ()),
    size: usize,
    align: usize,
    method: fn(*const ()) -> String,
}
```
---

## Exotically Sized Types: Summary

### Zero Sized Types
 * Compile to `nop`
 * Logical construct rather than data driven

---

## Exotically Sized Types: Summary

### Zero Sized Types
 * Compile to `nop`
 * Logical construct rather than data driven

### Dynamically Sized Types
 * Cannot access directly (must be behind a pointer)
 * Common use cases include:
   * Contiguous memory of dynamic size
   * Trait Objects
   * Structs containing Dynamically Sized Types
 * Compiler implements as fat pointers with either a size or a vtable
 * We can cast between statically sized and dynamically sized types.
---

## References

 * The rust book: https://doc.rust-lang.org/stable/book/unsized-types.html
 * The rustonomicon: https://doc.rust-lang.org/nomicon/exotic-sizes.html
 * Baby Steps: http://smallcultfollowing.com/babysteps/blog/2014/01/05/dst-take-5/
 * Huon on the Internet: http://huonw.github.io/blog/2015/01/peeking-inside-trait-objects/

