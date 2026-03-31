# 1. Principles Framework

[← Back to Index](frontend-engineering-framework.md)

---

## 1.1 Core Software Design Principles

| Principle | Definition | Frontend Application |
|-----------|-----------|----------------------|
| **SRP** — Single Responsibility | A module/component should have one reason to change | One component = one UI concern. Separate data fetching from rendering. |
| **OCP** — Open/Closed | Open for extension, closed for modification | Use composition, render props, slots, and higher-order components instead of modifying shared components. |
| **LSP** — Liskov Substitution | Subtypes must be substitutable for their base types | A `<PrimaryButton>` should be usable anywhere a `<Button>` is expected without breaking the interface contract. |
| **ISP** — Interface Segregation | No client should depend on methods it doesn't use | Keep component prop interfaces minimal. Avoid "god props" objects. Split large context providers. |
| **DIP** — Dependency Inversion | Depend on abstractions, not concretions | Components depend on service interfaces (hooks, context), not on `fetch()` or `localStorage` directly. |

### SRP — React Example

```tsx
// ❌ Violates SRP — one component does fetching, logic, AND rendering
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then((data) => {
        setUser(data)
        setLoading(false)
      })
  }, [userId])

  if (loading) return <Spinner />
  if (!user) return <ErrorMessage />

  const fullName = `${user.firstName} ${user.lastName}`
  const initials = `${user.firstName[0]}${user.lastName[0]}`

  return (
    <div className="profile">
      <Avatar initials={initials} />
      <h2>{fullName}</h2>
      <p>{user.email}</p>
    </div>
  )
}

// ✅ Follows SRP — separated into hook (data), util (logic), component (UI)

// Hook: data fetching concern
function useUser(userId: string) {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then(setUser)
      .finally(() => setLoading(false))
  }, [userId])

  return { user, loading }
}

// Util: business logic concern
function formatUser(user: User) {
  return {
    fullName: `${user.firstName} ${user.lastName}`,
    initials: `${user.firstName[0]}${user.lastName[0]}`,
  }
}

// Component: presentation concern only
function UserProfile({ userId }: { userId: string }) {
  const { user, loading } = useUser(userId)

  if (loading) return <Spinner />
  if (!user) return <ErrorMessage />

  const { fullName, initials } = formatUser(user)

  return (
    <div className="profile">
      <Avatar initials={initials} />
      <h2>{fullName}</h2>
      <p>{user.email}</p>
    </div>
  )
}
```

### OCP — React Example

```tsx
// ❌ Violates OCP — must modify this component every time a new alert type is added
function Alert({ type, message }: { type: string; message: string }) {
  if (type === 'success') {
    return <div className="alert-success">✅ {message}</div>
  }
  if (type === 'error') {
    return <div className="alert-error">❌ {message}</div>
  }
  if (type === 'warning') {
    return <div className="alert-warning">⚠️ {message}</div>
  }
  return <div>{message}</div>
}

// ✅ Follows OCP — extensible via composition without modifying the base
interface AlertProps {
  icon: ReactNode
  className: string
  children: ReactNode
}

function Alert({ icon, className, children }: AlertProps) {
  return (
    <div className={`alert ${className}`}>
      {icon}
      <span>{children}</span>
    </div>
  )
}

// Extend by creating new components — no modification to Alert needed
function SuccessAlert({ children }: { children: ReactNode }) {
  return <Alert icon={<CheckIcon />} className="alert-success">{children}</Alert>
}

function ErrorAlert({ children }: { children: ReactNode }) {
  return <Alert icon={<XIcon />} className="alert-error">{children}</Alert>
}

// New type? Just add a new component:
function InfoAlert({ children }: { children: ReactNode }) {
  return <Alert icon={<InfoIcon />} className="alert-info">{children}</Alert>
}
```

### LSP — React Example

```tsx
// Base Button component
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  children: ReactNode
}

function Button({ children, className, ...rest }: ButtonProps) {
  return (
    <button className={`btn ${className ?? ''}`} {...rest}>
      {children}
    </button>
  )
}

// ✅ Follows LSP — PrimaryButton can replace Button anywhere without breaking
function PrimaryButton({ children, className, ...rest }: ButtonProps) {
  return (
    <Button className={`btn-primary ${className ?? ''}`} {...rest}>
      {children}
    </Button>
  )
}

// ✅ Follows LSP — IconButton extends the contract, doesn't break it
interface IconButtonProps extends ButtonProps {
  icon: ReactNode
}

function IconButton({ icon, children, ...rest }: IconButtonProps) {
  return (
    <Button {...rest}>
      {icon}
      {children}
    </Button>
  )
}

// Usage — any Button subtype works interchangeably
function Toolbar({ actions }: { actions: { label: string; onClick: () => void }[] }) {
  return (
    <div>
      {actions.map((action) => (
        <Button key={action.label} onClick={action.onClick}>
          {action.label}
        </Button>
      ))}
    </div>
  )
}

// Can swap Button for PrimaryButton or IconButton — no breakage
```

### ISP — React Example

```tsx
// ❌ Violates ISP — UserCard depends on the entire User object but only uses name/avatar
interface User {
  id: string
  name: string
  avatar: string
  email: string
  address: Address
  preferences: Preferences
  paymentMethods: PaymentMethod[]
}

function UserCard({ user }: { user: User }) {
  return (
    <div>
      <img src={user.avatar} alt={user.name} />
      <span>{user.name}</span>
    </div>
  )
}

// ✅ Follows ISP — component only requires what it actually uses
interface UserCardProps {
  name: string
  avatar: string
}

function UserCard({ name, avatar }: UserCardProps) {
  return (
    <div>
      <img src={avatar} alt={name} />
      <span>{name}</span>
    </div>
  )
}

// ❌ Violates ISP — one giant context for everything
const AppContext = createContext<{
  user: User
  theme: Theme
  locale: string
  cart: CartItem[]
  notifications: Notification[]
}>(/* ... */)

// ✅ Follows ISP — split into focused contexts
const AuthContext = createContext<{ user: User }>(/* ... */)
const ThemeContext = createContext<{ theme: Theme }>(/* ... */)
const CartContext = createContext<{ cart: CartItem[] }>(/* ... */)
```

