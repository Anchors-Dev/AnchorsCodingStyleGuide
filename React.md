# React 스타일 가이드 (React 18 + TypeScript)

## ESLint 설정

- 프로젝트에는 항상 ESLint와 TypeScript를 사용합니다.
- `.eslintrc.js` 파일 예시:

```javascript
module.exports = {
  env: {
    browser: true,
    es2024: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:@typescript-eslint/recommended-requiring-type-checking',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:jsx-a11y/recommended',
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
    project: './tsconfig.json',
    ecmaFeatures: {
      jsx: true,
    },
  },
  settings: {
    react: {
      version: '18',
    },
  },
  rules: {
    'react/react-in-jsx-scope': 'off',
    '@typescript-eslint/explicit-function-return-type': 'error',
    '@typescript-eslint/no-unused-vars': 'error',
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',
  },
};
```

## 파일 구조와 명명 규칙

### 디렉토리 구조

```
src/
├── app/                # 앱 라우팅 및 레이아웃
│   ├── (auth)/        # 인증 관련 라우트
│   ├── (dashboard)/   # 대시보드 관련 라우트
│   └── layout.tsx     # 루트 레이아웃
├── components/         # 컴포넌트
│   ├── ui/            # Shadcn UI 기본 컴포넌트
│   │   ├── button.tsx
│   │   ├── dialog.tsx
│   │   └── ...
│   ├── charts/        # 차트 관련 컴포넌트
│   ├── data-table/    # 테이블 관련 컴포넌트
│   ├── forms/         # 폼 관련 컴포넌트
│   └── shared/        # 공통 컴포넌트
├── config/            # 설정 파일
│   ├── site.ts       # 사이트 메타데이터
│   └── dashboard.ts  # 대시보드 설정
├── hooks/             # 커스텀 훅
├── lib/              # 유틸리티 함수
│   ├── utils.ts     # 공통 유틸리티
│   └── fonts.ts     # 폰트 설정
├── providers/        # Context Providers
├── styles/          # 글로벌 스타일
│   └── globals.css
└── types/           # 타입 정의
```

### 파일 명명 규칙

```
# 컴포넌트 파일
components/
├── ui/                          # Shadcn UI 컴포넌트
│   ├── button.tsx
│   ├── button.types.ts         # 타입 정의 (선택적)
│   └── button.test.tsx         # 테스트 (선택적)
├── data-table/                 # 복잡한 컴포넌트
│   ├── index.tsx
│   ├── columns.tsx            # 테이블 컬럼 정의
│   ├── toolbar.tsx           # 테이블 도구 모음
│   └── types.ts             # 타입 정의
└── forms/
    ├── user-form.tsx
    └── settings-form.tsx

# 페이지 파일
app/
├── (dashboard)/
│   ├── page.tsx              # 대시보드 메인 페이지
│   ├── users/
│   │   └── page.tsx         # 사용자 관리 페이지
│   └── settings/
│       └── page.tsx         # 설정 페이지
└── layout.tsx               # 루트 레이아웃

# 훅 파일
hooks/
├── use-theme.ts            # 테마 관련 훅
└── use-toast.ts           # 토스트 관련 훅

# 설정 파일
config/
├── site.ts                # 사이트 설정
└── dashboard.ts           # 대시보드 설정

# 타입 파일
types/
├── api.d.ts              # API 관련 타입
└── dashboard.d.ts        # 대시보드 관련 타입
```

## 타입 정의 가이드

### Props 타입 정의

```tsx
// types/components/Button.types.ts
export interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
  children: React.ReactNode;
}

// components/Button.tsx
import { ButtonProps } from '@/types/components/Button.types';

const Button: React.FC<ButtonProps> = ({ 
  variant = 'primary',
  size = 'md',
  isLoading = false,
  children,
  ...props 
}) => {
  return (
    <button
      className={cn(buttonStyles[variant], sizeStyles[size])}
      disabled={isLoading}
      {...props}
    >
      {isLoading ? <Spinner /> : children}
    </button>
  );
};
```

### 커스텀 훅 타입 정의

```tsx
// hooks/useUser.ts
interface User {
  id: number;
  name: string;
  email: string;
}

interface UseUserReturn {
  user: User | null;
  isLoading: boolean;
  error: Error | null;
  fetchUser: (id: number) => Promise<void>;
}

const useUser = (initialId?: number): UseUserReturn => {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState<boolean>(false);
  const [error, setError] = useState<Error | null>(null);

  // ... 구현
  
  return { user, isLoading, error, fetchUser };
};
```

### API 응답 타입 정의

