# ZaZen Website - Technical Implementation Guide
## React + GSAP Production Stack

---

## 1. PROJECT STRUCTURE

```
zazen-website/
├── public/
│   ├── images/
│   │   ├── products/
│   │   ├── industries/
│   │   ├── icons/
│   │   └── backgrounds/
│   ├── fonts/
│   └── models/           # 3D assets if using Three.js
├── src/
│   ├── components/
│   │   ├── ui/           # Reusable UI components
│   │   ├── sections/     # Page sections
│   │   ├── animations/   # Animation components
│   │   └── layout/       # Layout components
│   ├── hooks/            # Custom React hooks
│   ├── lib/              # Utilities
│   ├── styles/           # Global styles
│   └── types/            # TypeScript types
├── package.json
├── tailwind.config.ts
├── tsconfig.json
└── vite.config.ts
```

---

## 2. DEPENDENCIES

```bash
# Core
npm install react react-dom typescript

# Animation
npm install gsap @gsap/react lenis

# Styling
npm install tailwindcss @tailwindcss/typography clsx tailwind-merge

# Icons
npm install lucide-react

# Utilities
npm install @radix-ui/react-tabs @radix-ui/react-dialog @radix-ui/react-slot
```

---

## 3. CORE CONFIGURATION

### tailwind.config.ts
```typescript
import type { Config } from 'tailwindcss'

const config: Config = {
  content: ['./src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        midnight: '#0A1628',
        'navy-deep': '#0F1D32',
        'navy-light': '#1A2B47',
        slate: '#2A3A55',
        'electric-green': '#00D084',
        'electric-green-glow': '#00FF9D',
        'electric-blue': '#00A8FF',
        'ice-blue': '#E0F7FF',
        'frost-white': '#F8FBFF',
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
        mono: ['JetBrains Mono', 'monospace'],
        display: ['SF Pro Display', 'Inter', 'sans-serif'],
      },
      animation: {
        'sensor-pulse': 'sensorPulse 2s ease-in-out infinite',
        'spin-slow': 'spin 20s linear infinite',
        'float': 'float 6s ease-in-out infinite',
      },
      keyframes: {
        sensorPulse: {
          '0%, 100%': { boxShadow: '0 0 0 0 rgba(0, 208, 132, 0.4)' },
          '50%': { boxShadow: '0 0 20px 10px rgba(0, 208, 132, 0)' },
        },
        float: {
          '0%, 100%': { transform: 'translateY(0px)' },
          '50%': { transform: 'translateY(-20px)' },
        },
      },
    },
  },
  plugins: [],
}

export default config
```

### GSAP Setup (src/lib/gsap.ts)
```typescript
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import { SplitText } from 'gsap/SplitText' // Club GreenSock

if (typeof window !== 'undefined') {
  gsap.registerPlugin(ScrollTrigger, SplitText)
}

export { gsap, ScrollTrigger }
```

### Smooth Scroll Hook (src/hooks/useSmoothScroll.ts)
```typescript
import { useEffect } from 'react'
import Lenis from 'lenis'

export function useSmoothScroll() {
  useEffect(() => {
    const lenis = new Lenis({
      duration: 1.2,
      easing: (t: number) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
      smoothWheel: true,
    })

    function raf(time: number) {
      lenis.raf(time)
      requestAnimationFrame(raf)
    }

    requestAnimationFrame(raf)

    return () => {
      lenis.destroy()
    }
  }, [])
}
```

---

## 4. ANIMATION COMPONENTS