### DIP — React Example

```tsx
// ❌ Violates DIP — component directly depends on concrete implementations
function UserSettings() {
  const handleSave = async (settings: Settings) => {
    // Directly coupled to fetch and localStorage
    await fetch('/api/settings', {
      method: 'POST',
      body: JSON.stringify(settings),
    })
    localStorage.setItem('settings', JSON.stringify(settings))
  }

  return <SettingsForm onSave={handleSave} />
}

// ✅ Follows DIP — component depends on abstractions (hooks/context)

// Abstract service interface via hook
function useSettingsService() {
  const api = useApiClient()       // abstraction over fetch
  const storage = useStorage()     // abstraction over localStorage

  const saveSettings = async (settings: Settings) => {
    await api.post('/settings', settings)
    storage.set('settings', settings)
  }

  return { saveSettings }
}

// Component depends on the hook abstraction — easily testable & swappable
function UserSettings() {
  const { saveSettings } = useSettingsService()
  return <SettingsForm onSave={saveSettings} />
}

// In tests, provide mock implementations via context providers
// In production, provide real implementations
// The component never knows the difference
```

---

## 1.2 General Engineering Principles

### DRY — Don't Repeat Yourself

> "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system."

- Extract shared logic into custom hooks, utilities, or shared components.
- Centralize constants, type definitions, and configuration.
- **Frontend caveat**: Don't over-DRY presentational code. Two components that look the same today may diverge tomorrow. Duplicate first, abstract later — only when the pattern is proven.

```tsx
// ❌ Violates DRY — same validation logic duplicated across components
function LoginForm() {
  const [email, setEmail] = useState('')
  const [error, setError] = useState('')

  const validate = () => {
    if (!email.includes('@')) setError('Invalid email')
  }
  // ...
}

function SignupForm() {
  const [email, setEmail] = useState('')
  const [error, setError] = useState('')

  const validate = () => {
    if (!email.includes('@')) setError('Invalid email')
  }
  // ...
}

// ✅ Follows DRY — shared logic extracted into a custom hook
function useEmailField(initialValue = '') {
  const [email, setEmail] = useState(initialValue)
  const [error, setError] = useState('')

  const validate = () => {
    if (!email.includes('@')) {
      setError('Invalid email')
      return false
    }
    setError('')
    return true
  }

  return { email, setEmail, error, validate }
}

function LoginForm() {
  const { email, setEmail, error, validate } = useEmailField()
  // ...
}

function SignupForm() {
  const { email, setEmail, error, validate } = useEmailField()
  // ...
}
```

### WET — Write Everything Twice

> "Duplication is far cheaper than the wrong abstraction."

- Allow up to 2–3 duplications before extracting a shared abstraction.
- Premature abstraction creates rigid, hard-to-change code.
- Use WET as a **threshold signal**: the third duplication is when you abstract.

```tsx
// ✅ WET is OK here — two cards look similar but serve different purposes
// Don't abstract prematurely; they may diverge

function ProductCard({ product }: { product: Product }) {
  return (
    <div className="card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button>Add to Cart</button>
    </div>
  )
}

function WishlistCard({ item }: { item: WishlistItem }) {
  return (
    <div className="card">
      <img src={item.image} alt={item.name} />
      <h3>{item.name}</h3>
      <p>${item.price}</p>
      <button>Move to Cart</button>
    </div>
  )
}

// 🧠 Only when a THIRD similar card appears, consider abstracting:
interface GenericCardProps {
  image: string
  name: string
  price: number
  actionLabel: string
  onAction: () => void
}

function GenericCard({ image, name, price, actionLabel, onAction }: GenericCardProps) {
  return (
    <div className="card">
      <img src={image} alt={name} />
      <h3>{name}</h3>
      <p>${price}</p>
      <button onClick={onAction}>{actionLabel}</button>
    </div>
  )
}
```

### KISS — Keep It Simple, Stupid

- Choose the simplest solution that satisfies the requirement.
- Avoid clever one-liners that sacrifice readability.
- Prefer explicit code over implicit magic (e.g., explicit props over deep injection).
- A junior developer should understand your component within a few minutes.

```tsx
// ❌ Overly clever — hard to read and maintain
function StatusBadge({ status }: { status: string }) {
  return (
    <span
      className={[
        'badge',
        ...(['active', 'pending', 'inactive'].includes(status)
          ? [`badge-${status}`]
          : []),
        ...(status === 'active' ? ['pulse'] : []),
      ].join(' ')}
    >
      {status.replace(/^\w/, (c) => c.toUpperCase())}
    </span>
  )
}

// ✅ KISS — simple, readable, obvious
const badgeStyles: Record<string, string> = {
  active: 'badge badge-active pulse',
  pending: 'badge badge-pending',
  inactive: 'badge badge-inactive',
}

function StatusBadge({ status }: { status: string }) {
  const className = badgeStyles[status] ?? 'badge'

  return <span className={className}>{status}</span>
}
```

### YAGNI — You Aren't Gonna Need It

- Don't build features, abstractions, or configurations speculatively.
- Avoid adding optional parameters "just in case" — extend when the need is real.
- Applies to: premature generalization, unused component variants, over-configurable props, speculative API fields.

