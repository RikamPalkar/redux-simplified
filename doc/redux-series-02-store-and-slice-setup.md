# Article 2: Store and Slice Setup (First Working Redux State)

This article builds your first working Redux data layer.

Focus only on:

- `src/store/store.ts`
- `src/store/cart-slice.ts`

## Step 1: Define cart state shape in `cart-slice.ts`

Current core types:

```ts
export type CartItem = {
  id: string;
  title: string;
  price: number;
  quantity: number;
}

type CartState = {
  items: CartItem[];
}
```

Why this matters:

- every cart item must follow one shape
- reducer logic becomes predictable
- TypeScript catches wrong data early

## Step 2: Add initial state

```ts
const initialState: CartState = {
  items: [],
};
```

This means cart starts empty on first app load.

## Step 3: Create slice with reducers

```ts
export const cartSlice = createSlice({
  name: "cart",
  initialState,
  reducers: {
    addToCart(...),
    removeFromCart(...)
  },
});
```

Meaning:

- `name: "cart"` scopes action types (`cart/addToCart`)
- reducers describe state transitions
- Toolkit auto-generates action creators

## Step 4: Implement `addToCart`

Current logic summary:

1. find item index by id
2. if found, increment quantity
3. if not found, push new item with quantity 1

Key line idea:

```ts
state.items.push({ ...action.payload, quantity: 1 });
```

Payload provides `id`, `title`, `price`, and reducer sets first `quantity`.

## Step 5: Implement `removeFromCart`

Current logic summary:

1. find item by id
2. if quantity is 1, remove item from array
3. otherwise decrement quantity

This gives expected cart behavior for minus button clicks.

## Step 6: Export generated action creators

```ts
export const { addToCart, removeFromCart } = cartSlice.actions;
```

Now components can dispatch these actions.

## Step 7: Create global store in `store.ts`

Current setup:

```ts
export const store = configureStore({
  reducer: {
    cart: cartSlice.reducer
  },
});
```

Global state shape now starts as:

```ts
{
  cart: {
    items: []
  }
}
```

## Step 8: Export Redux TypeScript helper types

```ts
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

Why these are important:

- `RootState`: single source of truth for selector state type
- `AppDispatch`: typed dispatch function for action safety

## Practical notes for beginners

- Reducer code looks mutable (`push`, `quantity++`) but Toolkit uses Immer under the hood
- `PayloadAction<T>` defines what payload each reducer accepts
- Store and slice separation keeps architecture scalable

## Covered so far

- cart state types
- initial cart state
- slice creation
- add/remove reducer logic
- action creator exports
- store configuration
- RootState and AppDispatch exports

## Next article

In Article 3, you will connect Redux to React correctly:

- typed custom hooks in `hooks.ts`
- Provider setup in `App.tsx`
- why typed hooks reduce repetition in components