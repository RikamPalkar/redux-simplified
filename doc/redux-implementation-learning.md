# Redux Implementation: Learn as We Build

> Note: This long-form guide has been split into a beginner-friendly 5-part series.
> Start here: [Redux + TypeScript Learning Series](./redux-series.md)

This article is for beginners who want to build Redux step by step, not by copying a huge block of code.

We will start from the **current snapshot** in this project and understand why each file exists, why it is named this way, and what each line is doing.

In this first part, we only cover the new `store` folder and the two files inside it.

## Why do we create a `store` folder?

In Redux, the store is the central place that holds shared state.

So we usually keep Redux-related files together in one folder named `store/`.

That gives us:

- one clear place for Redux setup
- easier project structure
- easier scaling when more slices are added later

If the app grows, we can add more files here, such as:

- `user-slice.ts`
- `ui-slice.ts`
- `wishlist-slice.ts`

This naming makes it obvious that those files belong to Redux state management.

## Current snapshot: files inside `src/store/`

Right now, the folder has these files:

- `store.ts`
- `cart-slice.ts`
- `hooks.ts`

Let us understand each one.

## File 1: `store.ts`

Current snapshot:

```ts
import { configureStore } from "@reduxjs/toolkit";
import { cartSlice } from "./cart-slice";

export const store = configureStore({
  reducer: {
    cart: cartSlice.reducer
  },
});

const name = "Rikam";
type  NameType = typeof name; // Name is of type string

export type AppDispatch = typeof store.dispatch;
```

### Line-by-line explanation

```ts
import { configureStore } from "@reduxjs/toolkit";
```

- We import `configureStore` from Redux Toolkit.
- `configureStore` is the modern way to create the Redux store.
- It gives good defaults and makes setup easier than old Redux patterns.

```ts
import { cartSlice } from "./cart-slice";
```

- This imports the cart slice definition you created in `cart-slice.ts`.
- We need this because the store must know which reducer handles cart state.

```ts
export const store = configureStore({
  reducer: {
    cart: cartSlice.reducer
  },
});
```

- `configureStore(...)` creates the Redux store.
- `export const store` makes the store available to other files (like React root or App).
- `reducer` is an object map.
- `cart: cartSlice.reducer` means:
  the `cart` key in global state is managed by `cartSlice.reducer`.

So your Redux state shape now starts like this:

```ts
{
  cart: {
    items: []
  }
}
```

```ts
const name = "Rikam";
type NameType = typeof name;
```

- This is a TypeScript learning example of `typeof` in type position.
- `name` is a value (`"Rikam"`).
- `typeof name` means: get the TypeScript type of that value.
- So `NameType` becomes `string`.

This helps beginners understand how TypeScript can infer types from existing values.

```ts
export type AppDispatch = typeof store.dispatch;
```

- This creates a reusable dispatch type from your real store.
- `store.dispatch` is a function.
- `typeof store.dispatch` captures that exact function type.
- Exporting `AppDispatch` helps create typed hooks later.

```ts
export type RootState = ReturnType<typeof store.getState>;
```

- This creates a type that represents your entire Redux state.
- `store.getState()` is a function that returns the current state.
- `typeof store.getState` gets that function's type.
- `ReturnType<...>` means: extract what that function returns.
- So `RootState` becomes the exact shape of your store state.

Currently, that shape is:

```ts
{
  cart: {
    items: [...cart items...]
  }
}
```

As your app grows with more slices (user, UI, etc.), `RootState` automatically includes all of them.

Why export it?

- Components need to know state shape when reading data
- Exporting `RootState` keeps one single source of truth
- Avoids duplicating or guessing the state shape elsewhere

### Why this file is named `store.ts`

The file creates and configures the Redux store, so `store.ts` is a direct and clear name.

When another developer opens this folder, they can quickly guess:

"This is where the main Redux store setup is done."

## File 2: `cart-slice.ts`

Current snapshot:

```ts
import { createSlice, PayloadAction } from "@reduxjs/toolkit";

type CartItem = {
    id: string;
    title: string;
    price: number;
    quantity: number;
}

type CartState = {
    items: CartItem[];
}
const initialState: CartState = {
    items: [],
};

export const cartSlice = createSlice({
  name: "cart",
  initialState: initialState,
  reducers: {
  addToCart(state, action : PayloadAction<{id: string, title: string, price: number;}>) {
    const itemIndex = state.items.findIndex((item) => item.id === action.payload.id);
    if(itemIndex >= 0) {
      state.items[itemIndex].quantity += 1; // you can directly mutate the state in createSlice because it uses Immer under the hood
    } else {
      state.items.push({ ...action.payload, quantity: 1 });
    }
  },
  removeFromCart(state, action : PayloadAction<string>) {
    const itemIndex = state.items.findIndex((item) => item.id === action.payload);

    if(state.items[itemIndex].quantity === 1) {
      state.items.splice(itemIndex, 1);
    }
    else {
      state.items[itemIndex].quantity--;
    }
  },
  },
});

export const { addToCart, removeFromCart } = cartSlice.actions;
```

This version now has real business logic for adding and removing cart items. That is a major step because your reducers now do useful work, not just placeholders.

### Line-by-line explanation

```ts
import { createSlice, PayloadAction } from "@reduxjs/toolkit";
```

- `createSlice` helps us define one feature state (here: cart).
- It combines state setup + reducer logic in one place.
- `PayloadAction` is a TypeScript helper type from Redux Toolkit.
- It describes the shape of data sent with an action.

```ts
type CartItem = {
  id: string;
  title: string;
  price: number;
  quantity: number;
}
```

- This defines the shape of one cart item.
- `id` identifies the product.
- `title` is the product name.
- `price` stores item price.
- `quantity` tracks how many units are in cart.
- This gives TypeScript safety so each item follows the same structure.

```ts
type CartState = {
  items: CartItem[];
}
```

- This defines the overall cart slice state.
- Right now, the cart state has one field: `items`.
- `items` is an array of `CartItem`.

```ts
const initialState: CartState = {
  items: [],
};
```

- This is the starting cart state when the app first loads.
- It is an empty array, which means no products are in cart initially.
- Typing it as `CartState` ensures the state matches the shape you defined.

```ts
export const cartSlice = createSlice({
```

- We create a slice object named `cartSlice`.
- `export` is used so this slice can be imported in other files.

```ts
  name: "cart",
```

- This is the slice name.
- Redux uses this name when generating action type strings (like `cart/addItem`).

```ts
  initialState: initialState,
```

- This connects the `initialState` object into the slice.
- It is equivalent to writing the object directly, but cleaner when state grows.

```ts
  reducers: {
    addToCart(state, action : PayloadAction<{id: string, title: string, price: number;}>) {
      const itemIndex = state.items.findIndex((item) => item.id === action.payload.id);
      if(itemIndex >= 0) {
        state.items[itemIndex].quantity += 1;
      } else {
        state.items.push({ ...action.payload, quantity: 1 });
      }
    },
    removeFromCart(state, action : PayloadAction<string>) {
      const itemIndex = state.items.findIndex((item) => item.id === action.payload);

      if(state.items[itemIndex].quantity === 1) {
        state.items.splice(itemIndex, 1);
      }
      else {
        state.items[itemIndex].quantity--;
      }
    },
  },
```

- `reducers` is where state-changing logic lives.
- `addToCart` and `removeFromCart` are action handlers.
- `state` is the current cart state managed by this slice.
- `action` contains metadata and payload data for what happened.
- `PayloadAction<{ id, title, price }>` means `addToCart` expects those three values in payload.
- `PayloadAction<string>` in `removeFromCart` means payload should be only an item id.

## Deep dive: addToCart logic (beginner friendly)

Reducer code:

```ts
addToCart(state, action : PayloadAction<{id: string, title: string, price: number;}>) {
  const itemIndex = state.items.findIndex((item) => item.id === action.payload.id);
  if(itemIndex >= 0) {
    state.items[itemIndex].quantity += 1;
  } else {
    state.items.push({ ...action.payload, quantity: 1 });
  }
}
```

Let us break this into very small pieces.

### 1) Function parameters

- `state`: current cart state (`{ items: [...] }`)
- `action`: action object with payload data

Payload type here says we must receive:

- `id`
- `title`
- `price`