### ScrollReveal Component
```tsx
// src/components/animations/ScrollReveal.tsx
'use client'

import { useEffect, useRef } from 'react'
import { gsap } from '@/lib/gsap'

interface ScrollRevealProps {
  children: React.ReactNode
  className?: string
  delay?: number
  direction?: 'up' | 'down' | 'left' | 'right'
  duration?: number
}

export function ScrollReveal({
  children,
  className = '',
  delay = 0,
  direction = 'up',
  duration = 1,
}: ScrollRevealProps) {
  const ref = useRef<HTMLDivElement>(null)

  useEffect(() => {
    const el = ref.current
    if (!el) return

    const directions = {
      up: { y: 60 },
      down: { y: -60 },
      left: { x: 60 },
      right: { x: -60 },
    }

    gsap.fromTo(
      el,
      {
        opacity: 0,
        ...directions[direction],
      },
      {
        opacity: 1,
        x: 0,
        y: 0,
        duration,
        delay,
        ease: 'power3.out',
        scrollTrigger: {
          trigger: el,
          start: 'top 85%',
          toggleActions: 'play none none none',
        },
      }
    )

    return () => {
      ScrollTrigger.getAll().forEach(st => st.kill())
    }
  }, [delay, direction, duration])

  return (
    <div ref={ref} className={className}>
      {children}
    </div>
  )
}
```

### TextSplit Animation
```tsx
// src/components/animations/TextSplit.tsx
'use client'

import { useEffect, useRef } from 'react'
import { gsap } from '@/lib/gsap'

interface TextSplitProps {
  children: string
  className?: string
  tag?: 'h1' | 'h2' | 'h3' | 'p' | 'span'
}

export function TextSplit({ children, className = '', tag = 'h2' }: TextSplitProps) {
  const ref = useRef<HTMLElement>(null)

  useEffect(() => {
    const el = ref.current
    if (!el) return

    const split = new SplitText(el, { type: 'words,lines' })

    gsap.fromTo(
      split.words,
      {
        y: 100,
        opacity: 0,
      },
      {
        y: 0,
        opacity: 1,
        duration: 1.2,
        stagger: 0.1,
        ease: 'expo.out',
        scrollTrigger: {
          trigger: el,
          start: 'top 80%',
        },
      }
    )

    return () => {
      split.revert()
    }
  }, [])

  const Tag = tag

  return <Tag ref={ref as any} className={className}>{children}</Tag>
}
```

### Counter Animation
```tsx
// src/components/animations/Counter.tsx
'use client'

import { useEffect, useRef, useState } from 'react'
import { gsap } from '@/lib/gsap'

interface CounterProps {
  end: number
  prefix?: string
  suffix?: string
  duration?: number
  className?: string
}

export function Counter({
  end,
  prefix = '',
  suffix = '',
  duration = 2,
  className = '',
}: CounterProps) {
  const ref = useRef<HTMLSpanElement>(null)
  const [hasAnimated, setHasAnimated] = useState(false)

  useEffect(() => {
    const el = ref.current
    if (!el || hasAnimated) return

    const obj = { value: 0 }

    gsap.to(obj, {
      value: end,
      duration,
      ease: 'power2.out',
      scrollTrigger: {
        trigger: el,
        start: 'top 85%',
        onEnter: () => setHasAnimated(true),
      },
      onUpdate: () => {
        el.textContent = prefix + Math.round(obj.value).toLocaleString() + suffix
      },
    })
  }, [end, prefix, suffix, duration, hasAnimated])

  return <span ref={ref} className={className}>{prefix}0{suffix}</span>
}
```

### Horizontal Scroll Section
```tsx
// src/components/animations/HorizontalScroll.tsx
'use client'

import { useEffect, useRef } from 'react'
import { gsap } from '@/lib/gsap'

export function HorizontalScroll({ children }: { children: React.ReactNode }) {
  const containerRef = useRef<HTMLDivElement>(null)
  const scrollRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    const container = containerRef.current
    const scroll = scrollRef.current
    if (!container || !scroll) return

    const scrollWidth = scroll.scrollWidth - window.innerWidth

    gsap.to(scroll, {
      x: -scrollWidth,
      ease: 'none',
      scrollTrigger: {
        trigger: container,
        start: 'top top',
        end: () => `+=${scrollWidth}`,
        pin: true,
        scrub: 1,
        snap: {
          snapTo: 1 / (scroll.children.length - 1),
          duration: 0.5,
          ease: 'power2.inOut',
        },
      },
    })

    return () => {
      ScrollTrigger.getAll().forEach(st => st.kill())
    }
  }, [])

  return (
    <div ref={containerRef} className="h-screen overflow-hidden">
      <div ref={scrollRef} className="flex h-full">
        {children}
      </div>
    </div>
  )
}
```

