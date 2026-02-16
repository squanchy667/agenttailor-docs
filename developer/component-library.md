# Component Library

## Component Conventions

- Functional components with named exports
- Props interface named `{ComponentName}Props`
- Tailwind CSS for styling
- Handle loading, error, and empty states
- Accessible: aria labels, keyboard navigation, semantic HTML

## Dashboard Components

### UI Primitives (dashboard/src/components/ui/)
- Button, Input, TextArea, Card, Modal, Spinner, Badge
- Toast (notification system), Skeleton (loading placeholders)
- EmptyState, ErrorBanner

### Layout (dashboard/src/components/layout/)
- AppLayout — Sidebar + header + content
- Sidebar — Navigation links
- Header — Branding + user menu

### Feature Components
- **Projects** — ProjectCard, ProjectForm
- **Documents** — DocumentUploader (drag-and-drop), DocumentList, DocumentStatus, ProcessingProgress
- **Tailoring** — TailorForm, ContextViewer, SourceCard, SessionHistory
- **Analytics** — UsageChart, QualityTrend, ProjectStats, PlanUsage
- **Settings** — ApiKeyManager, ContextPreferences, NotificationSettings

## Extension Components

### Side Panel (extension/src/sidepanel/)
- StatusIndicator — Tailoring progress
- ContextPreview — Assembled context display
- SourceList — Source origins with relevance scores
- EditableContext — User-editable context before injection

### Popup (extension/src/popup/)
- ProjectSelector — Active project picker
- QuickSettings — Toggle auto-tailor, web search

## Shared Patterns

### Loading State
```tsx
if (isLoading) return <Skeleton variant="card" />;
```

### Error State
```tsx
if (error) return <ErrorBanner message={error.message} onRetry={refetch} />;
```

### Empty State
```tsx
if (items.length === 0) return <EmptyState message="No projects yet" action={<Button>Create Project</Button>} />;
```
