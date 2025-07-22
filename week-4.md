# 🗓️ Tuần 4: Ứng dụng TypeScript trong Frontend / AI

Tuần cuối cùng! Chúng ta sẽ áp dụng tất cả kiến thức TypeScript đã học vào việc xây dựng ứng dụng React và tích hợp với AI APIs. Đây là phần thực hành nhiều nhất.

---

## 28. 🔥 TSX là gì trong React?

### Setup React với TypeScript
```bash
# Create React App với TypeScript
npx create-react-app my-app --template typescript

# Hoặc với Vite (faster)
npm create vite@latest my-app -- --template react-ts

# Hoặc với Next.js
npx create-next-app@latest my-app --typescript
```

### TSX vs JSX
```tsx
// TSX = TypeScript + JSX
import React from 'react';

// Component với props typing
interface ButtonProps {
    text: string;
    onClick: () => void;
    variant?: 'primary' | 'secondary';
    disabled?: boolean;
}

// Function component với TypeScript
const Button: React.FC<ButtonProps> = ({ 
    text, 
    onClick, 
    variant = 'primary',
    disabled = false 
}) => {
    return (
        <button
            className={`btn btn-${variant}`}
            onClick={onClick}
            disabled={disabled}
        >
            {text}
        </button>
    );
};

// Usage with type checking
function App() {
    const handleClick = () => {
        console.log('Clicked!');
    };

    return (
        <div>
            <Button text="Save" onClick={handleClick} />
            <Button 
                text="Delete" 
                onClick={handleClick} 
                variant="secondary" 
            />
            {/* Error: missing required prop 'text' */}
            {/* <Button onClick={handleClick} /> */}
        </div>
    );
}

export default App;
```

**So với Python**: Giống như typing cho functions, nhưng cho UI components.

---

## 29. 📦 Props typing trong React

```tsx
import React, { ReactNode } from 'react';

// Basic props interface
interface UserCardProps {
    user: {
        id: number;
        name: string;
        email: string;
        avatar?: string;
    };
    onEdit: (userId: number) => void;
    onDelete: (userId: number) => void;
}

// Props with children
interface ModalProps {
    isOpen: boolean;
    onClose: () => void;
    title: string;
    children: ReactNode;  // hoặc React.ReactNode
}

const Modal: React.FC<ModalProps> = ({ isOpen, onClose, title, children }) => {
    if (!isOpen) return null;

    return (
        <div className="modal-overlay" onClick={onClose}>
            <div className="modal" onClick={e => e.stopPropagation()}>
                <div className="modal-header">
                    <h2>{title}</h2>
                    <button onClick={onClose}>×</button>
                </div>
                <div className="modal-body">
                    {children}
                </div>
            </div>
        </div>
    );
};

// Props với event handlers
interface FormProps {
    onSubmit: (data: FormData) => void;
    onFieldChange: (field: string, value: string) => void;
    initialData?: Partial<FormData>;
}

interface FormData {
    name: string;
    email: string;
    age: number;
}

// Generic component props
interface ListProps<T> {
    items: T[];
    renderItem: (item: T, index: number) => ReactNode;
    keyExtractor: (item: T) => string | number;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
    return (
        <ul>
            {items.map((item, index) => (
                <li key={keyExtractor(item)}>
                    {renderItem(item, index)}
                </li>
            ))}
        </ul>
    );
}

// Usage
interface Product {
    id: number;
    name: string;
    price: number;
}

const products: Product[] = [
    { id: 1, name: "Laptop", price: 999 },
    { id: 2, name: "Mouse", price: 29 }
];

<List<Product>
    items={products}
    keyExtractor={item => item.id}
    renderItem={(product) => (
        <span>{product.name} - ${product.price}</span>
    )}
/>
```

---

## 30. 🎣 useState, useEffect có generic