```tsx
// ❌ Violates YAGNI — over-engineered for hypothetical future needs
interface ButtonProps {
  label: string
  icon?: ReactNode
  iconPosition?: 'left' | 'right' | 'top' | 'bottom'
  variant?: 'primary' | 'secondary' | 'tertiary' | 'ghost' | 'link' | 'danger'
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl'
  loading?: boolean
  loadingText?: string
  tooltip?: string
  tooltipPosition?: 'top' | 'bottom' | 'left' | 'right'
  analytics?: { event: string; category: string }
  as?: React.ElementType  // polymorphic — we only use <button>
}

// ✅ Follows YAGNI — only what the current requirements demand
interface ButtonProps {
  label: string
  variant?: 'primary' | 'secondary'
  onClick: () => void
}

function Button({ label, variant = 'primary', onClick }: ButtonProps) {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {label}
    </button>
  )
}

// When you actually need an icon button later, THEN add it.
```

### LoD — Law of Demeter (Principle of Least Knowledge)

> "Only talk to your immediate friends."

- A component should not reach deep into nested objects: avoid `user.address.city.zipCode` chains.
- Pass only what the component needs — not entire parent objects.
- Use selectors or derived state to flatten data before passing to UI.

```jsx
// ❌ Violates LoD — component knows too much about data shape
<OrderCard total={order.payment.summary.total} />

// ✅ Respects LoD — parent extracts what's needed
const total = selectOrderTotal(order)
<OrderCard total={total} />
```

### CQS — Command-Query Separation

- **Queries** return data and have no side effects (getters, selectors, computed values).
- **Commands** perform actions and change state (event handlers, mutations, dispatches).
- Never mix both: a function should either return something or change something, not both.

```ts
// ❌ Mixed — returns value AND mutates
function addItemAndGetTotal(item: Item): number {
  cart.push(item)        // command
  return cart.totalPrice // query
}

// ✅ Separated
function addItem(item: Item): void { cart.push(item) }
function getTotal(): number { return cart.totalPrice }
```

```tsx
// ✅ CQS in React — separate query hooks from command handlers

// Query: returns data, no side effects
function useCartTotal(items: CartItem[]): number {
  return useMemo(
    () => items.reduce((sum, item) => sum + item.price * item.quantity, 0),
    [items]
  )
}

// Command: performs action, returns void
function useCartActions() {
  const dispatch = useCartDispatch()

  const addItem = (item: CartItem) => dispatch({ type: 'ADD_ITEM', item })
  const removeItem = (id: string) => dispatch({ type: 'REMOVE_ITEM', id })

  return { addItem, removeItem }
}

// Usage — queries and commands are clearly separated
function Cart() {
  const { items } = useCart()
  const total = useCartTotal(items)        // query
  const { removeItem } = useCartActions()  // commands

  return (
    <div>
      {items.map((item) => (
        <CartRow key={item.id} item={item} onRemove={() => removeItem(item.id)} />
      ))}
      <p>Total: ${total}</p>
    </div>
  )
}
```

### Separation of Concerns (Broad Principle)

- Each module, layer, or component addresses a distinct concern.
- In frontend: separate **data fetching** from **presentation**, **business logic** from **UI logic**, **styling** from **structure**.
- Framework-specific: React hooks separate logic from JSX; Vue composables separate logic from templates; Angular services separate logic from components.

```tsx
// ❌ Mixed concerns — data fetching, business logic, and UI all in one
function ProductPage({ id }: { id: string }) {
  const [product, setProduct] = useState<Product | null>(null)

  useEffect(() => {
    fetch(`/api/products/${id}`)
      .then((res) => res.json())
      .then(setProduct)
  }, [id])

  if (!product) return <p>Loading...</p>

  const discountedPrice = product.price * (1 - product.discount / 100)
  const inStock = product.inventory > 0

  return (
    <div style={{ padding: 20, border: '1px solid #ccc' }}>
      <h1>{product.name}</h1>
      <p>${discountedPrice.toFixed(2)}</p>
      {inStock ? <button>Buy Now</button> : <span>Out of Stock</span>}
    </div>
  )
}

// ✅ Separated concerns

// Data layer — hook handles fetching
function useProduct(id: string) {
  const [product, setProduct] = useState<Product | null>(null)
  useEffect(() => {
    fetch(`/api/products/${id}`)
      .then((res) => res.json())
      .then(setProduct)
  }, [id])
  return product
}

// Business logic — pure functions
function calcDiscountedPrice(price: number, discount: number) {
  return price * (1 - discount / 100)
}

// Presentation — only renders UI
function ProductPage({ id }: { id: string }) {
  const product = useProduct(id)

  if (!product) return <p>Loading...</p>

  return (
    <ProductCard
      name={product.name}
      price={calcDiscountedPrice(product.price, product.discount)}
      inStock={product.inventory > 0}
    />
  )
}
```

### Encapsulation

- Hide internal implementation details; expose only a public API.
- Components expose props/events — not internal state or refs.
- CSS Modules/scoped styles encapsulate visual details.
- Barrel exports (`index.ts`) define the public surface of a feature module.

```
feature/
├── components/       ← internal
├── hooks/            ← internal
├── utils/            ← internal
└── index.ts          ← public API (only this is importable)
```

```tsx
// ❌ Poor encapsulation — exposes internal state and implementation details
function useCounter() {
  const [state, setState] = useState({ count: 0, history: [] as number[] })

  return {
    state,       // ❌ exposes raw state object
    setState,    // ❌ consumers can mutate anything
  }
}

// ✅ Good encapsulation — exposes only a controlled public API
function useCounter(initial = 0) {
  const [count, setCount] = useState(initial)

  const increment = () => setCount((c) => c + 1)
  const decrement = () => setCount((c) => c - 1)
  const reset = () => setCount(initial)

  return { count, increment, decrement, reset } as const
}

// Consumer only sees the public API — cannot access or break internals
function Counter() {
  const { count, increment, decrement, reset } = useCounter(0)

  return (
    <div>
      <span>{count}</span>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  )
}
```

