# 컴포넌트 패턴 가이드

이 문서는 **Next.js 16 + React 19** 환경에서 효율적이고 재사용 가능한 컴포넌트 작성 패턴을 제공합니다.

> ⚠️ **스택 사실의 단일 출처는 `CLAUDE.md`(2·4장)다.** 이 가이드의 서술이 `CLAUDE.md`와 어긋나면 `CLAUDE.md`를 따른다.
> Server Components/클라이언트 컴포넌트 전략은 `nextjs-app-router.md`를 먼저 읽으십시오 — 이 프로젝트는 **클라이언트 컴포넌트 우선**입니다.

## 기본 설계 원칙

### 1. 단일 책임 원칙 (Single Responsibility)

```tsx
// ✅ 각 컴포넌트가 하나의 명확한 책임
export function PriorityBadge({ priority }: { priority: Priority }) {
  return (
    <span className={cn('rounded-full px-2 py-0.5 text-xs', priorityColors[priority])}>
      {priorityLabels[priority]}
    </span>
  )
}

export function TodoDueDate({ dueDate }: { dueDate: string | null }) {
  if (!dueDate) return null
  return <span className="text-muted-foreground text-sm">{formatDate(dueDate)}</span>
}

// ❌ 여러 책임이 섞인 컴포넌트
export function TodoCard({ todo }) {
  // 뱃지 + 날짜 포맷 + 토글 로직 + 삭제 로직 + 애니메이션... (너무 많은 책임)
}
```

### 2. 컴포지션 우선 (Composition over Inheritance)

```tsx
// ✅ 컴포지션 패턴
export function Card({ children, className, ...props }: React.ComponentProps<'div'>) {
  return (
    <div className={cn('rounded-lg border bg-card p-6', className)} {...props}>
      {children}
    </div>
  )
}

export function CardHeader({ children, className, ...props }: React.ComponentProps<'div'>) {
  return (
    <div className={cn('flex flex-col space-y-1.5 pb-6', className)} {...props}>
      {children}
    </div>
  )
}

// 사용법
<Card>
  <CardHeader>
    <CardTitle>제목</CardTitle>
  </CardHeader>
  <CardContent>내용</CardContent>
</Card>
```

## 이 프로젝트의 Server/Client 경계

이 프로젝트는 `nextjs-app-router.md`에서 설명한 대로 **클라이언트 컴포넌트를 기본값**으로 씁니다. 인증 토큰이 `localStorage`에만 있어 서버 컴포넌트가 사용자별 데이터를 가져올 방법이 없기 때문입니다.

```tsx
// ✅ 이 프로젝트의 기본 패턴 — page.tsx도 클라이언트 컴포넌트
'use client'

import { useTodos } from '@/hooks/useTodos'

export default function TodosPage() {
  const { data, isLoading } = useTodos({ page: 1 })

  if (isLoading) return <TodoListSkeleton />
  return <TodoList todos={data?.content ?? []} />
}
```

서버 컴포넌트로 남겨도 되는 것은 인증·사용자 데이터와 무관한 순수 껍데기뿐입니다(루트 `layout.tsx` 등). `useState`, `useEffect`, 이벤트 핸들러, React Query 훅을 쓰는 컴포넌트는 전부 `'use client'`가 필요합니다.

```tsx
// ❌ 이 프로젝트에는 없는 패턴 — 서버에서 사용자 데이터를 페칭
export default async function UserDashboard() {
  const user = await getUser() // 서버는 토큰을 모른다
  return <div>{user.name}</div>
}

// ❌ Server Actions도 쓰지 않는다 — 쓰기는 React Query mutation + REST API 호출로 처리
async function createTodo(formData: FormData) {
  'use server'
  // 이 프로젝트에는 존재하지 않는 패턴
}
```

## Props 설계 패턴

### 1. Props Interface 정의

```tsx
// ✅ 명확한 Props 타입 정의
interface ButtonProps {
  children: React.ReactNode
  variant?: 'default' | 'destructive' | 'outline' | 'secondary' | 'ghost' | 'link'
  size?: 'default' | 'sm' | 'lg' | 'icon'
  disabled?: boolean
  loading?: boolean
  onClick?: () => void
  className?: string
}

export function Button({
  children,
  variant = 'default',
  size = 'default',
  disabled = false,
  loading = false,
  onClick,
  className,
  ...props
}: ButtonProps) {
  return (
    <button
      className={cn(buttonVariants({ variant, size }), className)}
      disabled={disabled || loading}
      onClick={onClick}
      {...props}
    >
      {loading ? <Spinner className="mr-2" /> : null}
      {children}
    </button>
  )
}
```

### 2. 제네릭 컴포넌트

