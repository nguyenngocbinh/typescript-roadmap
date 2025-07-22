# 🧰 BONUS – Chủ đề mở rộng

Chúc mừng bạn đã hoàn thành 4 tuần cơ bản! Phần bonus này sẽ giúp bạn trở thành TypeScript developer hoàn thiện với các kỹ năng cần thiết trong môi trường production.

---

## 37. 🧪 Viết test với Jest có type

### Setup Jest với TypeScript
```bash
# Install dependencies
npm install --save-dev jest @types/jest ts-jest typescript

# Generate Jest config
npx ts-jest config:init
```

### jest.config.js
```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/__tests__/**/*.ts', '**/?(*.)+(spec|test).ts'],
  transform: {
    '^.+\\.ts$': 'ts-jest',
  },
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
  ],
};
```

### Viết tests có type
```typescript
// src/utils/math.ts
export function add(a: number, b: number): number {
    return a + b;
}

export function divide(a: number, b: number): number {
    if (b === 0) {
        throw new Error('Cannot divide by zero');
    }
    return a / b;
}

export class Calculator {
    private history: string[] = [];

    add(a: number, b: number): number {
        const result = add(a, b);
        this.history.push(`${a} + ${b} = ${result}`);
        return result;
    }

    getHistory(): readonly string[] {
        return [...this.history];
    }

    clear(): void {
        this.history = [];
    }
}

// src/utils/__tests__/math.test.ts
import { add, divide, Calculator } from '../math';

describe('Math utilities', () => {
    describe('add function', () => {
        test('should add two positive numbers', () => {
            expect(add(2, 3)).toBe(5);
        });

        test('should add negative numbers', () => {
            expect(add(-2, -3)).toBe(-5);
        });

        test('should add zero', () => {
            expect(add(5, 0)).toBe(5);
        });
    });

    describe('divide function', () => {
        test('should divide two numbers', () => {
            expect(divide(10, 2)).toBe(5);
        });

        test('should throw error when dividing by zero', () => {
            expect(() => divide(10, 0)).toThrow('Cannot divide by zero');
        });

        test('should handle decimal results', () => {
            expect(divide(1, 3)).toBeCloseTo(0.333, 2);
        });
    });

    describe('Calculator class', () => {
        let calculator: Calculator;

        beforeEach(() => {
            calculator = new Calculator();
        });

        test('should perform addition and track history', () => {
            const result = calculator.add(2, 3);
            expect(result).toBe(5);
            
            const history = calculator.getHistory();
            expect(history).toHaveLength(1);
            expect(history[0]).toBe('2 + 3 = 5');
        });

        test('should clear history', () => {
            calculator.add(1, 1);
            calculator.clear();
            expect(calculator.getHistory()).toHaveLength(0);
        });

        test('should return readonly history', () => {
            calculator.add(1, 1);
            const history = calculator.getHistory();
            
            // TypeScript should prevent this modification
            // history.push('invalid'); // This would be a TypeScript error
            expect(history).toHaveLength(1);
        });
    });
});

// Advanced testing với mocks
// src/services/__tests__/api.test.ts
import { ApiClient } from '../api';

// Mock fetch
global.fetch = jest.fn();
const mockFetch = fetch as jest.MockedFunction<typeof fetch>;

interface User {
    id: number;
    name: string;
    email: string;
}

describe('ApiClient', () => {
    let apiClient: ApiClient;

    beforeEach(() => {
        apiClient = new ApiClient('https://api.example.com');
        mockFetch.mockClear();
    });

    test('should fetch user successfully', async () => {
        const mockUser: User = {
            id: 1,
            name: 'John Doe',
            email: 'john@example.com'
        };

        mockFetch.mockResolvedValueOnce({
            ok: true,
            json: async () => mockUser,
        } as Response);

        const user = await apiClient.getUser(1);
        
        expect(mockFetch).toHaveBeenCalledWith(
            'https://api.example.com/users/1',
            expect.objectContaining({
                method: 'GET',
                headers: expect.objectContaining({
                    'Content-Type': 'application/json'
                })
            })
        );
        
        expect(user).toEqual(mockUser);
    });

    test('should handle API errors', async () => {
        mockFetch.mockResolvedValueOnce({
            ok: false,
            status: 404,
            statusText: 'Not Found'
        } as Response);

        await expect(apiClient.getUser(999))
            .rejects
            .toThrow('API request failed: 404 Not Found');
    });
});

// React component testing
// src/components/__tests__/Button.test.tsx
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import '@testing-library/jest-dom';
import { Button } from '../Button';

describe('Button component', () => {
    test('renders with correct text', () => {
        render(<Button text="Click me" onClick={() => {}} />);
        expect(screen.getByText('Click me')).toBeInTheDocument();
    });

    test('calls onClick when clicked', () => {
        const handleClick = jest.fn();
        render(<Button text="Click me" onClick={handleClick} />);
        
        fireEvent.click(screen.getByText('Click me'));
        expect(handleClick).toHaveBeenCalledTimes(1);
    });

    test('is disabled when disabled prop is true', () => {
        render(<Button text="Click me" onClick={() => {}} disabled />);
        expect(screen.getByText('Click me')).toBeDisabled();
    });

    test('applies correct variant class', () => {
        render(<Button text="Click me" onClick={() => {}} variant="secondary" />);
        expect(screen.getByText('Click me')).toHaveClass('btn-secondary');
    });
});
```

