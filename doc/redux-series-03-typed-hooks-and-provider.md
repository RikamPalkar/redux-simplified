# Article 3: Typed Hooks and Provider Wiring

In Article 2, you created working Redux state.

In this article, you connect that state to React in a TypeScript-safe way.

Focus files:

- `src/store/hooks.ts`
- `src/App.tsx`

## Step 1: Create typed hooks in `hooks.ts`

Current code:

```ts
import { useDispatch, useSelector, type TypedUseSelectorHook } from "react-redux";
import { AppDispatch, RootState } from "./store";

type DispatchFunc = () => AppDispatch;
export const useCartDispatch: DispatchFunc = useDispatch;
export const useCartSelector: TypedUseSelectorHook<RootState> = useSelector;
```

What this gives you:

- `useCartDispatch()` returns correctly typed dispatch
- `useCartSelector(...)` knows full `RootState` shape

Why this is better than raw hooks everywhere:

- avoids repeated type code in each component
- gives autocomplete for `state.cart...`
- prevents wrong selector paths

## Step 2: Wire Provider in `App.tsx`

Current structure:

```tsx
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
```

Provider role:

- injects Redux store into React tree
- enables hooks (`useCartDispatch`, `useCartSelector`) in child components

Without Provider, Redux hooks cannot access store context.

## Step 3: Understand dispatch vs selector responsibilities

- dispatch hook: write/update state via actions
- selector hook: read state snapshots

Both are needed for complete Redux UI behavior.

## Beginner checklist after this step

You should now be able to say:

- "My app is wrapped with Provider"
- "My components can access Redux through typed hooks"
- "TypeScript validates my state and dispatch usage"

## Covered so far

- typed custom hooks
- RootState-driven typed selectors
- AppDispatch-driven typed dispatch
- Provider wiring in app root tree

## Next article

In Article 4, you will use these hooks in real components to dispatch actions:

- add to cart from `Product.tsx`
- add/remove from `CartItems.tsx`
- action payload flow from UI to reducer