```tsx
// ✅ 타입 안전한 제네릭 컴포넌트
interface SelectProps<T> {
  options: T[]
  value?: T
  onChange: (value: T) => void
  getLabel: (option: T) => string
  getValue: (option: T) => string
}

export function Select<T>({ options, value, onChange, getLabel, getValue }: SelectProps<T>) {
  return (
    <select
      value={value ? getValue(value) : ''}
      onChange={(e) => {
        const selected = options.find((option) => getValue(option) === e.target.value)
        if (selected) onChange(selected)
      }}
    >
      {options.map((option) => (
        <option key={getValue(option)} value={getValue(option)}>
          {getLabel(option)}
        </option>
      ))}
    </select>
  )
}
```

## 재사용성 패턴 — 컴포넌트 변형 (Variants)

```tsx
import { cva, type VariantProps } from 'class-variance-authority'

// ✅ CVA로 변형 정의
const cardVariants = cva('rounded-lg border bg-card text-card-foreground shadow-sm', {
  variants: {
    variant: {
      default: 'border-border',
      outline: 'border-2',
      ghost: 'border-transparent shadow-none',
    },
    size: {
      sm: 'p-4',
      md: 'p-6',
      lg: 'p-8',
    },
  },
  defaultVariants: { variant: 'default', size: 'md' },
})

interface CardProps extends VariantProps<typeof cardVariants> {
  children: React.ReactNode
  className?: string
}

export function Card({ variant, size, className, children }: CardProps) {
  return <div className={cn(cardVariants({ variant, size }), className)}>{children}</div>
}
```

## 성능 최적화 패턴

### 메모이제이션

```tsx
import { memo, useMemo, useCallback } from 'react'

// ✅ React.memo로 불필요한 리렌더링 방지
export const TodoItem = memo(function TodoItem({
  todo,
  onToggle,
}: {
  todo: Todo
  onToggle: (id: number) => void
}) {
  const handleToggle = useCallback(() => onToggle(todo.id), [todo.id, onToggle])

  return (
    <div>
      <span className={cn(todo.completed && 'text-muted-foreground line-through')}>
        {todo.title}
      </span>
      <button onClick={handleToggle}>완료</button>
    </div>
  )
})
```

목록이 큰 컴포넌트(Todo 목록 등)에서 항목 하나하나가 불필요하게 리렌더링되지 않도록, 콜백은 `useCallback`으로 감싸고 목록 항목 컴포넌트는 `memo`로 감쌉니다.

## 타입 안전성 패턴

### 조건부 타입

```tsx
// ✅ 조건부 props 타입
type ButtonProps<T extends boolean = false> = {
  children: React.ReactNode
  loading?: T
} & (T extends true
  ? { onClick?: never; disabled?: boolean }
  : { onClick: () => void; disabled?: boolean })
```

## 고급 패턴 — Hook 기반 상태 관리

```tsx
// ✅ 커스텀 훅으로 로직 분리
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue)
  const toggle = useCallback(() => setValue((prev) => !prev), [])
  return { value, toggle, setValue }
}

export function ConfirmDialog({ children }: { children: React.ReactNode }) {
  const { value: isOpen, toggle } = useToggle()
  return (
    <>
      <button onClick={toggle}>열기</button>
      {isOpen && <Dialog onClose={toggle}>{children}</Dialog>}
    </>
  )
}
```

## 안티패턴 및 금지사항

```tsx
// ❌ 불필요한 'use client' — 이 프로젝트는 어차피 대부분 클라이언트 컴포넌트이므로
//    이 항목의 실익은 적지만, 순수 서버 레이아웃 껍데기에는 여전히 붙이지 않는다
'use client'
export default function StaticFooter() {
  return <footer>© Todo App</footer> // 상태·이벤트가 없다면 서버 컴포넌트로 둔다
}

// ❌ 깊은 props drilling
function App() {
  const user = useUser()
  return <Level1 user={user} />
}
function Level1({ user }) {
  return <Level2 user={user} />
}

// ❌ 거대한 컴포넌트 (300줄 초과 시 분할 고려)
function GiantTodoForm() {
  // 500줄 이상의 JSX와 로직
}

// ❌ 인라인 객체/함수로 매 렌더링마다 새 참조 생성
function BadComponent() {
  return <ExpensiveComponent config={{ option: 'value' }} onUpdate={() => {}} />
}
```

## 컴포넌트 작성 체크리스트

- [ ] 단일 책임 원칙 준수
- [ ] Props 인터페이스 정의(`any` 금지, 필요 시 `unknown` + 타입가드)
- [ ] 상태·이벤트가 있으면 `'use client'`, 순수 정적 껍데기면 서버 컴포넌트로 유지
- [ ] 데이터 패칭은 React Query 훅 경유(컴포넌트 안에서 `fetch` 직접 호출 금지)
- [ ] 300줄 이하 유지, 컴포넌트 파일명은 PascalCase.tsx
- [ ] 목록 항목은 필요 시 `memo` + `useCallback`으로 리렌더링 최소화