**💡 Benefits của TypeScript trong testing**:
- Type safety cho test data
- Better autocomplete trong test IDE
- Compile-time error cho invalid test scenarios
- Mock typing cho external dependencies

---

## 38. 🛡️ Linter và Type Checker: ESLint + tsc

### Setup ESLint với TypeScript
```bash
# Install ESLint và TypeScript support
npm install --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin

# Install additional plugins
npm install --save-dev eslint-plugin-react eslint-plugin-react-hooks
```

### .eslintrc.js
```javascript
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 2020,
    sourceType: 'module',
    ecmaFeatures: {
      jsx: true,
    },
  },
  settings: {
    react: {
      version: 'detect',
    },
  },
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
  ],
  plugins: [
    '@typescript-eslint',
    'react',
    'react-hooks',
  ],
  rules: {
    // TypeScript specific rules
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-non-null-assertion': 'warn',
    
    // General rules
    'no-console': 'warn',
    'no-debugger': 'error',
    'prefer-const': 'error',
    'no-var': 'error',
    
    // React specific rules
    'react/react-in-jsx-scope': 'off', // Not needed in React 17+
    'react/prop-types': 'off', // We use TypeScript for prop validation
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',
  },
  env: {
    browser: true,
    es6: true,
    node: true,
    jest: true,
  },
};
```

### package.json scripts
```json
{
  "scripts": {
    "lint": "eslint src/**/*.{ts,tsx}",
    "lint:fix": "eslint src/**/*.{ts,tsx} --fix",
    "type-check": "tsc --noEmit",
    "type-check:watch": "tsc --noEmit --watch",
    "build": "npm run type-check && npm run lint && react-scripts build"
  }
}
```

### Prettier integration
```bash
npm install --save-dev prettier eslint-config-prettier eslint-plugin-prettier
```

### .prettierrc
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false
}
```

### VS Code settings.json
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "typescript.preferences.noSemicolons": false
}
```

### Pre-commit hooks với husky
```bash
npm install --save-dev husky lint-staged

# Setup husky
npx husky install
npx husky add .husky/pre-commit "npx lint-staged"
```

### package.json lint-staged config
```json
{
  "lint-staged": {
    "src/**/*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "git add"
    ]
  }
}
```

---

## 39. ⛑️ Xử lý lỗi có type