---

## 5. SECTION IMPLEMENTATIONS

### Hero Section
```tsx
// src/components/sections/Hero.tsx
'use client'

import { useEffect, useRef } from 'react'
import { gsap } from '@/lib/gsap'
import { Button } from '@/components/ui/Button'

export function Hero() {
  const headlineRef = useRef<HTMLHeadingElement>(null)
  const subheadRef = useRef<HTMLParagraphElement>(null)
  const ctaRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    const tl = gsap.timeline({ defaults: { ease: 'expo.out' } })

    // Headline animation
    tl.fromTo(
      headlineRef.current,
      { y: 100, opacity: 0 },
      { y: 0, opacity: 1, duration: 1.2 }
    )
      .fromTo(
        subheadRef.current,
        { y: 40, opacity: 0 },
        { y: 0, opacity: 1, duration: 1 },
        '-=0.8'
      )
      .fromTo(
        ctaRef.current,
        { y: 30, opacity: 0 },
        { y: 0, opacity: 1, duration: 0.8 },
        '-=0.6'
      )
  }, [])

  return (
    <section className="relative min-h-screen flex items-center justify-center overflow-hidden bg-midnight">
      {/* Animated background particles */}
      <div className="absolute inset-0">
        <ParticleField />
      </div>

      {/* Gradient glow */}
      <div className="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-[800px] h-[800px] bg-gradient-glow opacity-50" />

      <div className="relative z-10 max-w-7xl mx-auto px-6 text-center">
        {/* Product visualization */}
        <div className="mb-12">
          <ProductVisualization />
        </div>

        <h1
          ref={headlineRef}
          className="text-5xl md:text-7xl lg:text-8xl font-display font-bold text-white tracking-tight"
        >
          INTELLIGENT COLD CHAIN
          <br />
          <span className="text-electric-green">FOR MODERN INDIA</span>
        </h1>

        <p
          ref={subheadRef}
          className="mt-8 text-xl md:text-2xl text-text-secondary max-w-3xl mx-auto"
        >
          40% energy savings. ±0.5°C precision. Solar-ready architecture.
          The only commercial freezer built for India&apos;s reality.
        </p>

        <div ref={ctaRef} className="mt-12 flex flex-col sm:flex-row gap-4 justify-center">
          <Button size="lg" variant="primary">
            Request Demo
          </Button>
          <Button size="lg" variant="secondary">
            Explore Technology
          </Button>
        </div>

        {/* Trust bar */}
        <TrustBar />
      </div>
    </section>
  )
}
```

