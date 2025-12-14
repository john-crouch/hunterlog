---
name: pywebview-bridge-etiquette
description: Build desktop apps with pywebview (React/TypeScript frontend + Python backend) focusing on reliable bridge architecture. Use when: (1) Building pywebview desktop applications, (2) Creating/refactoring JS↔Python bridge APIs, (3) Implementing typed API wrappers, (4) Adding endpoints or debugging bridge communication, (5) Working with window.pywebview.api integration.
---

# pywebview Bridge Etiquette

Clean bridge architecture for pywebview desktop apps (React 18 + TypeScript ↔ Python).

## Core Principles

**Architecture**: React/TS frontend ↔ `window.pywebview.api` ↔ Python backend

**Goals**: Clean separation, reliable typed calls, stable API contract

## Hard Rules

* **Never bypass the bridge** - All backend actions through pywebview API
* **Bridge is a public contract** - Changes require compatibility handling
* **Never block UI thread** - All calls async, assume they may be slow

## API Contract & Typing

Define single TypeScript interface for `window.pywebview.api`:

```typescript
interface PyWebViewAPI {
  getUserData(params: FooParams): Promise<FooResult>;
}
```

Create typed wrapper service:

```typescript
class APIService {
  async getUserData(userId: string): Promise<Result<UserData>> {
    const correlationId = uuidv4();
    try {
      const data = await window.pywebview.api.getUserData({ userId, correlationId });
      return { success: true, data };
    } catch (error) {
      return { success: false, error: this.normalizeError(error) };
    }
  }
}
```

Normalize errors to consistent shape:
```typescript
type Result<T> = { success: true; data: T } | { success: false; error: APIError };
interface APIError { code: string; message: string; correlationId?: string; }
```

## Call Patterns

* **Wrap with try/catch** - Return user-friendly errors
* **Add correlation IDs** - `const correlationId = crypto.randomUUID()` for debugging
* **Batch calls** - Prefer `getUserData()` over `getUser()` + `getPermissions()` + `getSettings()`
* **Cache safely** - Use client-side cache with explicit TTL and invalidation

```typescript
// Cache pattern
class DataCache {
  private cache = new Map<string, { data: any; timestamp: number }>();
  private ttl = 5 * 60 * 1000;
  
  get(key: string) {
    const entry = this.cache.get(key);
    if (!entry || Date.now() - entry.timestamp > this.ttl) return null;
    return entry.data;
  }
}
```

## Data Validation

**Client-side** (fail fast):
```typescript
const validation = validatePayload(data);
if (!validation.valid) return { success: false, error: { code: 'VALIDATION_ERROR', message: validation.error } };
```

**Server-side** (source of truth):
```python
class PayloadSchema(Schema):
    name = fields.Str(required=True, validate=validate.Length(min=1))
    age = fields.Int(validate=validate.Range(min=0, max=150))
```

**Schema stability**: Avoid renaming fields. Add backward compatibility when migrating.

## Security

* **No eval** - Never run arbitrary code
* **Sanitize HTML** - Use DOMPurify for user content: `DOMPurify.sanitize(html)`
* **Safe logging** - Never log passwords, tokens, or sensitive data

## UI Integration

**Data layer pattern** - Use hooks/services, never call `window.pywebview.api` directly from components:

```typescript
export function useUserData(userId: string) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;
    apiService.getUserData(userId).then(result => {
      if (cancelled) return;
      if (result.success) setData(result.data);
      else setError(result.error);
      setLoading(false);
    });
    return () => { cancelled = true; };
  }, [userId]);

  return { data, loading, error };
}
```

**Components**:
```typescript
function UserProfile({ userId }) {
  const { data, loading, error } = useUserData(userId);
  if (loading) return <Skeleton />;
  if (error) return <ErrorMessage error={error} />;
  return <div>{data.name}</div>;
}
```

**Loading states**: Use consistent patterns - skeletons for content, spinners for actions, disabled states for forms

## Deliverables

For each bridge change:

1. **Updated TS typings** - Document API version and changes
2. **Python changelog** - List added/deprecated/breaking changes
3. **Test checklist** - Manual verification steps for key workflows

Example changelog:
```markdown
## API v2.1.0
Added: getUserData() - batches user/permissions/settings
Deprecated: getUser(), getPermissions(), getSettings() 
Breaking: None (deprecated methods work until v3.0)
```

## Verification

Before finalizing:
- [ ] All API calls use typed wrappers
- [ ] Error handling is consistent  
- [ ] No components call `window.pywebview.api` directly
- [ ] Validation on both TS and Python sides
- [ ] No sensitive data in logs
- [ ] End-to-end tests pass in packaged app
- [ ] Errors surface clearly without crashing UI

## References

See `references/examples.md` for complete implementation examples including:
- Full TypeScript API layer with types, services, and cache
- React hooks and component patterns
- Python API class with Marshmallow schemas
- Validation, security, and testing examples
