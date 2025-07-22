# 🗓️ Tuần 2: Làm việc với project thực tế (Node.js / Deno)

Tuần này chúng ta sẽ học cách áp dụng TypeScript vào các project thực tế, từ module system đến việc xây dựng REST API đơn giản.

---

## 11. 📦 Module system trong TypeScript

### Python imports
```python
# Python
from utils import helper_function
from models.user import User
import json
```

### TypeScript imports/exports
```typescript
// TypeScript - Named exports/imports
export function helperFunction() { /* ... */ }
export class User { /* ... */ }

import { helperFunction, User } from './utils';
import { readFile } from 'fs/promises';

// Default export/import
export default class Database { /* ... */ }
import Database from './database';

// Import everything
import * as fs from 'fs/promises';
```

**💡 Takeaway**: TS module system rõ ràng hơn Python, ít bị conflict namespace.

---

## 12. 🔃 Async/Await trong TS

```typescript
// Giống Python async/await
async function fetchUser(id: number): Promise<User> {
    try {
        const response = await fetch(`/api/users/${id}`);
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);
        }
        const userData = await response.json();
        return userData as User;
    } catch (error) {
        console.error('Fetch failed:', error);
        throw error;
    }
}

// Promise chaining (ít dùng)
function fetchUserOldWay(id: number): Promise<User> {
    return fetch(`/api/users/${id}`)
        .then(response => response.json())
        .then(data => data as User);
}
```

**So với Python**:
```python
async def fetch_user(id: int) -> User:
    async with httpx.AsyncClient() as client:
        response = await client.get(f"/api/users/{id}")
        return response.json()
```

---

## 13. 🧾 Đọc/ghi file với Node.js

```typescript
import { readFile, writeFile } from 'fs/promises';
import { existsSync } from 'fs';

// Đọc file
async function loadConfig(): Promise<Config> {
    try {
        const data = await readFile('config.json', 'utf8');
        return JSON.parse(data) as Config;
    } catch (error) {
        console.error('Cannot load config:', error);
        return getDefaultConfig();
    }
}

// Ghi file
async function saveData(data: any[], filename: string): Promise<void> {
    const jsonData = JSON.stringify(data, null, 2);
    await writeFile(filename, jsonData, 'utf8');
    console.log(`Saved ${data.length} records to ${filename}`);
}

// Kiểm tra file tồn tại
if (existsSync('data.csv')) {
    const csvContent = await readFile('data.csv', 'utf8');
    // Process CSV...
}
```

**So với Python**:
```python
# Python
with open('config.json', 'r') as f:
    config = json.load(f)
```

---

## 14. 🌐 Gọi HTTP API với fetch

```typescript
interface ApiResponse<T> {
    data: T;
    status: number;
    message?: string;
}

class ApiClient {
    private baseUrl: string;
    
    constructor(baseUrl: string) {
        this.baseUrl = baseUrl;
    }
    
    async get<T>(endpoint: string): Promise<T> {
        const response = await fetch(`${this.baseUrl}${endpoint}`, {
            method: 'GET',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${this.getToken()}`
            }
        });
        
        if (!response.ok) {
            throw new Error(`GET ${endpoint} failed: ${response.status}`);
        }
        
        return response.json() as T;
    }
    
    async post<T>(endpoint: string, data: any): Promise<T> {
        const response = await fetch(`${this.baseUrl}${endpoint}`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(data)
        });
        
        return response.json() as T;
    }
    
    private getToken(): string {
        return process.env.API_TOKEN || '';
    }
}

