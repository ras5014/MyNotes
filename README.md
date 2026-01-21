# i18n Todo App – Complete Step-by-Step Guide

This README walks you through setting up internationalization (i18n) in a basic React Todo application using i18next, including packages to install, folder structure, configuration, and example components.

---

## Overview

- Adds multi-language support (English, French, Spanish)
- Uses `i18next` and `react-i18next`
- Loads translations from `public/locales`
- Organizes translations by language and namespace (e.g., `common`, `todo`, `header`)

---

## Prerequisites

- Node.js and npm installed
- A React app (Create React App or similar)

---

## Install Packages

```bash
npm install i18next react-i18next i18next-http-backend i18next-browser-languagedetector
```

- `i18next`: Core internationalization engine
- `react-i18next`: React bindings (hooks/components)
- `i18next-http-backend`: Loads translation files via HTTP (from `public/locales`)
- `i18next-browser-languagedetector`: Detects browser language (with localStorage support)

---

## Project Structure

```
project-root/
├── public/
│   └── locales/
│       ├── en/
│       │   ├── common.json
│       │   ├── todo.json
│       │   └── header.json
│       ├── fr/
│       │   ├── common.json
│       │   ├── todo.json
│       │   └── header.json
│       └── es/
│           ├── common.json
│           ├── todo.json
│           └── header.json
├── src/
│   ├── i18n/
│   │   └── config.ts
│   ├── components/
│   │   ├── Header.tsx
│   │   ├── LanguageSwitcher.tsx
│   │   ├── TodoForm.tsx
│   │   ├── TodoItem.tsx
│   │   └── TodoList.tsx
│   ├── types/
│   │   └── todo.ts
│   ├── App.tsx
│   └── index.tsx
└── package.json
```

---

## i18n Configuration (`src/i18n/config.ts`)

```ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import HttpBackend from 'i18next-http-backend';
import LanguageDetector from 'i18next-browser-languagedetector';

i18n
  .use(HttpBackend)
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    fallbackLng: 'en',
    supportedLngs: ['en', 'fr', 'es'],
    ns: ['common', 'todo', 'header'],
    defaultNS: 'common',
    nsSeparator: ':',
    interpolation: { escapeValue: false },
    backend: { loadPath: '/locales/{{lng}}/{{ns}}.json' },
    detection: { order: ['querystring', 'localStorage', 'navigator'], caches: ['localStorage'] },
    react: { useSuspense: false },
    debug: false,
  });

export default i18n;
```

- `fallbackLng`: Language used if a translation is missing
- `supportedLngs`: Whitelist of supported languages
- `ns`: Namespaces for translation grouping
- `backend.loadPath`: File path pattern to translations
- `detection`: How to detect and cache language
- `react.useSuspense`: Avoid suspense for smoother UX

---

## Bootstrapping (`src/index.tsx`)

```ts
import React from 'react';
import ReactDOM from 'react-dom/client';
import './i18n/config'; // IMPORTANT: initialize i18n before rendering
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root') as HTMLElement);
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

---

## Translation Files (Examples)

### `public/locales/en/common.json`
```json
{
  "app": {
    "title": "My Todo App",
    "description": "Organize your daily tasks"
  },
  "buttons": {
    "add": "Add",
    "delete": "Delete",
    "edit": "Edit",
    "save": "Save",
    "cancel": "Cancel",
    "clear": "Clear All",
    "submit": "Submit"
  },
  "messages": {
    "welcome": "Welcome, {{name}}!",
    "noData": "No data available",
    "loading": "Loading...",
    "success": "Operation successful!",
    "error": "An error occurred",
    "confirmDelete": "Are you sure?"
  },
  "language": {
    "label": "Select Language",
    "english": "English",
    "french": "Français",
    "spanish": "Español"
  }
}
```

### `public/locales/en/todo.json`
```json
{
  "form": {
    "title": "Add New Todo",
    "label": "Task Title",
    "placeholder": "What needs to be done?",
    "priority": "Priority",
    "priorityHigh": "High",
    "priorityMedium": "Medium",
    "priorityLow": "Low",
    "dueDate": "Due Date",
    "submit": "Add Todo"
  },
  "list": {
    "title": "Your Tasks",
    "empty": "Your todo list is empty",
    "completed": "{{count}} task completed",
    "completed_plural": "{{count}} tasks completed",
    "pending": "{{count}} pending task",
    "pending_plural": "{{count}} pending tasks"
  },
  "item": {
    "markComplete": "Mark as complete",
    "markPending": "Mark as pending",
    "delete": "Delete task",
    "edit": "Edit task"
  },
  "status": {
    "pending": "Pending",
    "inProgress": "In Progress",
    "completed": "Completed"
  },
  "validation": {
    "required": "Task title is required",
    "minLength": "Task must be at least {{count}} characters",
    "maxLength": "Task cannot exceed {{count}} characters"
  }
}
```

### `public/locales/en/header.json`
```json
{
  "navigation": {
    "home": "Home",
    "about": "About",
    "contact": "Contact"
  },
  "search": { "placeholder": "Search tasks..." },
  "profile": {
    "myProfile": "My Profile",
    "settings": "Settings",
    "logout": "Logout"
  }
}
```

(Provide equivalent `fr` and `es` files mirroring the same keys in French and Spanish.)

---

## Components

### Language Switcher (`src/components/LanguageSwitcher.tsx`)
```tsx
import React from 'react';
import { useTranslation } from 'react-i18next';