```typescript
// Custom error types
abstract class AppError extends Error {
    abstract readonly statusCode: number;
    abstract readonly isOperational: boolean;

    constructor(message: string, public readonly context?: Record<string, any>) {
        super(message);
        this.name = this.constructor.name;
        Error.captureStackTrace(this, this.constructor);
    }
}

class ValidationError extends AppError {
    readonly statusCode = 400;
    readonly isOperational = true;

    constructor(message: string, public readonly field?: string) {
        super(message);
    }
}

class NotFoundError extends AppError {
    readonly statusCode = 404;
    readonly isOperational = true;
}

class DatabaseError extends AppError {
    readonly statusCode = 500;
    readonly isOperational = false;
}

// Result type pattern (like Rust Result<T, E>)
type Result<T, E = Error> = 
    | { success: true; data: T }
    | { success: false; error: E };

// Safe async function wrapper
async function safeAsync<T>(
    fn: () => Promise<T>
): Promise<Result<T, Error>> {
    try {
        const data = await fn();
        return { success: true, data };
    } catch (error) {
        return { 
            success: false, 
            error: error instanceof Error ? error : new Error(String(error))
        };
    }
}

// Either type (functional programming approach)
type Either<L, R> = Left<L> | Right<R>;

class Left<L> {
    constructor(public readonly value: L) {}
    isLeft(): this is Left<L> { return true; }
    isRight(): this is Right<any> { return false; }
}

class Right<R> {
    constructor(public readonly value: R) {}
    isLeft(): this is Left<any> { return false; }
    isRight(): this is Right<R> { return true; }
}

// API service với proper error handling
class UserService {
    async getUser(id: string): Promise<Result<User, AppError>> {
        if (!id.trim()) {
            return { 
                success: false, 
                error: new ValidationError('User ID is required', 'id') 
            };
        }

        const result = await safeAsync(async () => {
            const response = await fetch(`/api/users/${id}`);
            
            if (!response.ok) {
                if (response.status === 404) {
                    throw new NotFoundError(`User with ID ${id} not found`);
                }
                throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }
            
            return response.json() as Promise<User>;
        });

        if (!result.success) {
            const error = result.error;
            if (error instanceof AppError) {
                return { success: false, error };
            }
            return { 
                success: false, 
                error: new DatabaseError('Failed to fetch user', { originalError: error.message })
            };
        }

        return { success: true, data: result.data };
    }

    async createUser(userData: CreateUserRequest): Promise<Either<ValidationError[], User>> {
        const errors: ValidationError[] = [];

        if (!userData.name?.trim()) {
            errors.push(new ValidationError('Name is required', 'name'));
        }

        if (!userData.email?.trim()) {
            errors.push(new ValidationError('Email is required', 'email'));
        } else if (!isValidEmail(userData.email)) {
            errors.push(new ValidationError('Invalid email format', 'email'));
        }

        if (errors.length > 0) {
            return new Left(errors);
        }

        try {
            const response = await fetch('/api/users', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(userData)
            });

            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }

            const user = await response.json() as User;
            return new Right(user);
        } catch (error) {
            return new Left([new ValidationError('Failed to create user')]);
        }
    }
}

// React error boundary với TypeScript
interface ErrorBoundaryState {
    hasError: boolean;
    error?: Error;
    errorInfo?: React.ErrorInfo;
}

class ErrorBoundary extends React.Component<
    React.PropsWithChildren<{}>,
    ErrorBoundaryState
> {
    constructor(props: React.PropsWithChildren<{}>) {
        super(props);
        this.state = { hasError: false };
    }

    static getDerivedStateFromError(error: Error): ErrorBoundaryState {
        return { hasError: true, error };
    }

    componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
        this.setState({ error, errorInfo });
        
        // Log to error reporting service
        console.error('Error caught by boundary:', error, errorInfo);
    }

    render() {
        if (this.state.hasError) {
            return (
                <div className="error-boundary">
                    <h2>Something went wrong.</h2>
                    <details style={{ whiteSpace: 'pre-wrap' }}>
                        <summary>Error details</summary>
                        {this.state.error?.toString()}
                        <br />
                        {this.state.errorInfo?.componentStack}
                    </details>
                </div>
            );
        }

        return this.props.children;
    }
}

// Hook for error handling
function useErrorHandler() {
    const [error, setError] = useState<AppError | null>(null);

    const handleError = useCallback((error: unknown) => {
        if (error instanceof AppError) {
            setError(error);
        } else if (error instanceof Error) {
            setError(new DatabaseError(error.message));
        } else {
            setError(new DatabaseError('Unknown error occurred'));
        }
    }, []);

    const clearError = useCallback(() => {
        setError(null);
    }, []);

    return { error, handleError, clearError };
}

// Usage trong component
function UserProfile({ userId }: { userId: string }) {
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState(false);
    const { error, handleError, clearError } = useErrorHandler();

    useEffect(() => {
        async function fetchUser() {
            setLoading(true);
            clearError();

            const userService = new UserService();
            const result = await userService.getUser(userId);

            if (result.success) {
                setUser(result.data);
            } else {
                handleError(result.error);
            }

            setLoading(false);
        }

        fetchUser();
    }, [userId, handleError, clearError]);

    if (loading) return <div>Loading...</div>;
    
    if (error) {
        if (error instanceof ValidationError) {
            return <div className="error">Validation Error: {error.message}</div>;
        }
        if (error instanceof NotFoundError) {
            return <div className="error">User not found</div>;
        }
        return <div className="error">Something went wrong: {error.message}</div>;
    }

    return user ? <div>Hello, {user.name}!</div> : null;
}

function isValidEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

---

## 40. 🧠 Case study: TypeScript vs Python

### Cùng một chức năng: URL Shortener Service

#### Python version
```python
# url_shortener.py
from typing import Dict, Optional, List
from dataclasses import dataclass
from datetime import datetime
import hashlib
import json

