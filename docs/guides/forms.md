# 폼 처리 가이드 (라이브러리 없는 수동 검증)

이 문서는 이 프로젝트가 폼을 처리하는 방식을 정리합니다.

> ⚠️ **스택 사실의 단일 출처는 `CLAUDE.md`(3·4장)다.** 이 가이드의 서술이 `CLAUDE.md`와 어긋나면 `CLAUDE.md`를 따른다.

## 왜 라이브러리를 쓰지 않는가

`react-hook-form`, `zod`, `@hookform/resolvers`는 이 프로젝트에 **설치하지 않습니다**(`CLAUDE.md` 3장에서 확정). 이 프로젝트의 폼은 로그인·회원가입·Todo 작성/수정 정도로 개수와 필드가 제한적이라, 폼 상태 관리 라이브러리 없이 `useState` + 직접 검증으로 충분합니다.

```bash
# ❌ 설치 금지 — 필요하면 먼저 이유와 함께 제안하고 승인받는다 (CLAUDE.md 절대 규칙 11)
npm install react-hook-form zod @hookform/resolvers

# ❌ shadcn form 컴포넌트도 금지 — react-hook-form이 함께 딸려온다
npx shadcn@latest add form
```

## 검증 규칙은 `lib/validation.ts` 한 곳에 모은다

폼 라이브러리가 없으므로 검증 로직이 화면마다 흩어지기 쉽습니다. `CLAUDE.md` 4장의 입력값 제약 표를 그대로 코드로 옮긴 `lib/validation.ts`를 두고, 모든 화면이 이 함수만 가져다 씁니다.

```typescript
// lib/validation.ts
export function validateEmail(value: string): string | null {
  if (!value) return '이메일을 입력해주세요.'
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) return '올바른 이메일 형식이 아닙니다.'
  return null
}

export function validatePassword(value: string): string | null {
  if (value.length < 6) return '비밀번호는 6자 이상이어야 합니다.'
  const byteLength = new TextEncoder().encode(value).length
  if (byteLength > 72) return '비밀번호가 너무 깁니다. (한글 1자 = 3바이트, 최대 72바이트)'
  return null
}

export function validateTitle(value: string): string | null {
  if (!value.trim()) return '제목을 입력해주세요.'
  if (value.length > 200) return '제목은 200자 이하로 입력해주세요.'
  return null
}
```

> ⚠️ **비밀번호 상한은 문자 수가 아니라 바이트다.** `maxLength={64}` 같은 문자 수 제한만 걸면 한글 25자(=75바이트)가 통과해 서버 BCrypt 단계에서 400 에러를 냅니다. `new TextEncoder().encode(v).length`로 UTF-8 바이트 수를 세야 합니다. 안내 문구에도 "한글 1자 = 3바이트"임을 명시해 사용자가 왜 막히는지 알 수 있게 합니다(`AUTH-02`).

## 필드 상태 관리 — `useState`로 직접

각 필드를 개별 `useState`로 관리하고, 제출 시점 또는 `onBlur`에서 검증 함수를 호출해 에러 메시지를 별도 state에 저장합니다. 과도한 추상화(자체 `useForm` 훅 등)를 만들지 않고, 화면마다 필요한 필드만 직접 선언합니다.

```tsx
// app/(auth)/signup/page.tsx
'use client'

import { useState } from 'react'
import { validateEmail, validatePassword } from '@/lib/validation'
import { useSignup } from '@/hooks/useSignup'

export default function SignupPage() {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [emailError, setEmailError] = useState<string | null>(null)
  const [passwordError, setPasswordError] = useState<string | null>(null)

  const signup = useSignup()

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault()

    const emailErr = validateEmail(email)
    const passwordErr = validatePassword(password)
    setEmailError(emailErr)
    setPasswordError(passwordErr)
    if (emailErr || passwordErr) return

    signup.mutate({ email, password })
  }

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label htmlFor="email">이메일</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          onBlur={() => setEmailError(validateEmail(email))}
        />
        {emailError && <p className="text-destructive text-sm">{emailError}</p>}
      </div>

      <div>
        <label htmlFor="password">비밀번호</label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          onBlur={() => setPasswordError(validatePassword(password))}
        />
        {passwordError && <p className="text-destructive text-sm">{passwordError}</p>}
      </div>

      <button type="submit" disabled={signup.isPending}>
        {signup.isPending ? '가입 중...' : '가입하기'}
      </button>
    </form>
  )
}
```

`onBlur`에서 실시간 검증(`AUTH-02` — 제출 전 인라인 안내)을 하고, `onSubmit`에서 다시 한 번 전체 검증 후 API를 호출합니다. `onSubmit`에서만 검증하면 이미 입력을 끝낸 필드를 사용자가 다시 건드리지 않는 한 오류를 보지 못합니다.

## React Query mutation과 결합해 제출 처리

폼 제출은 React Query의 `useMutation`으로 감싼 훅을 통해 처리합니다. 컴포넌트가 직접 `fetch`나 `lib/api/`를 호출하지 않습니다.

```typescript
// hooks/useSignup.ts
import { useMutation } from '@tanstack/react-query'
import { useRouter } from 'next/navigation'
import { signup } from '@/lib/api/auth'
import { saveToken } from '@/lib/auth/token'

export function useSignup() {
  const router = useRouter()

  return useMutation({
    mutationFn: signup,
    onSuccess: (res) => {
      saveToken(res.data.accessToken)
      router.push('/todos')
    },
  })
}
```

## 에러 표시 — `lib/errorMessages.ts`를 통해서만

서버가 반환한 `error.code`(`INVALID_INPUT`, `EMAIL_DUPLICATED` 등)나 네트워크 실패를 화면 문구로 바꾸는 책임은 Phase 6에서 만드는 `lib/errorMessages.ts` 단일 함수가 맡습니다. 화면(컴포넌트)에서 직접 에러 문구를 하드코딩하지 않습니다 — 그러면 화면마다 문구가 달라져 매핑 표가 사문화됩니다.

```tsx
// mutation의 onError에서 lib/errorMessages.ts를 거쳐 문구를 얻는 형태
onError: (error) => {
  const message = toErrorMessage(error) // lib/errorMessages.ts
  setFormError(message)
}
```

이 문서는 `lib/errorMessages.ts`의 구체적인 구현까지 다루지 않습니다. 해당 파일이 만들어지면 그 문서를 참조하십시오.

## 제출 중 상태

`useMutation`이 제공하는 `isPending`을 그대로 버튼 비활성화·로딩 텍스트에 사용합니다. 별도의 `isSubmitting` state를 만들 필요가 없습니다.

```tsx
<button type="submit" disabled={mutation.isPending}>
  {mutation.isPending ? '저장 중...' : '저장'}
</button>
```

## 체크리스트

- [ ] `lib/validation.ts`의 함수로만 검증하고, 화면에 검증 로직을 직접 작성하지 않았는지
- [ ] 비밀번호 검증이 문자 수뿐 아니라 UTF-8 바이트 수(72바이트)도 확인하는지
- [ ] `onBlur`(실시간) + `onSubmit`(제출 시) 이중으로 검증하는지
- [ ] 제출은 React Query `useMutation` 훅을 통해서만 하는지(컴포넌트에서 `fetch` 직접 호출 금지)
- [ ] 에러 문구는 `lib/errorMessages.ts`를 거쳐서만 표시하는지
- [ ] `react-hook-form`/`zod`/`@hookform/resolvers`가 `package.json`에 없는지
