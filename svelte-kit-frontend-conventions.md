# SvelteKit Style Guide for LLM Code Generation

## Environment

- **Target Svelte Version**: **SvelteKit 1.0**
- **Target TypeScript Version**: **4.x**

## General Principles

1. **Use Svelte components as the primary building blocks.**
2. **Prefer composition over inheritance at all times.**
3. **Use TypeScript for type safety and interfaces consistently throughout the codebase.**
4. **Follow Svelte's conventions for naming and code formatting rigorously.**
5. **Utilize reactive declarations and stores appropriately to manage state.**

## Design Principles

### 1. High Cohesion and Low Coupling

- **Ensure each component or module has a single, well-defined responsibility.**
- **Minimize dependencies between components using props, events, and the context API.**

### 2. Favor Composition Over Inheritance

- **Build complex UIs from simpler components using composition.**
- **Avoid class inheritance entirely; Svelte favors functional and reactive programming.**
- **Utilize slots and props to compose and customize components effectively.**

### 3. Start with the Data

- **Define data structures first using TypeScript interfaces or types.**
- **Use type annotations to specify data types clearly and consistently.**
- **Design components and stores around data structures, not behaviors.**

### 4. Depend on Abstractions

- **Use TypeScript interfaces and types to define contracts between components and services.**
- **Code against abstractions, not concrete implementations, to enhance flexibility.**
- **Leverage dependency injection via the context API or module imports to decouple components.**

### 5. Separate Creation from Use

- **Initialize data and services outside of components whenever possible.**
- **Pass dependencies as props or through the context API; do not hardcode dependencies within components.**
- **Use service modules for shared logic and API interactions.**

### 6. Keep Things Simple

- **Write simple and readable code, avoiding clever or complex solutions.**
- **Do not add unnecessary functionality (YAGNI principle).**
- **Refactor and simplify code regularly to maintain clarity and efficiency.**

## Coding Standards

### TypeScript Type Annotations

1. **Use built-in types and generics like `Array<T>`, `Promise<T>`, and `Record<K, V>`.**
2. **Define custom types and interfaces for all complex data structures.**
3. **Use union types (e.g., `string | number`) where necessary.**
4. **Use `type | undefined` for optional properties instead of `null`.**
5. **Annotate all variables, function parameters, and return types explicitly.**
6. **Avoid using `any`; if the type cannot be determined, use `unknown` and refine as soon as possible.**

**Example:**

```typescript
interface User {
  id: number;
  name: string;
  email?: string;
}

function greet(user: User): string {
  return `Hello, ${user.name}!`;
}
```

### Components

1. **Use PascalCase for component names and filenames (e.g., `MyComponent.svelte`).**
2. **Keep components focused and small, adhering strictly to the Single Responsibility Principle.**
3. **Use reactive statements and declarations to manage state changes effectively.**
4. **Avoid using `bind:this`; use Svelte's reactivity and refs for DOM manipulation instead.**

### Props and State

1. **Define props using the `export` keyword in components, and annotate their types.**
2. **Use default values for props when appropriate to ensure components behave predictably.**
3. **Manage local state within components using reactive variables and statements.**
4. **Avoid modifying props directly; treat them as immutable within the component.**

**Example:**

```svelte
<script lang="ts">
  export let count: number = 0;

  $: doubled = count * 2;
</script>
```

### Stores

1. **Use Svelte's `writable`, `readable`, and `derived` stores for shared state management.**
2. **Create custom stores for complex state logic and encapsulate related functionality.**
3. **Subscribe to stores using the `$` prefix for automatic reactivity in components.**
4. **Unsubscribe from stores manually only if subscribing imperatively; otherwise, let Svelte handle it.**

**Example:**

```typescript
// store.ts
import { writable } from 'svelte/store';

export const count = writable(0);
```

```svelte
<script lang="ts">
  import { count } from './store';

  function increment() {
    count.update(n => n + 1);
  }
</script>

<button on:click={increment}>Count is {$count}</button>
```

### Events

1. **Use Svelte's event system for communication between components.**
2. **Dispatch custom events using `createEventDispatcher` with typed event payloads.**
3. **Handle events using the `on:eventName` directive in parent components.**
4. **Name events descriptively, using camelCase, to reflect their purpose clearly.**

