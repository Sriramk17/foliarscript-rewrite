# Pattern: Supabase Queries

## Server Component Queries (Next.js)

```tsx
// app/dashboard/scripts/page.tsx
import { createClient } from '@/lib/supabase/server'

export default async function ScriptsPage() {
  const supabase = await createClient()

  const { data: scripts, error } = await supabase
    .from('scripts')
    .select(`
      *,
      client:clients(name),
      field:fields(name)
    `)
    .order('created_at', { ascending: false })

  if (error) {
    console.error('Error fetching scripts:', error)
    return <div>Error loading scripts</div>
  }

  return (
    <div>
      {scripts.map((script) => (
        <div key={script.id}>
          {script.sample_id} - {script.client?.name}
        </div>
      ))}
    </div>
  )
}
```

## Client Component Queries

```tsx
'use client'

import { useEffect, useState } from 'react'
import { createClient } from '@/lib/supabase/client'

export function ScriptsList() {
  const [scripts, setScripts] = useState([])
  const [loading, setLoading] = useState(true)
  const supabase = createClient()

  useEffect(() => {
    async function fetchScripts() {
      const { data, error } = await supabase
        .from('scripts')
        .select('*')
        .order('created_at', { ascending: false })

      if (!error) {
        setScripts(data || [])
      }
      setLoading(false)
    }
    fetchScripts()
  }, [])

  if (loading) return <div>Loading...</div>

  return (
    <div>
      {scripts.map((script) => (
        <div key={script.id}>{script.sample_id}</div>
      ))}
    </div>
  )
}
```

## Common Query Patterns

### Select with Relations

```typescript
// Get scripts with related client and field
const { data } = await supabase
  .from('scripts')
  .select(`
    *,
    client:clients(id, name),
    field:fields(id, name),
    recommendation:recommendations(*)
  `)
```

### Filtering

```typescript
// Filter by status
const { data } = await supabase
  .from('scripts')
  .select('*')
  .eq('status', 'paid')

// Multiple filters
const { data } = await supabase
  .from('scripts')
  .select('*')
  .eq('crop', 'corn')
  .gte('test_date', '2024-01-01')
  .order('test_date', { ascending: false })

// Filter by field ID
const { data } = await supabase
  .from('scripts')
  .select('*')
  .eq('field_id', fieldId)
```

### Pagination

```typescript
const page = 1
const pageSize = 10

const { data, count } = await supabase
  .from('scripts')
  .select('*', { count: 'exact' })
  .order('created_at', { ascending: false })
  .range((page - 1) * pageSize, page * pageSize - 1)
```

### Upsert

```typescript
// Insert or update (for fields with unique constraint)
const { data } = await supabase
  .from('fields')
  .upsert({ user_id: userId, name: fieldName })
  .select()
```

### Single Record

```typescript
// Get single record (throws if not found)
const { data, error } = await supabase
  .from('scripts')
  .select('*')
  .eq('id', scriptId)
  .single()
```

### Count Only

```typescript
const { count } = await supabase
  .from('scripts')
  .select('*', { count: 'exact', head: true })
  .eq('status', 'unpaid')
```

## Real-time Subscriptions

```tsx
'use client'

import { useEffect, useState } from 'react'
import { createClient } from '@/lib/supabase/client'

export function RealtimeScripts() {
  const [scripts, setScripts] = useState([])
  const supabase = createClient()

  useEffect(() => {
    // Initial fetch
    supabase
      .from('scripts')
      .select('*')
      .then(({ data }) => setScripts(data || []))

    // Subscribe to changes
    const channel = supabase
      .channel('scripts-changes')
      .on(
        'postgres_changes',
        {
          event: '*',
          schema: 'public',
          table: 'scripts',
        },
        (payload) => {
          if (payload.eventType === 'INSERT') {
            setScripts((prev) => [payload.new, ...prev])
          } else if (payload.eventType === 'UPDATE') {
            setScripts((prev) =>
              prev.map((s) => (s.id === payload.new.id ? payload.new : s))
            )
          } else if (payload.eventType === 'DELETE') {
            setScripts((prev) =>
              prev.filter((s) => s.id !== payload.old.id)
            )
          }
        }
      )
      .subscribe()

    return () => {
      supabase.removeChannel(channel)
    }
  }, [])

  return (
    <div>
      {scripts.map((script) => (
        <div key={script.id}>{script.sample_id}</div>
      ))}
    </div>
  )
}
```

## Storage Operations

```typescript
// Upload file
const { data, error } = await supabase.storage
  .from('uploads')
  .upload(`${userId}/${filename}`, file, {
    cacheControl: '3600',
    upsert: false,
  })

// Get signed URL (for private buckets)
const { data } = await supabase.storage
  .from('uploads')
  .createSignedUrl(`${userId}/${filename}`, 3600) // 1 hour expiry

// Download file
const { data } = await supabase.storage
  .from('uploads')
  .download(`${userId}/${filename}`)

// Delete file
const { error } = await supabase.storage
  .from('uploads')
  .remove([`${userId}/${filename}`])
```

## Error Handling

```typescript
const { data, error } = await supabase
  .from('scripts')
  .select('*')

if (error) {
  // Supabase error object
  console.error('Code:', error.code)
  console.error('Message:', error.message)
  console.error('Details:', error.details)

  // Common error codes:
  // PGRST116 - No rows returned (single() on empty result)
  // 23505 - Unique constraint violation
  // 42501 - RLS policy violation

  if (error.code === '23505') {
    return { error: 'This record already exists' }
  }
}
```
