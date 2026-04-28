# Article 4: Dispatching Actions from Components

Now your Redux setup is connected to React.

In this article, you use it to send real cart actions from UI events.

Focus files:

- `src/components/Product.tsx`
- `src/components/CartItems.tsx`

## Step 1: Dispatch from `Product.tsx`

Current behavior:

```tsx
const dispatch = useCartDispatch();

function handleAddToCart() {
  dispatch(addToCart({ id, title, price }));
}
```

Flow:

1. user clicks Add to Cart
2. component dispatches `addToCart` action with payload
3. reducer in `cart-slice.ts` updates state

Why payload contains only `id`, `title`, `price`:

- reducer is responsible for quantity logic
- first add creates `quantity: 1`

## Step 2: Dispatch plus/minus in `CartItems.tsx`

Current handlers:

```tsx
function handleAddToCart(item: CartItem) {
  dispatch(addToCart(item));
}

function handleRemoveFromCart(id: string) {
  dispatch(removeFromCart(id));
}
```

How these map to reducer behavior:

- plus button -> `addToCart` -> increment existing quantity or insert item
- minus button -> `removeFromCart` -> decrement quantity or remove item entirely

## Step 3: Connect dispatch to button events

Current wiring:

```tsx
<button onClick={() => handleRemoveFromCart(item.id)}>-</button>
<button onClick={() => handleAddToCart(item)}>+</button>
```

This is where user interaction enters Redux flow.

## Step 4: Why this design is beginner-friendly

- UI component handles user events
- Action creators describe intent
- Reducers contain state-transition rules
- Store remains the central source of truth

This separation is the main reason Redux scales better than ad-hoc shared state.

## Covered so far

- dispatch from `Product.tsx`
- dispatch from `CartItems.tsx`
- payload flow from UI to reducers
- mapping button events to state transitions

## Next article

In Article 5, you will finish the learning path with read-side logic:

- selector usage in Header and CartItems
- `reduce` for derived values (item count, total price)
- complete end-to-end Redux update diagram