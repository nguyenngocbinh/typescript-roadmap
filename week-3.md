# 🗓️ Tuần 3: Type nâng cao – Lập trình mạnh mẽ hơn

Tuần này chúng ta sẽ khám phá sức mạnh thực sự của TypeScript system - từ Generic đến Conditional Types. Đây là những tính năng giúp TS vượt trội so với Python typing.

---

## 19. ⚙️ Generic Type là gì?

### Python Generic (từ 3.5+)
```python
from typing import TypeVar, Generic, List

T = TypeVar('T')

class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: List[T] = []
    
    def push(self, item: T) -> None:
        self._items.append(item)
    
    def pop(self) -> T:
        return self._items.pop()
```

### TypeScript Generic (mạnh mẽ hơn)
```typescript
// Function generic
function identity<T>(arg: T): T {
    return arg;
}

const num = identity<number>(42);     // T = number
const str = identity("hello");        // T được infer = string

// Class generic
class Stack<T> {
    private items: T[] = [];
    
    push(item: T): void {
        this.items.push(item);
    }
    
    pop(): T | undefined {
        return this.items.pop();
    }
    
    peek(): T | undefined {
        return this.items[this.items.length - 1];
    }
}

const numberStack = new Stack<number>();
const stringStack = new Stack<string>();

// Interface generic
interface ApiResponse<TData> {
    data: TData;
    status: number;
    error?: string;
}

const userResponse: ApiResponse<User[]> = {
    data: [{ name: "Alice", age: 30 }],
    status: 200
};
```

**💡 Takeaway**: TS Generic linh hoạt và mạnh mẽ hơn Python, có type inference tốt hơn.

---

## 20. 📐 Utility types phổ biến

```typescript
interface User {
    id: number;
    name: string;
    email: string;
    password: string;
    createdAt: Date;
}

// Partial<T> - tất cả properties optional
type UserUpdate = Partial<User>;
const update: UserUpdate = { name: "New Name" }; // OK

// Pick<T, K> - chọn một số properties
type UserProfile = Pick<User, 'id' | 'name' | 'email'>;
const profile: UserProfile = {
    id: 1,
    name: "Alice",
    email: "alice@example.com"
    // password không cần
};

// Omit<T, K> - loại bỏ một số properties
type CreateUserRequest = Omit<User, 'id' | 'createdAt'>;
const createRequest: CreateUserRequest = {
    name: "Bob",
    email: "bob@example.com",
    password: "secret123"
};

// Record<K, V> - tạo object type với key/value specific
type UserRoles = Record<string, 'admin' | 'user' | 'guest'>;
const roles: UserRoles = {
    "alice": "admin",
    "bob": "user",
    "charlie": "guest"
};

// Required<T> - tất cả properties required
type RequiredUser = Required<Partial<User>>;

// Readonly<T> - tất cả properties readonly
type ImmutableUser = Readonly<User>;
```

**Python tương đương** (hạn chế hơn):
```python
from typing import TypedDict, Optional

class UserUpdate(TypedDict, total=False):  # Similar to Partial
    name: Optional[str]
    email: Optional[str]
```

---

## 21. 🔎 Type inference & narrowing

```typescript
// Type narrowing với typeof
function processValue(value: string | number) {
    if (typeof value === "string") {
        // TS biết value là string trong block này
        return value.toUpperCase();
    }
    // TS biết value là number ở đây
    return value.toFixed(2);
}

// Type narrowing với instanceof
class Dog {
    bark(): void { console.log("Woof!"); }
}

class Cat {
    meow(): void { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat) {
    if (animal instanceof Dog) {
        animal.bark();  // TS biết là Dog
    } else {
        animal.meow();  // TS biết là Cat
    }
}

// Type narrowing với in operator
interface Bird {
    fly(): void;
    feathers: boolean;
}

interface Fish {
    swim(): void;
    scales: boolean;
}

function move(animal: Bird | Fish) {
    if ('fly' in animal) {
        animal.fly();   // TS biết là Bird
    } else {
        animal.swim();  // TS biết là Fish
    }
}

// Type guards (custom)
function isString(value: unknown): value is string {
    return typeof value === "string";
}

function processUnknown(value: unknown) {
    if (isString(value)) {
        // TS biết value là string
        console.log(value.length);
    }
}
```