### Problem Section
```tsx
// src/components/sections/Problem.tsx
'use client'

import { ScrollReveal } from '@/components/animations/ScrollReveal'
import { Counter } from '@/components/animations/Counter'
import { ProblemChart } from '@/components/visualizations/ProblemChart'

const problemData = [
  {
    value: 40,
    suffix: '%',
    label: 'FOOD WASTE RATE',
    description: 'India loses 40% of perishable food to temperature failures',
  },
  {
    value: 3,
    prefix: '±',
    suffix: '°C',
    label: 'VARIANCE INDUSTRY STANDARD',
    description: 'Traditional freezers swing wildly, destroying product quality',
  },
  {
    value: 2.1,
    prefix: '₹',
    suffix: 'L',
    label: 'ANNUAL POWER COST (200L)',
    description: 'Fixed-speed compressors guzzle electricity 24/7',
  },
]

export function Problem() {
  return (
    <section className="py-32 bg-midnight">
      <div className="max-w-7xl mx-auto px-6">
        <ScrollReveal>
          <span className="text-electric-green font-mono text-sm tracking-wider">
            THE PROBLEM
          </span>
          <h2 className="mt-4 text-4xl md:text-6xl font-display font-bold text-white">
            INDIA&apos;S COLD CHAIN
            <br />
            IS BROKEN
          </h2>
        </ScrollReveal>

        <div className="mt-16 grid md:grid-cols-3 gap-8">
          {problemData.map((item, i) => (
            <ScrollReveal key={item.label} delay={i * 0.15}>
              <div className="p-8 rounded-2xl bg-navy-light border border-slate/30 backdrop-blur-sm hover:border-electric-green/50 transition-all hover:-translate-y-2">
                <Counter
                  end={item.value}
                  prefix={item.prefix}
                  suffix={item.suffix}
                  className="text-5xl md:text-6xl font-bold text-electric-green"
                />
                <h3 className="mt-4 text-lg font-semibold text-white">
                  {item.label}
                </h3>
                <p className="mt-2 text-text-muted">{item.description}</p>
              </div>
            </ScrollReveal>
          ))}
        </div>

        <ScrollReveal delay={0.4}>
          <div className="mt-16 p-8 rounded-3xl bg-navy-deep border border-slate/20">
            <ProblemChart />
          </div>
        </ScrollReveal>
      </div>
    </section>
  )
}
```

### Product Tabs Section
```tsx
// src/components/sections/Products.tsx
'use client'

import { useState } from 'react'
import { motion, AnimatePresence } from 'framer-motion'
import { ScrollReveal } from '@/components/animations/ScrollReveal'

const products = [
  {
    id: 'zf-series',
    name: 'ZF-Series',
    title: 'Smart Chest Freezers',
    description: 'Complete intelligent units with BLDC compressor, IoT monitoring, and solar compatibility.',
    models: [
      { name: 'ZF-200', size: '200L', price: '₹85,000' },
      { name: 'ZF-400', size: '400L', price: '₹1,25,000' },
      { name: 'ZF-700', size: '700L', price: '₹1,85,000' },
      { name: 'ZF-1200', size: '1200L', price: '₹2,45,000' },
      { name: 'ZF-1500', size: '1500L', price: '₹3,15,000' },
    ],
  },
  {
    id: 'retrofit',
    name: 'Retrofit',
    title: 'Upgrade Solutions',
    description: 'Transform existing equipment at 60% lower cost than replacement.',
    brands: ['Blue Star', 'Voltas', 'Haier', 'Godrej', 'Rockwell'],
    priceRange: '₹50,000 - ₹80,000',
  },
  {
    id: 'solar',
    name: 'Solar',
    title: 'Power Packages',
    description: 'True energy independence with LiFePO4 batteries and hybrid controllers.',
    configs: [
      { name: 'ZF-200 Solar', power: '300W' },
      { name: 'ZF-400 Solar', power: '450W' },
      { name: 'ZF-700 Solar', power: '600W' },
    ],
  },
  {
    id: 'cloud',
    name: 'IoT Platform',
    title: 'ZaZen Cloud',
    description: '24/7 operational visibility with predictive maintenance AI.',
    features: ['24/7 monitoring', 'SMS/email alerts', 'Mobile app', 'API integration', 'OTA updates'],
  },
]

export function Products() {
  const [activeTab, setActiveTab] = useState('zf-series')
  const activeProduct = products.find(p => p.id === activeTab)

  return (
    <section className="py-32 bg-frost-white">
      <div className="max-w-7xl mx-auto px-6">
        <ScrollReveal>
          <span className="text-electric-green font-mono text-sm tracking-wider">
            PRODUCTS
          </span>
          <h2 className="mt-4 text-4xl md:text-6xl font-display font-bold text-midnight">
            CHOOSE YOUR
            <br />
            SOLUTION
          </h2>
        </ScrollReveal>

        {/* Tabs */}
        <div className="mt-12 flex flex-wrap gap-2">
          {products.map((product) => (
            <button
              key={product.id}
              onClick={() => setActiveTab(product.id)}
              className={`relative px-6 py-3 rounded-full font-medium transition-all ${
                activeTab === product.id
                  ? 'text-white'
                  : 'text-text-secondary hover:text-midnight'
              }`}
            >
              {activeTab === product.id && (
                <motion.div
                  layoutId="activeTab"
                  className="absolute inset-0 bg-midnight rounded-full"
                  transition={{ type: 'spring', bounce: 0.2, duration: 0.6 }}
                />
              )}
              <span className="relative z-10">{product.name}</span>
            </button>
          ))}
        </div>

        {/* Content */}
        <AnimatePresence mode="wait">
          <motion.div
            key={activeTab}
            initial={{ opacity: 0, y: 20 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: -20 }}
            transition={{ duration: 0.3 }}
            className="mt-12 grid lg:grid-cols-2 gap-12"
          >
            {/* Product Image */}
            <div className="relative aspect-square rounded-3xl bg-navy-light overflow-hidden">
              <ProductImage product={activeProduct} />
            </div>

            {/* Product Details */}
            <div>
              <h3 className="text-3xl font-bold text-midnight">
                {activeProduct?.title}
              </h3>
              <p className="mt-4 text-text-secondary text-lg">
                {activeProduct?.description}
              </p>

              {activeProduct?.models && (
                <div className="mt-8 grid grid-cols-2 sm:grid-cols-3 gap-4">
                  {activeProduct.models.map((model) => (
                    <div
                      key={model.name}
                      className="p-4 rounded-xl bg-white border border-slate/20 hover:border-electric-green hover:shadow-lg transition-all cursor-pointer"
                    >
                      <div className="font-semibold text-midnight">{model.name}</div>
                      <div className="text-sm text-text-muted">{model.size}</div>
                      <div className="mt-2 text-electric-green font-medium">{model.price}</div>
                    </div>
                  ))}
                </div>
              )}

              <button className="mt-8 text-midnight font-medium inline-flex items-center gap-2 hover:gap-4 transition-all">
                View Full Specifications
                <ArrowRight />
              </button>
            </div>
          </motion.div>
        </AnimatePresence>
      </div>
    </section>
  )
}
```

