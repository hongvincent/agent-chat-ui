# CLAUDE.md - Agent Chat UI 코드베이스 가이드 (AI 어시스턴트용)

*[English](./CLAUDE.md) | 한국어*

이 문서는 AI 어시스턴트가 Agent Chat UI 코드베이스를 작업할 때 필요한 종합적인 가이드를 제공합니다. 아키텍처, 컨벤션, 패턴 및 모범 사례를 다룹니다.

## 목차

1. [프로젝트 개요](#프로젝트-개요)
2. [코드베이스 구조](#코드베이스-구조)
3. [개발 환경](#개발-환경)
4. [아키텍처 및 패턴](#아키텍처-및-패턴)
5. [코딩 컨벤션](#코딩-컨벤션)
6. [컴포넌트 패턴](#컴포넌트-패턴)
7. [상태 관리](#상태-관리)
8. [API 통합](#api-통합)
9. [스타일링 접근 방식](#스타일링-접근-방식)
10. [일반적인 개발 작업](#일반적인-개발-작업)
11. [테스트 및 품질](#테스트-및-품질)
12. [주요 파일 참조](#주요-파일-참조)

---

## 프로젝트 개요

**Agent Chat UI**는 LangGraph 에이전트를 위한 고도로 커스터마이징 가능한 채팅 인터페이스를 제공하는 프로덕션 수준의 Next.js 15 애플리케이션입니다. 원본 LangChain 프로젝트의 포크로, 추가적인 UX/UI 기능이 강화되었습니다.

### 기술 스택

- **프레임워크**: Next.js 15.2.3 (App Router)
- **런타임**: React 19.0.0
- **언어**: TypeScript 5.7.2 (strict mode)
- **스타일링**: Tailwind CSS 4.0.13
- **UI 컴포넌트**: Radix UI primitives (shadcn/ui 패턴)
- **에이전트 통합**: LangGraph SDK 0.2.63
- **상태 관리**: React Context + nuqs (URL 상태)
- **패키지 매니저**: pnpm 10.5.1
- **애니메이션**: Framer Motion 12.4.9

### 주요 기능

- LangGraph 에이전트와 실시간 스트리밍 채팅
- 파일 업로드 지원 (이미지, PDF)
- 스레드/대화 기록 관리
- 사이드 패널에서 아티팩트 렌더링
- 에이전트 상호작용을 위한 중단(Interrupt) 처리
- YAML 기반 런타임 구성
- 다크 모드 지원
- 반응형 디자인 (모바일/데스크톱)
- 수식(KaTeX) 및 코드 하이라이팅을 포함한 마크다운 렌더링

---

## 코드베이스 구조

### 디렉터리 레이아웃

```
/home/user/agent-chat-ui/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── api/[..._path]/    # 모든 요청을 처리하는 API 프록시 라우트
│   │   ├── layout.tsx         # NuqsAdapter가 있는 루트 레이아웃
│   │   ├── page.tsx           # 메인 채팅 페이지
│   │   └── globals.css        # 전역 스타일 (Tailwind + CSS 변수)
│   │
│   ├── components/            # React 컴포넌트
│   │   ├── ui/               # shadcn/ui primitives (22개 컴포넌트)
│   │   │   ├── button.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── input.tsx
│   │   │   └── ...
│   │   │
│   │   ├── thread/           # 채팅 스레드 컴포넌트
│   │   │   ├── index.tsx     # 메인 Thread 오케스트레이터 (655줄)
│   │   │   ├── artifact.tsx  # 포털 기반 아티팩트 시스템
│   │   │   │
│   │   │   ├── messages/     # 메시지 렌더링
│   │   │   │   ├── ai.tsx           # AI 메시지 표시
│   │   │   │   ├── human.tsx        # 사용자 메시지 표시
│   │   │   │   ├── tool-calls.tsx   # 도구 실행 표시
│   │   │   │   └── shared.tsx       # 공유 메시지 컴포넌트
│   │   │   │
│   │   │   ├── history/      # 스레드 히스토리 사이드바
│   │   │   │   ├── index.tsx
│   │   │   │   ├── ThreadList.tsx
│   │   │   │   ├── DesktopSidebar.tsx
│   │   │   │   ├── MobileSidebar.tsx
│   │   │   │   └── components/thread-item/
│   │   │   │
│   │   │   └── agent-inbox/  # 중단(Interrupt) 처리
│   │   │       └── components/
│   │   │
│   │   ├── settings/         # 설정 다이얼로그
│   │   └── icons/            # 커스텀 SVG 아이콘
│   │
│   ├── hooks/                # 커스텀 React 훅
│   │   ├── useMediaQuery.tsx        # 반응형 중단점
│   │   └── use-file-upload.tsx      # 파일 업로드 관리 (168줄)
│   │
│   ├── lib/                  # 유틸리티 및 API 클라이언트
│   │   ├── utils.ts                 # Tailwind용 cn() 유틸리티
│   │   ├── config.ts                # YAML 설정 로더
│   │   ├── assistant-api.ts         # Assistant API 함수
│   │   ├── multimodal-utils.ts      # 파일/이미지 처리
│   │   ├── constants.ts             # 앱 상수
│   │   └── ...
│   │
│   └── providers/            # React Context 프로바이더
│       ├── Stream.tsx        # LangGraph 스트리밍 (308줄)
│       ├── Thread.tsx        # 스레드 목록 관리
│       ├── Settings.tsx      # 앱 구성 + 사용자 설정
│       ├── AssistantConfig.tsx      # Assistant별 설정
│       └── client.ts         # LangGraph 클라이언트 생성
│
├── public/                   # 정적 자산
│   ├── settings.yaml        # 런타임 앱 구성 ⚠️ 중요
│   ├── full-description.md  # 사용자 가이드 (랜딩 페이지에 표시)
│   ├── logo.png            # 앱 로고
│   └── ...
│
├── .env.example             # 환경 변수 템플릿
├── next.config.mjs         # Next.js 구성 (10mb body limit)
├── tsconfig.json           # TypeScript 설정 (strict mode, @ 별칭)
├── tailwind.config.js      # Tailwind 구성
├── eslint.config.js        # ESLint flat config
├── prettier.config.js      # Prettier 구성
├── components.json         # shadcn/ui 구성
└── package.json            # 의존성 및 스크립트
```

### 경로 별칭

코드베이스는 `tsconfig.json`에 설정된 경로 별칭을 사용합니다:

```typescript
// 임포트 패턴:
import { Button } from "@/components/ui/button";
import { useSettings } from "@/providers/Settings";
import { cn } from "@/lib/utils";
```

**항상 `@/` 임포트를 사용하세요** - 상대 경로 대신 사용하면 더 깔끔하고 유지보수하기 쉬운 코드가 됩니다.

---

## 개발 환경

### 전제 조건

- **Node.js**: 18.x 이상
- **pnpm**: 10.x (필수 - npm 또는 yarn 사용 금지)
- **백엔드**: LangGraph 서버가 http://localhost:2024에서 실행 중이어야 함

### 초기 설정

1. **의존성 설치**:
   ```bash
   pnpm install
   ```

2. **환경 변수 구성**:
   ```bash
   cp .env.example .env
   ```

   `.env` 편집:
   ```env
   # 로컬 개발
   NEXT_PUBLIC_API_URL=http://localhost:2024
   NEXT_PUBLIC_ASSISTANT_ID=agent

   # 서버 측 전용 (NEXT_PUBLIC_로 접두사 사용 금지)
   LANGSMITH_API_KEY=lsv2_...
   ```

3. **개발 서버 시작**:
   ```bash
   pnpm dev
   ```

### NPM 스크립트

```bash
pnpm dev           # Next.js 개발 서버 시작 (http://localhost:3000)
pnpm build         # 프로덕션 빌드 (.next 디렉터리)
pnpm start         # 프로덕션 서버 시작
pnpm lint          # ESLint 검사 실행
pnpm lint:fix      # ESLint 이슈 자동 수정
pnpm format        # Prettier로 코드 포맷팅
pnpm format:check  # 코드 포맷팅 검사
```

### 환경 변수

#### Public 변수 (브라우저에 노출됨)
- `NEXT_PUBLIC_API_URL`: API 엔드포인트 (로컬: http://localhost:2024, 프로덕션: https://yoursite.com/api)
- `NEXT_PUBLIC_ASSISTANT_ID`: Assistant/graph ID (기본값: "agent")

#### Private 변수 (서버 측 전용)
- `LANGGRAPH_API_URL`: LangGraph 배포 URL
- `LANGSMITH_API_KEY`: LangSmith API 키 (lsv2_로 시작)

**중요**: 서버 전용 변수에는 절대 `NEXT_PUBLIC_` 접두사를 사용하지 마세요 - 클라이언트에 노출됩니다!

---

## 아키텍처 및 패턴

### 1. Next.js App Router 패턴

이 프로젝트는 **App Router**를 사용합니다 (Pages Router 아님). 주요 차이점:

```typescript
// 파일 시스템 라우팅:
src/app/page.tsx          → /
src/app/layout.tsx        → 루트 레이아웃
src/app/api/[..._path]/route.ts → /api/* (catch-all)
```

- **기본적으로 Server Components**: 필요한 경우에만 `"use client"` 지시어 사용
- **getServerSideProps/getStaticProps 없음**: 대신 async Server Components 사용
- **Layouts**: 라우트 간에 유지되는 공유 UI

### 2. API 프록시 패턴

모든 LangGraph API 호출은 Next.js edge 함수 프록시를 통과합니다:

```typescript
// src/app/api/[..._path]/route.ts
export const { GET, POST, PUT, PATCH, DELETE, OPTIONS } =
  initApiPassthrough({
    apiUrl: process.env.LANGGRAPH_API_URL,
    apiKey: process.env.LANGSMITH_API_KEY,
    runtime: "edge",
  });
```

**목적**:
- 클라이언트로부터 API 키 숨기기
- CORS 활성화
- 글로벌 성능을 위한 Edge 런타임

**클라이언트 측 사용법**:
```typescript
const client = new Client({
  apiUrl: NEXT_PUBLIC_API_URL,  // LangGraph가 아닌 /api를 가리킴
});
```

### 3. 포털 기반 아티팩트 시스템

아티팩트 패널은 **헤드리스 포털 패턴**을 사용하여 prop drilling을 방지합니다:

```typescript
// src/components/thread/artifact.tsx
export function useArtifact() {
  // 튜플 반환: [Component, { open, setOpen, title, setTitle }]
}

// Thread 컴포넌트에서 사용:
const [ArtifactContent, artifact] = useArtifact();

// 컴포넌트 트리 깊숙한 곳에서:
<ArtifactContent>
  <CodeEditor code={artifactCode} />
</ArtifactContent>
```

**장점**:
- 여러 계층을 통한 prop drilling 없음
- 중앙 집중식 아티팩트 상태
- 깔끔한 컴포넌트 계층 구조

### 4. 스트리밍 + 낙관적 업데이트

Stream 프로바이더는 실시간 LangGraph 스트리밍을 처리합니다:

```typescript
// src/providers/Stream.tsx
const stream = useTypedStream({
  apiUrl, apiKey, assistantId, threadId,
  fetchStateHistory: true,
  onCustomEvent: handleCustomEvent,
});

// 즉각적인 UI 피드백을 위한 낙관적 업데이트:
stream.submit(input, {
  optimisticValues: (prev) => ({
    ...prev,
    messages: [...prev.messages, newMessage],
  }),
});
```

### 5. DO_NOT_RENDER 패턴

LangGraph 요구 사항을 충족하면서 UI에서 메시지를 숨기는 특수 접두사:

```typescript
// src/lib/constants.ts
export const DO_NOT_RENDER_ID_PREFIX = "do-not-render-";

// 사용법:
const toolResponseId = `${DO_NOT_RENDER_ID_PREFIX}${uuid()}`;
```

컴포넌트는 이 접두사를 확인하고 렌더링을 건너뜁니다.

### 6. YAML 기반 구성

`public/settings.yaml`의 런타임 구성을 통해 재빌드 없이 커스터마이징 가능:

```typescript
// src/lib/config.ts
export async function loadConfig(): Promise<ChatConfig> {
  const response = await fetch("/settings.yaml");
  const yaml = await response.text();
  return parseYaml(yaml);
}
```

**중요**: `settings.yaml` 변경 사항은 즉시 적용됩니다 (캐시 지우기 위해 하드 리프레시 필요할 수 있음).

### 7. 드래그 카운터 패턴

파일 업로드는 자식 요소 위로 호버할 때 깜빡임을 방지하기 위해 드래그 카운터를 사용합니다:

```typescript
// src/hooks/use-file-upload.tsx
const dragCounter = useRef(0);

const handleDragEnter = (e: DragEvent) => {
  e.preventDefault();
  dragCounter.current++;
  if (dragCounter.current === 1) setIsDragging(true);
};

const handleDragLeave = (e: DragEvent) => {
  e.preventDefault();
  dragCounter.current--;
  if (dragCounter.current === 0) setIsDragging(false);
};
```

### 8. 미디어 쿼리를 사용한 반응형 디자인

중단점에 대한 단일 진실 공급원:

```typescript
// src/hooks/useMediaQuery.tsx
export function useMediaQuery(query: string): boolean

// 사용법:
const isLargeScreen = useMediaQuery("(min-width: 1024px)");
```

---

## 코딩 컨벤션

### TypeScript 표준

1. **Strict Mode**: 항상 활성화 (`tsconfig.json`에서 `"strict": true`)
   - 암시적 any 없음
   - 엄격한 null 검사
   - 엄격한 함수 타입

2. **명시적 타입**: 내보낸 함수에는 명시적 반환 타입 선호
   ```typescript
   // 좋음
   export function getAssistant(id: string): Promise<Assistant> {
     // ...
   }

   // 피하기
   export function getAssistant(id: string) {
     // ...
   }
   ```

3. **타입 가드**: 런타임 검증에 사용
   ```typescript
   export function isBase64ContentBlock(block: unknown): block is Base64ContentBlock {
     return (
       typeof block === "object" &&
       block !== null &&
       "type" in block &&
       block.type === "image"
     );
   }
   ```

4. **Interface vs Type**:
   - 객체 형태에는 `interface` 사용 (확장 가능)
   - 유니온, 교차, 원시 타입에는 `type` 사용

5. **`any` 사용 금지**: ESLint 설정에서 허용되지만 권장하지 않음 - 대신 `unknown` 사용

### React 컨벤션

1. **Client Components**: 필요한 경우 파일 상단에 `"use client"` 지시어 사용
   ```typescript
   "use client";

   import { useState } from "react";
   ```

2. **컴포넌트 구조**:
   ```typescript
   // 1. 임포트 (React, 외부, 내부, 타입)
   // 2. 타입 정의
   // 3. 컴포넌트 정의
   // 4. 내보내기

   "use client";

   import React from "react";
   import { Button } from "@/components/ui/button";
   import type { Message } from "@langchain/langgraph-sdk";

   interface Props {
     message: Message;
   }

   export function MessageComponent({ message }: Props) {
     // 훅 먼저
     const [expanded, setExpanded] = useState(false);

     // 이벤트 핸들러
     const handleClick = () => setExpanded(!expanded);

     // 렌더
     return <div onClick={handleClick}>{/* ... */}</div>;
   }
   ```

3. **훅 순서**:
   - 상태 훅 (`useState`, `useReducer`)
   - Context 훅 (`useContext`, 커스텀 context 훅)
   - Ref 훅 (`useRef`)
   - Effect 훅 (`useEffect`, `useLayoutEffect`)
   - 커스텀 훅
   - 파생 값 (useMemo, useCallback)

4. **Props 명명**:
   - 이벤트 핸들러: `on[Event]` (예: `onClick`, `onSubmit`)
   - Boolean props: `is[State]`, `has[State]`, `enable[Feature]`
   - Render props: `render[Element]` 또는 요소 이름만

### ESLint 구성

```javascript
// eslint.config.js (flat config 형식)
rules: {
  ...reactHooks.configs.recommended.rules,
  "@typescript-eslint/no-explicit-any": 0,  // 경고 (오류 아님)
  "@typescript-eslint/no-unused-vars": ["warn", {
    args: "none",
    argsIgnorePattern: "^_",
    varsIgnorePattern: "^_"
  }],
  "react-refresh/only-export-components": ["warn", {
    allowConstantExport: true
  }],
}
```

**주요 사항**:
- `_`로 시작하는 미사용 변수는 무시됨
- React Hooks 규칙 강제 적용
- Fast Refresh를 위해 컴포넌트를 내보내야 함

### Prettier 구성

```javascript
// prettier.config.js
{
  endOfLine: "auto",
  singleAttributePerLine: true,  // 각 JSX prop을 별도 줄에
  plugins: ["prettier-plugin-tailwindcss"],  // Tailwind 클래스 자동 정렬
}
```

**항상 실행하세요** 코드 커밋 전에 `pnpm format`.

### 파일 명명

- **컴포넌트**: PascalCase (`ThreadList.tsx`, `ArtifactPanel.tsx`)
- **훅**: `use` 접두사와 함께 camelCase (`useMediaQuery.tsx`, `use-file-upload.tsx`)
- **Utils/Libs**: kebab-case (`multimodal-utils.ts`, `file-validation.ts`)
- **타입**: PascalCase (`Message.ts`, `AssistantConfig.ts`)
- **상수**: kebab-case (`constants.ts`)

### 임포트 구성

```typescript
// 1. React 임포트
import React, { useState, useEffect } from "react";

// 2. 외부 라이브러리 (알파벳순)
import { Button } from "@/components/ui/button";
import { motion } from "framer-motion";
import { Loader2 } from "lucide-react";

// 3. 내부 임포트 (@ 별칭, 알파벳순)
import { useSettings } from "@/providers/Settings";
import { cn } from "@/lib/utils";

// 4. 타입 (값과 분리)
import type { Message } from "@langchain/langgraph-sdk";
import type { ChatConfig } from "@/lib/config";
```

---

## 컴포넌트 패턴

### shadcn/ui 패턴

모든 UI 컴포넌트는 shadcn/ui 패턴을 따릅니다:

```typescript
// src/components/ui/button.tsx
import * as React from "react";
import { Slot } from "@radix-ui/react-slot";
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

const buttonVariants = cva(
  "inline-flex items-center justify-center...",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground...",
        destructive: "bg-destructive text-destructive-foreground...",
        outline: "border border-input...",
        // ...
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : "button";
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  }
);
Button.displayName = "Button";

export { Button, buttonVariants };
```

**주요 특성**:
- 변형 관리를 위해 `class-variance-authority` (cva) 사용
- 접근성을 위한 Radix UI primitives
- 부모 컴포넌트 접근을 위한 Forward refs
- 컴포지션을 위한 `asChild` prop
- 클래스 병합을 위한 `cn()` 유틸리티

### 새 UI 컴포넌트 추가

shadcn/ui CLI를 사용하거나 (사용 가능한 경우) 패턴을 따라 수동으로 생성:

```bash
# shadcn CLI가 사용 가능한 경우:
npx shadcn-ui@latest add [component-name]

# 컴포넌트는 다음을 사용하도록 구성됨:
# - 스타일: New York
# - 색상: Neutral
# - Tailwind config: tailwind.config.js
# - CSS 변수: src/app/globals.css
```

### 커스텀 컴포넌트 예제

```typescript
// src/components/thread/MessageBubble.tsx
"use client";

import React from "react";
import { motion } from "framer-motion";
import { cn } from "@/lib/utils";
import type { Message } from "@langchain/langgraph-sdk";

interface MessageBubbleProps {
  message: Message;
  isUser: boolean;
  className?: string;
}

export function MessageBubble({
  message,
  isUser,
  className
}: MessageBubbleProps) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.3 }}
      className={cn(
        "rounded-lg p-4",
        isUser ? "bg-primary text-primary-foreground" : "bg-muted",
        className
      )}
    >
      {message.content}
    </motion.div>
  );
}
```

**패턴 노트**:
- 컴포넌트 전에 Props 인터페이스 정의
- 커스터마이징을 위한 `className` prop
- 조건부 클래스를 위한 `cn()`
- 애니메이션을 위한 Motion
- 시맨틱 HTML

---

## 상태 관리

### 프로바이더 아키텍처

앱은 서로 다른 관심사에 대해 React Context 프로바이더를 사용합니다:

#### 1. Stream Provider (LangGraph 통합)

```typescript
// src/providers/Stream.tsx
export function StreamProvider({ children }: { children: ReactNode }) {
  const stream = useTypedStream({
    apiUrl, apiKey, assistantId, threadId,
    fetchStateHistory: true,
    onCustomEvent: (event) => {
      // 커스텀 이벤트 처리 (제목 생성 등)
    },
  });

  return (
    <StreamContext.Provider value={stream}>
      {children}
    </StreamContext.Provider>
  );
}

export function useStream() {
  return useContext(StreamContext);
}
```

**사용법**:
```typescript
const stream = useStream();
stream.submit(input);
stream.regenerate();
stream.setBranch(branch);
```

#### 2. Settings Provider (구성 + 사용자 환경설정)

```typescript
// src/providers/Settings.tsx
export function SettingsProvider({ children }: { children: ReactNode }) {
  const [config, setConfig] = useState<ChatConfig | null>(null);
  const [userSettings, setUserSettings] = useLocalStorage("userSettings");

  return (
    <SettingsContext.Provider value={{
      config,
      userSettings,
      updateUserSettings,
      resetUserSettings
    }}>
      {children}
    </SettingsContext.Provider>
  );
}

export function useSettings() {
  return useContext(SettingsContext);
}
```

**2계층 설정**:
- `config`: YAML 기반 앱 구성 (`settings.yaml`에서 읽기 전용)
- `userSettings`: 사용자 환경설정 (localStorage에 저장)

#### 3. Thread Provider (대화 관리)

```typescript
// src/providers/Thread.tsx
export function ThreadProvider({ children }: { children: ReactNode }) {
  const [threads, setThreads] = useState<Thread[]>([]);

  const getThreads = useCallback(async () => {
    const result = await client.threads.list();
    setThreads(result);
  }, [client]);

  return (
    <ThreadContext.Provider value={{ threads, getThreads, setThreads }}>
      {children}
    </ThreadContext.Provider>
  );
}
```

#### 4. Assistant Config Provider

```typescript
// src/providers/AssistantConfig.tsx
export function AssistantConfigProvider({ children }: { children: ReactNode }) {
  const [config, setConfig] = useState<Assistant | null>(null);
  const [schemas, setSchemas] = useState<AssistantSchemas | null>(null);

  return (
    <AssistantConfigContext.Provider value={{
      config,
      schemas,
      updateConfig,
      refetchConfig
    }}>
      {children}
    </AssistantConfigContext.Provider>
  );
}
```

### URL 상태 관리 (nuqs)

공유 가능/북마크 가능한 상태에는 nuqs 사용:

```typescript
import { useQueryState, parseAsBoolean } from "nuqs";

// 간단한 문자열 상태
const [threadId, setThreadId] = useQueryState("threadId");

// 기본값이 있는 파싱된 상태
const [hideToolCalls, setHideToolCalls] = useQueryState(
  "hideToolCalls",
  parseAsBoolean.withDefault(false)
);

// URL 상태 업데이트
setThreadId("thread-123");  // URL: ?threadId=thread-123
setHideToolCalls(true);     // URL: ?threadId=thread-123&hideToolCalls=true
```

**루트 레이아웃에 NuqsAdapter 필요**:

```typescript
// src/app/layout.tsx
import { NuqsAdapter } from "nuqs/adapters/next/app";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <NuqsAdapter>{children}</NuqsAdapter>
      </body>
    </html>
  );
}
```

### Local Storage

지속적인 사용자 환경설정:

```typescript
// 패턴: localStorage 확인, 기본값으로 폴백
const savedSettings = localStorage.getItem("userSettings");
const userSettings = savedSettings ? JSON.parse(savedSettings) : defaultSettings;

// 업데이트
localStorage.setItem("userSettings", JSON.stringify(newSettings));
```

**사용 용도**:
- 사용자 테마 환경설정
- 글꼴 크기/패밀리 재정의
- UI 환경설정 (사이드바 상태 등)

---

## API 통합

### LangGraph 클라이언트 설정

```typescript
// src/providers/client.ts
import { Client } from "@langchain/langgraph-sdk";

export function createClient(apiUrl: string, apiKey?: string) {
  return new Client({ apiKey, apiUrl });
}

// 사용법
const client = createClient(
  process.env.NEXT_PUBLIC_API_URL!,
  process.env.LANGSMITH_API_KEY
);
```

### Assistant API 함수

```typescript
// src/lib/assistant-api.ts

// ID 또는 graph_id로 assistant 가져오기
export async function getAssistant(
  client: Client,
  assistantId: string
): Promise<Assistant> {
  // 먼저 assistant_id로 시도
  const assistant = await client.assistants.get(assistantId);
  if (assistant) return assistant;

  // graph_id 검색으로 폴백
  return await searchAssistants(client, assistantId);
}

// assistant 구성 업데이트
export async function updateAssistantConfig(
  client: Client,
  assistantId: string,
  config: Partial<AssistantConfig>
): Promise<Assistant> {
  return await client.assistants.update(assistantId, { config });
}

// 입력/출력 스키마 가져오기
export async function getAssistantSchemas(
  client: Client,
  graphId: string
): Promise<AssistantSchemas> {
  return await client.assistants.getSchemas(graphId);
}
```

### 스레드 작업

```typescript
// 새 스레드 생성
const thread = await client.threads.create();

// 스레드 상태 가져오기
const state = await client.threads.getState(threadId);

// 스레드 목록
const threads = await client.threads.list({ limit: 50 });

// 스레드 메타데이터 업데이트
await client.threads.update(threadId, {
  metadata: { title: "New Title" }
});

// 스레드 삭제
await client.threads.delete(threadId);
```

### 메시지 스트리밍

```typescript
// src/providers/Stream.tsx
const stream = useTypedStream({
  apiUrl,
  apiKey,
  assistantId,
  threadId,
  fetchStateHistory: true,

  // 커스텀 이벤트 처리 (예: 제목 생성)
  onCustomEvent: (event) => {
    if (event.event === "title_generated") {
      setTitle(event.data.title);
    }
  },
});

// 메시지 제출
stream.submit({
  messages: [{ role: "user", content: "Hello" }]
}, {
  // 즉각적인 UI 피드백을 위한 낙관적 업데이트
  optimisticValues: (prev) => ({
    ...prev,
    messages: [...prev.messages, newMessage],
  }),
});
```

### 오류 처리

```typescript
import { toast } from "sonner";

try {
  const result = await client.threads.create();
} catch (error) {
  console.error("Failed to create thread:", error);
  toast.error("대화 생성 실패", {
    description: error instanceof Error ? error.message : "알 수 없는 오류",
  });
}
```

---

## 스타일링 접근 방식

### Tailwind CSS 4.x

이 프로젝트는 v3과 다른 아키텍처를 가진 **Tailwind CSS 4.0**을 사용합니다:

```css
/* src/app/globals.css */
@import "tailwindcss";

/* 테마를 위한 CSS 변수 */
:root {
  --background: 0 0% 100%;
  --foreground: 0 0% 3.9%;
  --primary: 0 0% 9%;
  --primary-foreground: 0 0% 98%;
  /* ... */
}

.dark {
  --background: 0 0% 3.9%;
  --foreground: 0 0% 98%;
  /* ... */
}
```

### 디자인 시스템

**색상 팔레트**:
- 지각적으로 균일한 색상을 위해 `oklch()` 색 공간 사용
- 모노크롬 중립 팔레트 (Claude.ai/ChatGPT에서 영감)
- 시맨틱 명명: `background`, `foreground`, `muted`, `accent`, `primary`, `destructive`

**테마 변수**:
```css
/* 라이트 모드 */
--background: oklch(99% 0.01 297);
--foreground: oklch(31% 0.01 297);

/* 다크 모드 */
.dark {
  --background: oklch(22% 0.01 297);
  --foreground: oklch(96% 0.01 297);
}
```

### 유틸리티 클래스

```typescript
// src/lib/utils.ts
import { clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

**사용법**:
```typescript
<div className={cn(
  "base-classes",
  condition && "conditional-classes",
  anotherCondition ? "true-classes" : "false-classes",
  className  // 부모 재정의 허용
)} />
```

### 커스텀 유틸리티

```css
/* src/app/globals.css */
.scrollbar-pretty {
  scrollbar-width: thin;
  scrollbar-color: hsl(var(--border)) transparent;
}

.shadow-inner-right {
  box-shadow: inset -10px 0 10px -10px rgba(0, 0, 0, 0.1);
}
```

### 반응형 디자인

Tailwind 중단점:
- `sm`: 640px
- `md`: 768px
- `lg`: 1024px
- `xl`: 1280px
- `2xl`: 1536px

**사용법**:
```typescript
<div className="px-4 md:px-6 lg:px-8">
  <h1 className="text-2xl md:text-3xl lg:text-4xl">제목</h1>
</div>
```

### 다크 모드

```typescript
// 다크 모드를 위해 next-themes 사용
import { ThemeProvider } from "next-themes";

<ThemeProvider attribute="class" defaultTheme="system">
  {children}
</ThemeProvider>

// 테마 토글
import { useTheme } from "next-themes";

const { theme, setTheme } = useTheme();
setTheme("dark");  // "light" | "dark" | "system"
```

### 애니메이션

**Framer Motion** 복잡한 애니메이션용:

```typescript
import { motion } from "framer-motion";

<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  exit={{ opacity: 0, y: -20 }}
  transition={{ duration: 0.3 }}
>
  콘텐츠
</motion.div>
```

**Tailwind Animate** 간단한 전환용:

```typescript
<div className="transition-all duration-300 hover:scale-105">
  호버하세요
</div>
```

---

## 일반적인 개발 작업

### 새 컴포넌트 추가

1. **컴포넌트 파일 생성**:
   ```bash
   # UI primitive (shadcn/ui 패턴)
   touch src/components/ui/new-component.tsx

   # 기능 컴포넌트
   touch src/components/thread/NewFeature.tsx
   ```

2. **컴포넌트 패턴 따르기**:
   ```typescript
   "use client";  // 훅/상태 사용 시

   import React from "react";
   import { cn } from "@/lib/utils";

   interface NewComponentProps {
     // Props 정의
   }

   export function NewComponent({ }: NewComponentProps) {
     return <div>컴포넌트</div>;
   }
   ```

3. **인덱스에서 내보내기** (모듈 생성 시):
   ```typescript
   // src/components/ui/index.ts
   export { Button } from "./button";
   export { NewComponent } from "./new-component";
   ```

### 새 훅 추가

1. **훅 파일 생성**:
   ```bash
   touch src/hooks/use-new-feature.tsx
   ```

2. **훅 패턴 따르기**:
   ```typescript
   import { useState, useEffect } from "react";

   export function useNewFeature(param: string) {
     const [state, setState] = useState<Type>(initialValue);

     useEffect(() => {
       // Effect 로직
     }, [param]);

     return { state, setState };
   }
   ```

### 구성 업데이트

**앱 구성** (`public/settings.yaml`):

```yaml
branding:
  appName: "My Chat App"
  logoPath: "/logo.png"
  description: "무엇이든 물어보세요"
  chatOpeners:
    - "날씨는 어때요?"
    - "농담 하나 해줘"

buttons:
  enableFileUpload: true
  chatInputPlaceholder: "메시지를 입력하세요..."

threads:
  showHistory: true
  enableDeletion: true

theme:
  fontFamily: "sans"  # sans | serif | mono
  fontSize: "medium"   # small | medium | large
  colorScheme: "light" # light | dark | auto

ui:
  autoCollapseToolCalls: true
  chatWidth: "wide"    # default | wide
```

**변경 사항은 즉시 적용됩니다** (브라우저 캐시 하드 리프레시).

### 파일 업로드 지원 추가

파일 업로드는 `use-file-upload.tsx`에서 처리:

```typescript
const {
  files,
  isDragging,
  addFiles,
  removeFile,
  clearFiles,
  handleDrop,
  handleDragEnter,
  handleDragLeave,
  handleDragOver,
  handlePaste,
} = useFileUpload();

// 지원 형식: JPEG, PNG, GIF, WebP, PDF
// src/lib/file-validation.ts에서 검증
```

### 커스텀 이벤트 추가

```typescript
// 백엔드에서 커스텀 이벤트 전송:
{
  "event": "custom_event_name",
  "data": { "key": "value" }
}

// Stream 프로바이더에서 프론트엔드 처리:
const stream = useTypedStream({
  // ...
  onCustomEvent: (event) => {
    if (event.event === "custom_event_name") {
      // 이벤트 처리
      console.log(event.data);
    }
  },
});
```

### 메시지 렌더링 수정

```typescript
// src/components/thread/messages/ai.tsx
export function AIMessage({ message }: { message: Message }) {
  // 커스텀 렌더링 로직 추가

  if (message.metadata?.customType) {
    return <CustomRenderer data={message.metadata} />;
  }

  // 기본 렌더링
  return <DefaultRenderer content={message.content} />;
}
```

### 아티팩트 타입 추가

```typescript
// 아티팩트는 사이드 패널에 렌더링됨
const [ArtifactContent, artifact] = useArtifact();

// 아티팩트 콘텐츠 설정
artifact.setTitle("코드 미리보기");
artifact.setOpen(true);

<ArtifactContent>
  <CodeEditor code={code} language="typescript" />
</ArtifactContent>
```

---

## 테스트 및 품질

### 린팅

```bash
# 이슈 검사
pnpm lint

# 이슈 자동 수정
pnpm lint:fix
```

**ESLint 규칙**:
- React Hooks 규칙 (강제 적용)
- TypeScript strict mode
- 미사용 변수 경고 (`_` 접두사는 무시)
- React Refresh 컴포넌트 내보내기

### 포맷팅

```bash
# 모든 파일 포맷팅
pnpm format

# 쓰기 없이 포맷팅 검사
pnpm format:check
```

**Prettier 규칙**:
- 줄 끝: auto
- JSX에서 속성당 한 줄
- Tailwind 클래스 자동 정렬

### 타입 검사

```bash
# 빌드 없이 타입 검사
npx tsc --noEmit

# 프로덕션 빌드 (타입 검사 포함)
pnpm build
```

### 커밋 전 체크리스트

코드 커밋 전:

1. ✅ `pnpm lint:fix` 실행
2. ✅ `pnpm format` 실행
3. ✅ 브라우저에서 테스트 (개발 모드)
4. ✅ TypeScript 오류 검사 (`npx tsc --noEmit`)
5. ✅ 콘솔 오류 없음 확인
6. ✅ 반응형 디자인 테스트 (모바일/데스크톱)
7. ✅ UI 변경 시 다크 모드 테스트

### 일반적인 문제

**문제**: 의존성 추가 후 TypeScript 오류
**해결책**: IDE에서 TypeScript 서버 재시작

**문제**: Tailwind 클래스가 작동하지 않음
**해결책**: `tailwind.config.js` content 경로 확인, 개발 서버 재시작

**문제**: 환경 변수를 사용할 수 없음
**해결책**: 클라이언트 접근을 위해 `NEXT_PUBLIC_` 접두사 추가, 개발 서버 재시작

**문제**: 파일 업로드가 작동하지 않음
**해결책**: `next.config.mjs`에 `bodySizeLimit: "10mb"` 있는지 확인

**문제**: 설정 변경사항이 반영되지 않음
**해결책**: 브라우저 하드 리프레시 (Cmd+Shift+R / Ctrl+Shift+R)

---

## 주요 파일 참조

### 구성 파일

| 파일 | 목적 |
|------|---------|
| `next.config.mjs` | Next.js 구성 (body size limit, 실험적 기능) |
| `tsconfig.json` | TypeScript 구성 (strict mode, 경로 별칭) |
| `tailwind.config.js` | Tailwind CSS 구성 (테마, 플러그인) |
| `eslint.config.js` | ESLint 규칙 (flat config 형식) |
| `prettier.config.js` | Prettier 포맷팅 규칙 |
| `components.json` | shadcn/ui 구성 |
| `.env.example` | 환경 변수 템플릿 |
| `public/settings.yaml` | **런타임 앱 구성** ⚠️ |

### 핵심 애플리케이션 파일

| 파일 | 목적 | 줄 수 |
|------|---------|-------|
| `src/app/layout.tsx` | NuqsAdapter가 있는 루트 레이아웃 | 24 |
| `src/app/page.tsx` | 메인 채팅 페이지 | - |
| `src/app/globals.css` | 전역 스타일 + CSS 변수 | 171 |
| `src/app/api/[..._path]/route.ts` | LangGraph로의 API 프록시 | - |

### 프로바이더 파일

| 파일 | 목적 | 줄 수 |
|------|---------|-------|
| `src/providers/Stream.tsx` | LangGraph 스트리밍 + 낙관적 업데이트 | 308 |
| `src/providers/Settings.tsx` | 앱 구성 + 사용자 환경설정 | 150 |
| `src/providers/Thread.tsx` | 스레드 목록 관리 | 80 |
| `src/providers/AssistantConfig.tsx` | Assistant별 구성 | 167 |
| `src/providers/client.ts` | LangGraph 클라이언트 생성 | - |

### 컴포넌트 파일

| 파일 | 목적 | 줄 수 |
|------|---------|-------|
| `src/components/thread/index.tsx` | 메인 Thread 오케스트레이터 | 655 |
| `src/components/thread/artifact.tsx` | 포털 기반 아티팩트 시스템 | 190 |
| `src/components/thread/messages/ai.tsx` | AI 메시지 렌더링 | - |
| `src/components/thread/messages/human.tsx` | 사용자 메시지 렌더링 | - |
| `src/components/thread/messages/tool-calls.tsx` | 도구 실행 표시 | - |
| `src/components/thread/history/` | 스레드 히스토리 사이드바 | - |
| `src/components/thread/agent-inbox/` | 중단(Interrupt) 처리 | - |

### 유틸리티 파일

| 파일 | 목적 |
|------|---------|
| `src/lib/utils.ts` | Tailwind 클래스 병합을 위한 `cn()` 유틸리티 |
| `src/lib/config.ts` | YAML 구성 로더 |
| `src/lib/assistant-api.ts` | Assistant API 헬퍼 함수 |
| `src/lib/multimodal-utils.ts` | 파일/이미지 처리 유틸리티 |
| `src/lib/constants.ts` | 앱 상수 (DO_NOT_RENDER 접두사 등) |
| `src/lib/file-validation.ts` | 파일 업로드 검증 |

### 훅 파일

| 파일 | 목적 | 줄 수 |
|------|---------|-------|
| `src/hooks/useMediaQuery.tsx` | 반응형 중단점 감지 | - |
| `src/hooks/use-file-upload.tsx` | 파일 업로드 관리 (드래그/드롭/붙여넣기) | 168 |

---

## AI 어시스턴트를 위한 모범 사례

### 변경 시

1. **쓰기 전에 읽기**: 편집하기 전에 항상 전체 파일 읽기
2. **패턴 유지**: 파일의 기존 패턴 따르기
3. **경로 별칭 사용**: 항상 `@/` 임포트 사용, 상대 경로 사용 금지
4. **타입 안전성**: 새 코드에 적절한 TypeScript 타입 추가
5. **코드 포맷팅**: 변경 후 `pnpm format` 실행
6. **철저한 테스트**: 브라우저에서 확인, 반응형 디자인 테스트, 다크 모드 확인

### 기능 추가 시

1. **기존 패턴 확인**: 먼저 유사한 기능 찾기
2. **기존 컴포넌트 사용**: shadcn/ui 컴포넌트 활용
3. **프로바이더 패턴 따르기**: 적절한 프로바이더에 상태 추가
4. **settings.yaml 업데이트**: 기능에 구성이 필요한 경우
5. **오류 우아하게 처리**: 사용자 피드백을 위해 toast 알림 사용
6. **반응형 디자인 고려**: 모바일과 데스크톱에서 테스트

### 디버깅 시

1. **콘솔 확인**: TypeScript/ESLint 오류 찾기
2. **환경 변수 확인**: 적절한 접두사 확인
3. **프로바이더 계층 확인**: 컴포넌트가 필요한 프로바이더로 래핑되었는지 확인
4. **네트워크 검사**: API 프록시가 작동하는지 확인
5. **하드 리프레시로 테스트**: 구성 변경을 위해 브라우저 캐시 지우기

### 일반적인 함정

- **환경 변수**: 변경 후 개발 서버 재시작 필요
- **Public 변수**: 클라이언트 접근을 위해 `NEXT_PUBLIC_` 접두사 필요
- **Settings.yaml**: 변경 사항은 캐시를 지우기 위해 하드 리프레시 필요
- **Tailwind 클래스**: 완전한 문자열이어야 함 (문자열 연결 불가)
- **Next.js 캐싱**: 때때로 `.next` 디렉터리 삭제 필요
- **파일 업로드**: `next.config.mjs`에서 10MB로 제한

---

## 추가 리소스

- **README.md**: 사용자 대상 문서 (한국어)
- **public/full-description.md**: 최종 사용자 가이드
- **LangGraph SDK 문서**: https://langchain-ai.github.io/langgraph/
- **Next.js 문서**: https://nextjs.org/docs
- **Tailwind CSS 문서**: https://tailwindcss.com/docs
- **shadcn/ui 문서**: https://ui.shadcn.com/
- **Radix UI 문서**: https://www.radix-ui.com/

---

**최종 업데이트**: 2025-11-15
**프로젝트 버전**: 0.0.0
**Next.js 버전**: 15.2.3
**React 버전**: 19.0.0