@dataclass
class UrlRecord:
    id: str
    original_url: str
    short_code: str
    created_at: datetime
    clicks: int = 0

class UrlShortener:
    def __init__(self):
        self.urls: Dict[str, UrlRecord] = {}
        self.codes: Dict[str, str] = {}  # short_code -> id mapping
    
    def shorten(self, url: str) -> Optional[str]:
        if not self._is_valid_url(url):
            return None
        
        # Check if URL already exists
        for record in self.urls.values():
            if record.original_url == url:
                return record.short_code
        
        # Generate new short code
        url_id = self._generate_id(url)
        short_code = self._generate_short_code(url)
        
        record = UrlRecord(
            id=url_id,
            original_url=url,
            short_code=short_code,
            created_at=datetime.now()
        )
        
        self.urls[url_id] = record
        self.codes[short_code] = url_id
        
        return short_code
    
    def expand(self, short_code: str) -> Optional[str]:
        url_id = self.codes.get(short_code)
        if not url_id:
            return None
        
        record = self.urls.get(url_id)
        if record:
            record.clicks += 1
            return record.original_url
        
        return None
    
    def get_stats(self, short_code: str) -> Optional[Dict]:
        url_id = self.codes.get(short_code)
        if not url_id:
            return None
        
        record = self.urls.get(url_id)
        if record:
            return {
                "original_url": record.original_url,
                "short_code": record.short_code,
                "created_at": record.created_at.isoformat(),
                "clicks": record.clicks
            }
        
        return None
    
    def _is_valid_url(self, url: str) -> bool:
        return url.startswith(('http://', 'https://'))
    
    def _generate_id(self, url: str) -> str:
        return hashlib.md5(url.encode()).hexdigest()[:8]
    
    def _generate_short_code(self, url: str) -> str:
        return hashlib.sha256(url.encode()).hexdigest()[:6]
```

#### TypeScript version (tương đương)
```typescript
// urlShortener.ts
interface UrlRecord {
    id: string;
    originalUrl: string;
    shortCode: string;
    createdAt: Date;
    clicks: number;
}

interface UrlStats {
    originalUrl: string;
    shortCode: string;
    createdAt: string;
    clicks: number;
}

type ShortenResult = 
    | { success: true; shortCode: string }
    | { success: false; error: string };

type ExpandResult = 
    | { success: true; originalUrl: string }
    | { success: false; error: string };

class UrlShortener {
    private urls = new Map<string, UrlRecord>();
    private codes = new Map<string, string>(); // shortCode -> id mapping

    shorten(url: string): ShortenResult {
        if (!this.isValidUrl(url)) {
            return { success: false, error: 'Invalid URL format' };
        }

        // Check if URL already exists
        for (const record of this.urls.values()) {
            if (record.originalUrl === url) {
                return { success: true, shortCode: record.shortCode };
            }
        }

        // Generate new short code
        const urlId = this.generateId(url);
        const shortCode = this.generateShortCode(url);

        const record: UrlRecord = {
            id: urlId,
            originalUrl: url,
            shortCode,
            createdAt: new Date(),
            clicks: 0
        };

        this.urls.set(urlId, record);
        this.codes.set(shortCode, urlId);

        return { success: true, shortCode };
    }