// Sử dụng
const api = new ApiClient('https://api.example.com');
const users = await api.get<User[]>('/users');
```

---

## 15. ⚙️ Giải thích tsconfig.json

```json
{
  "compilerOptions": {
    // Target JavaScript version
    "target": "ES2020",
    
    // Module system
    "module": "ESNext",
    "moduleResolution": "node",
    
    // Type checking
    "strict": true,                    // Bật tất cả strict checks
    "noImplicitAny": true,            // Không cho phép any ngầm định
    "strictNullChecks": true,         // Kiểm tra null/undefined
    
    // Output
    "outDir": "./dist",               // Thư mục output
    "rootDir": "./src",               // Thư mục source
    
    // Advanced
    "esModuleInterop": true,          // Tương thích CommonJS
    "skipLibCheck": true,             // Skip checking .d.ts files
    "forceConsistentCasingInFileNames": true
  },
  
  // Files to include/exclude
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

**💡 Tip**: Luôn bật `strict: true` để có type safety tốt nhất.

---

## 16. 🏃 Cách chạy file .ts nhanh nhất

### Option 1: ts-node (Node.js)
```bash
# Install
npm install -g ts-node typescript

# Run directly
ts-node script.ts

# With tsconfig
ts-node --project tsconfig.json script.ts
```

### Option 2: Bun (fastest)
```bash
# Install Bun
curl -fsSL https://bun.sh/install | bash

# Run directly (no compilation needed)
bun run script.ts
```

### Option 3: Deno (secure by default)
```bash
# Install Deno
curl -fsSL https://deno.land/x/install/install.sh | sh

# Run with permissions
deno run --allow-read --allow-write script.ts

# Or allow all
deno run --allow-all script.ts
```

**💡 Recommendation**: Dùng Bun cho development, Node.js cho production.

---

## 17. 🧪 Bài tập: Viết REST API đơn giản

### Với Express (Node.js)
```typescript
import express, { Request, Response } from 'express';

interface User {
    id: number;
    name: string;
    email: string;
}

const app = express();
app.use(express.json());

let users: User[] = [
    { id: 1, name: 'Alice', email: 'alice@example.com' },
    { id: 2, name: 'Bob', email: 'bob@example.com' }
];

// GET /users
app.get('/users', (req: Request, res: Response) => {
    res.json(users);
});

// GET /users/:id
app.get('/users/:id', (req: Request, res: Response) => {
    const id = parseInt(req.params.id);
    const user = users.find(u => u.id === id);
    
    if (!user) {
        return res.status(404).json({ error: 'User not found' });
    }
    
    res.json(user);
});

// POST /users
app.post('/users', (req: Request, res: Response) => {
    const { name, email }: Partial<User> = req.body;
    
    if (!name || !email) {
        return res.status(400).json({ error: 'Name and email required' });
    }
    
    const newUser: User = {
        id: users.length + 1,
        name,
        email
    };
    
    users.push(newUser);
    res.status(201).json(newUser);
});

app.listen(3000, () => {
    console.log('Server running on http://localhost:3000');
});
```

### Với Deno (Modern approach)
```typescript
interface User {
    id: number;
    name: string;
    email: string;
}

const users: User[] = [
    { id: 1, name: 'Alice', email: 'alice@example.com' }
];

async function handler(req: Request): Promise<Response> {
    const url = new URL(req.url);
    
    if (req.method === 'GET' && url.pathname === '/users') {
        return new Response(JSON.stringify(users), {
            headers: { 'Content-Type': 'application/json' }
        });
    }
    
    if (req.method === 'POST' && url.pathname === '/users') {
        const body = await req.json();
        const newUser: User = {
            id: users.length + 1,
            name: body.name,
            email: body.email
        };
        users.push(newUser);
        return new Response(JSON.stringify(newUser), { status: 201 });
    }
    
    return new Response('Not Found', { status: 404 });
}

Deno.serve({ port: 3000 }, handler);
```

---

## 18. 📋 Cấu trúc project chuẩn trong TS

```
my-typescript-project/
├── src/
│   ├── controllers/
│   │   └── userController.ts
│   ├── models/
│   │   └── User.ts
│   ├── routes/
│   │   └── userRoutes.ts
│   ├── services/
│   │   └── userService.ts
│   ├── types/
│   │   └── index.ts
│   ├── utils/
│   │   └── helpers.ts
│   └── index.ts
├── tests/
│   └── user.test.ts
├── dist/                  # Compiled output
├── node_modules/
├── package.json
├── tsconfig.json
├── .eslintrc.js
├── .gitignore
└── README.md
```

### package.json mẫu
```json
{
  "name": "my-typescript-project",
  "version": "1.0.0",
  "scripts": {
    "dev": "ts-node src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "jest",
    "lint": "eslint src/**/*.ts"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.0",
    "@types/node": "^18.0.0",
    "typescript": "^5.0.0",
    "ts-node": "^10.9.0"
  }
}
```

---

## 🎯 Bài tập tuần 2

### Bài 1: File Operations
Viết chương trình TypeScript đọc file CSV, parse thành array objects, và ghi ra JSON.

### Bài 2: API Client
Tạo class `WeatherAPI` để gọi API thời tiết và trả về dữ liệu có type.

### Bài 3: Mini REST API
Mở rộng API ở bài 17 thêm PUT và DELETE endpoints.

### Bài 4: Project Setup
Setup một project TypeScript hoàn chỉnh với ESLint, testing framework.

---

## 🚀 Tuần tới

👉 **[Tuần 3: Type nâng cao – Lập trình mạnh mẽ hơn](./week-3.md)**

Tuần tới chúng ta sẽ đi sâu vào các tính năng type system nâng cao của TypeScript!

---

*🎉 Chúc mừng bạn đã hoàn thành tuần 2!*