export const LanguageSwitcher: React.FC = () => {
  const { i18n, t } = useTranslation('common');
  const languages = [
    { code: 'en', name: t('language.english'), flag: '🇬🇧' },
    { code: 'fr', name: t('language.french'), flag: '🇫🇷' },
    { code: 'es', name: t('language.spanish'), flag: '🇪🇸' },
  ];
  return (
    <div>
      <label>{t('language.label')}</label>
      {languages.map(lang => (
        <button
          key={lang.code}
          className={i18n.language === lang.code ? 'active' : ''}
          onClick={() => i18n.changeLanguage(lang.code)}
        >
          {lang.flag} {lang.name}
        </button>
      ))}
    </div>
  );
};
```

### Header (`src/components/Header.tsx`)
```tsx
import React from 'react';
import { useTranslation } from 'react-i18next';
import { LanguageSwitcher } from './LanguageSwitcher';

export const Header: React.FC = () => {
  const { t: tHeader } = useTranslation('header');
  const { t: tCommon } = useTranslation('common');
  return (
    <header>
      <div>
        <h1>{tCommon('app.title')}</h1>
        <p>{tCommon('app.description')}</p>
      </div>
      <nav>
        <a href="/">{tHeader('navigation.home')}</a>
        <a href="/about">{tHeader('navigation.about')}</a>
        <a href="/contact">{tHeader('navigation.contact')}</a>
      </nav>
      <LanguageSwitcher />
    </header>
  );
};
```

### Todo Form (`src/components/TodoForm.tsx`)
```tsx
import React, { useState } from 'react';
import { useTranslation } from 'react-i18next';

interface TodoFormProps { onAddTodo: (title: string, priority: string) => void; }

export const TodoForm: React.FC<TodoFormProps> = ({ onAddTodo }) => {
  const { t: tTodo } = useTranslation('todo');
  const { t: tCommon } = useTranslation('common');
  const [input, setInput] = useState('');
  const [priority, setPriority] = useState('medium');
  const [errors, setErrors] = useState<string[]>([]);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    setErrors([]);
    if (!input.trim()) { setErrors([tTodo('validation.required')]); return; }
    if (input.length < 3) { setErrors([tTodo('validation.minLength', { count: 3 })]); return; }
    onAddTodo(input, priority); setInput(''); setPriority('medium');
    alert(tCommon('messages.success'));
  };

  return (
    <form onSubmit={handleSubmit}>
      <h2>{tTodo('form.title')}</h2>
      {errors.map((e, i) => (<p key={i}>{e}</p>))}
      <label>{tTodo('form.label')}</label>
      <input
        placeholder={tTodo('form.placeholder')}
        value={input}
        onChange={e => setInput(e.target.value)}
      />
      <label>{tTodo('form.priority')}</label>
      <select value={priority} onChange={e => setPriority(e.target.value)}>
        <option value="low">{tTodo('form.priorityLow')}</option>
        <option value="medium">{tTodo('form.priorityMedium')}</option>
        <option value="high">{tTodo('form.priorityHigh')}</option>
      </select>
      <button type="submit">{tTodo('form.submit')}</button>
    </form>
  );
};
```

### Todo Item (`src/components/TodoItem.tsx`)
```tsx
import React from 'react';
import { useTranslation } from 'react-i18next';

interface Todo { id: number; title: string; status: 'pending' | 'inProgress' | 'completed'; priority: 'low' | 'medium' | 'high'; }

interface TodoItemProps { todo: Todo; onDelete: (id: number) => void; onStatusChange: (id: number, status: Todo['status']) => void; }

export const TodoItem: React.FC<TodoItemProps> = ({ todo, onDelete, onStatusChange }) => {
  const { t: tTodo } = useTranslation('todo');
  const { t: tCommon } = useTranslation('common');
  const nextStatus = todo.status === 'pending' ? 'inProgress' : todo.status === 'inProgress' ? 'completed' : 'pending';
  return (
    <li>
      <span>{todo.title}</span>
      <span>{tTodo(`status.${todo.status}`)}</span>
      <button onClick={() => onStatusChange(todo.id, nextStatus)} title={tTodo('item.markComplete')}>✓</button>
      <button onClick={() => onDelete(todo.id)} title={tCommon('buttons.delete')}>🗑️</button>
    </li>
  );
};
```

### Todo List (`src/components/TodoList.tsx`)
```tsx
import React from 'react';
import { useTranslation } from 'react-i18next';
import { TodoItem } from './TodoItem';

