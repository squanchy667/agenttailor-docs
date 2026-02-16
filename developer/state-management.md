# State Management

## Approach

- Zod schemas as single source of truth for data shapes
- Server state managed by React Query
- Client state managed by React state/context
- Form state with controlled components and Zod validation

## Server State

Managed with React Query (@tanstack/react-query):
- Automatic caching and refetching
- Optimistic updates for mutations
- Loading and error states built-in

```tsx
const { data, isLoading, error } = useQuery({
  queryKey: ['resource', id],
  queryFn: () => fetchResource(id),
});
```

## Client State

- React `useState` for local component state
- React Context for shared UI state (theme, sidebar, modals)
- Avoid global state when React Query covers the use case

## Form State

- Controlled form components with Zod schema validation
- Validation on blur and submit
- Error messages derived from Zod parse errors
