# Pattern: React Components

## Basic Component Structure

```tsx
// components/feature-card.tsx
import { cn } from '@/lib/utils'

interface FeatureCardProps {
  title: string
  description: string
  className?: string
  children?: React.ReactNode
}

export function FeatureCard({
  title,
  description,
  className,
  children,
}: FeatureCardProps) {
  return (
    <div className={cn('rounded-lg border bg-card p-6', className)}>
      <h3 className="font-semibold">{title}</h3>
      <p className="text-sm text-muted-foreground mt-1">{description}</p>
      {children && <div className="mt-4">{children}</div>}
    </div>
  )
}
```

## Component with Loading State

```tsx
// components/data-card.tsx
import { Skeleton } from '@/components/ui/skeleton'

interface DataCardProps {
  title: string
  value: number | string
  loading?: boolean
}

export function DataCard({ title, value, loading }: DataCardProps) {
  return (
    <div className="rounded-lg border bg-card p-6">
      <p className="text-sm text-muted-foreground">{title}</p>
      {loading ? (
        <Skeleton className="h-8 w-20 mt-1" />
      ) : (
        <p className="text-2xl font-bold mt-1">{value}</p>
      )}
    </div>
  )
}
```

## Component with Variants (using CVA)

```tsx
// components/status-badge.tsx
import { cva, type VariantProps } from 'class-variance-authority'
import { cn } from '@/lib/utils'

const badgeVariants = cva(
  'inline-flex items-center rounded-md px-2.5 py-0.5 text-xs font-semibold',
  {
    variants: {
      variant: {
        default: 'bg-primary/10 text-primary',
        success: 'bg-green-500/10 text-green-500',
        warning: 'bg-yellow-500/10 text-yellow-500',
        error: 'bg-red-500/10 text-red-500',
      },
    },
    defaultVariants: {
      variant: 'default',
    },
  }
)

interface StatusBadgeProps extends VariantProps<typeof badgeVariants> {
  children: React.ReactNode
  className?: string
}

export function StatusBadge({ variant, className, children }: StatusBadgeProps) {
  return (
    <span className={cn(badgeVariants({ variant }), className)}>
      {children}
    </span>
  )
}

// Usage:
// <StatusBadge variant="success">Paid</StatusBadge>
// <StatusBadge variant="warning">Unpaid</StatusBadge>
```

## Empty State Component

```tsx
// components/empty-state.tsx
import { Button } from '@/components/ui/button'
import { LucideIcon } from 'lucide-react'

interface EmptyStateProps {
  icon?: LucideIcon
  title: string
  description: string
  action?: {
    label: string
    onClick: () => void
  }
}

export function EmptyState({
  icon: Icon,
  title,
  description,
  action,
}: EmptyStateProps) {
  return (
    <div className="flex flex-col items-center justify-center py-12 text-center">
      {Icon && (
        <div className="rounded-full bg-muted p-3 mb-4">
          <Icon className="h-6 w-6 text-muted-foreground" />
        </div>
      )}
      <h3 className="font-semibold">{title}</h3>
      <p className="text-sm text-muted-foreground mt-1 max-w-sm">
        {description}
      </p>
      {action && (
        <Button onClick={action.onClick} className="mt-4">
          {action.label}
        </Button>
      )}
    </div>
  )
}

// Usage:
// <EmptyState
//   icon={FileText}
//   title="No scripts yet"
//   description="Create your first script to get nutrient recommendations"
//   action={{ label: "Create Script", onClick: () => router.push('/new') }}
// />
```

## Data Table Component

```tsx
// components/data-table.tsx
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table'

interface Column<T> {
  key: keyof T | string
  header: string
  render?: (item: T) => React.ReactNode
}

interface DataTableProps<T> {
  data: T[]
  columns: Column<T>[]
  onRowClick?: (item: T) => void
}

export function DataTable<T extends { id: string }>({
  data,
  columns,
  onRowClick,
}: DataTableProps<T>) {
  return (
    <Table>
      <TableHeader>
        <TableRow>
          {columns.map((column) => (
            <TableHead key={column.key as string}>{column.header}</TableHead>
          ))}
        </TableRow>
      </TableHeader>
      <TableBody>
        {data.map((item) => (
          <TableRow
            key={item.id}
            onClick={() => onRowClick?.(item)}
            className={onRowClick ? 'cursor-pointer hover:bg-muted/50' : ''}
          >
            {columns.map((column) => (
              <TableCell key={column.key as string}>
                {column.render
                  ? column.render(item)
                  : (item[column.key as keyof T] as React.ReactNode)}
              </TableCell>
            ))}
          </TableRow>
        ))}
      </TableBody>
    </Table>
  )
}

// Usage:
// <DataTable
//   data={scripts}
//   columns={[
//     { key: 'sample_id', header: 'Sample ID' },
//     { key: 'crop', header: 'Crop', render: (s) => s.crop.toUpperCase() },
//     { key: 'status', header: 'Status', render: (s) => <StatusBadge>{s.status}</StatusBadge> },
//   ]}
//   onRowClick={(script) => router.push(`/scripts/${script.id}`)}
// />
```

## Modal/Dialog Component

```tsx
// components/confirm-dialog.tsx
'use client'

import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from '@/components/ui/alert-dialog'

interface ConfirmDialogProps {
  trigger: React.ReactNode
  title: string
  description: string
  confirmLabel?: string
  cancelLabel?: string
  onConfirm: () => void
  destructive?: boolean
}

export function ConfirmDialog({
  trigger,
  title,
  description,
  confirmLabel = 'Confirm',
  cancelLabel = 'Cancel',
  onConfirm,
  destructive = false,
}: ConfirmDialogProps) {
  return (
    <AlertDialog>
      <AlertDialogTrigger asChild>{trigger}</AlertDialogTrigger>
      <AlertDialogContent>
        <AlertDialogHeader>
          <AlertDialogTitle>{title}</AlertDialogTitle>
          <AlertDialogDescription>{description}</AlertDialogDescription>
        </AlertDialogHeader>
        <AlertDialogFooter>
          <AlertDialogCancel>{cancelLabel}</AlertDialogCancel>
          <AlertDialogAction
            onClick={onConfirm}
            className={destructive ? 'bg-destructive hover:bg-destructive/90' : ''}
          >
            {confirmLabel}
          </AlertDialogAction>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  )
}

// Usage:
// <ConfirmDialog
//   trigger={<Button variant="destructive">Delete</Button>}
//   title="Delete script?"
//   description="This action cannot be undone."
//   confirmLabel="Delete"
//   onConfirm={handleDelete}
//   destructive
// />
```

## Page Layout Component

```tsx
// components/page-header.tsx
interface PageHeaderProps {
  title: string
  description?: string
  actions?: React.ReactNode
}

export function PageHeader({ title, description, actions }: PageHeaderProps) {
  return (
    <div className="flex items-center justify-between mb-6">
      <div>
        <h1 className="text-2xl font-semibold">{title}</h1>
        {description && (
          <p className="text-muted-foreground mt-1">{description}</p>
        )}
      </div>
      {actions && <div className="flex items-center gap-2">{actions}</div>}
    </div>
  )
}

// Usage:
// <PageHeader
//   title="Scripts"
//   description="Manage your nutrient test scripts"
//   actions={<Button>New Script</Button>}
// />
```