### 2) `findIndex` explanation

This line:

```ts
const itemIndex = state.items.findIndex((item) => item.id === action.payload.id);
```

`findIndex` is a JavaScript array method.

- It checks each item in the array.
- It returns the position (index) of the first matching item.
- If nothing matches, it returns `-1`.

So here it means:

- look for an existing cart item whose `id` equals the incoming product id.

### 3) `if (itemIndex >= 0)`

If index is `0` or more, item already exists in cart.

Then this line runs:

```ts
state.items[itemIndex].quantity += 1;
```

It increases quantity by 1.

### 4) `else` block and adding a new item

If index is `-1`, item is not in cart yet.

Then this runs:

```ts
state.items.push({ ...action.payload, quantity: 1 });
```

`push` adds a new item to the end of the array.

`...action.payload` means:

- copy all payload fields (`id`, `title`, `price`)

Then we also add:

- `quantity: 1`

So the first time an item is added, quantity starts at 1.

## Deep dive: removeFromCart logic

Reducer code:

```ts
removeFromCart(state, action : PayloadAction<string>) {
  const itemIndex = state.items.findIndex((item) => item.id === action.payload);

  if(state.items[itemIndex].quantity === 1) {
    state.items.splice(itemIndex, 1);
  }
  else {
    state.items[itemIndex].quantity--;
  }
}
```

### 1) Payload type

`PayloadAction<string>` means this action only expects one string value.

In this case, that string is the product id to remove.

### 2) Find target item

Again you use `findIndex` to locate the item in cart by id.

### 3) Two remove cases

Case A: quantity is exactly 1

```ts
state.items.splice(itemIndex, 1);
```

- `splice(index, 1)` removes one item from array at that index.
- So if last quantity is being removed, item disappears from cart list.

Case B: quantity is more than 1

```ts
state.items[itemIndex].quantity--;
```

- `--` means decrease by 1.
- Item stays in cart, only quantity becomes smaller.

## Important beginner question: why are we directly updating state?

In classic Redux, direct mutation is not allowed.

But in Redux Toolkit `createSlice`, this is allowed because of **Immer**.

Immer lets you write code that looks like mutation:

- `state.items.push(...)`
- `state.items[itemIndex].quantity += 1`

Under the hood, Immer creates a safe immutable update for Redux.

So your code stays easy to read, and Redux still gets immutable state updates.

## Beginner syntax notes from this reducer

- `===` means strict equality check.
- `>= 0` means valid array index found.
- `findIndex` returns index or `-1`.
- `push` appends a new array item.
- `splice(index, 1)` removes one array item.
- `...obj` spreads object properties into a new object.
- `+= 1` increments by one.
- `--` decrements by one.

## Small safety note you should add next

Right now `removeFromCart` assumes the item exists.

In real projects, we usually add a guard like "if item index is -1, return" to avoid errors if a wrong id is dispatched.

That is a good next beginner improvement.

```ts
});
```

- This closes `createSlice` and completes the slice definition.

```ts
export const { addToCart, removeFromCart } = cartSlice.actions;
```

- `cartSlice.actions` is an object automatically generated by `createSlice`.
- It contains action creator functions for every reducer name.
- Because your reducers are `addToCart` and `removeFromCart`, action creators with same names are generated.
- This line uses object destructuring to pull those two functions out.
- Then it exports them so components can import and dispatch them.

Think of this line as:

- "give me the generated action creator functions from this slice, and export them for use in UI files"

When you call `addToCart({ id, title, price })`, Redux Toolkit builds an action object like:

```ts
{
  type: "cart/addToCart",
  payload: { id, title, price }
}
```

That action object is what gets dispatched.

## File 3: `hooks.ts`

Current snapshot:

```ts
import { useDispatch } from "react-redux";
import { AppDispatch } from "./store";

type DispatchFunc = () => AppDispatch;
export const useCartDispatch: DispatchFunc = useDispatch;
```

### What is a custom hook?

A custom hook is a function you create (usually starting with `use`) to reuse React hook logic.

In plain words:

- built-in hook: `useDispatch`, `useSelector`, `useState`
- custom hook: your own wrapper like `useCartDispatch`

### Why create a custom hook here?

