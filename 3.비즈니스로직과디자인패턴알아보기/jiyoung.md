## Ch 8. 데이터 관리

### 8.1 비즈니스 로직 누수 현상

원래 백엔드에 위치해야 할 비즈니스 로직이 프론트엔드로 새어 나오는 것.

- 문제: 로직과 강하게 결합된 컴포넌트를 만들어 재사용성과 유지보수를 어렵게 함.

### 8.2 ACL (오류 방지 계층)

- Access Control Layer
- 중간에서 데이터를 조정하는 역할의 계층

ACL 계층을 사용해 프론트 ↔ 백앤드 사이에서 데이터를 사용하기 좋게 변환하는 역할도 가능.

<aside>
💡

useQuery의 select를 통해 데이터를 변환해서 UI 컴포넌트에선 받아먹기만 하면 되도록 함.

</aside>

- 예외 상황 대응법
  - 만약 백앤드에서 타입과 다른 값을 준다면? 런타임 에러 발생 시 어떻게 대응할 것인가?

→ zod 사용

<aside>
💡

개인취향의 영역같기도 한데, 런타임 에러가 발생하게 둔다.

타입과 다른 응답값이 오는 건 에러를 찍지 않고 넘어갈 경우, 영원히 모를 수도 있음. 따라서 에러를 발생시켜 백앤드나 프론트에서 빠르게 대응하도록 하는 편.

</aside>

### 8.3 Prop Drilling

prop을 전달하기만하는 중간다리 역할의 컴포넌트가 생겨나는 것.

- 문제: 특정 컴포넌트는 쓰지 않는 prop을 받아서 처리해야 한다. 이는 단일책임원칙과 멀어지는 방향

### 8.4 Context API

Context Provider와 Consumer를 통해 동일한 부모 컴포넌트 이하에서 상태를 공유할 수 있게 한다.

- 컴파운드 패턴

## Ch 9. 리액트 설계 원칙

시간이 지날 수록 코드베이스를 지속적으로 안정화시키는 방식.

(왜케 쓸데없이 영어 쓰고 줄이는겨 🥵🥵🥵)

### 9.1 단일 책임 원칙

객체 지향 설계의 SOLID 원칙 중 첫번째 원칙

- 모든 모듈 또는 클래스가 오직 하나의 변경 이유만 가져야 한다는 원칙.
- 즉 컴포넌트나 함수는 하나의 책임만.

- render prop 패턴

```tsx
// 고대
const Title = () => <div>Title || This is a Title</div>

// 성숙기
const Title = ({ title, children }: 대충타입) => <div>{children(title)}</div>

// 외부
<Title title='This is a title'>
  {(s: string) => {
    const formatted = s.toUpperCase();
    return <h3>{formatted}</h3>
  }}
</Title>
```

<aside>
💡

사실 render props 많이 안써봄. 😰😰😰

우리팀에선 뭔가 컴포넌트 props가 복잡해진다 느껴져서 잘 안쓰는 듯.

완전 추상화할 것인지 VS 일부 추상화할 것인지

</aside>

중요한 것은 요구사항이 많아져도 **핵심 로직을 변경하지 않고** 컴포넌트 동작을 쉽게 확장할 수 있게 하는 것.

- 합성

컴포넌트 내부에 로직을 추가하지 않고, 각자의 역할을 하는 컴포넌트를 만들어 핵심 로직 유지하기. (p.216 참고)

### 9.2 의존관계 역전 원칙

구체적인 구현에 대한 제어를 외부로 넘기는 것.

고수준 모듈이 구체적인 저수준 구현에 직접 의존하지 않고 추상화에 의존하도록 만드는 것.

- 로깅을 위해 Context API를 추가하는 건가. 요런 방식 써보신분

### 9.3 명령과 조회 책임 분리 원칙

조회와 명령을 분리하기.

- useReducer + context API
  - 이거 사용할 바에 zustand 쓴다.

## Ch 10. 합성 패턴

- 최근에 SearchForm 리팩토링하다가 던진 거 기억난다…
- SearchForm이 너무 복잡해서 zustand로 각 역할에 맞게 분리하려다 애초에 zustand가 문제가 아니고 각 컴포넌트들에 역할이 개 복잡해서 포기.

### 10.1 고차 컴포넌트 & 고차 함수

컴포넌트를 인자로 받아서 개선된 버전의 컴포넌트를 반환하는 함수.

- 이건 마치 콜백지옥

### 10.2 리액트 훅

### 10.4 헤드리스 컴포넌트 패턴

로직과 UI를 명확하게 분리하는 것.

- 가장 윗부분에서 애플리케이션의 look and feel ㅋㅋ 영역을 담당.
- 도메인 모델 ↔ 훅 - 상태관리 로직 ↔ 뷰