**Example:**

```svelte
<script lang="ts">
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher<{ increment: { value: number } }>();

  function handleClick() {
    dispatch('increment', { value: 1 });
  }
</script>

<button on:click={handleClick}>Increment</button>
```

### Naming Conventions

1. **Use camelCase for variables, functions, and methods.**
2. **Use PascalCase for component names and TypeScript classes or interfaces.**
3. **Use UPPER_SNAKE_CASE for constants that are intended to remain unchanged.**
4. **Choose meaningful and descriptive names for all identifiers to enhance code readability.**

### Reactive Declarations and Statements

1. **Use reactive declarations (`$:`) for values that depend on reactive variables.**
2. **Avoid side effects within reactive statements; keep them pure and predictable.**
3. **Keep reactive statements simple and focused to maintain clarity.**

**Example:**

```svelte
<script lang="ts">
  let count = 0;

  $: doubled = count * 2;
</script>
```

### Comments and Documentation

1. **Use JSDoc comments for functions, interfaces, and any complex logic that requires explanation.**
2. **Document component APIs, including props, events, and slots, thoroughly.**
3. **Write comments to explain non-obvious code; avoid redundant or obvious comments.**

**Example:**

```typescript
/**
 * Fetches user data from the API.
 * @param id - The ID of the user to fetch.
 * @returns A Promise that resolves to a User object.
 */
async function fetchUser(id: number): Promise<User> {
  // ...
}
```

## Data Modeling

### TypeScript Interfaces and Types

1. **Define interfaces for object shapes, component props, and API responses.**
2. **Use types for unions, intersections, and more complex type manipulations.**
3. **Prefer interfaces when you need to extend object types through inheritance.**
4. **Use `readonly` and `Partial<>` modifiers when you want to enforce immutability or optionality.**

**Example:**

```typescript
interface Address {
  street: string;
  city: string;
  zipCode: string;
}

interface User {
  id: number;
  name: string;
  address?: Address;
}
```

### API Integration

1. **Define data models that mirror backend API schemas using TypeScript interfaces.**
2. **Use the Fetch API for network requests, leveraging `async`/`await` syntax.**
3. **Handle API responses and errors gracefully, providing meaningful feedback to the user.**
4. **Ensure type safety by explicitly typing API response data.**

**Example:**

```typescript
async function getUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new Error(`Failed to fetch user with id ${id}`);
  }
  return response.json() as Promise<User>;
}
```

### Composition Patterns

1. **Compose components using props and slots to build flexible and reusable UIs.**
2. **Use the context API for dependency injection to avoid prop drilling.**
3. **Create higher-order components (HOCs) only when necessary; prefer composition over HOCs.**
4. **Utilize custom stores to manage shared logic and state effectively.**

**Example:**

```svelte
<!-- Modal.svelte -->
<div class="modal">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

## Application Structure

### File Organization

1. **Organize files by feature or domain, grouping related components, stores, and utilities together.**
2. **Use consistent and descriptive naming conventions for files and directories.**
3. **Separate concerns by keeping components, stores, utilities, and styles in their respective directories.**
4. **Avoid deep directory nesting to maintain simplicity and ease of navigation.**

**Example Directory Structure:**

```
src/
├── components/
│   ├── Button.svelte
│   └── Modal.svelte
├── routes/
│   ├── +layout.svelte
│   └── +page.svelte
├── stores/
│   └── userStore.ts
├── lib/
│   └── utils.ts
```

### Routing

1. **Use SvelteKit's file-based routing system exclusively.**
2. **Organize routes within the `src/routes` directory, using `+page.svelte` and `+layout.svelte` appropriately.**
3. **Use nested layouts and pages to create a hierarchical structure.**
4. **Leverage dynamic routes and parameters for flexible navigation and data retrieval.**

**Example:**

```
src/routes/
├── +layout.svelte
├── index.svelte
├── about.svelte
└── items/
    ├── +page.svelte
    └── [id]/
        └── +page.svelte