---

## 6. UTILITY COMPONENTS

### Frosted Card
```tsx
// src/components/ui/FrostedCard.tsx
interface FrostedCardProps {
  children: React.ReactNode
  className?: string
  hover?: boolean
}

export function FrostedCard({ children, className = '', hover = true }: FrostedCardProps) {
  return (
    <div
      className={`
        p-8 rounded-2xl
        bg-navy-light/80
        backdrop-blur-xl
        border border-white/10
        ${hover ? 'hover:-translate-y-2 hover:border-electric-green/30 hover:shadow-xl hover:shadow-electric-green/10' : ''}
        transition-all duration-300
        ${className}
      `}
    >
      {children}
    </div>
  )
}
```

### Button Component
```tsx
// src/components/ui/Button.tsx
import { cva, type VariantProps } from 'class-variance-authority'
import { cn } from '@/lib/utils'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-full font-medium transition-all focus:outline-none focus:ring-2 focus:ring-offset-2',
  {
    variants: {
      variant: {
        primary: 'bg-electric-green text-midnight hover:bg-electric-green-glow hover:-translate-y-1 hover:shadow-lg hover:shadow-electric-green/25',
        secondary: 'bg-transparent border-2 border-white text-white hover:bg-white hover:text-midnight',
        outline: 'bg-transparent border-2 border-midnight text-midnight hover:bg-midnight hover:text-white',
      },
      size: {
        default: 'px-6 py-3 text-base',
        sm: 'px-4 py-2 text-sm',
        lg: 'px-8 py-4 text-lg',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'default',
    },
  }
)

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

export function Button({ className, variant, size, ...props }: ButtonProps) {
  return (
    <button
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  )
}
```

---

## 7. ACCESSIBILITY IMPLEMENTATIONS

