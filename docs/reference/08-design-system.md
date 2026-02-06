# FoliarScript Design System

> Built on **Tailwind CSS**, **Radix UI primitives**, and **shadcn/ui** patterns.

---

## Technology Stack

```json
{
  "core": {
    "next": "15.x",
    "react": "19.x",
    "typescript": "5.x",
    "tailwindcss": "3.4.x"
  },
  "ui": {
    "@radix-ui/react-*": "latest",
    "class-variance-authority": "0.7.x",
    "clsx": "2.1.x",
    "tailwind-merge": "3.x",
    "tailwindcss-animate": "1.0.x"
  },
  "icons": {
    "lucide-react": "0.542.x"
  },
  "fonts": {
    "family": "Poppins",
    "weights": [400, 500, 600, 700]
  }
}
```

---

## Color Palette

### Primary Colors

| Name | Hex | HSL | Usage |
|------|-----|-----|-------|
| Primary Blue | `#146EF5` | `hsl(217, 91%, 60%)` | Primary actions, links, focus states |
| Primary Hover | `#0D5FD4` | `hsl(217, 91%, 44%)` | Hover state for primary |
| Primary Active | `#0A4FB3` | `hsl(217, 91%, 37%)` | Active/pressed state |

### Secondary Colors

| Name | Hex | HSL | Usage |
|------|-----|-----|-------|
| Secondary Gray | `#454C64` | `hsl(228, 18%, 33%)` | Secondary text, icons |
| Secondary Hover | `#3A4154` | `hsl(228, 18%, 28%)` | Hover state |

### Semantic Colors

| Name | Hex | HSL | Usage |
|------|-----|-----|-------|
| Success | `#10B981` / `#11C819` | `hsl(160, 84%, 39%)` | Sufficient nutrients, confirmations |
| Warning | `#F59E0B` / `#FF9822` | `hsl(38, 92%, 50%)` | Low nutrients, cautions |
| Error | `#EF4444` | `hsl(0, 84%, 60%)` | Errors, excess warnings |
| Info | `#3B82F6` | `hsl(217, 91%, 60%)` | Informational messages |

### Background Colors (Dark Theme)

| Name | HSL | Usage |
|------|-----|-------|
| Background | `hsl(240, 10%, 10%)` | Main app background |
| Card | `hsl(240, 15%, 15%)` | Card backgrounds |
| Popover | `hsl(240, 15%, 15%)` | Popover/dropdown backgrounds |
| Muted | `hsl(240, 10%, 20%)` | Muted/disabled backgrounds |

### Text Colors

| Name | HSL | Usage |
|------|-----|-------|
| Foreground | `hsl(0, 0%, 98%)` | Primary text (white) |
| Muted Foreground | `hsl(240, 5%, 65%)` | Secondary/muted text |

### Border Colors

| Name | HSL | Usage |
|------|-----|-------|
| Border | `hsl(240, 10%, 25%)` | Default borders |
| Input | `hsl(240, 10%, 20%)` | Input field borders |
| Ring | `hsl(217, 91%, 60%)` | Focus rings (primary blue) |

---

## CSS Variables

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    /* Background colors */
    --background: 240 10% 10%;
    --foreground: 0 0% 98%;

    /* Card */
    --card: 240 15% 15%;
    --card-foreground: 0 0% 98%;

    /* Popover */
    --popover: 240 15% 15%;
    --popover-foreground: 0 0% 98%;

    /* Primary */
    --primary: 217 91% 60%;
    --primary-foreground: 0 0% 100%;

    /* Secondary */
    --secondary: 240 10% 20%;
    --secondary-foreground: 0 0% 98%;

    /* Muted */
    --muted: 240 10% 20%;
    --muted-foreground: 240 5% 64.9%;

    /* Accent */
    --accent: 217 91% 60%;
    --accent-foreground: 0 0% 98%;

    /* Destructive */
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 0 0% 98%;

    /* Border & Input */
    --border: 240 10% 25%;
    --input: 240 10% 20%;
    --ring: 217 91% 60%;

    /* Border Radius */
    --radius: 0.75rem;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}

/* Custom Scrollbar */
.scrollbar-thin {
  scrollbar-width: thin;
  scrollbar-color: hsl(var(--muted-foreground) / 0.3) transparent;
}

.scrollbar-thin::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

.scrollbar-thin::-webkit-scrollbar-track {
  background: transparent;
}