Purpose:

- keep dispatch typing in one place
- avoid repeating type code in many components
- make component code cleaner and easier to read

### Line-by-line explanation

```ts
import { useDispatch } from "react-redux";
```

- Imports React-Redux dispatch hook.
- `useDispatch()` gives access to store dispatch function.

```ts
import { AppDispatch } from "./store";
```

- Imports the dispatch type derived from your store.
- This keeps dispatch strongly typed.

```ts
type DispatchFunc = () => AppDispatch;
```

- Defines a function type alias.
- It means: a function with no arguments that returns `AppDispatch`.

```ts
export const useCartDispatch: DispatchFunc = useDispatch;
```

```ts
export const useCartDispatch: DispatchFunc = useDispatch;
export const useCartSelector: TypedUseSelectorHook<RootState> = useSelector;
```

- Exports your custom typed dispatch hook.
- You can now call `useCartDispatch()` inside components.
- It behaves like `useDispatch`, but with your app's dispatch type.
- Also exports a custom typed selector hook for reading state.

### Understanding useCartSelector and RootState

This is where "reading" from Redux comes in. Until now, you learned about dispatching (writing).

**Dispatching** = sending actions to change state (like addToCart)

**Selecting** = reading state values from the store (like getting cart items to display)

```ts
import { useDispatch, useSelector, type TypedUseSelectorHook} from "react-redux";
import { AppDispatch, RootState } from "./store";

type DispatchFunc = () => AppDispatch;
export const useCartDispatch: DispatchFunc = useDispatch;
export const useCartSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### Line-by-line explanation

```ts
import { useSelector, type TypedUseSelectorHook} from "react-redux";
```

- `useSelector` is React-Redux hook for reading Redux state.
- `TypedUseSelectorHook` is a TypeScript type helper.
- This type helps make selectors type-safe.

```ts
import { AppDispatch, RootState } from "./store";
```

- Imports dispatch type and state type from store.
- `RootState` tells components the exact shape of Redux state.

```ts
export const useCartSelector: TypedUseSelectorHook<RootState> = useSelector;
```

- Creates a custom selector hook specific to your app.
- `TypedUseSelectorHook<RootState>` wraps `useSelector` with your state type.
- This means TypeScript knows about your entire state.
- So when you use it, TypeScript catches wrong state paths.

### How useCartSelector is used in components

Inside a component, you would use it like:

```tsx
const cartItems = useCartSelector((state) => state.cart.items);
```

- `useCartSelector` takes a function called a selector.
- The selector receives the entire `state` as parameter.
- It returns a piece of that state (here: just the items array).
- Component re-renders whenever that specific piece changes.

Why is this typed?

- TypeScript knows `state` shape is `RootState`.
- So it suggests `state.cart` and knows it has `items`.
- If you try `state.notexist`, TypeScript catches the error.
- This prevents bugs where you read from wrong state path.

## Connecting it all: dispatch and select pattern

Now you have both tools:

- `useCartDispatch()` to send actions
- `useCartSelector(...)` to read state

Together they form the complete Redux flow:

1. Component reads state with `useCartSelector`
2. User clicks button
3. Component dispatches action with `useCartDispatch`
4. Reducer updates state
5. `useCartSelector` detects change and re-renders component

This is the core of Redux in React.

## Understanding the Complete Redux Flow: Header.tsx Example

Now let's look at a real example from your Header component and understand the **entire data flow** from dispatch to UI update.

Here's the line from Header.tsx:

```tsx
const cartItems = useCartSelector((state) => state.cart.items.reduce((total, item) => total + item.quantity, 0));
```

This looks complicated, but it's really just 3 steps combined. Let's break it down piece by piece.

### Step 1: Understanding `reduce()`

The `reduce()` method is a JavaScript array function that **combines all items in an array into a single value**.

Imagine you have a list of numbers and want to find the total:

```ts
const numbers = [1, 2, 3, 4];
const total = numbers.reduce((sum, num) => sum + num, 0);
// Result: 10
```

Here's how `reduce()` works step-by-step:

```
Step 1: sum = 0 (starting value),  num = 1  →  sum = 0 + 1 = 1
Step 2: sum = 1 (from previous),   num = 2  →  sum = 1 + 2 = 3
Step 3: sum = 3 (from previous),   num = 3  →  sum = 3 + 3 = 6
Step 4: sum = 6 (from previous),   num = 4  →  sum = 6 + 4 = 10
Final: 10
```

The `0` at the end is the **starting value**.

### Step 2: Applying reduce to cart items

In your Header, you're doing the same thing but with cart items:

```tsx
state.cart.items.reduce((total, item) => total + item.quantity, 0)
```

- `state.cart.items` is an array of items: `[{id, title, price, quantity: 2}, {id, title, price, quantity: 1}]`
- Instead of adding numbers, you're adding **quantities**
- `total` starts at `0`
- For each `item`, you add its `quantity` to `total`

So if you have 2 items (one with quantity 2, one with quantity 1):

```
Step 1: total = 0,  item.quantity = 2  →  total = 0 + 2 = 2
Step 2: total = 2,  item.quantity = 1  →  total = 2 + 1 = 3
Final: 3 (total items in cart)
```

### Step 3: Using useCartSelector to read the result

```tsx
const cartItems = useCartSelector((state) => state.cart.items.reduce((total, item) => total + item.quantity, 0));
```

Breaking this down:

- `useCartSelector` reads from Redux store
- The selector function receives the entire `state`
- It calculates the total quantity using `reduce()`
- It returns just the number (not the whole state)
- `cartItems` now contains that number: `3`

### Why calculate it this way?

You could store `totalQuantity` in Redux, but calculating it on-the-fly is better because:

1. **Single source of truth**: Only store individual items, not a duplicate count
2. **Automatic accuracy**: Total is always correct (calculated from current items)
3. **Less code**: No need to update count every time an item quantity changes

---

## The Complete Data Flow: From Product Click to Header Update

Now let's visualize the **complete flow** from when a user clicks "Add to Cart" to when the Header shows the new count.

Here's what happens:

### 1. User clicks "Add to Cart" button in Product component

```tsx
// Product.tsx
function handleAddToCart() {
  dispatch(addToCart({ id, title, price }));
  //  ↑ Action is dispatched to Redux store
}
```

### 2. Redux Reducer processes the action

```ts
// cart-slice.ts
addToCart: (state, action) => {
  const item = state.items.find(item => item.id === action.payload.id);
  if (item) {
    item.quantity += 1;  // ← State is updated here
  } else {
    state.items.push({ ...action.payload, quantity: 1 });
  }
}
```

### 3. Redux Store notifies all subscribers

The store says: "State changed! Anyone listening, here's the new state."

### 4. Header component's `useCartSelector` detects the change

```tsx
const cartItems = useCartSelector((state) => state.cart.items.reduce(...));
```

- Redux calls the selector function
- Selector runs `reduce()` with the **new** state
- Returns the new total: `3` items
- Component re-renders with the new number

### 5. UI updates automatically

```tsx
<button onClick={handleOpenCartClick}>Cart ({cartItems})</button>
// Now shows: Cart (3)
```

---

## Visual Diagram: Complete Redux Flow

Here's how data flows through your entire app:

```
┌─────────────────────────────────────────────────────────────┐
│                    USER INTERACTION                          │
│          User clicks "Add to Cart" button                    │
└───────────────────┬─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────┐
│              PRODUCT COMPONENT                               │
│  handleAddToCart() calls:                                   │
│  dispatch(addToCart({ id, title, price }))                 │
└───────────────────┬─────────────────────────────────────────┘
                    │
                    ▼  (Action: {type: 'cart/addToCart', payload: {...}})