```tsx
import React, { useState, useEffect } from 'react';

// useState với generic
function UserProfile() {
    // Type inference tự động
    const [name, setName] = useState('');  // string
    const [age, setAge] = useState(0);     // number
    
    // Explicit generic khi cần
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState<boolean>(false);
    
    // Array state
    const [items, setItems] = useState<string[]>([]);
    
    // Object state
    const [config, setConfig] = useState<{
        theme: 'light' | 'dark';
        language: string;
    }>({
        theme: 'light',
        language: 'en'
    });

    // useEffect với dependencies typing
    useEffect(() => {
        async function fetchUser() {
            setLoading(true);
            try {
                const response = await fetch('/api/user');
                const userData = await response.json() as User;
                setUser(userData);
            } catch (error) {
                console.error('Failed to fetch user:', error);
            } finally {
                setLoading(false);
            }
        }

        fetchUser();
    }, []); // Empty deps array

    // useEffect với cleanup
    useEffect(() => {
        const timer = setInterval(() => {
            setAge(prev => prev + 1);
        }, 1000);

        return () => clearInterval(timer);  // Cleanup function
    }, []);

    return (
        <div>
            {loading ? (
                <div>Loading...</div>
            ) : (
                <div>
                    <h1>{user?.name || 'Unknown'}</h1>
                    <p>Age: {age}</p>
                </div>
            )}
        </div>
    );
}

// Custom hook với TypeScript
interface UseApiResult<T> {
    data: T | null;
    loading: boolean;
    error: string | null;
    refetch: () => void;
}

function useApi<T>(url: string): UseApiResult<T> {
    const [data, setData] = useState<T | null>(null);
    const [loading, setLoading] = useState<boolean>(true);
    const [error, setError] = useState<string | null>(null);

    const fetchData = async () => {
        try {
            setLoading(true);
            setError(null);
            const response = await fetch(url);
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }
            const result = await response.json() as T;
            setData(result);
        } catch (err) {
            setError(err instanceof Error ? err.message : 'Unknown error');
        } finally {
            setLoading(false);
        }
    };

    useEffect(() => {
        fetchData();
    }, [url]);

    return {
        data,
        loading,
        error,
        refetch: fetchData
    };
}

// Usage
function UserList() {
    const { data: users, loading, error } = useApi<User[]>('/api/users');

    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error}</div>;

    return (
        <ul>
            {users?.map(user => (
                <li key={user.id}>{user.name}</li>
            ))}
        </ul>
    );
}
```

---

## 31. 🧰 Custom Hook trong TypeScript

```tsx
// useDebounce hook
function useDebounce<T>(value: T, delay: number): T {
    const [debouncedValue, setDebouncedValue] = useState<T>(value);

    useEffect(() => {
        const handler = setTimeout(() => {
            setDebouncedValue(value);
        }, delay);

        return () => {
            clearTimeout(handler);
        };
    }, [value, delay]);

    return debouncedValue;
}

// useLocalStorage hook
function useLocalStorage<T>(
    key: string, 
    initialValue: T
): [T, (value: T | ((val: T) => T)) => void] {
    // Get from local storage then parse stored json or return initialValue
    const [storedValue, setStoredValue] = useState<T>(() => {
        try {
            const item = window.localStorage.getItem(key);
            return item ? JSON.parse(item) : initialValue;
        } catch (error) {
            console.log(error);
            return initialValue;
        }
    });

    // Return a wrapped version of useState's setter function that persists the new value to localStorage
    const setValue = (value: T | ((val: T) => T)) => {
        try {
            // Allow value to be a function so we have the same API as useState
            const valueToStore = value instanceof Function ? value(storedValue) : value;
            setStoredValue(valueToStore);
            window.localStorage.setItem(key, JSON.stringify(valueToStore));
        } catch (error) {
            console.log(error);
        }
    };

    return [storedValue, setValue];
}

// useAsync hook for API calls
interface UseAsyncState<T> {
    data: T | null;
    loading: boolean;
    error: Error | null;
}

function useAsync<T>(
    asyncFunction: () => Promise<T>,
    dependencies: React.DependencyList = []
): UseAsyncState<T> {
    const [state, setState] = useState<UseAsyncState<T>>({
        data: null,
        loading: true,
        error: null
    });

    useEffect(() => {
        let isCancelled = false;

        setState({ data: null, loading: true, error: null });

        asyncFunction()
            .then(data => {
                if (!isCancelled) {
                    setState({ data, loading: false, error: null });
                }
            })
            .catch(error => {
                if (!isCancelled) {
                    setState({ data: null, loading: false, error });
                }
            });

        return () => {
            isCancelled = true;
        };
    }, dependencies);

    return state;
}

// Usage examples
function SearchComponent() {
    const [query, setQuery] = useState('');
    const debouncedQuery = useDebounce(query, 500);
    
    const [favorites, setFavorites] = useLocalStorage<string[]>('favorites', []);
    
    const searchResults = useAsync(
        () => fetch(`/api/search?q=${debouncedQuery}`).then(r => r.json()),
        [debouncedQuery]
    );

    return (
        <div>
            <input 
                value={query}
                onChange={e => setQuery(e.target.value)}
                placeholder="Search..."
            />
            
            {searchResults.loading && <div>Searching...</div>}
            {searchResults.error && <div>Error: {searchResults.error.message}</div>}
            {searchResults.data && (
                <ul>
                    {searchResults.data.map((item: any) => (
                        <li key={item.id}>{item.name}</li>
                    ))}
                </ul>
            )}
        </div>
    );
}
```

---

## 32. 🧪 App ToDo list có typing