.scrollbar-thin::-webkit-scrollbar-thumb {
  background: hsl(var(--muted-foreground) / 0.3);
  border-radius: 3px;
}

.scrollbar-thin::-webkit-scrollbar-thumb:hover {
  background: hsl(var(--muted-foreground) / 0.5);
}
```

---

## Typography

### Font Setup (Next.js)

```tsx
// app/layout.tsx
import { Poppins } from 'next/font/google'

const poppins = Poppins({
  subsets: ['latin'],
  weight: ['400', '500', '600', '700'],
  variable: '--font-poppins',
})

export default function RootLayout({ children }) {
  return (
    <html className={poppins.variable}>
      <body className="font-sans">{children}</body>
    </html>
  )
}
```

### Typography Scale

| Name | Size | Weight | Tailwind Classes |
|------|------|--------|------------------|
| Display Large | 32px | 700 | `text-[2rem] font-bold` |
| Heading 1 | 22px | 600 | `text-[1.375rem] font-semibold` |
| Heading 2 | 20px | 500 | `text-[1.25rem] font-medium` |
| Heading 3 | 24px | 600 | `text-2xl font-semibold` |
| Heading 4 | 20px | 600 | `text-xl font-semibold` |
| Body Large | 16px | 500 | `text-base font-medium` |
| Body Medium | 14px | 500 | `text-sm font-medium` |
| Body Small | 13px | 500 | `text-[0.8125rem] font-medium` |
| Caption | 12px | 500 | `text-xs font-medium` |

---

## Spacing System

**Base Unit**: 8px grid

| Token | Value | Tailwind |
|-------|-------|----------|
| xs | 8px | `p-2`, `m-2` |
| sm | 12px | `p-3`, `m-3` |
| md | 16px | `p-4`, `m-4` |
| lg | 24px | `p-6`, `m-6` |
| xl | 32px | `p-8`, `m-8` |
| 2xl | 64px | `p-16`, `m-16` |

### Component Spacing

| Component | Padding |
|-----------|---------|
| Button SM | 8px 16px |
| Button MD | 12px 24px |
| Card | 24px |
| Input | 12px 16px |
| Dialog | 24px |

---

## Border Radius

| Name | Value | Tailwind | Usage |
|------|-------|----------|-------|
| sm | 8px | `rounded-sm` | Small elements |
| md | 10px | `rounded-md` | Buttons, inputs |
| lg | 12px | `rounded-lg` | Cards, dialogs |
| xl | 16px | `rounded-xl` | Large cards |
| full | 9999px | `rounded-full` | Pills, avatars |

---

## Shadows

| Name | Value | Usage |
|------|-------|-------|
| sm | `0 1px 3px 0 rgb(0 0 0 / 0.3)` | Cards |
| md | `0 4px 6px -1px rgb(0 0 0 / 0.4)` | Elevated elements |
| lg | `0 20px 25px -5px rgb(0 0 0 / 0.5)` | Modals |

---

## Animations

### Transition Durations

| Name | Duration | Usage |
|------|----------|-------|
| Fast | 150ms | Micro-interactions |
| Base | 200ms | Standard transitions |
| Slow | 300ms | Complex animations |

### Animation Classes (tailwindcss-animate)

```css
/* Fade */
animate-in fade-in-0
animate-out fade-out-0

/* Zoom */
animate-in zoom-in-95
animate-out zoom-out-95

/* Slide */
animate-in slide-in-from-top-2
animate-in slide-in-from-bottom-2
```

---

## Responsive Breakpoints

| Name | Min Width | Tailwind Prefix |
|------|-----------|-----------------|
| sm | 640px | `sm:` |
| md | 768px | `md:` |
| lg | 1024px | `lg:` |
| xl | 1280px | `xl:` |
| 2xl | 1536px | `2xl:` |

---

## Icon System

**Library**: lucide-react

### Sizes

```tsx
// Small (14px)
<Icon className="h-3.5 w-3.5" />

// Standard (16px)
<Icon className="h-4 w-4" />

// Large (24px)
<Icon className="h-6 w-6" />
```

---

## Utility Function

```ts
// lib/utils.ts
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

---

## Component Patterns

### Button Variants

