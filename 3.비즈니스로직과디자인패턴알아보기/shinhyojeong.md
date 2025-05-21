# 3.비즈니스 로직과 디자인 패턴 알아보기

## 리액트 설계 원칙 적용

### 단일 책임 원칙(Single Responsibility Principle)

하나의 컴포넌트는 하나의 역할만 하도록 설게하기

### 의존 관계 역전 원칙(Dependency Inversion Principle)

상위 모듈이 하위 모듈에 의존하지 않고, 추상화에 초점을 맞춰야 한다. 추상화(interface) 정의와 실제 구현부를 분리해야 한다.

### 명령과 조회 책임 분리 원칙(Command and Query Responsibility Segregation)

변경은 복잡한 로직이 많고, 조회는 단순하기 때문에 변경 작업과 조회 작업을 분리한다.

```
/services
  ├── getMemoList.ts   ← 조회 전용
  └── editMemo.ts ← 변경 전용
/components
  ├── MemoList.tsx
  └── MemoForm.tsx
```

- 테스트 용이
- 유지보수 용이
- 동작에 대한 추론이 쉬움

```tsx
function TodoListView() {
  const [todos, setTodos] = useState([]);

  useEffect(() => {
    todoQueryService.getTodos().then(setTodos);
  }, []);

  return <TodoList todos={todos} />;
}

function TodoInputView({ onTodoAdded }: { onTodoAdded: () => void }) {
  const handleAdd = async (text: string) => {
    await todoCommandService.addTodo(text);
    onTodoAdded(); // 조회 갱신은 호출 측에서 명시적으로 결정
  };

  return <TodoForm onAdd={handleAdd} />;
}

function TodoApp() {
  const [refreshKey, setRefreshKey] = useState(0);

  return (
    <>
      <TodoInputView onTodoAdded={() => setRefreshKey((k) => k + 1)} />
      <TodoListView key={refreshKey} />
    </>
  );
}
```

## 합성 패턴

하나의 컴포넌트를 조각으로 나눠 다양하게 조합하여 사용

- 로직 간소화
- 재사용성 향상
- 관심사 분리

### 고차 컴포넌트

```tsx
export const withToggle = (Component) => (props) => {
  const [on, setOn] = useState(false);
  const toggle = () => setOn((prev) => !prev);

  return <Component {...props} on={on} toggle={toggle} />;
};

function RawButton({ on, toggle }) {
  return <button onClick={toggle}>{on ? "On" : "Off"}</button>;
}

const ToggleButton = withToggle(RawButton);
```

### 리액트 훅

```tsx
import { useState } from "react";

export const useToggle = (initial = false) => {
  const [on, setOn] = useState(initial);
  const toggle = () => setOn((prev) => !prev);
  return { on, toggle };
};

function ToggleButton() {
  const { on, toggle } = useToggle();

  return <button onClick={toggle}>{on ? "켜짐" : "꺼짐"}</button>;
}
```

### 헤드리스 컴포넌트

```tsx
export const Toggle = ({ children }) => {
  const [on, setOn] = useState(false);
  const toggle = () => setOn((prev) => !prev);

  return children({ on, toggle });
};

const ToggleButton = () => {
  return (
    <Toggle>
      {({ on, toggle }) => (
        <button onClick={toggle}>{on ? "활성화됨" : "비활성화됨"}</button>
      )}
    </Toggle>
  );
};
```

### 비교

| 패턴     | 핵심 장점                  | 단점/주의점                                     |
| -------- | -------------------------- | ----------------------------------------------- |
| HOC      | 기존 컴포넌트에 로직 주입  | JSX 중첩 구조가 많아 트리 깊이가 깊어질 수 있음 |
| Hook     | 사용이 가장 직관적         | 느슨한 UI 결합                                  |
| Headless | 완전한 UI 자유 + 상태 분리 | children 함수 사용해야해 복잡도 증가            |