```tsx
// types/todo.ts
export interface Todo {
    id: string;
    text: string;
    completed: boolean;
    createdAt: Date;
    updatedAt: Date;
}

export type TodoFilter = 'all' | 'active' | 'completed';

export interface TodoState {
    todos: Todo[];
    filter: TodoFilter;
}

// hooks/useTodos.ts
import { useState, useCallback } from 'react';
import { Todo, TodoFilter, TodoState } from '../types/todo';

export function useTodos() {
    const [state, setState] = useState<TodoState>({
        todos: [],
        filter: 'all'
    });

    const addTodo = useCallback((text: string) => {
        const newTodo: Todo = {
            id: crypto.randomUUID(),
            text: text.trim(),
            completed: false,
            createdAt: new Date(),
            updatedAt: new Date()
        };

        setState(prev => ({
            ...prev,
            todos: [...prev.todos, newTodo]
        }));
    }, []);

    const toggleTodo = useCallback((id: string) => {
        setState(prev => ({
            ...prev,
            todos: prev.todos.map(todo =>
                todo.id === id
                    ? { ...todo, completed: !todo.completed, updatedAt: new Date() }
                    : todo
            )
        }));
    }, []);

    const deleteTodo = useCallback((id: string) => {
        setState(prev => ({
            ...prev,
            todos: prev.todos.filter(todo => todo.id !== id)
        }));
    }, []);

    const updateTodo = useCallback((id: string, text: string) => {
        setState(prev => ({
            ...prev,
            todos: prev.todos.map(todo =>
                todo.id === id
                    ? { ...todo, text: text.trim(), updatedAt: new Date() }
                    : todo
            )
        }));
    }, []);

    const setFilter = useCallback((filter: TodoFilter) => {
        setState(prev => ({ ...prev, filter }));
    }, []);

    const clearCompleted = useCallback(() => {
        setState(prev => ({
            ...prev,
            todos: prev.todos.filter(todo => !todo.completed)
        }));
    }, []);

    // Computed values
    const filteredTodos = state.todos.filter(todo => {
        switch (state.filter) {
            case 'active':
                return !todo.completed;
            case 'completed':
                return todo.completed;
            default:
                return true;
        }
    });

    const stats = {
        total: state.todos.length,
        active: state.todos.filter(t => !t.completed).length,
        completed: state.todos.filter(t => t.completed).length
    };

    return {
        todos: filteredTodos,
        filter: state.filter,
        stats,
        actions: {
            addTodo,
            toggleTodo,
            deleteTodo,
            updateTodo,
            setFilter,
            clearCompleted
        }
    };
}

// components/TodoApp.tsx
import React from 'react';
import { TodoInput } from './TodoInput';
import { TodoList } from './TodoList';
import { TodoFilter } from './TodoFilter';
import { useTodos } from '../hooks/useTodos';

export const TodoApp: React.FC = () => {
    const { todos, filter, stats, actions } = useTodos();

    return (
        <div className="todo-app">
            <h1>Todo List</h1>
            
            <TodoInput onAdd={actions.addTodo} />
            
            <TodoFilter
                current={filter}
                onChange={actions.setFilter}
                stats={stats}
            />
            
            <TodoList
                todos={todos}
                onToggle={actions.toggleTodo}
                onDelete={actions.deleteTodo}
                onUpdate={actions.updateTodo}
            />
            
            {stats.completed > 0 && (
                <button onClick={actions.clearCompleted}>
                    Clear Completed ({stats.completed})
                </button>
            )}
        </div>
    );
};

// components/TodoItem.tsx
interface TodoItemProps {
    todo: Todo;
    onToggle: (id: string) => void;
    onDelete: (id: string) => void;
    onUpdate: (id: string, text: string) => void;
}

export const TodoItem: React.FC<TodoItemProps> = ({
    todo,
    onToggle,
    onDelete,
    onUpdate
}) => {
    const [isEditing, setIsEditing] = useState(false);
    const [editText, setEditText] = useState(todo.text);

    const handleUpdate = () => {
        if (editText.trim()) {
            onUpdate(todo.id, editText);
            setIsEditing(false);
        }
    };

    return (
        <li className={`todo-item ${todo.completed ? 'completed' : ''}`}>
            <input
                type="checkbox"
                checked={todo.completed}
                onChange={() => onToggle(todo.id)}
            />
            
            {isEditing ? (
                <input
                    value={editText}
                    onChange={e => setEditText(e.target.value)}
                    onBlur={handleUpdate}
                    onKeyDown={e => {
                        if (e.key === 'Enter') handleUpdate();
                        if (e.key === 'Escape') setIsEditing(false);
                    }}
                    autoFocus
                />
            ) : (
                <span onDoubleClick={() => setIsEditing(true)}>
                    {todo.text}
                </span>
            )}
            
            <button onClick={() => onDelete(todo.id)}>Delete</button>
        </li>
    );
};
```

---

