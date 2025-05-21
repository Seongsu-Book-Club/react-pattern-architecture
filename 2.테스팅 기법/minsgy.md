

스터디목표
- 실무 환경에서 테스트 코드를 통해 리팩토링하는 사례 3가지 이상 만들어보기

---

# 리액트 테스팅

## 테스트가 필요한 이유

테스트는 소프트웨어 안정성과 유지보수를 위한 핵심 요소
- 코드 검증
- 회귀 방지
- 리팩토링/유지보수 용이함
- 코드 품질 신뢰성 향상
- 문서화

## 여러 종류의 테스트 알아보기

- 정적 검사 (Linting)
  - 오류를 잡아내고 코딩 표준을 보장하기 위해 코드를 실행하지 않으며 구문을 분석
- 단위 테스트
  - 개별 컴포넌트나 기능들을 분리하여 동작하는 지 확인
- 통합 테스트
  - 다른 모듈과 서비스들이 원활하게 상호작용하며 동작하는 지 확인
- 시각적 회귀테스트 (storybook)
  - 스냅샷과 같은 픽셀 단위로 비교하여 시각적 차이점을 찾아냄
- E2E 테스트
  - 전체 애플리케이션 흐름 end-to-end 방식으로 전부 동작하는 지 확인

## jest 테스트 과정 

- test
- it
- describe 
- expect


## React 컴포넌트 테스트 

react-testing-library와 같은 전용 라이브러리를 활용하여 구현한다.
 - jest 단독으로도 가능하지만 복잡한 렌더 구조를 하나하나 다 구현해야 한다

## React 환경의 통합 테스트 예제

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

