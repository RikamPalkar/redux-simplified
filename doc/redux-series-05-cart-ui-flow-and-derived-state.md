# Article 5: Reading State, Derived Data, and Full Redux Flow

This article closes the Redux + TypeScript implementation path.

You now focus on reading state and deriving UI values safely.

Focus files:

- `src/components/Header.tsx`
- `src/components/CartItems.tsx`

## Step 1: Read cart quantity in Header

Current selector:

```tsx
const cartItems = useCartSelector((state) =>
  state.cart.items.reduce((total, item) => total + item.quantity, 0)
);
```

Meaning:

- read full cart items array from Redux state
- compute one number: total quantity across all items
- render this number in header button

`reduce` logic in plain words:

- start from `0`
- for each item, add `item.quantity`
- final output is total units in cart

## Step 2: Read cart items and total price in CartItems

Current code shape:

```tsx
const cartItems = useCartSelector((state) => state.cart.items);
const totalPrice = cartItems.reduce(
  (total, item) => total + item.price * item.quantity,
  0
);
const formattedTotalPrice = `$${totalPrice.toFixed(2)}`;
```

Why this is good design:

- base state remains minimal (`items` only)
- totals are derived at read time
- less duplication, less risk of inconsistent stored totals

## Step 3: Conditional rendering by selected state

Current UI behavior:

- if `cartItems.length === 0`, show empty-cart message
- otherwise render item list with plus/minus controls

Selector result drives rendering automatically.

## Step 4: Full end-to-end Redux flow in this app

```mermaid
flowchart LR
  A[Click Add or Remove button] --> B[dispatch addToCart/removeFromCart]
  B --> C[cart-slice reducer updates state]
  C --> D[store holds new cart state]
  D --> E[useCartSelector re-runs in Header and CartItems]
  E --> F[derived values recalculate via reduce]
  F --> G[React re-renders UI]
```

This is the exact behavior readers should observe while coding along.

## Step 5: What to improve next (optional)

For production hardening, add a guard in `removeFromCart`:

- if id is not found, return early

This prevents edge-case crashes from unexpected payloads.

## Covered so far

- selecting state with typed selectors
- deriving item count and total price with `reduce`
- conditional rendering from selected state
- complete UI update cycle from dispatch to re-render

## Next article

This is the final implementation article in the 5-part series.

Recommended next step for readers:

- add one new Redux feature (example: wishlist slice) using the same pattern
- repeat the same sequence: slice -> store -> typed hooks -> dispatch -> selectors