## 33. 🤖 Gọi API AI (OpenAI / Hugging Face)

```tsx
// types/ai.ts
export interface OpenAIMessage {
    role: 'system' | 'user' | 'assistant';
    content: string;
}

export interface OpenAIResponse {
    id: string;
    object: string;
    created: number;
    choices: Array<{
        index: number;
        message: OpenAIMessage;
        finish_reason: string;
    }>;
    usage: {
        prompt_tokens: number;
        completion_tokens: number;
        total_tokens: number;
    };
}

// services/openai.ts
class OpenAIService {
    private apiKey: string;
    private baseUrl = 'https://api.openai.com/v1';

    constructor(apiKey: string) {
        this.apiKey = apiKey;
    }

    async chat(messages: OpenAIMessage[]): Promise<string> {
        const response = await fetch(`${this.baseUrl}/chat/completions`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${this.apiKey}`
            },
            body: JSON.stringify({
                model: 'gpt-3.5-turbo',
                messages,
                max_tokens: 1000,
                temperature: 0.7
            })
        });

        if (!response.ok) {
            throw new Error(`OpenAI API error: ${response.status}`);
        }

        const data = await response.json() as OpenAIResponse;
        return data.choices[0]?.message?.content || '';
    }

    async generateImage(prompt: string): Promise<string> {
        const response = await fetch(`${this.baseUrl}/images/generations`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${this.apiKey}`
            },
            body: JSON.stringify({
                prompt,
                n: 1,
                size: '1024x1024'
            })
        });

        if (!response.ok) {
            throw new Error(`OpenAI API error: ${response.status}`);
        }

        const data = await response.json();
        return data.data[0]?.url || '';
    }
}

// hooks/useOpenAI.ts
export function useOpenAI(apiKey: string) {
    const [service] = useState(() => new OpenAIService(apiKey));
    
    const [state, setState] = useState({
        loading: false,
        error: null as string | null
    });

    const chat = async (messages: OpenAIMessage[]): Promise<string | null> => {
        setState({ loading: true, error: null });
        
        try {
            const result = await service.chat(messages);
            setState({ loading: false, error: null });
            return result;
        } catch (error) {
            const errorMessage = error instanceof Error ? error.message : 'Unknown error';
            setState({ loading: false, error: errorMessage });
            return null;
        }
    };

    const generateImage = async (prompt: string): Promise<string | null> => {
        setState({ loading: true, error: null });
        
        try {
            const result = await service.generateImage(prompt);
            setState({ loading: false, error: null });
            return result;
        } catch (error) {
            const errorMessage = error instanceof Error ? error.message : 'Unknown error';
            setState({ loading: false, error: errorMessage });
            return null;
        }
    };

    return {
        chat,
        generateImage,
        loading: state.loading,
        error: state.error
    };
}

// components/AIChat.tsx
export const AIChat: React.FC = () => {
    const [messages, setMessages] = useState<OpenAIMessage[]>([]);
    const [input, setInput] = useState('');
    const apiKey = process.env.REACT_APP_OPENAI_API_KEY || '';
    
    const { chat, loading, error } = useOpenAI(apiKey);

    const handleSend = async () => {
        if (!input.trim()) return;

        const userMessage: OpenAIMessage = {
            role: 'user',
            content: input
        };

        const newMessages = [...messages, userMessage];
        setMessages(newMessages);
        setInput('');

        const response = await chat(newMessages);
        
        if (response) {
            setMessages(prev => [...prev, {
                role: 'assistant',
                content: response
            }]);
        }
    };

    return (
        <div className="ai-chat">
            <div className="messages">
                {messages.map((message, index) => (
                    <div key={index} className={`message ${message.role}`}>
                        <strong>{message.role}:</strong> {message.content}
                    </div>
                ))}
                {loading && <div className="loading">AI is thinking...</div>}
                {error && <div className="error">Error: {error}</div>}
            </div>
            
            <div className="input-area">
                <input
                    value={input}
                    onChange={e => setInput(e.target.value)}
                    onKeyDown={e => e.key === 'Enter' && handleSend()}
                    placeholder="Type your message..."
                    disabled={loading}
                />
                <button onClick={handleSend} disabled={loading || !input.trim()}>
                    Send
                </button>
            </div>
        </div>
    );
};
```

---

## 34. 📸 Upload ảnh, xử lý AI

```tsx
// hooks/useImageUpload.ts
interface UseImageUploadResult {
    uploadImage: (file: File) => Promise<string | null>;
    uploading: boolean;
    error: string | null;
}

export function useImageUpload(): UseImageUploadResult {
    const [uploading, setUploading] = useState(false);
    const [error, setError] = useState<string | null>(null);

    const uploadImage = async (file: File): Promise<string | null> => {
        setUploading(true);
        setError(null);

        try {
            // Validate file
            if (!file.type.startsWith('image/')) {
                throw new Error('File must be an image');
            }

            if (file.size > 5 * 1024 * 1024) { // 5MB
                throw new Error('File size must be less than 5MB');
            }

            // Convert to base64 for API
            const base64 = await fileToBase64(file);
            
            // Upload to your API or cloud storage
            const formData = new FormData();
            formData.append('image', file);

            const response = await fetch('/api/upload', {
                method: 'POST',
                body: formData
            });

            if (!response.ok) {
                throw new Error(`Upload failed: ${response.status}`);
            }

            const result = await response.json();
            setUploading(false);
            return result.url;

        } catch (err) {
            const errorMessage = err instanceof Error ? err.message : 'Upload failed';
            setError(errorMessage);
            setUploading(false);
            return null;
        }
    };

    return { uploadImage, uploading, error };
}

function fileToBase64(file: File): Promise<string> {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => resolve(reader.result as string);
        reader.onerror = error => reject(error);
    });
}

// components/ImageUpload.tsx
interface ImageUploadProps {
    onImageProcessed: (result: string) => void;
}

export const ImageUpload: React.FC<ImageUploadProps> = ({ onImageProcessed }) => {
    const [selectedImage, setSelectedImage] = useState<string | null>(null);
    const [processing, setProcessing] = useState(false);
    
    const { uploadImage, uploading, error } = useImageUpload();
    const { chat } = useOpenAI(process.env.REACT_APP_OPENAI_API_KEY || '');

    const handleFileSelect = (event: React.ChangeEvent<HTMLInputElement>) => {
        const file = event.target.files?.[0];
        if (file) {
            const imageUrl = URL.createObjectURL(file);
            setSelectedImage(imageUrl);
        }
    };

    const processImage = async () => {
        const fileInput = document.querySelector('input[type="file"]') as HTMLInputElement;
        const file = fileInput?.files?.[0];
        
        if (!file) return;

        setProcessing(true);

        try {
            // Upload image
            const imageUrl = await uploadImage(file);
            
            if (imageUrl) {
                // Analyze with AI (example with vision model)
                const messages: OpenAIMessage[] = [
                    {
                        role: 'user',
                        content: `Analyze this image and describe what you see: ${imageUrl}`
                    }
                ];

                const analysis = await chat(messages);
                
                if (analysis) {
                    onImageProcessed(analysis);
                }
            }
        } catch (err) {
            console.error('Image processing failed:', err);
        } finally {
            setProcessing(false);
        }
    };

    return (
        <div className="image-upload">
            <div className="upload-area">
                <input
                    type="file"
                    accept="image/*"
                    onChange={handleFileSelect}
                    disabled={uploading || processing}
                />
                
                {selectedImage && (
                    <div className="preview">
                        <img src={selectedImage} alt="Preview" style={{ maxWidth: '300px' }} />
                    </div>
                )}
            </div>

            {selectedImage && (
                <button
                    onClick={processImage}
                    disabled={uploading || processing}
                >
                    {processing ? 'Processing...' : 'Analyze Image'}
                </button>
            )}

            {error && <div className="error">Error: {error}</div>}
        </div>
    );
};

// components/ImageAnalyzer.tsx
export const ImageAnalyzer: React.FC = () => {
    const [results, setResults] = useState<string[]>([]);

    const handleImageProcessed = (result: string) => {
        setResults(prev => [...prev, result]);
    };

    return (
        <div className="image-analyzer">
            <h2>AI Image Analyzer</h2>
            
            <ImageUpload onImageProcessed={handleImageProcessed} />
            
            <div className="results">
                <h3>Analysis Results:</h3>
                {results.map((result, index) => (
                    <div key={index} className="result-item">
                        <p>{result}</p>
                    </div>
                ))}
            </div>
        </div>
    );
};
```

---

## 35. 💬 Viết chatbot nhỏ

```tsx
// types/chat.ts
export interface ChatMessage {
    id: string;
    role: 'user' | 'bot';
    content: string;
    timestamp: Date;
    type?: 'text' | 'image' | 'file';
}

export interface ChatSession {
    id: string;
    title: string;
    messages: ChatMessage[];
    createdAt: Date;
    updatedAt: Date;
}

// hooks/useChat.ts
export function useChat() {
    const [sessions, setSessions] = useState<ChatSession[]>([]);
    const [currentSessionId, setCurrentSessionId] = useState<string | null>(null);
    const [isTyping, setIsTyping] = useState(false);

    const { chat } = useOpenAI(process.env.REACT_APP_OPENAI_API_KEY || '');

    const currentSession = sessions.find(s => s.id === currentSessionId);

    const createSession = useCallback((title: string = 'New Chat'): string => {
        const newSession: ChatSession = {
            id: crypto.randomUUID(),
            title,
            messages: [],
            createdAt: new Date(),
            updatedAt: new Date()
        };

        setSessions(prev => [newSession, ...prev]);
        setCurrentSessionId(newSession.id);
        return newSession.id;
    }, []);

    const deleteSession = useCallback((sessionId: string) => {
        setSessions(prev => prev.filter(s => s.id !== sessionId));
        if (currentSessionId === sessionId) {
            setCurrentSessionId(null);
        }
    }, [currentSessionId]);

    const sendMessage = useCallback(async (content: string) => {
        if (!currentSessionId) {
            createSession();
            return;
        }

        const userMessage: ChatMessage = {
            id: crypto.randomUUID(),
            role: 'user',
            content,
            timestamp: new Date(),
            type: 'text'
        };

        // Add user message
        setSessions(prev => prev.map(session =>
            session.id === currentSessionId
                ? {
                    ...session,
                    messages: [...session.messages, userMessage],
                    updatedAt: new Date()
                }
                : session
        ));

        setIsTyping(true);

        try {
            // Prepare context for AI
            const messages: OpenAIMessage[] = currentSession?.messages.map(msg => ({
                role: msg.role === 'user' ? 'user' : 'assistant',
                content: msg.content
            })) || [];

            messages.push({ role: 'user', content });

            const response = await chat(messages);

            if (response) {
                const botMessage: ChatMessage = {
                    id: crypto.randomUUID(),
                    role: 'bot',
                    content: response,
                    timestamp: new Date(),
                    type: 'text'
                };

                setSessions(prev => prev.map(session =>
                    session.id === currentSessionId
                        ? {
                            ...session,
                            messages: [...session.messages, botMessage],
                            updatedAt: new Date()
                        }
                        : session
                ));
            }
        } catch (error) {
            console.error('Chat error:', error);
            
            const errorMessage: ChatMessage = {
                id: crypto.randomUUID(),
                role: 'bot',
                content: 'Sorry, I encountered an error. Please try again.',
                timestamp: new Date(),
                type: 'text'
            };

            setSessions(prev => prev.map(session =>
                session.id === currentSessionId
                    ? {
                        ...session,
                        messages: [...session.messages, errorMessage],
                        updatedAt: new Date()
                    }
                    : session
            ));
        } finally {
            setIsTyping(false);
        }
    }, [currentSessionId, currentSession, chat, createSession]);

    return {
        sessions,
        currentSession,
        currentSessionId,
        isTyping,
        createSession,
        deleteSession,
        sendMessage,
        setCurrentSessionId
    };
}

// components/Chatbot.tsx
export const Chatbot: React.FC = () => {
    const {
        sessions,
        currentSession,
        currentSessionId,
        isTyping,
        createSession,
        deleteSession,
        sendMessage,
        setCurrentSessionId
    } = useChat();

    const [input, setInput] = useState('');
    const messagesEndRef = useRef<HTMLDivElement>(null);

    const scrollToBottom = () => {
        messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
    };

    useEffect(() => {
        scrollToBottom();
    }, [currentSession?.messages]);

    const handleSend = async () => {
        if (!input.trim()) return;
        
        const message = input.trim();
        setInput('');
        await sendMessage(message);
    };

    const handleKeyPress = (e: React.KeyboardEvent) => {
        if (e.key === 'Enter' && !e.shiftKey) {
            e.preventDefault();
            handleSend();
        }
    };

    if (!currentSessionId) {
        return (
            <div className="chatbot-welcome">
                <h2>Welcome to AI Chatbot</h2>
                <button onClick={() => createSession()}>
                    Start New Conversation
                </button>
            </div>
        );
    }

    return (
        <div className="chatbot">
            <div className="sidebar">
                <button onClick={() => createSession()}>
                    + New Chat
                </button>
                
                <div className="sessions">
                    {sessions.map(session => (
                        <div
                            key={session.id}
                            className={`session-item ${session.id === currentSessionId ? 'active' : ''}`}
                            onClick={() => setCurrentSessionId(session.id)}
                        >
                            <span className="session-title">{session.title}</span>
                            <button
                                onClick={(e) => {
                                    e.stopPropagation();
                                    deleteSession(session.id);
                                }}
                                className="delete-btn"
                            >
                                ×
                            </button>
                        </div>
                    ))}
                </div>
            </div>

            <div className="chat-area">
                <div className="messages">
                    {currentSession?.messages.map(message => (
                        <div
                            key={message.id}
                            className={`message ${message.role}`}
                        >
                            <div className="message-content">
                                {message.content}
                            </div>
                            <div className="message-time">
                                {message.timestamp.toLocaleTimeString()}
                            </div>
                        </div>
                    ))}
                    
                    {isTyping && (
                        <div className="message bot typing">
                            <div className="typing-indicator">
                                <span></span>
                                <span></span>
                                <span></span>
                            </div>
                        </div>
                    )}
                    
                    <div ref={messagesEndRef} />
                </div>

                <div className="input-area">
                    <textarea
                        value={input}
                        onChange={e => setInput(e.target.value)}
                        onKeyDown={handleKeyPress}
                        placeholder="Type your message..."
                        disabled={isTyping}
                        rows={1}
                    />
                    <button
                        onClick={handleSend}
                        disabled={isTyping || !input.trim()}
                    >
                        Send
                    </button>
                </div>
            </div>
        </div>
    );
};
```

---

## 36. 🧪 Refactor component JS sang TS

### JavaScript component cũ
```jsx
// UserProfile.jsx - Legacy JavaScript
import React, { useState, useEffect } from 'react';

const UserProfile = ({ userId, onUpdate }) => {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);
    const [editing, setEditing] = useState(false);
    const [formData, setFormData] = useState({});

    useEffect(() => {
        fetchUser();
    }, [userId]);

    const fetchUser = async () => {
        try {
            const response = await fetch(`/api/users/${userId}`);
            const userData = await response.json();
            setUser(userData);
            setFormData(userData);
        } catch (error) {
            console.error('Failed to fetch user:', error);
        } finally {
            setLoading(false);
        }
    };

    const handleSave = async () => {
        try {
            const response = await fetch(`/api/users/${userId}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(formData)
            });
            const updatedUser = await response.json();
            setUser(updatedUser);
            setEditing(false);
            onUpdate(updatedUser);
        } catch (error) {
            console.error('Failed to update user:', error);
        }
    };

    if (loading) return <div>Loading...</div>;
    if (!user) return <div>User not found</div>;

    return (
        <div className="user-profile">
            {editing ? (
                <EditForm
                    data={formData}
                    onChange={setFormData}
                    onSave={handleSave}
                    onCancel={() => setEditing(false)}
                />
            ) : (
                <DisplayMode
                    user={user}
                    onEdit={() => setEditing(true)}
                />
            )}
        </div>
    );
};
```

### TypeScript version được refactor
```tsx
// UserProfile.tsx - TypeScript version
import React, { useState, useEffect, useCallback } from 'react';