```tsx
// types/api.types.ts
export interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

export interface PaginatedResponse<T> extends ApiResponse<T> {
  pagination: {
    currentPage: number;
    totalPages: number;
    totalItems: number;
  };
}

// services/api.ts
async function fetchUsers(): Promise<ApiResponse<User[]>> {
  const response = await axios.get<ApiResponse<User[]>>('/api/users');
  return response.data;
}
```

## 컴포넌트 작성 규칙

### 함수형 컴포넌트

```tsx
// ❌ 나쁨
const UserProfile = (props: any) => {
  return <div>{props.name}</div>;
};

// 👍 좋음
interface UserProfileProps {
  name: string;
  age: number;
  email?: string;
}

const UserProfile: React.FC<UserProfileProps> = ({ name, age, email }) => {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      {email && <p>Email: {email}</p>}
    </div>
  );
};
```

### 제네릭 컴포넌트

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
}

const List = <T extends unknown>({ items, renderItem }: ListProps<T>): JSX.Element => {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{renderItem(item)}</li>
      ))}
    </ul>
  );
};

// 사용 예시
const UserList = (): JSX.Element => {
  const users: User[] = [/* ... */];
  return (
    <List
      items={users}
      renderItem={(user) => <UserCard user={user} />}
    />
  );
};
```

## 컴포넌트 작성 규칙

### 함수형 컴포넌트 사용

```jsx
// ❌ 나쁨
class UserProfile extends React.Component {
  render() {
    return <div>{this.props.name}</div>;
  }
}

// 👍 좋음
const UserProfile = ({ name }) => {
  return <div>{name}</div>;
};
```

### Props 타입 검사

```jsx
import PropTypes from 'prop-types';

const Button = ({ label, onClick, variant = 'primary' }) => {
  return (
    <button onClick={onClick} className={`btn-${variant}`}>
      {label}
    </button>
  );
};

Button.propTypes = {
  label: PropTypes.string.isRequired,
  onClick: PropTypes.func.isRequired,
  variant: PropTypes.oneOf(['primary', 'secondary', 'danger']),
};

export default Button;
```

### 구조 분해 할당 사용

```jsx
// ❌ 나쁨
const UserCard = (props) => {
  return (
    <div>
      <h2>{props.name}</h2>
      <p>{props.email}</p>
    </div>
  );
};

// 👍 좋음
const UserCard = ({ name, email }) => {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
    </div>
  );
};
```

## Hooks 사용 가이드

### 기본 규칙

1. 최상위에서만 Hook 호출
2. React 함수 내에서만 Hook 호출

```jsx
// ❌ 나쁨
const UserProfile = ({ userId }) => {
  if (userId) {
    useEffect(() => {
      // 조건부 Hook 사용
    }, [userId]);
  }
};

// 👍 좋음
const UserProfile = ({ userId }) => {
  useEffect(() => {
    if (userId) {
      // 조건부 로직
    }
  }, [userId]);
};
```

### 커스텀 Hook 작성

```jsx
// hooks/useWindowSize.js
const useWindowSize = () => {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
};
```

## 성능 최적화

### React.memo 사용

```jsx
// 👍 좋음
const ExpensiveComponent = React.memo(({ data }) => {
  return <div>{/* 복잡한 렌더링 로직 */}</div>;
});
```

### useMemo와 useCallback

```jsx
const MemoExample = ({ items }) => {
  // 복잡한 계산은 useMemo 사용
  const sortedItems = useMemo(() => {
    return [...items].sort((a, b) => b - a);
  }, [items]);

  // 이벤트 핸들러는 useCallback 사용
  const handleClick = useCallback(() => {
    console.log('Clicked!');
  }, []);

  return (
    <div>
      {sortedItems.map(item => (
        <Item key={item.id} onClick={handleClick} />
      ))}
    </div>
  );
};
```

## 컴포넌트 분할 기준

1. 단일 책임 원칙
2. 재사용성
3. 복잡도

```jsx
// ❌ 나쁨
const Page = () => {
  return (
    <div>
      <header>{/* 복잡한 헤더 로직 */}</header>
      <main>{/* 복잡한 컨텐츠 로직 */}</main>
      <footer>{/* 복잡한 푸터 로직 */}</footer>
    </div>
  );
};

// 👍 좋음
const Header = () => {/* 헤더 로직 */};
const Content = () => {/* 컨텐츠 로직 */};
const Footer = () => {/* 푸터 로직 */};

const Page = () => {
  return (
    <div>
      <Header />
      <Content />
      <Footer />
    </div>
  );
};
```

## 상태 관리

### 로컬 상태 vs 전역 상태

```jsx
// 로컬 상태 (useState)
const Counter = () => {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
};

// 전역 상태 (Context API)
const UserContext = createContext();