    expand(shortCode: string): ExpandResult {
        const urlId = this.codes.get(shortCode);
        if (!urlId) {
            return { success: false, error: 'Short code not found' };
        }

        const record = this.urls.get(urlId);
        if (!record) {
            return { success: false, error: 'URL record not found' };
        }

        record.clicks++;
        return { success: true, originalUrl: record.originalUrl };
    }

    getStats(shortCode: string): UrlStats | null {
        const urlId = this.codes.get(shortCode);
        if (!urlId) return null;

        const record = this.urls.get(urlId);
        if (!record) return null;

        return {
            originalUrl: record.originalUrl,
            shortCode: record.shortCode,
            createdAt: record.createdAt.toISOString(),
            clicks: record.clicks
        };
    }

    getAllUrls(): UrlRecord[] {
        return Array.from(this.urls.values());
    }

    private isValidUrl(url: string): boolean {
        try {
            new URL(url);
            return url.startsWith('http://') || url.startsWith('https://');
        } catch {
            return false;
        }
    }

    private generateId(url: string): string {
        return this.hash(url, 'md5').substring(0, 8);
    }

    private generateShortCode(url: string): string {
        return this.hash(url, 'sha256').substring(0, 6);
    }

    private hash(text: string, algorithm: 'md5' | 'sha256'): string {
        // In browser, use crypto.subtle or a library
        // In Node.js, use crypto module
        const crypto = require('crypto');
        return crypto.createHash(algorithm).update(text).digest('hex');
    }
}

// Advanced TypeScript version with generics and better error handling
class EnhancedUrlShortener<TMetadata = Record<string, any>> {
    private urls = new Map<string, UrlRecord & { metadata?: TMetadata }>();
    private codes = new Map<string, string>();

    async shortenWithMetadata(
        url: string, 
        metadata?: TMetadata
    ): Promise<ShortenResult> {
        const validation = await this.validateUrl(url);
        if (!validation.valid) {
            return { success: false, error: validation.error };
        }

        // Rest of the implementation with metadata support
        const urlId = this.generateId(url);
        const shortCode = this.generateShortCode(url);

        const record = {
            id: urlId,
            originalUrl: url,
            shortCode,
            createdAt: new Date(),
            clicks: 0,
            metadata
        };

        this.urls.set(urlId, record);
        this.codes.set(shortCode, urlId);

        return { success: true, shortCode };
    }

    private async validateUrl(url: string): Promise<{ valid: boolean; error?: string }> {
        if (!this.isValidUrl(url)) {
            return { valid: false, error: 'Invalid URL format' };
        }

        // Additional async validation (e.g., check if URL is reachable)
        try {
            const response = await fetch(url, { method: 'HEAD' });
            if (!response.ok) {
                return { valid: false, error: 'URL is not reachable' };
            }
        } catch {
            return { valid: false, error: 'URL validation failed' };
        }

        return { valid: true };
    }

    // ... rest of the methods
}