// Define types
interface User {
    id: string;
    name: string;
    email: string;
    avatar?: string;
    role: 'admin' | 'user' | 'guest';
    lastLoginAt: Date | null;
    createdAt: Date;
    updatedAt: Date;
}

interface UserFormData {
    name: string;
    email: string;
    role: User['role'];
}

interface UserProfileProps {
    userId: string;
    onUpdate: (user: User) => void;
    readonly?: boolean;
}

interface EditFormProps {
    data: UserFormData;
    onChange: (data: UserFormData) => void;
    onSave: () => Promise<void>;
    onCancel: () => void;
}

interface DisplayModeProps {
    user: User;
    onEdit: () => void;
    readonly?: boolean;
}

// API service with types
class UserService {
    static async fetchUser(userId: string): Promise<User> {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) {
            throw new Error(`Failed to fetch user: ${response.status}`);
        }
        const userData = await response.json();
        return {
            ...userData,
            lastLoginAt: userData.lastLoginAt ? new Date(userData.lastLoginAt) : null,
            createdAt: new Date(userData.createdAt),
            updatedAt: new Date(userData.updatedAt)
        };
    }

    static async updateUser(userId: string, data: UserFormData): Promise<User> {
        const response = await fetch(`/api/users/${userId}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data)
        });
        if (!response.ok) {
            throw new Error(`Failed to update user: ${response.status}`);
        }
        const userData = await response.json();
        return {
            ...userData,
            lastLoginAt: userData.lastLoginAt ? new Date(userData.lastLoginAt) : null,
            createdAt: new Date(userData.createdAt),
            updatedAt: new Date(userData.updatedAt)
        };
    }
}

// Main component
export const UserProfile: React.FC<UserProfileProps> = ({
    userId,
    onUpdate,
    readonly = false
}) => {
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState<boolean>(true);
    const [error, setError] = useState<string | null>(null);
    const [editing, setEditing] = useState<boolean>(false);
    const [formData, setFormData] = useState<UserFormData>({
        name: '',
        email: '',
        role: 'user'
    });

    const fetchUser = useCallback(async () => {
        try {
            setLoading(true);
            setError(null);
            const userData = await UserService.fetchUser(userId);
            setUser(userData);
            setFormData({
                name: userData.name,
                email: userData.email,
                role: userData.role
            });
        } catch (err) {
            const errorMessage = err instanceof Error ? err.message : 'Failed to fetch user';
            setError(errorMessage);
        } finally {
            setLoading(false);
        }
    }, [userId]);

    useEffect(() => {
        fetchUser();
    }, [fetchUser]);

    const handleSave = async (): Promise<void> => {
        try {
            const updatedUser = await UserService.updateUser(userId, formData);
            setUser(updatedUser);
            setEditing(false);
            onUpdate(updatedUser);
        } catch (err) {
            const errorMessage = err instanceof Error ? err.message : 'Failed to update user';
            setError(errorMessage);
        }
    };

    const handleFormChange = useCallback((data: UserFormData) => {
        setFormData(data);
    }, []);

    const handleEditStart = useCallback(() => {
        setEditing(true);
        setError(null);
    }, []);

    const handleEditCancel = useCallback(() => {
        setEditing(false);
        setError(null);
        // Reset form data
        if (user) {
            setFormData({
                name: user.name,
                email: user.email,
                role: user.role
            });
        }
    }, [user]);

    if (loading) return <div className="loading">Loading...</div>;
    if (error) return <div className="error">Error: {error}</div>;
    if (!user) return <div className="not-found">User not found</div>;

    return (
        <div className="user-profile">
            {editing ? (
                <EditForm
                    data={formData}
                    onChange={handleFormChange}
                    onSave={handleSave}
                    onCancel={handleEditCancel}
                />
            ) : (
                <DisplayMode
                    user={user}
                    onEdit={handleEditStart}
                    readonly={readonly}
                />
            )}
        </div>
    );
};

// Edit form component
const EditForm: React.FC<EditFormProps> = ({ data, onChange, onSave, onCancel }) => {
    const [saving, setSaving] = useState<boolean>(false);

    const handleSubmit = async (e: React.FormEvent) => {
        e.preventDefault();
        setSaving(true);
        try {
            await onSave();
        } finally {
            setSaving(false);
        }
    };

    const handleInputChange = (field: keyof UserFormData) => 
        (e: React.ChangeEvent<HTMLInputElement | HTMLSelectElement>) => {
            onChange({
                ...data,
                [field]: e.target.value
            });
        };

    return (
        <form onSubmit={handleSubmit} className="edit-form">
            <div className="form-group">
                <label htmlFor="name">Name:</label>
                <input
                    id="name"
                    type="text"
                    value={data.name}
                    onChange={handleInputChange('name')}
                    required
                />
            </div>

            <div className="form-group">
                <label htmlFor="email">Email:</label>
                <input
                    id="email"
                    type="email"
                    value={data.email}
                    onChange={handleInputChange('email')}
                    required
                />
            </div>

            <div className="form-group">
                <label htmlFor="role">Role:</label>
                <select
                    id="role"
                    value={data.role}
                    onChange={handleInputChange('role')}
                >
                    <option value="user">User</option>
                    <option value="admin">Admin</option>
                    <option value="guest">Guest</option>
                </select>
            </div>

            <div className="form-actions">
                <button type="submit" disabled={saving}>
                    {saving ? 'Saving...' : 'Save'}
                </button>
                <button type="button" onClick={onCancel} disabled={saving}>
                    Cancel
                </button>
            </div>
        </form>
    );
};

// Display mode component
const DisplayMode: React.FC<DisplayModeProps> = ({ user, onEdit, readonly }) => {
    return (
        <div className="display-mode">
            <div className="user-info">
                <h2>{user.name}</h2>
                <p>Email: {user.email}</p>
                <p>Role: {user.role}</p>
                <p>
                    Last Login: {user.lastLoginAt 
                        ? user.lastLoginAt.toLocaleDateString() 
                        : 'Never'
                    }
                </p>
                <p>Member since: {user.createdAt.toLocaleDateString()}</p>
            </div>
            
            {!readonly && (
                <div className="actions">
                    <button onClick={onEdit}>Edit Profile</button>
                </div>
            )}
        </div>
    );
};

export default UserProfile;
```

**💡 Improvements with TypeScript**:
- Type safety cho props và state
- Error handling tốt hơn
- API response typing
- Better IntelliSense và refactoring
- Runtime error giảm đáng kể

---

## 🎯 Bài tập tuần 4

### Bài 1: Todo App với Context
Mở rộng Todo app dùng React Context thay vì prop drilling.

### Bài 2: Real-time Chat
Thêm WebSocket support vào chatbot để có real-time messaging.

### Bài 3: Image Gallery với AI
Tạo gallery với AI image analysis và search functionality.

### Bài 4: Dashboard Component
Viết dashboard component với charts, metrics, và real-time data.

---

## 🚀 Tiếp theo

👉 **[BONUS – Chủ đề mở rộng](./bonus.md)**

Phần bonus sẽ cover testing, linting, error handling và các chủ đề nâng cao khác!

---

*🎉 Chúc mừng bạn đã hoàn thành tuần 4! Bạn đã có foundation vững chắc về TypeScript!*