### Principle of Least Astonishment (POLA)

- Code should behave the way other developers expect.
- Name functions and components clearly — `useAuth()` should relate to authentication, not analytics.
- Follow framework conventions: a React hook starts with `use`, a Vue composable starts with `use`, an Angular service is injectable.
- Default behavior should be safe and unsurprising.

```tsx
// ❌ Astonishing behavior — confusing names and unexpected side effects
function useAuth() {
  // Surprise! This also tracks analytics and fetches user preferences
  useEffect(() => {
    trackPageView()          // 😱 not auth-related
    fetchPreferences()       // 😱 not auth-related
  }, [])
  return { user: null }
}

function Input({ value, onChange }: { value: string; onChange: (v: string) => void }) {
  // Surprise! Automatically trims and lowercases input
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    onChange(e.target.value.trim().toLowerCase()) // 😱 unexpected transformation
  }
  return <input value={value} onChange={handleChange} />
}

// ✅ Unsurprising — does exactly what the name suggests
function useAuth() {
  const [user, setUser] = useState<User | null>(null)

  useEffect(() => {
    authService.getCurrentUser().then(setUser)
  }, [])

  return { user, isAuthenticated: !!user }
}

function Input({ value, onChange }: { value: string; onChange: (v: string) => void }) {
  return (
    <input
      value={value}
      onChange={(e) => onChange(e.target.value)} // passes raw value, no surprises
    />
  )
}
```

### Robustness Principle (Postel's Law)

> "Be conservative in what you send, be liberal in what you accept."

- Components should accept flexible input (optional props, sensible defaults) but produce strict, predictable output.
- API clients should handle unexpected response shapes gracefully (validate at the boundary with Zod/Valibot).
- Useful for building resilient shared libraries and design system components.

```tsx
// ✅ Liberal in what it accepts — flexible props with sensible defaults
interface AvatarProps {
  src?: string | null
  name?: string
  size?: number | 'sm' | 'md' | 'lg'
}

const sizeMap = { sm: 24, md: 40, lg: 64 }

function Avatar({ src, name = 'User', size = 'md' }: AvatarProps) {
  const px = typeof size === 'number' ? size : sizeMap[size]

  // Conservative in output — always renders a consistent, predictable result
  if (!src) {
    return (
      <div
        className="avatar avatar-fallback"
        style={{ width: px, height: px }}
        role="img"
        aria-label={name}
      >
        {name[0].toUpperCase()}
      </div>
    )
  }

  return (
    <img
      className="avatar"
      src={src}
      alt={name}
      width={px}
      height={px}
    />
  )
}

// ✅ Validate at the boundary — liberal accept, strict internal types
import { z } from 'zod'

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
})

async function fetchUser(id: string) {
  const res = await fetch(`/api/users/${id}`)
  const data = await res.json()
  return UserSchema.parse(data) // validates at boundary, strict type inside
}
```

### Inversion of Control (IoC)

- The framework calls your code, not the other way around.
- Frontend examples: React renders your components, the router calls your loaders, middleware processes your actions.
- **Hooks / composables** are IoC — the framework orchestrates their lifecycle.
- **Dependency injection** (Angular) and **context** (React/Vue) are IoC mechanisms for decoupling consumers from implementations.

```tsx
// ❌ No IoC — component controls everything directly
function App() {
  const [user, setUser] = useState<User | null>(null)

  useEffect(() => {
    const token = localStorage.getItem('token')
    if (token) {
      fetch('/api/me', { headers: { Authorization: `Bearer ${token}` } })
        .then((res) => res.json())
        .then(setUser)
    }
  }, [])

  return user ? <Dashboard user={user} /> : <Login />
}

// ✅ IoC via Context — framework manages auth, components consume it

// Provider controls the auth lifecycle (IoC container)
function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null)

  useEffect(() => {
    authService.getCurrentUser().then(setUser)
  }, [])

  const login = async (credentials: Credentials) => {
    const user = await authService.login(credentials)
    setUser(user)
  }

  const logout = async () => {
    await authService.logout()
    setUser(null)
  }

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  )
}

// Components simply consume — they don't control the auth lifecycle
function Dashboard() {
  const { user, logout } = useAuth()
  return (
    <div>
      <h1>Welcome, {user?.name}</h1>
      <button onClick={logout}>Logout</button>
    </div>
  )
}
```

### AHA — Avoid Hasty Abstractions

> "Prefer duplication over the wrong abstraction." — Sandi Metz

- Wait until you see the actual shape of variation before abstracting.
- Optimize for change, not for reducing lines of code.
- Wrong abstractions are more expensive to fix than duplication.

```tsx
// ❌ Hasty abstraction — created a "universal" modal after seeing just one use case
function Modal({
  type,
  title,
  body,
  confirmLabel,
  cancelLabel,
  showFooter,
  showHeader,
  size,
  onConfirm,
  onCancel,
  variant,
}: UniversalModalProps) {
  // 150 lines of conditional rendering for every possible modal layout...
}

// ✅ AHA — start with concrete, specific components
function DeleteConfirmModal({ itemName, onConfirm, onCancel }: {
  itemName: string
  onConfirm: () => void
  onCancel: () => void
}) {
  return (
    <div className="modal">
      <h2>Delete {itemName}?</h2>
      <p>This action cannot be undone.</p>
      <div className="modal-actions">
        <button onClick={onCancel}>Cancel</button>
        <button onClick={onConfirm} className="btn-danger">Delete</button>
      </div>
    </div>
  )
}

function EditProfileModal({ user, onSave, onClose }: {
  user: User
  onSave: (data: UserUpdate) => void
  onClose: () => void
}) {
  return (
    <div className="modal modal-lg">
      <h2>Edit Profile</h2>
      <ProfileForm user={user} onSubmit={onSave} />
      <button onClick={onClose}>Cancel</button>
    </div>
  )
}

// Only abstract when you see real, repeated patterns — not before.
```