**So với Python**: TS type narrowing mạnh mẽ hơn và tự động hơn.

---

## 22. 🔁 Mapped types

```typescript
// Tạo type mới dựa trên type cũ
type Optional<T> = {
    [K in keyof T]?: T[K];
};

type Nullable<T> = {
    [K in keyof T]: T[K] | null;
};

// Ví dụ sử dụng
interface User {
    name: string;
    age: number;
    email: string;
}

type OptionalUser = Optional<User>;
// = { name?: string; age?: number; email?: string; }

type NullableUser = Nullable<User>;
// = { name: string | null; age: number | null; email: string | null; }

// Advanced mapped type
type Getters<T> = {
    [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<User>;
// = {
//   getName: () => string;
//   getAge: () => number;
//   getEmail: () => string;
// }

// Conditional mapping
type NonFunctionKeys<T> = {
    [K in keyof T]: T[K] extends Function ? never : K;
}[keyof T];

type DataOnly<T> = Pick<T, NonFunctionKeys<T>>;
```

**💡 Takeaway**: Mapped types cho phép tạo types mới từ types cũ một cách tự động.

---

## 23. ❓ Conditional types

```typescript
// Conditional type cơ bản
type IsArray<T> = T extends any[] ? true : false;

type Test1 = IsArray<string[]>;    // true
type Test2 = IsArray<number>;      // false

// Với generic
type ArrayElement<T> = T extends (infer U)[] ? U : T;

type StringElement = ArrayElement<string[]>;  // string
type NumberElement = ArrayElement<number>;    // number

// Practical example: API response wrapper
type ApiResult<T> = T extends string 
    ? { message: T }
    : T extends number
    ? { count: T }
    : { data: T };

type MessageResult = ApiResult<string>;
// = { message: string }

type CountResult = ApiResult<number>;
// = { count: number }

type DataResult = ApiResult<User[]>;
// = { data: User[] }

// Distribute over union
type ToArray<T> = T extends any ? T[] : never;
type Test = ToArray<string | number>;  // string[] | number[]

// Exclude/Extract utilities
type WithoutNull<T> = T extends null ? never : T;
type NonNull = WithoutNull<string | null | number>;  // string | number
```

---

## 24. 🧬 Structural typing là gì?

```typescript
// TypeScript sử dụng structural typing (duck typing)
interface Point {
    x: number;
    y: number;
}

interface Vector {
    x: number;
    y: number;
}

function distance(p1: Point, p2: Point): number {
    return Math.sqrt((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2);
}

// Vector có cùng structure với Point
const vector: Vector = { x: 1, y: 2 };
const point: Point = { x: 3, y: 4 };

console.log(distance(vector, point));  // OK! Structural typing

// Object literal có thêm properties
const extendedPoint = { x: 1, y: 2, z: 3, color: "red" };
console.log(distance(extendedPoint, point));  // OK! Extra properties ignored

// Nhưng cẩn thận với excess property checks
function createPoint(config: Point): Point {
    return config;
}

// Error: excess property 'z'
// createPoint({ x: 1, y: 2, z: 3 });

// OK: assign to variable first
const config = { x: 1, y: 2, z: 3 };
createPoint(config);  // OK
```

**So với Python**: Giống duck typing nhưng được kiểm tra tại compile time.

---

## 25. 🧠 So sánh với Python TypedDict

### Python TypedDict
```python
from typing import TypedDict, Optional

class User(TypedDict):
    name: str
    age: int
    email: Optional[str]

# Usage
user: User = {"name": "Alice", "age": 30}  # email optional
```

### TypeScript Interface (mạnh mẽ hơn)
```typescript
interface User {
    name: string;
    age: number;
    email?: string;
}

// Advanced features không có trong Python
interface UserMethods {
    getName(): string;
    setAge(age: number): void;
}

// Inheritance
interface AdminUser extends User {
    permissions: string[];
}

// Index signatures
interface FlexibleUser {
    name: string;
    [key: string]: any;  // Allow additional properties
}

// Computed property names
type DynamicUser = {
    [K in `user_${string}`]: string;
};
```