```

### Data Fetching and Load Functions

1. **Use `load` functions in `+page.ts` or `+layout.ts` for data fetching required by the page.**
2. **Perform server-side fetching whenever possible to improve performance and SEO.**
3. **Handle errors and redirects within the `load` function using SvelteKit's conventions.**
4. **Use the `fetch` function provided by the `load` context to make requests, ensuring cookies and headers are preserved.**

**Example:**

```typescript
// +page.ts
import type { PageLoad } from './$types';

export const load: PageLoad = async ({ fetch, params }) => {
  const response = await fetch(`/api/items/${params.id}`);
  if (!response.ok) {
    return {
      status: response.status,
      error: new Error(`Failed to fetch item with id ${params.id}`),
    };
  }
  const item = await response.json() as Item;
  return { item };
};
```

### Stores and State Management

1. **Use Svelte stores for global or shared state across components or pages.**
2. **Avoid using stores for state that is local to a component; use component state instead.**
3. **Use `derived` stores for computed state based on other stores to avoid redundant computations.**
4. **Keep store logic encapsulated within the store to promote maintainability and reuse.**

### Error Handling

1. **Use SvelteKit's error handling by returning errors from `load` functions or throwing exceptions.**
2. **Create custom error pages (`__error.svelte`) to display user-friendly error messages.**
3. **Use try-catch blocks for asynchronous operations within components and `load` functions.**
4. **Provide informative and actionable error messages to enhance user experience.**

**Example:**

```typescript
// +page.ts
import type { PageLoad } from './$types';

export const load: PageLoad = async ({ fetch }) => {
  try {
    const response = await fetch('/api/data');
    if (!response.ok) {
      throw new Error(`Failed to fetch data: ${response.statusText}`);
    }
    const data = await response.json();
    return { data };
  } catch (error) {
    return {
      status: 500,
      error: new Error('Internal Server Error'),
    };
  }
};
```

### Dependency Management

1. **Use npm for package management consistently across the project.**
2. **Specify exact versions of dependencies in `package.json` to ensure consistency across environments.**
3. **Regularly update dependencies and test for compatibility issues.**
4. **Avoid adding unnecessary dependencies to keep the bundle size minimal and reduce potential security risks.**

### Asynchronous Programming

1. **Use `async`/`await` syntax for asynchronous code to enhance readability and maintainability.**
2. **Handle promise rejections using try-catch blocks or `.catch()` methods to prevent unhandled exceptions.**
3. **Avoid blocking the UI with heavy computations; use web workers if necessary.**
4. **Leverage Svelte's reactivity to update the UI upon data changes without manual DOM manipulation.**

### Styling

1. **Use scoped styles within components by default to avoid style conflicts.**
2. **Leverage CSS variables and themes to maintain consistent styling across the application.**
3. **Follow the BEM (Block, Element, Modifier) naming convention for class names to enhance readability.**
4. **Use SCSS as a preprocessor for advanced styling features and maintainable stylesheets.**

**Example:**

```svelte
<style lang="scss">
  .button {
    background-color: var(--primary-color);
    &:hover {
      background-color: var(--primary-color-dark);
    }
  }
</style>
```

## Best Practices

### Testing

1. **Use Vitest as the testing framework for unit and integration tests.**
2. **Use @testing-library/svelte for testing Svelte components.**
3. **Write tests for all critical logic and components to ensure reliability.**
4. **Mock API calls and external dependencies in tests to isolate units and avoid flakiness.**

**Example:**

```typescript
// MyComponent.test.ts
import { render, fireEvent } from '@testing-library/svelte';
import MyComponent from './MyComponent.svelte';

