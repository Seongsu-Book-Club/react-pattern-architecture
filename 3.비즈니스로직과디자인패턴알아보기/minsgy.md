# 리액트 설계 원칙 적용 
- Solid 원칙을 가지고 리액트 설계 원칙을 구성

## 단일 책임 원칙

- render Prop

```tsx
// MouseTracker.tsx
interface MouseTrackerProps {
  render: (position: { x: number; y: number }) => React.ReactNode;
}

const MouseTracker: React.FC<MouseTrackerProps> = ({ render }) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };

    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  return <>{render(position)}</>;
};

// 사용 예시
const App = () => {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <div>
          마우스 위치: ({x}, {y})
        </div>
      )}
    />
  );
};
```

- 합성을 통한 단일 책임 원칙
```tsx
// Button.tsx
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
}

const Button: React.FC<ButtonProps> = ({ children, onClick }) => {
  return <button onClick={onClick}>{children}</button>;
};

// Icon.tsx
interface IconProps {
  name: string;
}

const Icon: React.FC<IconProps> = ({ name }) => {
  return <span className={`icon-${name}`} />;
};

// IconButton.tsx
interface IconButtonProps {
  icon: string;
  onClick?: () => void;
}

const IconButton: React.FC<IconButtonProps> = ({ icon, onClick }) => {
  return (
    <Button onClick={onClick}>
      <Icon name={icon} />
    </Button>
  );
};
```
  
## 의존관계 역전 원칙

- 유지보수가 가능하며 유연, 확장 가능한 소프트웨어를 구성하기 위한 추상화에 초점을 맞추는 방식
- 결합도를 낮추고 응집도를 높이자.
  
```tsx
// 인터페이스 정의
interface UserRepository {
  getUser: (id: string) => Promise<User>;
  saveUser: (user: User) => Promise<void>;
}

// 컴포넌트에서 사용
interface UserProfileProps {
  userRepository: UserRepository;
  userId: string;
}

const UserProfile = ({ userRepository, userId }) => {
  const [user, setUser] = useState<User>(null);
   
   ...

  return user ? <div>{user.name}</div> : <div>로딩중...</div>;
};
```

## 명령과 조회 책임 분리

- 하나의 함수나 쿼리는 하나의 역할만하도록 한다.
- reducer가 그 예시가 될 수 있음

```tsx
// 상태 관리 예시
interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

// 명령 (Command)
const todoReducer = (state: Todo[], action: TodoAction): Todo[] => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.payload];
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.payload.id
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    default:
      return state;
  }
};

// 조회 (Query)
const TodoList: React.FC = () => {
  const [todos, dispatch] = useReducer(todoReducer, []);
  ...
}
```



# 합성 패턴

- 합성은 다양한 코드베이스에서 확장과 유지보수를 용이하게 하기 위한 방법들은 고민하면서 재사용성을 늘릴 수 있는 패턴


## 고차 컴포넌트를 통한 합성의 이해 (HOC)

```tsx
interface WithLoadingProps {
  isLoading: boolean;
}

const withLoading = <P extends object>(
  WrappedComponent: React.ComponentType<P>
) => {
  return ({ isLoading, ...props }: WithLoadingProps & P) => {
    if (isLoading) {
      return <div>로딩중...</div>;
    }
    return <WrappedComponent {...props} />;
  };
};

// 사용 예시
const UserProfile = ({ name }: { name: string }) => (
  <div>사용자: {name}</div>
);

const UserProfileWithLoading = withLoading(UserProfile);

const App = () => {
  const [isLoading, setIsLoading] = useState(true);
  const [userName, setUserName] = useState('');

  return <UserProfileWithLoading isLoading={isLoading} name={userName} />;
};
```


## 리액트 훅

```tsx
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

// 사용 예시
const ResponsiveComponent = () => {
  const { width, height } = useWindowSize();
  return (
    <div>
      현재 창 크기: {width} x {height}
    </div>
  );
};
```


## 헤드리스 컴포넌트

```tsx
interface AccordionState {
  isOpen: boolean;
  toggle: () => void;
}

const useAccordion = (initialState = false): AccordionState => {
  const [isOpen, setIsOpen] = useState(initialState);
  
  const toggle = () => setIsOpen(!isOpen);
  
  return { isOpen, toggle };
};

// 사용 예시
const Accordion = () => {
  const { isOpen, toggle } = useAccordion();
  
  return (
    <div>
      <button onClick={toggle}>
        {isOpen ? '접기' : '펼치기'}
      </button>
      {isOpen && (
        <div className="content">
          아코디언 내용이 여기에 표시됩니다.
        </div>
      )}
    </div>
  );
};

// 다른 스타일로 재사용
const FancyAccordion = () => {
  const { isOpen, toggle } = useAccordion();
  
  return (
    <div className="fancy-accordion">
      <div 
        className={`header ${isOpen ? 'open' : ''}`}
        onClick={toggle}
      >
        <span>제목</span>
        <span className="arrow">{isOpen ? '▼' : '▶'}</span>
      </div>
      {isOpen && (
        <div className="fancy-content">
          다른 스타일의 아코디언 내용
        </div>
      )}
    </div>
  );
};
```