### Principle Tension Map

Principles often compete. Use judgment based on context:

```
DRY ◄──────────────────────────────► WET / AHA
"Remove all duplication"             "Duplicate until the pattern is clear"

YAGNI ◄────────────────────────────► Extensibility (OCP)
"Don't build it yet"                 "Design for future extension"

KISS ◄─────────────────────────────► Robustness
"Keep it simple"                     "Handle all edge cases"

Encapsulation ◄────────────────────► Transparency
"Hide internals"                     "Make behavior observable/debuggable"
```

**Rule of thumb**: Start simple (KISS, YAGNI, WET). Add complexity only when proven necessary (DRY, OCP, robustness). The best code is the minimum code that correctly solves the current problem.

---

## 1.3 Frontend-Specific Principles

### Separation of Concerns (SoC) — Applied

```
┌─────────────────────────────────────────────┐
│                  UI Layer                    │
│  Components · Styles · Animations · Layout  │
├─────────────────────────────────────────────┤
│              State Layer                     │
│  Local State · Global Store · Cache · URL   │
├─────────────────────────────────────────────┤
│              Service Layer                   │
│  API Clients · Auth · Validation · i18n     │
├─────────────────────────────────────────────┤
│            Infrastructure Layer              │
│  Routing · Build · Env Config · Analytics   │
└─────────────────────────────────────────────┘
```

- **Structure**: Markup/templates handle document structure.
- **Presentation**: CSS/styling handles visual appearance.
- **Behavior**: JavaScript handles interactivity and logic.
- **Data**: State management handles application data flow.

```tsx
// ✅ SoC Applied in React — each layer has a clear responsibility

// Service Layer: API client
const todoApi = {
  getAll: () => fetch('/api/todos').then((r) => r.json()),
  create: (title: string) =>
    fetch('/api/todos', {
      method: 'POST',
      body: JSON.stringify({ title }),
      headers: { 'Content-Type': 'application/json' },
    }).then((r) => r.json()),
}

// State Layer: custom hook manages state
function useTodos() {
  const [todos, setTodos] = useState<Todo[]>([])

  useEffect(() => {
    todoApi.getAll().then(setTodos)
  }, [])

  const addTodo = async (title: string) => {
    const newTodo = await todoApi.create(title)
    setTodos((prev) => [...prev, newTodo])
  }

  return { todos, addTodo }
}

// UI Layer: only handles rendering
function TodoList() {
  const { todos, addTodo } = useTodos()

  return (
    <div>
      <TodoForm onSubmit={addTodo} />
      {todos.map((todo) => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </div>
  )
}
```

### Composition Over Inheritance

- Prefer composing small components over extending base classes.
- Use hooks (React), composables (Vue 3), or services (Angular) for shared logic.
- Build complex UI from simple, composable primitives.

```tsx
// ❌ Inheritance-style thinking — rigid and hard to extend
class BaseInput extends React.Component {
  render() { return <input /> }
}
class ValidatedInput extends BaseInput {
  // overrides and tightly coupled to parent
}
class SearchInput extends ValidatedInput {
  // even more overrides, fragile hierarchy
}

// ✅ Composition — flexible, reusable building blocks

// Small, focused hooks for shared logic
function useDebounce<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value)
  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay)
    return () => clearTimeout(timer)
  }, [value, delay])
  return debounced
}

function useValidation(value: string, rules: ValidationRule[]) {
  const errors = rules
    .filter((rule) => !rule.test(value))
    .map((rule) => rule.message)
  return { errors, isValid: errors.length === 0 }
}

// Compose behaviors by combining hooks — no class hierarchy needed
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('')
  const debouncedQuery = useDebounce(query, 300)
  const { errors } = useValidation(query, [minLength(2)])

  useEffect(() => {
    if (debouncedQuery && errors.length === 0) {
      onSearch(debouncedQuery)
    }
  }, [debouncedQuery])

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      {errors.map((err) => <span key={err} className="error">{err}</span>)}
    </div>
  )
}
```

### Unidirectional Data Flow

```
Action → Dispatcher → Store → View → Action
```

- Data flows in one direction, making state changes predictable and traceable.
- Parent-to-child data passing via props; child-to-parent communication via events/callbacks.
- Eliminates two-way binding complexity in large applications.

```tsx
// ✅ Unidirectional data flow in React

// Parent owns the state, passes down via props
function ShoppingCart() {
  const [items, setItems] = useState<CartItem[]>([])

  // Action: handler passed down to children
  const removeItem = (id: string) => {
    setItems((prev) => prev.filter((item) => item.id !== id))
  }

  const updateQuantity = (id: string, quantity: number) => {
    setItems((prev) =>
      prev.map((item) => (item.id === id ? { ...item, quantity } : item))
    )
  }

  // Data flows DOWN via props
  return (
    <div>
      <CartItemList
        items={items}
        onRemove={removeItem}
        onUpdateQuantity={updateQuantity}
      />
      <CartSummary total={items.reduce((sum, i) => sum + i.price * i.quantity, 0)} />
    </div>
  )
}

// Child component: receives data and callbacks — never modifies parent state directly
function CartItemList({ items, onRemove, onUpdateQuantity }: CartItemListProps) {
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>
          {item.name} — Qty: {item.quantity}
          <button onClick={() => onUpdateQuantity(item.id, item.quantity + 1)}>+</button>
          <button onClick={() => onRemove(item.id)}>Remove</button>
        </li>
      ))}
    </ul>
  )
}
```

