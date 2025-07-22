---
layout: default
title: "Tuần 1: Làm quen với TypeScript & Tư duy kiểu tường minh"
description: "10 chủ đề cơ bản về TypeScript cho Python developers - Static typing, kiểu dữ liệu, interface, function"
---

# 🗓️ Tuần 1: Làm quen với TypeScript & Tư duy kiểu tường minh

Chào mừng đến với tuần đầu tiên! Tuần này chúng ta sẽ làm quen với những khái niệm cơ bản nhất của TypeScript, đặc biệt tập trung vào sự khác biệt với Python về quản lý kiểu dữ liệu.

---

## 1. 🔍 Static Typing vs Dynamic Typing

### Python (Dynamic Typing)
```python
# Python - kiểu được kiểm tra tại runtime
def add(a, b):
    return a + b

result = add(5, "hello")  # Lỗi runtime: TypeError
```

### TypeScript (Static Typing)
```typescript
// TypeScript - kiểu được kiểm tra tại compile time
function add(a: number, b: number): number {
    return a + b;
}

const result = add(5, "hello");  // Lỗi compile time ngay lập tức!
```

**💡 Takeaway**: TS giúp phát hiện lỗi sớm hơn, tốt cho codebase lớn và team work.

---

## 2. 🔠 Các kiểu dữ liệu cơ bản trong TS

```typescript
// Các kiểu cơ bản
let age: number = 30;
let name: string = "Alice";
let isActive: boolean = true;
let value: undefined = undefined;
let empty: null = null;

// Array
let numbers: number[] = [1, 2, 3];
let names: Array<string> = ["Alice", "Bob"];

// Object
let person: { name: string; age: number } = {
    name: "Alice",
    age: 30
};
```

**So với Python**: Python có `int`, `str`, `bool`, `None`. TS có thêm `undefined` và phân biệt rõ hơn.

---

## 3. 💥 `any` vs `unknown` vs `never`

```typescript
// any - tránh dùng (như object trong Python)
let anything: any = 42;
anything = "hello";
anything.foo.bar;  // Không lỗi, nhưng nguy hiểm!

// unknown - an toàn hơn
let something: unknown = 42;
if (typeof something === "string") {
    console.log(something.toUpperCase());  // OK
}

// never - không bao giờ xảy ra
function throwError(): never {
    throw new Error("Always throws");
}
```

**💡 Takeaway**: Dùng `unknown` thay vì `any` khi không chắc chắn kiểu.

---

## 4. 🧱 Interface vs Type Alias

```typescript
// Interface - dùng cho object
interface User {
    name: string;
    age: number;
    email?: string;  // optional
}

// Type Alias - linh hoạt hơn
type Status = "pending" | "approved" | "rejected";
type UserWithStatus = User & { status: Status };

// Cả hai đều work
const user1: User = { name: "Alice", age: 30 };
const user2: UserWithStatus = { 
    name: "Bob", 
    age: 25, 
    status: "pending" 
};
```

**So với Python**: Giống `TypedDict` và `dataclass` nhưng mạnh mẽ hơn.

---

## 5. 🧩 Union và Intersection Types

```typescript
// Union - HOẶC (như Union trong Python typing)
type StringOrNumber = string | number;
let value: StringOrNumber = "hello";
value = 42;  // OK

// Intersection - VÀ
type Person = { name: string };
type Employee = { id: number };
type PersonEmployee = Person & Employee;

const worker: PersonEmployee = {
    name: "Alice",
    id: 123
};
```

**Python tương đương**:
```python
from typing import Union
value: Union[str, int] = "hello"
```

---

## 6. 🏷️ Optional, Required, Readonly

```typescript
interface Config {
    host: string;
    port?: number;        // optional
    readonly secret: string;  // readonly
}

// Utility types
type PartialConfig = Partial<Config>;     // Tất cả optional
type RequiredConfig = Required<Config>;   // Tất cả required
type ReadonlyConfig = Readonly<Config>;   // Tất cả readonly
```

**💡 Takeaway**: TS có nhiều cách kiểm soát tính bắt buộc/readonly hơn Python.

---

## 7. 🧮 Viết hàm có khai báo kiểu

```typescript
// Function declaration
function add(x: number, y: number): number {
    return x + y;
}

// Arrow function
const multiply = (x: number, y: number): number => x * y;

// Optional parameters
function greet(name: string, title?: string): string {
    return title ? `${title} ${name}` : name;
}

// Default parameters
function createUser(name: string, age: number = 18): User {
    return { name, age };
}
```

**So với Python**:
```python
def add(x: int, y: int) -> int:
    return x + y
```

---

## 8. 🔁 Function Overloading

```typescript
// Function overloads
function process(input: string): string;
function process(input: number): number;
function process(input: string | number): string | number {
    if (typeof input === "string") {
        return input.toUpperCase();
    }
    return input * 2;
}

const result1 = process("hello");  // TypeScript biết return string
const result2 = process(5);        // TypeScript biết return number
```

**Python tương đương**:
```python
from typing import overload

@overload
def process(input: str) -> str: ...

@overload  
def process(input: int) -> int: ...
```

---

## 9. 🛠️ Dùng TypeScript Playground

**Link**: [https://www.typescriptlang.org/play](https://www.typescriptlang.org/play)

**Cách dùng hiệu quả**:
1. Copy code examples từ lộ trình này
2. Thử sửa đổi và xem lỗi compile
3. Hover mouse để xem type inference
4. Dùng tab "Examples" để học thêm

---

## 10. 🤖 Prompt ChatGPT hiệu quả cho TS

**Prompts gợi ý**:

```
Convert this Python function to TypeScript and add types:
[paste your Python code]

Explain this TypeScript error to me:
[paste error message]

How to type this pattern in TypeScript:
[describe your use case]

What's the TypeScript equivalent of Python's [feature]?
```

**💡 Tip**: Luôn mention "TypeScript" và "add types" trong prompt để có kết quả tốt nhất.

---

## 🎯 Bài tập tuần 1

### Bài 1: Chuyển đổi Python sang TypeScript
```python
# Python code
def calculate_discount(price, discount_percent, is_member=False):
    discount = price * discount_percent / 100
    if is_member:
        discount += 5
    return price - discount
```

### Bài 2: Tạo interface cho dữ liệu
Tạo interface cho một sản phẩm có: `id`, `name`, `price`, `category`, `inStock` (optional).

### Bài 3: Viết function overloading
Viết hàm `format` có thể nhận `Date` (return string) hoặc `number` (return formatted number).

---

## 🚀 Tuần tới

👉 **[Tuần 2: Làm việc với project thực tế (Node.js / Deno)](./week-2.md)**

Tuần tới chúng ta sẽ học cách áp dụng TypeScript vào project thực tế với Node.js/Deno!

---

*🎉 Chúc mừng bạn đã hoàn thành tuần 1!*