// Usage comparison
async function demo() {
    const shortener = new UrlShortener();
    
    // TypeScript provides better autocomplete and error checking
    const result = shortener.shorten('https://example.com');
    
    if (result.success) {
        console.log('Short code:', result.shortCode);
        
        const expandResult = shortener.expand(result.shortCode);
        if (expandResult.success) {
            console.log('Original URL:', expandResult.originalUrl);
        }
    } else {
        console.error('Error:', result.error);
    }

    // Type-safe stats
    const stats = shortener.getStats(result.success ? result.shortCode : '');
    if (stats) {
        console.log(`URL has been clicked ${stats.clicks} times`);
    }
}
```

### So sánh key differences:

| Aspect | Python | TypeScript |
|--------|--------|------------|
| **Type Safety** | Runtime checking | Compile-time checking |
| **Error Handling** | Exceptions | Result types, better error modeling |
| **Tooling** | Good (mypy, pylint) | Excellent (native TS, IDE support) |
| **Performance** | Slower startup | Faster (compiled to JS) |
| **Deployment** | Need Python runtime | Compiles to JS, universal |
| **Learning Curve** | Easier | Steeper (type system) |
| **Ecosystem** | Mature, broad | Large, frontend-focused |

**💡 Conclusion**: TypeScript provides better developer experience và compile-time safety, Python có ecosystem rộng hơn cho data science/ML.

---

## 41. 📚 Tự tạo type-challenges của riêng bạn

```typescript
// Challenge 1: Deep Readonly
// Make all properties (including nested) readonly
type DeepReadonly<T> = {
    readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// Test
interface User {
    name: string;
    profile: {
        age: number;
        address: {
            street: string;
            city: string;
        };
    };
}

type ReadonlyUser = DeepReadonly<User>;
// Should not allow modifications at any level

// Challenge 2: Function Curry
// Create a curry type that converts function arguments to curried form
type Curry<T> = T extends (...args: infer A) => infer R
    ? A extends [infer First, ...infer Rest]
        ? (arg: First) => Curry<(...args: Rest) => R>
        : R
    : never;

// Test
declare function curry<T extends (...args: any[]) => any>(fn: T): Curry<T>;

const add = (a: number, b: number, c: number) => a + b + c;
const curriedAdd = curry(add);
// Should work: curriedAdd(1)(2)(3) === 6

// Challenge 3: Object Path
// Extract all possible paths from an object type
type Paths<T, K extends keyof T = keyof T> = K extends string
    ? T[K] extends Record<string, any>
        ? K | `${K}.${Paths<T[K]>}`
        : K
    : never;

// Test
interface Config {
    server: {
        host: string;
        port: number;
        ssl: {
            enabled: boolean;
            cert: string;
        };
    };
    database: {
        url: string;
    };
}

type ConfigPaths = Paths<Config>;
// Should be: "server" | "database" | "server.host" | "server.port" | 
//           "server.ssl" | "server.ssl.enabled" | "server.ssl.cert" | "database.url"

// Challenge 4: Type-safe Object.get()
// Create a function that can safely get nested properties
type GetType<T, P extends string> = P extends `${infer K}.${infer Rest}`
    ? K extends keyof T
        ? GetType<T[K], Rest>
        : undefined
    : P extends keyof T
    ? T[P]
    : undefined;

declare function get<T, P extends Paths<T>>(obj: T, path: P): GetType<T, P>;

// Test
const config: Config = {
    server: { host: 'localhost', port: 3000, ssl: { enabled: true, cert: 'path' } },
    database: { url: 'mongodb://localhost' }
};

const host = get(config, 'server.host'); // Should be string
const enabled = get(config, 'server.ssl.enabled'); // Should be boolean

// Challenge 5: Required Keys
// Extract only the required (non-optional) keys from a type
type RequiredKeys<T> = {
    [K in keyof T]-?: {} extends Pick<T, K> ? never : K;
}[keyof T];

type OptionalKeys<T> = {
    [K in keyof T]-?: {} extends Pick<T, K> ? K : never;
}[keyof T];

// Test
interface Person {
    name: string;
    age?: number;
    email: string;
    phone?: string;
}

type PersonRequiredKeys = RequiredKeys<Person>; // "name" | "email"
type PersonOptionalKeys = OptionalKeys<Person>; // "age" | "phone"

// Challenge 6: Tuple to Union
// Convert a tuple type to a union of its element types
type TupleToUnion<T extends readonly any[]> = T[number];

// Test
type Colors = ['red', 'green', 'blue'];
type ColorUnion = TupleToUnion<Colors>; // 'red' | 'green' | 'blue'

// Challenge 7: Merge Types
// Merge two types, with the second type overriding the first
type Merge<T, U> = {
    [K in keyof T | keyof U]: K extends keyof U
        ? U[K]
        : K extends keyof T
        ? T[K]
        : never;
};

// Test
type BaseUser = { id: number; name: string; age: number };
type UserUpdate = { name: string; email: string };
type UpdatedUser = Merge<BaseUser, UserUpdate>;
// Should be: { id: number; name: string; age: number; email: string }

// Challenge 8: Function Overload Builder
// Create a type that builds function overloads
type OverloadBuilder<T extends Record<string, (...args: any[]) => any>> = {
    [K in keyof T]: T[K];
}[keyof T];

// Test
interface ProcessorMap {
    string: (input: string) => string;
    number: (input: number) => number;
    boolean: (input: boolean) => boolean;
}

declare const process: OverloadBuilder<ProcessorMap>;
// Should support: process("hello"), process(42), process(true)

// Challenge 9: Recursive Object Validation
// Create a validation schema type
type ValidationSchema<T> = {
    [K in keyof T]: T[K] extends string
        ? { type: 'string'; minLength?: number; maxLength?: number }
        : T[K] extends number
        ? { type: 'number'; min?: number; max?: number }
        : T[K] extends boolean
        ? { type: 'boolean' }
        : T[K] extends object
        ? { type: 'object'; schema: ValidationSchema<T[K]> }
        : { type: 'any' };
};

// Test
const userSchema: ValidationSchema<User> = {
    name: { type: 'string', minLength: 1 },
    profile: {
        type: 'object',
        schema: {
            age: { type: 'number', min: 0 },
            address: {
                type: 'object',
                schema: {
                    street: { type: 'string' },
                    city: { type: 'string' }
                }
            }
        }
    }
};

// Challenge 10: State Machine Types
// Create a type-safe state machine
type StateMachine<TStates extends string, TEvents extends string> = {
    [State in TStates]: {
        [Event in TEvents]?: TStates;
    };
};

type CreateMachine<T extends StateMachine<any, any>> = T;

// Test
type TrafficLightStates = 'red' | 'yellow' | 'green';
type TrafficLightEvents = 'timer' | 'emergency';

const trafficLight = {
    red: { timer: 'green' as const },
    yellow: { timer: 'red' as const },
    green: { timer: 'yellow' as const, emergency: 'red' as const }
} satisfies StateMachine<TrafficLightStates, TrafficLightEvents>;

// Practical implementation helper
function createChallenge<T>(
    description: string,
    solution: T,
    tests: Array<{ input: any; expected: any }>
) {
    return {
        description,
        solution,
        tests,
        validate: () => {
            // Runtime validation logic
            return tests.every(test => {
                // Implement test validation
                return true;
            });
        }
    };
}

// Your turn: Create more challenges!
// Ideas:
// - JSON Parser types
// - SQL Query builder types  
// - Event emitter with typed events
// - Reactive state management types
// - API route parameter extraction
```

---

## 🎯 Final Project Ideas

### 1. **Full-stack Todo App**
- TypeScript backend (Node.js/Deno)
- React frontend với TypeScript
- Shared types package
- Real-time updates với WebSocket
- Authentication & authorization

### 2. **AI-powered Code Assistant**
- CLI tool với TypeScript
- OpenAI API integration
- Type-safe plugin system
- File analysis và suggestions

### 3. **Type-safe API Gateway**
- Express/Fastify server
- Automatic OpenAPI generation
- Request/response validation
- Rate limiting với typed configs

### 4. **Real-time Chat Application**
- Socket.io với TypeScript
- React frontend
- Message encryption
- File sharing support

### 5. **Data Visualization Dashboard**
- D3.js với TypeScript
- Multiple chart types
- Real-time data streaming
- Export functionality

---

## 🚀 Next Steps

**Congratulations!** 🎉 Bạn đã hoàn thành toàn bộ lộ trình TypeScript. Bây giờ bạn có thể:

1. **Contribute to open source**: Tìm TypeScript projects trên GitHub
2. **Build portfolio projects**: Áp dụng kiến thức vào project thực tế
3. **Learn advanced topics**: GraphQL, Microservices, Design Patterns
4. **Explore ecosystem**: Next.js, NestJS, Prisma, tRPC
5. **Share knowledge**: Viết blog, tạo tutorials

### 📚 Recommended Learning Path

- **Beginner → Intermediate**: Framework deep dive (React/Vue/Angular)
- **Intermediate → Advanced**: Backend frameworks (NestJS, Fastify)
- **Advanced → Expert**: System design, Performance optimization
- **Expert → Architect**: Lead teams, Design large systems

---

*🎉 Chúc mừng bạn đã trở thành TypeScript developer! Keep coding and keep learning!* 

**Remember**: The best way to learn is by building. Start your first TypeScript project today! 🚀