### Principle of Least Privilege (UI Security)

- Components receive only the data and permissions they need.
- Never trust client-side validation alone — always validate on the server.
- Sanitize user-generated content to prevent XSS.
- Avoid exposing sensitive data in client bundles, URLs, or local storage.

```tsx
// ❌ Violates least privilege — component receives entire user including sensitive data
function UserGreeting({ user }: { user: FullUser }) {
  // Component only needs the name, but has access to email, role, tokens...
  return <h1>Hello, {user.name}!</h1>
}

// ✅ Follows least privilege — only receives what it needs
function UserGreeting({ name }: { name: string }) {
  return <h1>Hello, {name}!</h1>
}

// ✅ Permission-based rendering — components only see allowed actions
function AdminPanel() {
  const { permissions } = useAuth()

  return (
    <div>
      <h2>Admin Panel</h2>
      {permissions.includes('users:read') && <UserList />}
      {permissions.includes('users:delete') && <DeleteUsersButton />}
      {permissions.includes('settings:write') && <SettingsEditor />}
    </div>
  )
}

// ✅ Sanitize user-generated content to prevent XSS
import DOMPurify from 'dompurify'

function UserBio({ htmlContent }: { htmlContent: string }) {
  // Never use dangerouslySetInnerHTML with raw user input
  const sanitized = DOMPurify.sanitize(htmlContent)
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />
}
```

### Colocation

- Place related code close together: component, styles, tests, types, stories in the same directory.
- Reduces cognitive overhead and makes refactoring safer.

```tsx
// ✅ Colocated feature directory structure
//
// features/
// └── ProductCard/
//     ├── ProductCard.tsx          ← component
//     ├── ProductCard.module.css   ← styles
//     ├── ProductCard.test.tsx     ← tests
//     ├── ProductCard.stories.tsx  ← storybook
//     ├── useProductCard.ts       ← hook (logic)
//     ├── types.ts                ← types
//     └── index.ts                ← public export

// index.ts — only exports the public API
export { ProductCard } from './ProductCard'
export type { ProductCardProps } from './types'

// useProductCard.ts — colocated hook, related logic stays close
import type { Product } from './types'

export function useProductCard(product: Product) {
  const formattedPrice = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(product.price)

  const isOnSale = product.discount > 0

  return { formattedPrice, isOnSale }
}
```

### Progressive Enhancement & Graceful Degradation

- **Progressive Enhancement**: Start with a baseline HTML experience, layer on CSS and JS.
- **Graceful Degradation**: Build the full experience first, ensure it doesn't break in constrained environments.
- Both strategies ensure maximum accessibility and resilience.

```tsx
// ✅ Progressive Enhancement — works without JS, enhanced with JS

// Base: a standard HTML form that works even if JS fails to load
function SearchForm() {
  const [results, setResults] = useState<Result[] | null>(null)

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    const formData = new FormData(e.currentTarget)
    const query = formData.get('q') as string
    const data = await searchApi.search(query)
    setResults(data)
  }

  return (
    // action/method = works without JS (server handles the form)
    // onSubmit = JS enhancement (client handles it)
    <form action="/search" method="GET" onSubmit={handleSubmit}>
      <input name="q" type="search" placeholder="Search..." required />
      <button type="submit">Search</button>

      {/* JS-enhanced: show results inline instead of full page reload */}
      {results && (
        <ul>
          {results.map((r) => (
            <li key={r.id}>{r.title}</li>
          ))}
        </ul>
      )}
    </form>
  )
}

// ✅ Graceful Degradation — feature detection with fallbacks
function CopyButton({ text }: { text: string }) {
  const [copied, setCopied] = useState(false)

  // Gracefully degrade if clipboard API is unavailable
  if (!navigator.clipboard) {
    return (
      <textarea readOnly value={text} onClick={(e) => (e.target as HTMLTextAreaElement).select()} />
    )
  }

  const handleCopy = async () => {
    await navigator.clipboard.writeText(text)
    setCopied(true)
    setTimeout(() => setCopied(false), 2000)
  }

  return <button onClick={handleCopy}>{copied ? 'Copied!' : 'Copy'}</button>
}
```

---

## 1.4 Functional Programming Principles in the Frontend

| Concept | Application |
|---------|-------------|
| **Pure Functions** | Reducers, selectors, utility functions — same input always produces same output. |
| **Immutability** | Never mutate state directly. Use spread operators, `structuredClone`, or libraries like Immer. |
| **Declarative UI** | Describe *what* the UI should look like, not *how* to manipulate the DOM. |
| **Higher-Order Functions** | HOCs, custom hooks, middleware, pipe/compose utilities. |
| **Referential Transparency** | Components with identical props render identically — enables memoization. |

### Pure Functions — React Example

```tsx
// ❌ Impure reducer — mutates state and uses external variable
let nextId = 1

function todoReducer(state: Todo[], action: TodoAction) {
  if (action.type === 'add') {
    state.push({ id: nextId++, title: action.title, done: false }) // ❌ mutates
    return state
  }
  return state
}

// ✅ Pure reducer — no mutation, no side effects, deterministic
function todoReducer(state: Todo[], action: TodoAction): Todo[] {
  switch (action.type) {
    case 'add':
      return [...state, { id: crypto.randomUUID(), title: action.title, done: false }]
    case 'toggle':
      return state.map((todo) =>
        todo.id === action.id ? { ...todo, done: !todo.done } : todo
      )
    case 'remove':
      return state.filter((todo) => todo.id !== action.id)
    default:
      return state
  }
}

// ✅ Pure selector — derives data without side effects
function selectActiveTodos(todos: Todo[]): Todo[] {
  return todos.filter((todo) => !todo.done)
}

function selectTodoStats(todos: Todo[]) {
  return {
    total: todos.length,
    done: todos.filter((t) => t.done).length,
    remaining: todos.filter((t) => !t.done).length,
  }
}
```