// 카운터 컴포넌트 통합 테스트 예시
describe('Counter 컴포넌트 테스트', () => {
  it('증가 버튼 클릭시 카운트가 1 증가해야 한다', async () => {
    // 컴포넌트 렌더링
    render(<Counter />);
    
    // 초기값 확인
    const countText = screen.getByText('카운트: 0');
    expect(countText).toBeInTheDocument();
    
    // 증가 버튼 클릭 이벤트 발생
    const increaseButton = screen.getByRole('button', { name: '증가' });
    await userEvent.click(increaseButton);
    
    // 증가된 값 확인 
    expect(screen.getByText('카운트: 1')).toBeInTheDocument();
  });

  it('감소 버튼 클릭시 카운트가 1 감소해야 한다', async () => {
    render(<Counter />);
    
    const decreaseButton = screen.getByRole('button', { name: '감소' });
    await userEvent.click(decreaseButton);
    
    expect(screen.getByText('카운트: -1')).toBeInTheDocument();
  });
});
```

## JSDOM이란?

순수 JavaScript로 구현된 HTML, DOM, CSSOM을 포함한 웹 브라우저 환경을 제공하는 헤드리스 브라우저

주요 특징
- Node.js 환경에서 브라우저와 유사한 DOM 환경을 시뮬레이션
- 실제 브라우저 없이도 DOM API를 사용할 수 있음
- Jest의 기본 테스트 환경으로 사용됨
- window, document와 같은 브라우저 전역 객체 제공

장점
- 브라우저 없이 빠른 테스트 실행 가능
- Node.js 기반으로 CLI 환경에서도 실행 가능
- 메모리 사용량이 적음
- CI/CD 파이프라인에 적합

단점
- 실제 브라우저와 100% 동일하지 않음
- 일부 브라우저 전용 API 지원하지 않음
- 레이아웃 엔진이 없어 스타일 계산 불가능

## Cypress를 활용한 E2E 예제

```typescript
// cypress/e2e/login.cy.ts
describe('로그인 페이지 테스트', () => {
  beforeEach(() => {
    // 테스트 전에 로그인 페이지로 이동
    cy.visit('/login')
  })

  it('로그인 성공 시나리오', () => {
    // 이메일과 비밀번호 입력
    cy.get('[data-testid="email-input"]')
      .type('test@example.com')

    cy.get('[data-testid="password-input"]')
      .type('password123')

    // 로그인 버튼 클릭
    cy.get('[data-testid="login-button"]')
      .click()

    // 로그인 성공 후 메인 페이지로 리다이렉트 확인
    cy.url().should('include', '/dashboard')
    
    // 환영 메시지 확인
    cy.get('[data-testid="welcome-message"]')
      .should('contain', '환영합니다')
  })

  it('유효하지 않은 로그인 시도', () => {
    // 잘못된 이메일과 비밀번호 입력
    cy.get('[data-testid="email-input"]')
      .type('invalid@example.com')

    cy.get('[data-testid="password-input"]')
      .type('wrongpassword')

    // 로그인 버튼 클릭
    cy.get('[data-testid="login-button"]')
      .click()

    // 에러 메시지 확인
    cy.get('[data-testid="error-message"]')
      .should('be.visible')
      .and('contain', '로그인에 실패했습니다')
  })
})
```
- cy.visit(): 특정 페이지로 이동
- cy.get(): DOM 요소 선택
- .type(): 입력 필드에 텍스트 입력
- .click(): 버튼 클릭
- .should(): 특정 조건 확인


## cy.intercept vs MSW

### cy.intercept

특징
- E2E 테스트 환경에서만 동작
- Cypress 테스트 실행 중에만 네트워크 요청을 가로챔
- 테스트 시나리오별로 다른 응답을 쉽게 설정 가능

```ts
// cy.intercept 예제
cy.intercept('GET', '/api/users', {
  statusCode: 200,
  body: {
    users: [
      { id: 1, name: '김철수' },
      { id: 2, name: '이영희' }
    ]
  }
}).as('getUsers');
```
### MSW (Mock Service Worker)

특징
- 개발 환경과 테스트 환경 모두에서 사용 가능
- Service Worker를 통해 브라우저 레벨에서 요청 가로챔
- 실제 API와 동일한 방식으로 모킹 가능
- 재사용 가능한 핸들러 정의 가능

```ts
// MSW 예제
import { rest } from 'msw';
export const handlers = [
  rest.get('/api/users', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json({
        users: [
          { id: 1, name: '김철수' },
          { id: 2, name: '이영희' }
        ]
      })
    );
  }),
];
```

### 주요 차이점

사용 범위
- cy.intercept: Cypress 테스트에서만 사용
- MSW: 개발, 테스트, 스토리북 등 다양한 환경에서 사용 가능
  
구현 방식
- cy.intercept: 네트워크 레벨에서 요청 가로챔
- MSW: Service Worker를 통해 브라우저 레벨에서 요청 가로챔

설정 방식
- cy.intercept: 테스트 케이스 내에서 직접 설정
- MSW: 별도의 핸들러 파일에서 중앙 집중식으로 관리

유지보수성
- cy.intercept: 테스트별로 독립적인 모킹 가능하나 재사용성 낮음
- MSW: 재사용 가능한 핸들러로 유지보수가 용이

실제 환경 유사성
- cy.intercept: 테스트용 모킹에 최적화
- MSW: 실제 API 동작과 유사한 환경 제공

### 사용 추천
- E2E 테스트만을 위한 경우: cy.intercept
- 개발 환경을 포함한 전반적인 모킹이 필요한 경우: MSW

# 일반적인 리팩터링 기법 살펴보기

리팩토링은 언어/프레임워크를 가리지 않고 모든 코드에 적용 가능한 개념을 뜻하고, 네이밍 수정, 함수 선언 변경, 추출, 필드 수정 등 여러 기법이 존재함



# 리액트에서의 테스트 주도 개발


## TDD (테스트 주도 개발발)

- 테스트 주도 개발은 실제 코드를 작성하기 전에 테스트를 먼저 작성하는 방법

### Flow

1. 테스트 작성 - 실패 여부 상관없음
2. 테스트 실행/실패 확인 - 구현 전 테스트 실패
3. 코드 구현 - 테스트를 통과하는 최소한의 코드 작성
4. 테스트 재실행/통과 - 테스트 통과하는 지 확인
5. 리팩토링 - 코드를 개선하면서 테스트 계속 통과 여부 확인
   - 기본적으로 RED-GREEN-REFACTOR 사이클을 반복하며 품질과 신뢰성을 올림

## ATDD (인수 테스트 주도 개발)

- 사용자 관점에서의 요구사항을 테스트로 작성하는 방법론
- 이해관계자와 협업하여 사용자 관점에서 완료 상태를 정의하고 테스트를 진행함

```ts
describe('할 일 관리 기능', () => {
  // 시나리오 1: 사용자가 새로운 할 일을 추가하는 경우
  describe('새로운 할 일 추가', () => {
    it('사용자가 할 일을 입력하고 추가 버튼을 클릭하면 목록에 추가되어야 한다', async () => {
      // Given: 할 일 관리 페이지가 열려있음
      // When: 사용자가 "회의 준비하기"를 입력하고 추가 버튼을 클릭함
      // Then: "회의 준비하기"가 할 일 목록에 표시되어야 함
    });

    it('사용자가 빈 할 일을 추가하려고 하면 추가되지 않아야 한다', async () => {
      // Given: 할 일 관리 페이지가 열려있음
      // When: 사용자가 빈 문자열을 입력하고 추가 버튼을 클릭함
      // Then: 할 일 목록이 비어있어야 함
    });
  });

  // 시나리오 2: 사용자가 할 일을 완료 처리하는 경우
  describe('할 일 완료 처리', () => {
    it('사용자가 할 일을 클릭하면 완료 상태로 표시되어야 한다', async () => {
      // Given: 할 일 목록에 "보고서 작성하기"가 있음
      // When: 사용자가 "보고서 작성하기"를 클릭함
      // Then: "보고서 작성하기"가 취소선으로 표시되어야 함
    });

    it('사용자가 완료된 할 일을 다시 클릭하면 미완료 상태로 돌아가야 한다', async () => {
      // Given: 완료된 "이메일 확인하기" 할 일이 있음
      // 완료 상태로 변경
      // When: 사용자가 완료된 "이메일 확인하기"를 다시 클릭함
      // Then: 취소선이 제거되어야 함
    });
  });

  // 시나리오 3: 사용자가 여러 할 일을 관리하는 경우
  describe('여러 할 일 관리', () => {
    it('사용자가 여러 할 일을 추가하면 모두 목록에 표시되어야 한다', async () => {
      // Given: 할 일 관리 페이지가 열려있음
      // When: 사용자가 여러 할 일을 순차적으로 추가함
      // Then: 모든 할 일이 목록에 표시되어야 함
    });
  });
```
  
## BDD (해동 주도 개발)

- 테스트 시나리오를 자연어와 가깝게 표현하는 방법
- Given/When/Then 패턴을 활용하여 표현할 수 있음

```text
Given: 사용자의 계좌 잔액이 1000원일 때
When: 사용자가 500원을 인출하면
Then: 계좌 잔액은 500원이 되어야 한다
```


## 하향식 TDD (Top-Down TDD)
- 시스템의 상위 수준(인터페이스나 API)부터 테스트하고 구현
- 사용자나 클라이언트 관점에서 시스템이 어떻게 동작해야 하는지 정의
- 필요한 하위 컴포넌트는 처음에는 목(mock) 객체로 대체
- 점차 하위 레벨로 내려가면서 실제 구현으로 대체
- 전체적인 시스템 설계를 먼저 고려할 수 있어 큰 그림을 놓치지 않도록 함

## 상향식 TDD (Bottom-Up TDD)
- 기본 구성 요소부터 시작하여 점차 상위 레벨로 올라가는 접근 방식
- 시스템의 가장 기본적인 컴포넌트부터 테스트하고 구현현
- 완성된 하위 컴포넌트를 조합하여 더 복잡한 기능을 구현
- 각 컴포넌트가 단독으로 제대로 동작함을 보장
- 기술적 세부 사항에 먼저 집중