```tsx
const buttonVariants = cva(
  "inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground shadow hover:bg-primary/90",
        destructive: "bg-destructive text-destructive-foreground shadow-sm hover:bg-destructive/90",
        outline: "border border-input bg-background shadow-sm hover:bg-accent hover:text-accent-foreground",
        secondary: "bg-secondary text-secondary-foreground shadow-sm hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-9 px-4 py-2",
        sm: "h-8 rounded-md px-3 text-xs",
        lg: "h-10 rounded-md px-8",
        icon: "h-9 w-9",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)
```

### Input Style

```tsx
<input
  className="flex h-9 w-full rounded-md border border-input bg-transparent px-3 py-1 text-sm shadow-sm transition-colors placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-ring disabled:cursor-not-allowed disabled:opacity-50"
/>
```

### Card Pattern

```tsx
<div className="rounded-xl border bg-card text-card-foreground shadow">
  <div className="flex flex-col space-y-1.5 p-6">{/* Header */}</div>
  <div className="p-6 pt-0">{/* Content */}</div>
</div>
```

### Badge - Nutrient Status

```tsx
// Sufficient (green)
<span className="px-3 py-2 rounded text-sm font-semibold bg-[rgba(14,155,21,0.5)] text-[#10CF18]">
  3.5 %
</span>

// Low (orange)
<span className="px-3 py-2 rounded text-sm font-semibold bg-[rgba(220,123,12,0.28)] text-[#FF9982]">
  2.1 %
</span>
```

---

## FoliarScript-Specific Components

### Concentration Slider

```tsx
// Container
<div className="relative w-full">
  {/* Tooltip positioned above slider */}
  <input
    type="text"
    className="absolute -top-8 transform -translate-x-1/2 bg-[#ADD1FF] border border-white/30 rounded w-12 h-7 text-center text-xs text-[#000A3C]"
    style={{ left: `${sliderValue}%` }}
  />

  {/* Range slider */}
  <input
    type="range"
    min="0"
    max="100"
    className="w-full h-1.5 rounded-full appearance-none cursor-pointer"
    style={{
      background: `linear-gradient(to right, #0372FF 0%, #0372FF ${value}%, #2D4A6E ${value}%, #2D4A6E 100%)`
    }}
  />

  {/* Labels */}
  <div className="flex justify-between text-sm text-white mt-1">
    <span>0</span>
    <span>100</span>
  </div>
</div>
```

### Application Timeline

```
┌─────────────────────────────────────────────┐
│              12.50 fl oz                    │
│                                             │
│    ┌───┐                                    │
│    │ 1 │ ────────  5.00 fl oz               │
│    └───┘           1st Application          │
│      │                                      │
│  After                                      │
│  1 week                                     │
│      │                                      │
│    ┌───┐                                    │
│    │ 2 │ ────────  5.00 fl oz               │
│    └───┘           2nd Application          │
│      │                                      │
│  After                                      │
│  1 week                                     │
│      │                                      │
│    ┌───┐                                    │
│    │ 3 │ ────────  2.50 fl oz               │
│    └───┘           3rd Application          │
└─────────────────────────────────────────────┘
```

### Circle Colors (Timeline)

```css
.circle-1 { background: rgba(0, 94, 214, 1); }    /* #005ED6 - darkest */
.circle-2 { background: rgba(56, 136, 237, 1); }  /* #3888ED - medium */
.circle-3 { background: rgba(124, 177, 245, 1); } /* #7CB1F5 - lightest */
```

---

## Tailwind Config

```ts
// tailwind.config.ts
import type { Config } from "tailwindcss"

const config: Config = {
  darkMode: ["class"],
  content: ["./src/**/*.{js,ts,jsx,tsx,mdx}"],
  theme: {
    extend: {
      fontFamily: {
        sans: ['var(--font-poppins)', 'system-ui', 'sans-serif'],
      },
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      keyframes: {
        "accordion-down": {
          from: { height: "0" },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: "0" },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
}

export default config
```

---

## Dependencies Installation

```bash
# Core
npm install tailwindcss postcss autoprefixer

# Radix UI
npm install @radix-ui/react-dialog @radix-ui/react-select @radix-ui/react-tabs @radix-ui/react-checkbox @radix-ui/react-radio-group @radix-ui/react-tooltip @radix-ui/react-dropdown-menu @radix-ui/react-separator @radix-ui/react-progress @radix-ui/react-slider @radix-ui/react-collapsible @radix-ui/react-alert-dialog @radix-ui/react-toast

# Utilities
npm install class-variance-authority clsx tailwind-merge tailwindcss-animate

# Icons
npm install lucide-react

# PDF Generation
npm install html2canvas jspdf
```