### Immutability — React Example

```tsx
// ❌ Mutation — React won't detect the change, no re-render
function addItem(items: Item[], newItem: Item) {
  items.push(newItem)  // ❌ mutates the original array
  return items         // same reference — React skips re-render
}

function updateUser(user: User) {
  user.name = 'New Name' // ❌ direct mutation
  return user
}

// ✅ Immutable updates — new references trigger re-renders

// Array operations
const addItem = (items: Item[], newItem: Item) => [...items, newItem]
const removeItem = (items: Item[], id: string) => items.filter((i) => i.id !== id)
const updateItem = (items: Item[], id: string, changes: Partial<Item>) =>
  items.map((i) => (i.id === id ? { ...i, ...changes } : i))

// Object updates
const updateUser = (user: User, changes: Partial<User>) => ({ ...user, ...changes })

// Nested updates with spread
const updateAddress = (user: User, city: string) => ({
  ...user,
  address: { ...user.address, city },
})

// Complex state with useReducer + Immer for deeply nested updates
import { useImmerReducer } from 'use-immer'

const [state, dispatch] = useImmerReducer((draft, action) => {
  switch (action.type) {
    case 'updateCity':
      // Immer allows "mutation" syntax on a draft — produces immutable result
      draft.user.address.city = action.city
      break
  }
}, initialState)
```

### Declarative UI — React Example

```tsx
// ❌ Imperative — manually manipulating the DOM
function showError(message: string) {
  const div = document.createElement('div')
  div.className = 'error'
  div.textContent = message
  document.getElementById('errors')?.appendChild(div)
  setTimeout(() => div.remove(), 3000)
}

// ✅ Declarative — describe WHAT the UI should look like based on state
function ErrorMessages({ errors }: { errors: string[] }) {
  return (
    <div id="errors">
      {errors.map((error, i) => (
        <div key={i} className="error">
          {error}
        </div>
      ))}
    </div>
  )
}

// ✅ Declarative conditional rendering
function UserDashboard({ user, isLoading, error }: DashboardProps) {
  if (isLoading) return <Skeleton />
  if (error) return <ErrorMessage message={error} />
  if (!user) return <EmptyState message="No user found" />

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      {user.isAdmin && <AdminTools />}
    </div>
  )
}
```

### Higher-Order Functions — React Example

```tsx
// ✅ Higher-Order Component (HOC) — adds loading behavior to any component
function withLoading<P extends object>(
  WrappedComponent: React.ComponentType<P>
) {
  return function WithLoading({ isLoading, ...props }: P & { isLoading: boolean }) {
    if (isLoading) return <Spinner />
    return <WrappedComponent {...(props as P)} />
  }
}

const UserListWithLoading = withLoading(UserList)
// Usage: <UserListWithLoading isLoading={loading} users={users} />

// ✅ Compose multiple behaviors
const EnhancedUserList = withLoading(withErrorBoundary(UserList))

// ✅ Higher-order hook pattern — composes data fetching logic
function createResourceHook<T>(fetcher: () => Promise<T>) {
  return function useResource() {
    const [data, setData] = useState<T | null>(null)
    const [loading, setLoading] = useState(true)
    const [error, setError] = useState<Error | null>(null)

    useEffect(() => {
      fetcher()
        .then(setData)
        .catch(setError)
        .finally(() => setLoading(false))
    }, [])

    return { data, loading, error }
  }
}

// Create specific hooks from the factory
const useUsers = createResourceHook(() => api.getUsers())
const useProducts = createResourceHook(() => api.getProducts())
```

### Referential Transparency — React Example

```tsx
// ✅ Referentially transparent component — same props = same output
// This enables React.memo to skip re-renders
const PriceTag = React.memo(function PriceTag({ amount, currency }: {
  amount: number
  currency: string
}) {
  const formatted = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
  }).format(amount)

  return <span className="price">{formatted}</span>
})

// ❌ NOT referentially transparent — output depends on external state
let discountRate = 0.1

function PriceTag({ amount }: { amount: number }) {
  // Same props can produce different output depending on external variable
  return <span>${(amount * (1 - discountRate)).toFixed(2)}</span>
}

// ✅ Make it transparent — pass all dependencies as props
const PriceTag = React.memo(function PriceTag({ amount, discountRate = 0 }: {
  amount: number
  discountRate?: number
}) {
  return <span>${(amount * (1 - discountRate)).toFixed(2)}</span>
})

// Now React.memo works correctly — identical props always mean identical output
```

---

## 1.5 Resilience Principles

- **Fail gracefully**: Use error boundaries and fallback UI.
- **Retry with backoff**: Transient network failures should not crash the app.
- **Timeout management**: Every external call should have a timeout.
- **Circuit breaker (client-side)**: Stop hammering a failing service; surface degraded state.
- **Offline-first mindset**: Leverage service workers, IndexedDB, and optimistic UI.

### Fail Gracefully — React Example

```tsx
// ✅ Error Boundary — catches render errors and shows fallback UI
import { Component, type ErrorInfo, type ReactNode } from 'react'

interface Props {
  fallback: ReactNode
  children: ReactNode
}

interface State {
  hasError: boolean
}

class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false }

  static getDerivedStateFromError(): State {
    return { hasError: true }
  }

  componentDidCatch(error: Error, info: ErrorInfo) {
    console.error('ErrorBoundary caught:', error, info)
  }

  render() {
    if (this.state.hasError) return this.props.fallback
    return this.props.children
  }
}

// Usage — wrap sections so failures don't take down the whole page
function App() {
  return (
    <div>
      <Header />
      <ErrorBoundary fallback={<p>Something went wrong in the dashboard.</p>}>
        <Dashboard />
      </ErrorBoundary>
      <ErrorBoundary fallback={<p>Could not load sidebar.</p>}>
        <Sidebar />
      </ErrorBoundary>
    </div>
  )
}
```