test('renders with default props', () => {
  const { getByText } = render(MyComponent);
  expect(getByText('Default Text')).toBeInTheDocument();
});
```

### Accessibility

1. **Follow WAI-ARIA guidelines strictly to ensure the application is accessible to all users.**
2. **Use semantic HTML elements (e.g., `<button>`, `<nav>`, `<main>`) for better accessibility and SEO.**
3. **Ensure keyboard navigation works for all interactive elements and components.**
4. **Provide alternative text for images and media content using the `alt` attribute.**

### Security

1. **Sanitize all user inputs and outputs to prevent Cross-Site Scripting (XSS) attacks.**
2. **Avoid exposing sensitive data or secrets in the frontend codebase.**
3. **Use HTTPS for all network requests and API interactions to secure data in transit.**
4. **Implement proper authentication and authorization checks, especially on pages that access sensitive information.**

### Performance

1. **Optimize images and assets using appropriate formats (e.g., WebP) and compression techniques.**
2. **Lazy-load components and resources that are not immediately needed to improve initial load times.**
3. **Minimize unnecessary reactivity by avoiding overuse of reactive variables and statements.**
4. **Use code splitting and dynamic imports to reduce the bundle size of the initial page load.**

### Code Readability

1. **Use Prettier for consistent code formatting across the project, adhering to a shared configuration.**
2. **Break down large components into smaller, reusable ones to enhance maintainability.**
3. **Organize code logically within components, grouping related variables and functions together.**
4. **Avoid deeply nested code structures; flatten logic where possible for clarity.**

### Functional Programming

1. **Write pure functions for data transformations and avoid side effects to ensure predictability.**
2. **Avoid side effects in reactive statements; keep them focused on data computation.**
3. **Use immutable data patterns, avoiding direct mutation of objects or arrays.**
4. **Leverage array and object methods (e.g., `map`, `filter`, `reduce`) for data manipulation instead of loops when appropriate.**

### When to Use Classes and OOP

1. **Use classes only when integrating with libraries or APIs that require them.**
2. **Prefer functional and reactive programming paradigms within Svelte components.**
3. **Use classes for complex data models or services that benefit from encapsulation and inheritance, but only when necessary.**

## Example Structures

### Component with Props and Events

```svelte
<!-- Counter.svelte -->
<script lang="ts">
  import { createEventDispatcher } from 'svelte';

  export let count: number = 0;
  const dispatch = createEventDispatcher<{ increment: number }>();

  function increment() {
    count += 1;
    dispatch('increment', count);
  }
</script>

<button on:click={increment}>Count is {count}</button>
```

### Using Stores

```svelte
<!-- App.svelte -->
<script lang="ts">
  import { count } from './store';

  function increment() {
    count.update(n => n + 1);
  }
</script>

<button on:click={increment}>Count is {$count}</button>
```

### Fetching Data in Load Function

```typescript
// +page.ts
import type { PageLoad } from './$types';

export const load: PageLoad = async ({ fetch }) => {
  const response = await fetch('/api/items');
  if (!response.ok) {
    throw new Error('Failed to fetch items');
  }
  const items = await response.json() as Item[];
  return { items };
};
```

### Custom Store

```typescript
// userStore.ts
import { writable } from 'svelte/store';
import type { User } from './types';

function createUserStore() {
  const { subscribe, set } = writable<User | null>(null);

  return {
    subscribe,
    login: async (username: string, password: string) => {
      const user = await apiLogin(username, password);
      set(user);
    },
    logout: () => set(null),
  };
}

export const userStore = createUserStore();
```

### Context API

```svelte
<!-- ThemeProvider.svelte -->
<script lang="ts">
  import { setContext } from 'svelte';

  export let theme: 'light' | 'dark' = 'light';
  setContext('theme', theme);
</script>

<slot></slot>
```

```svelte
<!-- ChildComponent.svelte -->
<script lang="ts">
  import { getContext } from 'svelte';

  const theme = getContext<'light' | 'dark'>('theme');
</script>

<div class={`theme-${theme}`}>
  <!-- content -->
</div>
```

### Error Handling in Load Function

```typescript
// +page.ts
import type { PageLoad } from './$types';

export const load: PageLoad = async ({ fetch }) => {
  try {
    const response = await fetch('/api/data');
    if (!response.ok) {
      throw new Error(`Failed to fetch data: ${response.statusText}`);
    }
    const data = await response.json();
    return { data };
  } catch (error) {
    return {
      status: 500,
      error: new Error('Internal Server Error'),
    };
  }
};
```

### Unit Testing a Component

```typescript
// Button.test.ts
import { render, fireEvent } from '@testing-library/svelte';
import Button from './Button.svelte';

test('increments count on click', async () => {
  const { getByText, component } = render(Button, { props: { count: 0 } });
  const button = getByText('Count is 0');

  await fireEvent.click(button);

  expect(component.$$.ctx[0]).toBe(1);
  expect(getByText('Count is 1')).toBeInTheDocument();
});
```