### Reduced Motion Hook
```tsx
// src/hooks/useReducedMotion.ts
import { useEffect, useState } from 'react'

export function useReducedMotion(): boolean {
  const [reducedMotion, setReducedMotion] = useState(false)

  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)')
    setReducedMotion(mediaQuery.matches)

    const handler = (e: MediaQueryListEvent) => setReducedMotion(e.matches)
    mediaQuery.addEventListener('change', handler)

    return () => mediaQuery.removeEventListener('change', handler)
  }, [])

  return reducedMotion
}
```

### Accessible Animation Wrapper
```tsx
// src/components/animations/AccessibleAnimation.tsx
import { useReducedMotion } from '@/hooks/useReducedMotion'

export function AccessibleAnimation({
  children,
  animation,
  fallback,
}: {
  children: React.ReactNode
  animation: React.ReactNode
  fallback: React.ReactNode
}) {
  const reducedMotion = useReducedMotion()
  return reducedMotion ? fallback : animation
}
```

---

## 8. PERFORMANCE OPTIMIZATIONS

### Image Optimization
```tsx
// Use Next.js Image or similar
import Image from 'next/image'

<Image
  src="/products/zf-series.png"
  alt="ZF-Series Smart Freezer"
  width={800}
  height={600}
  priority // For hero images
  loading="lazy" // For below-fold
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,..."
/>
```

### Lazy Load Sections
```tsx
// src/components/LazySection.tsx
import { lazy, Suspense } from 'react'

const HeavySection = lazy(() => import('./HeavySection'))

export function LazySection() {
  return (
    <Suspense fallback={<div className="h-screen animate-pulse bg-navy-light" />}>
      <HeavySection />
    </Suspense>
  )
}
```

### Will-Change Management
```tsx
// src/hooks/useWillChange.ts
import { useEffect, useRef } from 'react'

export function useWillChange() {
  const ref = useRef<HTMLDivElement>(null)

  useEffect(() => {
    const el = ref.current
    if (!el) return

    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            el.style.willChange = 'transform, opacity'
          } else {
            el.style.willChange = 'auto'
          }
        })
      },
      { threshold: 0.1 }
    )

    observer.observe(el)
    return () => observer.disconnect()
  }, [])

  return ref
}
```

---

## 9. TESTING CHECKLIST

### Animation Tests
- [ ] All scroll animations trigger correctly
- [ ] Counter animations count up properly
- [ ] Text split animations don't break text selection
- [ ] Horizontal scroll works on touch devices
- [ ] Tab transitions are smooth

### Performance Tests
- [ ] Lighthouse score > 90
- [ ] First Contentful Paint < 1.5s
- [ ] Animations run at 60fps
- [ ] No layout shift during load
- [ ] Memory usage doesn't leak

### Accessibility Tests
- [ ] Keyboard navigation works
- [ ] Screen reader announces content
- [ ] Reduced motion preference respected
- [ ] Color contrast meets WCAG AA
- [ ] Focus states visible

### Cross-Browser Tests
- [ ] Chrome/Edge (Chromium)
- [ ] Safari (WebKit)
- [ ] Firefox (Gecko)
- [ ] Mobile Safari (iOS)
- [ ] Chrome Mobile (Android)

---

## 10. DEPLOYMENT

### Build Configuration
```json
// package.json scripts
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext ts,tsx",
    "type-check": "tsc --noEmit"
  }
}
```

### Environment Variables
```bash
# .env.production
VITE_API_URL=https://api.zazensystems.com
VITE_ANALYTICS_ID=GA-XXXXXXXX
VITE_CONTACT_FORM_ENDPOINT=/contact
```

### CDN Configuration
```javascript
// vercel.json or netlify.toml
{
  "headers": [
    {
      "source": "/fonts/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    },
    {
      "source": "/images/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=86400"
        }
      ]
    }
  ]
}
```

---

*This technical guide provides the implementation details for building the ZaZen website with React + GSAP. Follow these patterns for consistent, performant, and accessible animations.*
