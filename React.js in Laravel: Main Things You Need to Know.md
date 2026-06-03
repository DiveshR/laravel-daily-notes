React.js in Laravel: Main Things You Need to Know

React.js is the most popular front-end JS framework. There are a few ways to use React in Laravel projects, and in this tutorial, we will cover so-called RILT stack with examples, also touching the alternative of using Next.js with Laravel API.

So, if you're a Laravel back-ender and want to get familiar with React, let's begin.

---

## **Setting Up Your First RILT Stack Project**

The RILT stack (React, Inertia, Laravel, Tailwind) comes pre-configured in Laravel 12's React starter kit with React 19, TypeScript, Tailwind 4, and shadcn/ui components. Here's how to get started:

`laravel new my-react-app # Choose React when prompted for starter kit # Navigate to project and install dependenciescd my-react-appnpm install && npm run buildcomposer run dev`

![](https://laraveldaily.com/uploads/2025/07/laravel-new-project-react-kit.png)

Your application will be available at **`http://localhost:8000`** with authentication pages already working.

---

## **Understanding Your First React Component**

Let's examine a simple React component to understand the fundamentals. The majority of frontend code is located in the **`resources/js`** directory.

**resources/js/pages/welcome.tsx**:

`import { Head, usePage } from '@inertiajs/react';import { type SharedData } from '@/types'; export default function Welcome({    laravelVersion,    phpVersion,    message}: {    laravelVersion: string;    phpVersion: string;    message: string;}) {    const { auth } = usePage<SharedData>().props;     return (        <>            <Head title="Welcome" />            <div className="bg-gray-50 text-black/50 dark:bg-black dark:text-white/50">                <div className="relative min-h-screen flex flex-col items-center justify-center">                    <div className="relative w-full max-w-2xl px-6 lg:max-w-7xl">                        <header className="grid grid-cols-2 items-center gap-2 py-10 lg:grid-cols-3">                            <h1 className="text-3xl font-bold">                                Hello from React!                            </h1>                            {auth.user ? (                                <span>Welcome back, {auth.user.name}!</span>                            ) : (                                <span>Please log in to continue</span>                            )}                        </header>                         <main className="mt-6">                            <p>{message}</p>                            <p>Laravel version: {laravelVersion}</p>                            <p>PHP version: {phpVersion}</p>                        </main>                    </div>                </div>            </div>        </>    );}`

### **Component Breakdown**

**Import Statements**:

- **`Head`** from Inertia handles page metadata like title and meta tags
- **`PageProps`** provides TypeScript typing for component props

**Component Function**:

- Receives props including authentication data and version information
- Uses destructuring to extract specific props with TypeScript typing

**JSX Return**:

- Wrapped in React Fragment (**`<>...</>`**) to return multiple elements
- Uses Tailwind CSS classes for styling
- Conditional rendering shows different content for authenticated users

---

## **How Inertia.js Bridges Laravel and React**

Inertia.js allows you to build modern single-page React applications using classic server-side routing and Controllers. Here's how data flows from Laravel to React:

**app/Http/Controllers/WelcomeController.php**:

`<?php namespace App\Http\Controllers; use Illuminate\Foundation\Application;use Inertia\Inertia;use Inertia\Response; class WelcomeController extends Controller{    public function index(): Response    {        return Inertia::render('welcome', [            'laravelVersion' => Application::VERSION,            'phpVersion' => PHP_VERSION,            'message' => 'Hello from Laravel backend!'        ]);    }}`

**routes/web.php**

`<?php use App\Http\Controllers\WelcomeController;use Illuminate\Support\Facades\Route; Route::get('/', [WelcomeController::class, 'index'])->name('welcome');`

When a user visits your route, Laravel:

1. Processes the request through the Controller
2. Gathers data and passes it to **`Inertia::render()`**
3. Inertia sends this data as props to your React component
4. React renders the component with the provided data

![](https://laraveldaily.com/uploads/2025/07/laravel-vilt-homepage.png)

---

## **Building Interactive Components**

Let's create a practical component that handles user input and communicates with Laravel:

**resources/js/pages/tasks/create.tsx**:

`import { Head } from '@inertiajs/react';import { useForm } from '@inertiajs/react';import { FormEventHandler } from 'react'; export default function CreateTask() {    const { data, setData, post, processing, errors } = useForm({        title: '',        description: '',        due_date: ''    });     const submit: FormEventHandler = (e) => {        e.preventDefault();        post(route('tasks.store'));    };     return (        <>            <Head title="Create Task" />            <div className="max-w-2xl mx-auto p-6">                <h1 className="text-2xl font-bold mb-6">Create New Task</h1>                 <form onSubmit={submit} className="space-y-4">                    <div>                        <label className="block text-sm font-medium mb-1">                            Task Title                        </label>                        <input                            type="text"                            value={data.title}                            onChange={(e) => setData('title', e.target.value)}                            className="w-full px-3 py-2 border border-gray-300 rounded-md"                            required                        />                        {errors.title && (                            <p className="text-red-500 text-sm mt-1">{errors.title}</p>                        )}                    </div>                     <div>                        <label className="block text-sm font-medium mb-1">                            Description                        </label>                        <textarea                            value={data.description}                            onChange={(e) => setData('description', e.target.value)}                            className="w-full px-3 py-2 border border-gray-300 rounded-md"                            rows={4}                        />                        {errors.description && (                            <p className="text-red-500 text-sm mt-1">{errors.description}</p>                        )}                    </div>                     <div>                        <label className="block text-sm font-medium mb-1">                            Due Date                        </label>                        <input                            type="date"                            value={data.due_date}                            onChange={(e) => setData('due_date', e.target.value)}                            className="w-full px-3 py-2 border border-gray-300 rounded-md"                        />                    </div>                     <button                        type="submit"                        disabled={processing}                        className="bg-blue-500 text-white px-4 py-2 rounded-md hover:bg-blue-600 disabled:opacity-50"                    >                        {processing ? 'Creating...' : 'Create Task'}                    </button>                </form>            </div>        </>    );}`

In the Controller you call Inertia:

`use Inertia\Inertia;use Inertia\Response; class TaskController extends Controller{    public function create(): Response    {        return Inertia::render('tasks/create');    }}`

### **Key React Concepts Explained**

**useForm Hook**: Inertia's useForm hook provides automatic CSRF protection and validation handling. It manages form state and submission.

**State Management**: Each form field uses controlled components where React manages the input values through state.

**Error Handling**: Laravel validation errors are automatically passed to the React component and displayed conditionally.

**Event Handling**: The **`onChange`** handlers update component state, while **`onSubmit`** sends data to Laravel.

![](https://laraveldaily.com/uploads/2025/07/laravel-vilt-interactive-form.png)

---

## **Creating Reusable Components**

Good React development involves creating **reusable** components. Laravel's React starter kit organizes reusable components in the **`resources/js/components`** directory:

**resources/js/components/task-card.tsx**:

`interface Task {    id: number;    title: string;    description: string;    due_date: string;    completed: boolean;} interface TaskCardProps {    task: Task;    onToggleComplete: (id: number) => void;    onDelete: (id: number) => void;} export default function TaskCard({ task, onToggleComplete, onDelete }: TaskCardProps) {    return (        <div className="bg-white p-4 rounded-lg shadow-md border">            <div className="flex items-start justify-between">                <div className="flex-1">                    <h3 className={`font-semibold ${task.completed ? 'line-through text-gray-500' : ''}`}>                        {task.title}                    </h3>                    <p className="text-gray-600 mt-1">{task.description}</p>                    <p className="text-sm text-gray-500 mt-2">Due: {task.due_date.split('T')[0]}</p>                </div>                 <div className="flex gap-2 ml-4">                    <button                        onClick={() => onToggleComplete(task.id)}                        className={`px-3 py-1 rounded text-sm ${                            task.completed                                ? 'bg-gray-200 text-gray-700'                                : 'bg-green-500 text-white'                        }`}                    >                        {task.completed ? 'Undo' : 'Complete'}                    </button>                     <button                        onClick={() => onDelete(task.id)}                        className="px-3 py-1 bg-red-500 text-white rounded text-sm hover:bg-red-600"                    >                        Delete                    </button>                </div>            </div>        </div>    );}`

**Using the Component in a Page:**

**resources/js/Pages/tasks/index.tsx**

`import { Head } from '@inertiajs/react';import { router } from '@inertiajs/react';import TaskCard from '@/components/task-card';  interface Task {    id: number;    title: string;    description: string;    due_date: string;    completed: boolean;} export default function TaskIndex({ tasks }: { tasks: Task[] }) {    const handleToggleComplete = (id: number) => {        router.patch(`/tasks/${id}/toggle`, {}, {            preserveScroll: true        });    };     const handleDelete = (id: number) => {        if (confirm('Are you sure you want to delete this task?')) {            router.delete(`/tasks/${id}`, {                preserveScroll: true            });        }    };     return (        <>            <Head title="My Tasks" />            <div className="max-w-4xl mx-auto p-6">                <div className="flex justify-between items-center mb-6">                    <h1 className="text-3xl font-bold">My Tasks</h1>                    <a                        href={route('tasks.create')}                        className="bg-blue-500 text-white px-4 py-2 rounded-md hover:bg-blue-600"                    >                        Add New Task                    </a>                </div>                 <div className="space-y-4">                    {tasks.length === 0 ? (                        <p className="text-gray-500 text-center py-8">                            No tasks yet. Create your first task!                        </p>                    ) : (                        tasks.map(task => (                            <TaskCard                                 key={task.id}                                task={task}                                onToggleComplete={handleToggleComplete}                                onDelete={handleDelete}                            />                         ))                    )}                </div>            </div>        </>    );}`

![](https://laraveldaily.com/uploads/2025/07/laravel-vilt-reusable-component.png)

---

## **Working with TypeScript**

Laravel 12's React starter kit comes with TypeScript by default, but using types is optional. Here's how to define and use types effectively:

**resources/js/types/index.d.ts**

`export interface User {    id: number;    name: string;    email: string;    email_verified_at: string;} export interface Task {    id: number;    title: string;    description: string;    due_date: string;    completed: boolean;    user_id: number;    created_at: string;    updated_at: string;} export type PageProps<T extends Record<string, unknown> = Record<string, unknown>> = T & {    auth: {        user: User;    };    flash: {        message?: string;        error?: string;    };};`

We have a separate tutorial: **TypeScript in Laravel 12 Starter Kits: Main Things To Know**

---

## **Navigation and Layouts**

Create a consistent layout for your application:

**resources/js/layouts/AppLayout.tsx**

`import { PropsWithChildren } from 'react';import { Link } from '@inertiajs/react'; export default function AppLayout({ children }: PropsWithChildren) {    return (        <div className="min-h-screen bg-gray-100">            <nav className="bg-white shadow">                <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">                    <div className="flex justify-between h-16">                        <div className="flex items-center space-x-4">                            <Link href="/" className="text-xl font-bold">                                TaskApp                            </Link>                            <Link                                href={route('tasks.index')}                                className="text-gray-700 hover:text-gray-900"                            >                                Tasks                            </Link>                        </div>                         <div className="flex items-center space-x-4">                            <Link                                href={route('profile.edit')}                                className="text-gray-700 hover:text-gray-900"                            >                                Profile                            </Link>                            <Link                                href={route('logout')}                                method="post"                                className="text-red-600 hover:text-red-800"                            >                                Logout                            </Link>                        </div>                    </div>                </div>            </nav>             <main className="py-6">                {children}            </main>        </div>    );}`

An essential part here is to use **`Link`** from the **`@inertiajs/react`** for the links instead of the **`a`** anchor tag so that your application would have that SPA feeling.

**Using the Layout:**

`import AppLayout from '@/layouts/AppLayout'; export default function TaskIndex({ tasks }) {    return (        <AppLayout>            <div className="max-w-4xl mx-auto">                {/* Your page content */}            </div>        </AppLayout>    );}`

![](https://laraveldaily.com/uploads/2025/07/laravel-vilt-applayout.png)

---

## **Essential React Functionality for Laravel Developers**

**Conditional Rendering:**

`{user.role === 'admin' && (    <button>Admin Only Button</button>)} {tasks.length > 0 ? (    <TaskList tasks={tasks} />) : (    <EmptyState />)}`

**Lists and Keys:**

`{tasks.map(task => (    <TaskCard key={task.id} task={task} />))}`

**Event Handling:**

`const handleClick = (id: number) => {    router.visit(`/tasks/${id}`);}; <button onClick={() => handleClick(task.id)}>    View Task</button>`

**Flash Messages:**

`import { PageProps } from '@/types'; export default function MyPage({ flash }: PageProps) {    return (        <>            {flash.message && (                <div className="bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded mb-4">                    {flash.message}                </div>            )}            {/* Rest of your component */}        </>    );}`

**Route Helpers:**

The starter kit has a **ziggy** package installed, which provides helper functions for working with routes.

`import { route } from 'ziggy-js'; // Navigate to named routes<Link href={route('tasks.show', task.id)}>View Task</Link> // Form submission to routespost(route('tasks.store'), data);`

---

## **Alternative: Laravel API + Next.js Frontend**

While the RILT stack (React + Inertia + Laravel + Tailwind) provides a seamless full-stack solution, many developers prefer separating the backend and frontend completely. This approach uses Laravel purely as an API and Next.js as a standalone frontend application.

### **Key Differences from Laravel + Inertia.js**

**Separate Applications:**

- Laravel runs on one port (e.g., **`http://localhost:8000`**)
- Next.js runs on another port (e.g., **`http://localhost:3000`**)
- Complete separation of concerns between backend and frontend

**API Communication:**

`// Laravel Controller (API Resource)namespace App\Http\Controllers\Api; use App\Http\Controllers\Controller;use App\Models\Task;use Illuminate\Http\Request; class TaskController extends Controller{    public function index(Request $request)    {        $tasks = Task::where('user_id', $request->user()->id)->get();         return response()->json($tasks);    }     public function store(Request $request)    {        $validated = $request->validate([            'title' => 'required|string|max:255',            'description' => 'nullable|string',            'due_date' => 'nullable|date'        ]);         $task = Task::create([            ...$validated,            'user_id' => $request->user()->id        ]);         return response()->json($task, 201);    }}`

**Next.js Frontend Data Fetching:**

`// pages/tasks/index.tsx// pages/tasks/index.tsximport { useState, useEffect } from 'react';import { useRouter } from 'next/router'; interface Task {    id: number;    title: string;    description: string;    due_date: string;    completed: boolean;} export default function TasksPage() {    const [tasks, setTasks] = useState<Task[]>([]);    const [loading, setLoading] = useState(true);    const router = useRouter();     useEffect(() => {        fetchTasks();    }, []);     const fetchTasks = async () => {        try {            const response = await fetch('/api/tasks', {                headers: {                    'Authorization': `Bearer ${localStorage.getItem('token')}`,                    'Content-Type': 'application/json'                }            });             if (!response.ok) throw new Error('Failed to fetch tasks');             const tasks = await response.json();            setTasks(tasks);        } catch (error) {            console.error('Error fetching tasks:', error);        } finally {            setLoading(false);        }    };     const createTask = async (taskData: Omit<Task, 'id' | 'completed'>) => {        try {            const response = await fetch('/api/tasks', {                method: 'POST',                headers: {                    'Authorization': `Bearer ${localStorage.getItem('token')}`,                    'Content-Type': 'application/json'                },                body: JSON.stringify(taskData)            });             if (!response.ok) throw new Error('Failed to create task');             const newTask = await response.json();            setTasks([...tasks, newTask]);        } catch (error) {            console.error('Error creating task:', error);        }    };     if (loading) return <div>Loading...</div>;     return (        <div className="max-w-4xl mx-auto p-6">            <h1 className="text-3xl font-bold mb-6">My Tasks</h1>             <div className="space-y-4">                {tasks.map(task => (                    <div key={task.id} className="bg-white p-4 rounded-lg shadow-md">                        <h3 className="font-semibold">{task.title}</h3>                        <p className="text-gray-600">{task.description}</p>                        <p className="text-sm text-gray-500">Due: {task.due_date}</p>                    </div>                ))}            </div>        </div>    );}`

### **Authentication with Laravel Sanctum**

**Laravel API Routes:**

`// routes/api.php<?php use App\Http\Controllers\Api\AuthController;use App\Http\Controllers\Api\TaskController;use Illuminate\Support\Facades\Route; Route::post('/login', [AuthController::class, 'login']);Route::post('/register', [AuthController::class, 'register']); Route::middleware('auth:sanctum')->group(function () {    Route::get('/user', [AuthController::class, 'user']);    Route::post('/logout', [AuthController::class, 'logout']);    Route::apiResource('tasks', TaskController::class);});`

**Next.js Authentication Hook:**

`// hooks/useAuth.tsimport { useState, useEffect } from 'react';import { useRouter } from 'next/router'; export interface User {    id: number;    name: string;    email: string;} export default function useAuth() {    const [user, setUser] = useState<User | null>(null);    const [loading, setLoading] = useState(true);    const router = useRouter();     useEffect(() => {        checkAuth();    }, []);     const checkAuth = async () => {        const token = localStorage.getItem('token');        if (!token) {            setLoading(false);            return;        }         try {            const response = await fetch('/api/user', {                headers: {                    'Authorization': `Bearer ${token}`,                    'Content-Type': 'application/json'                }            });             if (response.ok) {                const userData = await response.json();                setUser(userData.data);            } else {                localStorage.removeItem('token');            }        } catch (error) {            console.error('Auth check failed:', error);            localStorage.removeItem('token');        } finally {            setLoading(false);        }    };     const login = async (email: string, password: string) => {        try {            const response = await fetch('/api/login', {                method: 'POST',                headers: { 'Content-Type': 'application/json' },                body: JSON.stringify({ email, password })            });             if (!response.ok) throw new Error('Login failed');             const data = await response.json();            localStorage.setItem('token', data.token);            setUser(data.user);            router.push('/dashboard');        } catch (error) {            console.error('Login error:', error);            throw error;        }    };     const logout = () => {        localStorage.removeItem('token');        setUser(null);        router.push('/login');    };     return { user, login, logout, loading };}`

### **When to Choose Laravel API + Next.js**

**Advantages:**

- Complete separation of concerns
- Frontend can be deployed separately (Vercel, Netlify)
- Better for mobile app development (shared API)
- Better for large teams with specialized frontend/backend developers

**Disadvantages:**

- More complex setup and deployment
- Additional authentication complexity
- CORS configuration needed
- No automatic CSRF protection
- More API endpoints to maintain