**💡 Takeaway**: TS interfaces linh hoạt hơn Python TypedDict rất nhiều.

---

## 26. 🧪 Refactor JS cũ sang TS

### JavaScript code cũ
```javascript
// Legacy JavaScript
function processUsers(users) {
    return users
        .filter(user => user.active)
        .map(user => ({
            id: user.id,
            displayName: user.firstName + ' ' + user.lastName,
            email: user.email.toLowerCase()
        }))
        .sort((a, b) => a.displayName.localeCompare(b.displayName));
}

const apiCall = async (url, options) => {
    const response = await fetch(url, options);
    const data = await response.json();
    return data;
};
```

### Refactored TypeScript
```typescript
// TypeScript with proper typing
interface RawUser {
    id: number;
    firstName: string;
    lastName: string;
    email: string;
    active: boolean;
}

interface ProcessedUser {
    id: number;
    displayName: string;
    email: string;
}

function processUsers(users: RawUser[]): ProcessedUser[] {
    return users
        .filter((user): user is RawUser & { active: true } => user.active)
        .map(user => ({
            id: user.id,
            displayName: `${user.firstName} ${user.lastName}`,
            email: user.email.toLowerCase()
        }))
        .sort((a, b) => a.displayName.localeCompare(b.displayName));
}

// Generic API call
async function apiCall<T = unknown>(
    url: string, 
    options?: RequestInit
): Promise<T> {
    const response = await fetch(url, options);
    
    if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    
    const data = await response.json();
    return data as T;
}

// Usage with type safety
const users = await apiCall<RawUser[]>('/api/users');
const processed = processUsers(users);
```

---

## 27. 🧪 Bài tập: Viết groupBy() có kiểu

```typescript
// Challenge: Implement typed groupBy function
function groupBy<T, K extends keyof T>(
    array: T[], 
    key: K
): Record<string, T[]> {
    return array.reduce((groups, item) => {
        const groupKey = String(item[key]);
        if (!groups[groupKey]) {
            groups[groupKey] = [];
        }
        groups[groupKey].push(item);
        return groups;
    }, {} as Record<string, T[]>);
}

// More advanced version with custom key function
function groupByFn<T, K extends string | number>(
    array: T[],
    keyFn: (item: T) => K
): Record<K, T[]> {
    return array.reduce((groups, item) => {
        const key = keyFn(item);
        if (!groups[key]) {
            groups[key] = [] as T[];
        }
        groups[key].push(item);
        return groups;
    }, {} as Record<K, T[]>);
}

// Usage examples
interface Person {
    name: string;
    age: number;
    department: string;
}

const people: Person[] = [
    { name: "Alice", age: 30, department: "Engineering" },
    { name: "Bob", age: 25, department: "Marketing" },
    { name: "Charlie", age: 35, department: "Engineering" }
];

// Group by property
const byDepartment = groupBy(people, 'department');
// Type: Record<string, Person[]>

// Group by custom function
const byAgeGroup = groupByFn(people, person => 
    person.age < 30 ? 'young' : 'experienced'
);
// Type: Record<'young' | 'experienced', Person[]>
```

---

## 🎯 Bài tập tuần 3

### Bài 1: Utility Type Creator
Tạo utility type `DeepPartial<T>` làm tất cả properties (kể cả nested) thành optional.

### Bài 2: API Response Wrapper
Viết generic type `ApiResponse<T>` và function `handleApiResponse<T>()` với error handling.

### Bài 3: Form Validation
Tạo type system cho form validation với conditional types.

### Bài 4: Database Model
Thiết kế typed database model với relationships sử dụng mapped types.

---

## 🚀 Tuần tới

👉 **[Tuần 4: Ứng dụng TypeScript trong Frontend / AI](./week-4.md)**

Tuần cuối chúng ta sẽ áp dụng tất cả kiến thức đã học vào React và AI applications!

---

*🎉 Chúc mừng bạn đã hoàn thành tuần 3 - phần khó nhất!*