┌─────────────────────────────────────────────────────────────┐
│              REDUX REDUCER                                   │
│  cart-slice.ts addToCart reducer:                           │
│  - Finds item in state.items by id                          │
│  - Increments quantity OR                                   │
│  - Adds new item with quantity 1                            │
└───────────────────┬─────────────────────────────────────────┘
                    │
                    ▼  (New State stored in store)
┌─────────────────────────────────────────────────────────────┐
│              REDUX STORE                                     │
│  store = {                                                   │
│    cart: {                                                   │
│      items: [                                                │
│        {id: 1, title: 'Shirt', price: 29, quantity: 2},    │
│        {id: 2, title: 'Jeans', price: 49, quantity: 1}     │
│      ]                                                       │
│    }                                                         │
│  }                                                           │
└───────────────────┬─────────────────────────────────────────┘
                    │
                    ▼  (Store notifies subscribers)
┌─────────────────────────────────────────────────────────────┐
│          HEADER COMPONENT (Reading State)                    │
│  useCartSelector runs selector:                             │
│  (state) => state.cart.items.reduce((total, item) =>       │
│     total + item.quantity, 0)                               │
│                                                              │
│  Calculation:                                                │
│  - Item 1: quantity = 2                                     │
│  - Item 2: quantity = 1                                     │
│  - Result: 2 + 1 = 3                                        │
└───────────────────┬─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────┐
│              UI UPDATES                                      │
│  cartItems = 3                                              │
│  Display: <button>Cart (3)</button>                         │
│  ✓ Header shows new count automatically                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Key Insights