const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
};
```

## 에러 처리

### Error Boundary 사용

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <h1>문제가 발생했습니다.</h1>;
    }
    return this.props.children;
  }
}

// 사용
const App = () => {
  return (
    <ErrorBoundary>
      <MyComponent />
    </ErrorBoundary>
  );
};
```

## UI 컴포넌트 가이드 (Shadcn UI)

### 컴포넌트 구성

```tsx
// components/ui/button.tsx
import { cn } from "@/lib/utils";
import { ButtonProps } from "./button.types";

const Button = ({ 
  className, 
  variant = "default", 
  size = "default", 
  ...props 
}: ButtonProps) => {
  return (
    <button
      className={cn(
        "rounded-lg font-medium transition-colors",
        "focus-visible:outline-none focus-visible:ring-2",
        {
          "bg-primary text-white hover:bg-primary/90": variant === "default",
          "h-9 px-4": size === "default",
        },
        className
      )}
      {...props}
    />
  );
};

export { Button };
```

### 컴포넌트 구조화

```
components/
├── ui/                 # 기본 UI 컴포넌트
│   ├── button.tsx
│   ├── card.tsx
│   └── dialog.tsx
├── common/            # 공통 복합 컴포넌트
│   ├── DataTable/
│   └── ThemeToggle/
└── features/         # 기능별 컴포넌트
    ├── dashboard/
    └── settings/
```

## Tailwind CSS 스타일 가이드

### 기본 규칙

1. 유틸리티 클래스 순서
```tsx
// 👍 좋음
<div className="
  flex items-center justify-between  // 레이아웃
  p-4 m-2                           // 스페이싱
  bg-white dark:bg-gray-800         // 배경
  text-sm font-medium               // 타이포그래피
  rounded-lg border                 // 테두리
  hover:bg-gray-50                  // 상호작용
  transition-colors duration-200    // 애니메이션
"/>
```

2. 조건부 스타일링
```tsx
// 👍 좋음
import { cn } from "@/lib/utils";

const Card = ({ isActive, className }) => {
  return (
    <div
      className={cn(
        "rounded-lg p-4",
        isActive ? "bg-primary text-white" : "bg-gray-100",
        className
      )}
    />
  );
};
```

### 다크 모드

```tsx
// 👍 좋음
<div className="
  bg-white dark:bg-gray-800
  text-gray-900 dark:text-gray-100
"/>

// ❌ 나쁨
<div className={isDark ? "bg-gray-800 text-gray-100" : "bg-white text-gray-900"} />
```

### 반응형 디자인

```tsx
// 👍 좋음
<div className="
  grid
  grid-cols-1          // 모바일
  sm:grid-cols-2       // 태블릿
  md:grid-cols-3       // 작은 데스크톱
  lg:grid-cols-4       // 큰 데스크톱
  gap-4
"/>
```

### 재사용 가능한 스타일

```tsx
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: {
          DEFAULT: '#0070f3',
          dark: '#0761d1',
        },
      },
      spacing: {
        container: '2rem',
      },
    },
  },
};

// 컴포넌트에서 사용
<div className="
  text-primary
  dark:text-primary-dark
  px-container
"/>
```

## 컴포넌트 최적화

### 스타일 분리

```tsx
// styles/card.ts
export const cardStyles = {
  base: "rounded-lg border bg-card text-card-foreground shadow-sm",
  header: "flex flex-col space-y-1.5 p-6",
  content: "p-6 pt-0",
  footer: "flex items-center p-6 pt-0",
};

// components/Card.tsx
import { cardStyles } from "@/styles/card";

const Card = ({ children }) => {
  return <div className={cardStyles.base}>{children}</div>;
};
```

### 성능 고려사항

```tsx
// 👍 좋음
const TableCell = React.memo(({ content }) => (
  <td className="p-4 align-middle [&:has([role=checkbox])]:pr-0">
    {content}
  </td>
));
```

## 접근성

### ARIA 속성 사용

```tsx
// 👍 좋음
<button
  aria-label="메뉴 열기"
  className="inline-flex items-center justify-center rounded-md p-2"
>
  <MenuIcon className="h-6 w-6" />
</button>
```

### 키보드 네비게이션

```tsx
// 👍 좋음
const MenuItem = ({ children }) => {
  return (
    <li>
      <a
        className="flex select-none items-center rounded-sm px-2 py-1.5 outline-none
                   focus-visible:bg-accent focus-visible:text-accent-foreground"
        href="#"
      >
        {children}
      </a>
    </li>
  );
};
```

## 참고

* [React 공식 문서](https://react.dev/)
* [TypeScript 공식 문서](https://www.typescriptlang.org/)
* [Shadcn UI](https://ui.shadcn.com/)
* [Tailwind CSS](https://tailwindcss.com/)
* [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