### Retry with Backoff — React Example

```tsx
// ✅ Custom hook with exponential backoff retry
function useRetryFetch<T>(url: string, maxRetries = 3) {
  const [data, setData] = useState<T | null>(null)
  const [error, setError] = useState<Error | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    let cancelled = false

    async function fetchWithRetry(attempt = 0) {
      try {
        const res = await fetch(url)
        if (!res.ok) throw new Error(`HTTP ${res.status}`)
        const json = await res.json()
        if (!cancelled) {
          setData(json)
          setLoading(false)
        }
      } catch (err) {
        if (attempt < maxRetries && !cancelled) {
          const delay = Math.min(1000 * 2 ** attempt, 10000) // exponential backoff, max 10s
          await new Promise((resolve) => setTimeout(resolve, delay))
          return fetchWithRetry(attempt + 1)
        }
        if (!cancelled) {
          setError(err as Error)
          setLoading(false)
        }
      }
    }

    fetchWithRetry()
    return () => { cancelled = true }
  }, [url, maxRetries])

  return { data, error, loading }
}

// Usage
function UserProfile({ userId }: { userId: string }) {
  const { data: user, error, loading } = useRetryFetch<User>(`/api/users/${userId}`)

  if (loading) return <Spinner />
  if (error) return <ErrorMessage message="Failed after multiple attempts" />
  return <ProfileCard user={user!} />
}
```

### Timeout Management — React Example

```tsx
// ✅ Fetch with timeout using AbortController
function fetchWithTimeout(url: string, timeoutMs = 5000): Promise<Response> {
  const controller = new AbortController()
  const timeoutId = setTimeout(() => controller.abort(), timeoutMs)

  return fetch(url, { signal: controller.signal }).finally(() => {
    clearTimeout(timeoutId)
  })
}

// ✅ Custom hook with timeout support
function useTimedFetch<T>(url: string, timeoutMs = 5000) {
  const [data, setData] = useState<T | null>(null)
  const [error, setError] = useState<string | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    const controller = new AbortController()
    const timeoutId = setTimeout(() => controller.abort(), timeoutMs)

    fetch(url, { signal: controller.signal })
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`)
        return res.json()
      })
      .then(setData)
      .catch((err) => {
        if (err.name === 'AbortError') {
          setError('Request timed out. Please try again.')
        } else {
          setError(err.message)
        }
      })
      .finally(() => {
        clearTimeout(timeoutId)
        setLoading(false)
      })

    return () => {
      controller.abort()
      clearTimeout(timeoutId)
    }
  }, [url, timeoutMs])

  return { data, error, loading }
}
```

### Circuit Breaker — React Example

```tsx
// ✅ Simple client-side circuit breaker
class CircuitBreaker {
  private failures = 0
  private lastFailureTime = 0
  private state: 'closed' | 'open' | 'half-open' = 'closed'

  constructor(
    private threshold: number = 3,
    private resetTimeMs: number = 30000
  ) {}

  async call<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'open') {
      if (Date.now() - this.lastFailureTime > this.resetTimeMs) {
        this.state = 'half-open'
      } else {
        throw new Error('Circuit is open — service unavailable')
      }
    }

    try {
      const result = await fn()
      this.reset()
      return result
    } catch (error) {
      this.recordFailure()
      throw error
    }
  }

  private recordFailure() {
    this.failures++
    this.lastFailureTime = Date.now()
    if (this.failures >= this.threshold) {
      this.state = 'open'
    }
  }

  private reset() {
    this.failures = 0
    this.state = 'closed'
  }
}

// Usage in a hook
const searchBreaker = new CircuitBreaker(3, 30000)

function useSearch(query: string) {
  const [results, setResults] = useState<SearchResult[]>([])
  const [degraded, setDegraded] = useState(false)

  useEffect(() => {
    if (!query) return

    searchBreaker
      .call(() => searchApi.search(query))
      .then(setResults)
      .catch(() => {
        setDegraded(true) // show degraded UI instead of crashing
      })
  }, [query])

  return { results, degraded }
}
```

### Offline-First & Optimistic UI — React Example

```tsx
// ✅ Optimistic UI — update UI immediately, sync with server in background
function useTodos() {
  const [todos, setTodos] = useState<Todo[]>([])

  const addTodo = async (title: string) => {
    // Optimistic: add to UI immediately with a temporary ID
    const optimisticTodo: Todo = {
      id: `temp-${Date.now()}`,
      title,
      done: false,
    }
    setTodos((prev) => [...prev, optimisticTodo])

    try {
      // Sync with server
      const savedTodo = await todoApi.create(title)
      // Replace optimistic entry with real server response
      setTodos((prev) =>
        prev.map((t) => (t.id === optimisticTodo.id ? savedTodo : t))
      )
    } catch {
      // Rollback on failure
      setTodos((prev) => prev.filter((t) => t.id !== optimisticTodo.id))
      toast.error('Failed to add todo. Please try again.')
    }
  }

  const toggleTodo = async (id: string) => {
    // Optimistic toggle
    setTodos((prev) =>
      prev.map((t) => (t.id === id ? { ...t, done: !t.done } : t))
    )

    try {
      await todoApi.toggle(id)
    } catch {
      // Rollback: toggle back
      setTodos((prev) =>
        prev.map((t) => (t.id === id ? { ...t, done: !t.done } : t))
      )
      toast.error('Failed to update. Please try again.')
    }
  }

  return { todos, addTodo, toggleTodo }
}
```

---

Next: [Pattern Catalog →](02-pattern-catalog.md)
