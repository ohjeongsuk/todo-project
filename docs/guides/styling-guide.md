# 스타일링 가이드

이 문서는 Tailwind CSS 4 + shadcn/ui를 활용한 스타일링 규칙과 모범 사례를 제공합니다.

> ⚠️ **스택 사실의 단일 출처는 `CLAUDE.md`(2·4장)다.** 이 가이드의 서술이 `CLAUDE.md`와 어긋나면 `CLAUDE.md`를 따른다.
> `components.json`의 `style`·`baseColor` 등 실제 설정값이 이 문서와 다르면 `components.json`이 사실이다.

## 기술 스택 개요

- **Tailwind CSS 4**: 유틸리티 기반 CSS 프레임워크. `@theme`으로 CSS-first 설정(v3의 `tailwind.config.js` 방식 아님)
- **shadcn/ui**: Radix 기반 컴포넌트 라이브러리. `components.json` 기준 `style`·`baseColor`를 따른다(설치 시점 값을 그대로 사실로 취급하고, 이 문서에서 임의로 다른 값을 상정하지 않는다)
- **lucide-react**: 아이콘
- **CSS 변수**: 색상·간격 등 디자인 토큰을 CSS 변수로 관리

## Tailwind CSS 4 사용 규칙

### 기본 원칙

```tsx
// ✅ 올바른 Tailwind 클래스 사용
<div className="flex items-center justify-between rounded-lg bg-background p-4 shadow-md">
  <h2 className="text-lg font-semibold text-foreground">제목</h2>
  <Button variant="outline" size="sm">버튼</Button>
</div>

// ❌ 인라인 스타일 사용 금지
<div style={{ display: 'flex', padding: '16px' }}>
```

클래스 순서는 `prettier-plugin-tailwindcss`가 자동 정렬합니다(설치되어 있다면 `package.json`의 devDependencies로 확인). 수동으로 신경 쓸 필요는 없습니다.

### 반응형 디자인

이 프로젝트는 **360px ~ 1920px** 범위에서 정상 동작해야 합니다(`UX-05`). 모바일 우선으로 작성합니다.

```tsx
<div className={cn(
  'flex flex-col space-y-4 p-4',        // 기본(모바일)
  'md:flex-row md:space-y-0 md:space-x-6 md:p-6',  // 태블릿
  'lg:max-w-6xl lg:mx-auto lg:p-8'      // 데스크톱
)}>
```

## 다크모드 — 미디어쿼리 전략 (필수)

**이 프로젝트에는 다크모드 토글 UI가 없습니다.** OS/브라우저의 `prefers-color-scheme` 설정만으로 자동 전환됩니다(`UX-07`). `next-themes` 같은 토글 라이브러리는 설치하지 않습니다 — 토글 대상이 없는데 상태 관리 라이브러리를 넣으면 죽은 코드가 됩니다.

```css
/* app/globals.css */
@theme {
  --color-background: #fafafa;
  --color-foreground: #0a0a0a;
  /* ... 라이트 모드 토큰 */
}

@media (prefers-color-scheme: dark) {
  @theme {
    --color-background: #0a0a0a;
    --color-foreground: #fafafa;
    /* ... 다크 모드 토큰 */
  }
}
```

```tsx
// ✅ 시맨틱 색상 변수 사용 — 미디어쿼리가 알아서 전환해준다
<div className="bg-background text-foreground">
  <h1 className="text-primary">제목</h1>
</div>

// ❌ 금지: class 전략 (`.dark` 셀렉터, `@custom-variant dark (&:is(.dark *))`)
//    토글 UI가 없는데 클래스 전략을 쓰면 아무 것도 전환되지 않거나,
//    OS가 다크인데 화면은 라이트로 굳어버리는 FOUC만 남는다.
```

```tsx
// ❌ 금지: next-themes의 ThemeProvider/useTheme/토글 버튼
//    이 프로젝트에는 테마를 전환할 UI 자체가 없다.
import { ThemeProvider } from 'next-themes' // 이 프로젝트에서 쓰지 않음
```

> shadcn/ui `init`이 만드는 기본 `globals.css`는 보통 `class` 전략(`.dark` 셀렉터 또는 `@custom-variant dark (&:is(.dark *))`)으로 생성됩니다. 초기화 직후에는 반드시 이 절의 미디어쿼리 방식으로 바꿔야 합니다 — 그대로 두면 이 문서의 규범과 어긋난 상태로 남습니다.

## 색상 시스템

색상 팔레트(배경·액센트·우선순위 3색 등)는 `CLAUDE.md`에 확정된 값이 있으면 그 값을, 없다면 `app/globals.css`의 `@theme` 토큰을 정본으로 삼습니다. 이 문서에 임의의 hex 값을 못 박지 않습니다 — 토큰 값 자체가 바뀌어도 이 가이드를 다시 고칠 필요가 없도록, 색상은 항상 CSS 변수(시맨틱 이름)로만 참조합니다.

```tsx
// ✅ 시맨틱 색상 클래스 사용
<div className="bg-background border-border">
  <h1 className="text-foreground">메인 텍스트</h1>
  <p className="text-muted-foreground">보조 텍스트</p>
</div>

// ❌ 직접 색상 지정 — 다크모드에서 깨진다
<div className="bg-white border-gray-200">
  <h1 className="text-gray-900">메인 텍스트</h1>
</div>
```

## shadcn/ui 컴포넌트 활용

```tsx
import { Button } from '@/components/ui/button'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'

export function TodoCard({ todo }: { todo: Todo }) {
  return (
    <Card>
      <CardHeader>
        <CardTitle>{todo.title}</CardTitle>
      </CardHeader>
      <CardContent>
        <Button variant="outline">상세 보기</Button>
      </CardContent>
    </Card>
  )
}
```

새 컴포넌트를 추가할 때는 `npx shadcn@latest add [component-name]`을 씁니다. 단, **`form` 컴포넌트는 추가하지 않습니다** — `react-hook-form`을 함께 설치하는데, 이 프로젝트는 폼 라이브러리를 쓰지 않기로 확정했습니다(`CLAUDE.md` 3장, `forms.md` 참조).

```bash
# ✅ 개별 컴포넌트 추가
npx shadcn@latest add button
npx shadcn@latest add dialog

# ❌ 금지 — react-hook-form 유입 경로
npx shadcn@latest add form
```

## 유틸리티 함수

```tsx
import { cn } from '@/lib/utils'

// ✅ cn()으로 클래스 조합
<div className={cn(
  'base-classes',
  condition && 'conditional-classes',
  className
)}>

// ❌ 수동 문자열 조합
<div className={`base-classes ${condition ? 'conditional-classes' : ''}`}>
```

## 금지사항

```tsx
// 인라인 스타일 사용
<div style={{ backgroundColor: 'red' }}>

// 하드코딩된 색상 (다크모드 미고려)
<div className="bg-white text-black">

// !important 남용
<div className="!text-red-500">
```

## 스타일링 체크리스트

- [ ] Tailwind 유틸리티 클래스 우선 사용, 인라인 스타일 없음
- [ ] 시맨틱 색상 변수 사용, 하드코딩된 색상 없음
- [ ] `prefers-color-scheme` 미디어쿼리로만 다크모드 전환(토글 UI·`class`·`next-themes` 없음)
- [ ] 360px ~ 1920px 반응형 확인
- [ ] 충분한 색상 대비, 포커스 상태 스타일링