**What makes this work:**

1. **Unidirectional Data Flow**: Data only flows one direction (down from store to components)
   - Components dispatch actions
   - Reducer updates store
   - Store notifies components
   - Components re-render
   - Never the other way around

2. **Automatic Re-renders**: You don't manually update the UI
   - `useCartSelector` watches the selected state
   - When it changes, component automatically re-renders
   - All components using that selector update instantly

3. **Separation of Concerns**:
   - Product.tsx only knows how to dispatch actions
   - Header.tsx only knows how to read state
   - Redux Store manages all shared data
   - No need to pass props through multiple components

4. **Why `reduce()` in selector**:
   - Derived state (calculated from base state)
   - Selector only runs when `state.cart.items` changes
   - Result is a single number (total quantity)
   - Header only re-renders if this number changes

---

## Why this file is named `store.ts`

Current snapshot:

```tsx
import { addToCart } from "../store/cart-slice";
import { useCartDispatch } from "../store/hooks";

type ProductProps = {
  id: string;
  image: string;
  title: string;
  price: number;
  description: string;
};

export default function Product({
  id,
  image,
  title,
  price,
  description,
}: ProductProps) {
  const dispatch = useCartDispatch();

  function handleAddToCart() {
    dispatch(addToCart({ id, title, price }));
  }

  return (
    <article className="product">
      <img src={image} alt={title} />
      <div className="product-content">
        <div>
          <h3>{title}</h3>
          <p className="product-price">${price}</p>
          <p>{description}</p>
        </div>
        <p className="product-actions">
          <button onClick={handleAddToCart}>Add to Cart</button>
        </p>
      </div>
    </article>
  );
}
```

### Line-by-line explanation for beginners

```tsx
import { addToCart } from "../store/cart-slice";
```

- Imports the action creator you exported from slice actions.
- This is not the reducer function itself.
- It is a helper that creates the action object to dispatch.

```tsx
import { useCartDispatch } from "../store/hooks";
```

- Imports your custom typed dispatch hook.

```tsx
type ProductProps = { ... }
```

- Defines prop types for the component.
- Ensures each product has id, image, title, price, description.

```tsx
export default function Product({...}: ProductProps) {
```

- React component receives one product's data as props.

```tsx
const dispatch = useCartDispatch();
```

- Gets dispatch function from Redux store.
- Because this is typed, TypeScript helps validate dispatched actions.

```tsx
function handleAddToCart() {
  dispatch(addToCart({ id, title, price }));
}
```

- `addToCart({ id, title, price })` builds action object.
- `dispatch(...)` sends that action to Redux store.
- Store passes action to cart reducer.
- Reducer updates cart state.

```tsx
<button onClick={handleAddToCart}>Add to Cart</button>
```

- Connects click event to the dispatch flow.
- When user clicks button, the add-to-cart action runs.

So the user interaction flow is:

1. click button in `Product`
2. `handleAddToCart` dispatches action
3. `cartSlice` reducer handles action
4. store state updates
5. any subscribed UI can re-render with updated cart state

## Understanding state and action in simple words

Inside a Redux reducer function, you usually see two parameters:

- `state`
- `action`

Think of it like this:

- `state` is your current data
- `action` is the instruction telling Redux what happened and what data came with it