interface Todo { id: number; title: string; status: 'pending' | 'inProgress' | 'completed'; priority: 'low' | 'medium' | 'high'; }
interface TodoListProps { todos: Todo[]; onDelete: (id: number) => void; onStatusChange: (id: number, status: Todo['status']) => void; }

export const TodoList: React.FC<TodoListProps> = ({ todos, onDelete, onStatusChange }) => {
  const { t: tTodo } = useTranslation('todo');
  if (todos.length === 0) return <div>{tTodo('list.empty')}</div>;
  const completedCount = todos.filter(t => t.status === 'completed').length;
  const pendingCount = todos.filter(t => t.status === 'pending').length;
  return (
    <div>
      <h2>{tTodo('list.title')}</h2>
      <p>{tTodo('list.completed', { count: completedCount })}</p>
      <p>{tTodo('list.pending', { count: pendingCount })}</p>
      <ul>
        {todos.map(todo => (
          <TodoItem key={todo.id} todo={todo} onDelete={onDelete} onStatusChange={onStatusChange} />
        ))}
      </ul>
    </div>
  );
};
```

### App (`src/App.tsx`)
```tsx
import React, { useState } from 'react';
import { useTranslation } from 'react-i18next';
import { Header } from './components/Header';
import { TodoForm } from './components/TodoForm';
import { TodoList } from './components/TodoList';

interface Todo { id: number; title: string; status: 'pending' | 'inProgress' | 'completed'; priority: 'low' | 'medium' | 'high'; createdAt: Date; }

const App: React.FC = () => {
  const { t: tCommon } = useTranslation('common');
  const [todos, setTodos] = useState<Todo[]>([]);

  const handleAddTodo = (title: string, priority: string) => {
    const newTodo: Todo = { id: Date.now(), title, status: 'pending', priority: priority as 'low' | 'medium' | 'high', createdAt: new Date() };
    setTodos(prev => [...prev, newTodo]);
    alert(tCommon('messages.success'));
  };

  const handleDeleteTodo = (id: number) => {
    if (window.confirm(tCommon('messages.confirmDelete'))) {
      setTodos(prev => prev.filter(t => t.id !== id));
    }
  };

  const handleStatusChange = (id: number, status: Todo['status']) => {
    setTodos(prev => prev.map(t => (t.id === id ? { ...t, status } : t)));
  };

  return (
    <div>
      <Header />
      <main>
        <TodoForm onAddTodo={handleAddTodo} />
        <TodoList todos={todos} onDelete={handleDeleteTodo} onStatusChange={handleStatusChange} />
      </main>
      <footer>
        <p>{tCommon('messages.welcome', { name: 'Developer' })}</p>
      </footer>
    </div>
  );
};

export default App;
```

---

## Using `useTranslation()`

- Basic: `const { t } = useTranslation();`
- With namespace: `const { t } = useTranslation('todo');`
- Multiple namespaces: `const { t: tTodo } = useTranslation('todo'); const { t: tCommon } = useTranslation('common');`
- Change language: `const { i18n } = useTranslation(); i18n.changeLanguage('fr');`
- Current language: `i18n.language`
- Load extra namespace: `i18n.loadNamespaces(['header']);`

---

## Debugging & Troubleshooting

- Enable debug in config: `debug: true`
- Check loaded resources in console:
  - `i18next.language`
  - `i18next.loadedNamespaces`
  - `i18next.getResourceBundle('en', 'common')`
- Common issues:
  - Translations not loading → verify `backend.loadPath` and file existence
  - Hook not working → ensure `./i18n/config` is imported in `index.tsx`
  - Wrong key → confirm JSON structure and namespace
  - Language not switching → ensure correct codes (`en`, `fr`, `es`) and files exist

---

## Quick Start

1) Install packages
```bash
npm install i18next react-i18next i18next-http-backend i18next-browser-languagedetector
```

2) Add `src/i18n/config.ts` and import it in `src/index.tsx`

3) Create translation files under `public/locales/{lng}/{ns}.json`

4) Use `useTranslation('namespace')` in components and call `t('key.path')`

5) Switch language via `i18n.changeLanguage('fr')`

---

## Best Practices

- Use namespaces to organize by feature/module
- Keep keys lowercase with dot notation (`form.title`)
- Favor interpolation over string concatenation (`{{name}}`, `{{count}}`)
- Always provide `fallbackLng`
- Mirror keys across languages to avoid missing translations