In cart terms:

- `state` might be `{ items: [...] }`
- `action` might say: "user clicked add to cart" plus product details

## What exactly is action?

An action is a plain object. It usually has:

- `type`: a string like `cart/addToCart`
- `payload`: extra data needed by that action

Example mental model:

- type: what happened
- payload: data needed to handle it

So `addToCart` needs product info. That product info is sent in the payload.

## Why use PayloadAction?

When using TypeScript, `PayloadAction<T>` tells TypeScript what payload shape is expected.

That gives you:

- better autocomplete
- compile-time safety
- fewer bugs from wrong payload data

In your code, this intent is:

- addToCart expects payload with id, title, and price

If someone dispatches addToCart with missing fields or wrong types, TypeScript can warn early.

## Why this matters in real apps

Without typed payloads, reducers can receive unexpected data and fail at runtime.

With `PayloadAction`, many mistakes are caught while coding, before users ever run the app.

For beginners, this is one of the biggest advantages of Redux Toolkit with TypeScript.

## Why this file is named `cart-slice.ts`

The name has two useful parts:

- `cart` tells us this slice belongs to cart feature state
- `slice` tells us this is a Redux slice file

Using `feature-slice.ts` naming is a common and helpful convention.

It keeps files readable when the app has many slices.

## Why we do not put all Redux code in one file

A beginner might ask: why not write everything in `store.ts`?

Good question.

We separate files because:

- `store.ts` handles global store setup
- `cart-slice.ts` handles cart-specific state logic

This separation keeps code cleaner and easier to maintain.

## What is the use case of this structure?

For an e-commerce app:

- cart state lives in `cart-slice.ts`
- user state can live in `user-slice.ts`
- UI state can live in `ui-slice.ts`

Then `store.ts` combines all of them.

This structure is easier to grow than one huge state file.

## New step: using Provider in `App.tsx`

Current snapshot:

```tsx
import Header from './components/Header.tsx';
import Shop from './components/Shop.tsx';
import Product from './components/Product.tsx';
import { DUMMY_PRODUCTS } from './dummy-products.ts';
import { Provider } from 'react-redux';
import { store } from  './store/store.ts'

function App() {
  return (
    <>
    <Provider store={store}>
      <Header />
      <Shop>
        {DUMMY_PRODUCTS.map((product) => (
          <li key={product.id}>
            <Product {...product} />
          </li>
        ))}
      </Shop>
      </Provider>
    </>
  );
}
```

### What is `Provider` in simple words?

`Provider` comes from `react-redux`.

Its job is to make the Redux store available to React components below it in the component tree.

Think of it as a bridge between React and Redux.

Without this bridge:

- hooks like `useSelector` and `useDispatch` cannot access the store
- components will not be connected to Redux state

### Why do we pass `store={store}`?

This line:

```tsx
<Provider store={store}>
```

tells React-Redux which Redux store to use.

In your project, that `store` object is the one created in `store.ts` with `configureStore`.

### What needs to be wrapped inside Provider?

Rule of thumb for beginners:

- wrap every component that needs Redux access
- easiest approach is to wrap your whole app UI tree

In your code, `Header`, `Shop`, `Product`, and their children are all inside `Provider`, so all of them can use Redux hooks if needed.

### Do we wrap in `App.tsx` or `main.tsx`?

Both can work.

- Common pattern: wrap in `main.tsx` at the app root
- Your current pattern: wrap in `App.tsx`

Your current setup is valid because the main app content is inside `Provider`.

### What is "store" in general?

The Redux store is the central container that holds your app's shared state.

It does three main things:

- keeps current state
- receives dispatched actions
- runs reducers to produce next state

So when user clicks "Add to cart":

1. component dispatches action
2. store sends action to reducer
3. reducer updates state
4. connected components re-render with new state

That full cycle is the core Redux flow.

## What comes next (without dumping all code at once)

In the next step of implementation, we will do only three focused actions:

1. export actions from `cart-slice.ts` (for dispatching)
2. use `useDispatch` in UI components to trigger add/remove
3. use `useSelector` to read cart data in components like header/cart

We will continue one small step at a time so learning stays clear and practical.