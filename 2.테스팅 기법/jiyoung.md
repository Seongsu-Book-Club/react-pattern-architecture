# Ch.5 리액트테스팅

## keyword

`stubbing`

`회귀(regression)` : 새로운 코드의 추가로 인해 기존에 잘 동작하던 기능을 깨트리는 것

## 5.1 테스트가 필요한 이유

## 5.2 테스트 종류

단위 테스트: 개별 컴포넌트 & 함수의 기능 **격리** 테스트 → 권장

통합 테스트: 다른 모듈과의 상호작용 체크 → 중요도가 높아짐

E2E 테스트: 전체 애플리케이션의 흐름. 시작 ~ 종료의 사용자 행동 묘사 → 중요도가 높아짐

시각적 회귀 테스트: 시각적인 부분 스냅샷 & 이전 버전과 비교

정적 검사: 린팅 & 타입 검사 & 눈으로 분석

## 5.3 단위 테스팅

- **BDD**: 개발자와 QA, 비개발자간의 협업을 강조.
  - 개발 구현 시작 전 요구사항의 일치를 정리
  - 일반 언어의 중요성 강조 → 가이드라인 기준 설정
  - ex) test VS it → it으로 영어문장처럼 이해하기 쉽게.
- describe 중첩
  - 관련된 테스트를 그룹으로 묶을 때 사용
- React Testing Library
  - 단순 렌더링 테스트

## 5.4 통합 테스트

컴포넌트 간의 상호작용 / 서버 ↔ 클라이언트 사이의 상호작용

- 실제 테스트하는 것에 맞게 테스트명을 정확하게 정의하기
- jsdom: 메모리 위에서 동작하는 헤드리스 브라우저
  - Node.js 환경에서 테스트가 실행될 때 가상의 DOM을 통해 브라우저와 유사한 환경 제공

## 5.5 cypress

- cy.intercept API로 네트워크 요청을 가로채 고정된 응답값을 반환한다
  - 반환되는 데이터를 제어해 실제 API 환경의 불안정한 상황으로부터 격리

```tsx
it('display the quote content', () => {
	cy.intercept('GET', 'https://api.quotable.io/quotes/random*', {
		statusCode: 200,
		body: 배열
	}
})
```

<aside>
💡

너무 easy. e2e 예시도 너무 짧음. 맛보기 느낌?

</aside>

# Ch.6 리팩터링 기법

리팩터링: 기존 코드베이스에서 기능 동작의 **외적인 변화 없이** 구조를 개선하는 것

- 작고 점진적인 개선을 오랜 시간 지속해서 수행하는 것

<aside>
💡

작업하면서 리팩터링할 시간이 없다~ 이러는데 사실 한번에 큰걸 변경하려는 부담감 때문에 시간내어 시도하기가 어려운 듯. 작고 점진적인 변화부터 시도하면 어렵지 않을 것 같다.

</aside>

### 리팩터링 vs 리스트럭처링(restructuring)

- 구조를 바꾸는 경우는 restructuring.
- 가독성을 높이는 것이 리팩터링

- 리팩터링
  - 작은 컴포넌트로 쪼개기. 복잡한 분기 조건을 정리(기능은 동일)
- 리스트럭처링
  - 대규모 변경작업으로 내부 구조 & 외부 특성까지 영향을 미칠 수 있는 경우.
  - 설계, 데이터 모델, 인터페이스 등의 변경
  - 성능 향상 또는 심각한 기술 부채 해결

<aside>
💡

두 개를 같다고 생각해왔던 것 같다.
최근에 recoil → zustand 변경작업을 진행했는데 대놓고 리팩터 라벨 붙임
ㅋㅋㅋ

![image.png](attachment:5a8a434c-44e6-4235-b8ab-59a80879edf8:image.png)

</aside>

## 리팩터링 전에 부족한 테스트를 추가한다

<aside>
💡

기존 코드 베이스 그대로 리팩터링을 그대로 진행했는데,
리팩터링으로 인한 변경사항을 정량적으로 체크하기 위해선 부족한 테스트를 추가하는 것이 좋을 것 같다.

생각지도 못한 꿀팁

</aside>

```tsx
interface Item {
  id: string;
  price: number;
  quantity: number;
}

class ShoppingCart {
  cartItems: Item[] = [];

  addItemToCart(id: string, price: number, quantity: number) {
    this.cartItems.push({ id, price, quantity });
  }

  calculateTotal() {
    let total = 0;
    for (let i = 0; i < this.cartItems.length; i++) {
      const item = this.cartItems[i];
      let subTotal = item.price * item.quantity;
      if (item.quantity > 10) {
        subTotal *= 0.9;
      }
      total += subTotal;
    }
    return total;
  }
}

export { ShoppingCart };
```

일단 입력과 출력이 동일해야 하는데 내부적으로 cartItems라는 배열에 push를 해서 원본 배열을 손상시키고 있다. addItemToCart는 배열을 params로 받아서 변경된 배열을 리턴해야 함.

```tsx
addItemToCart(cartItems: Item[], newItem: Item) {
  return cartItems.concat(newItem);
}
```

calculateTotal는 불필요한 let을 쓰고 있고, 마찬가지로 배열을 내부적으로 참조하고 있음. 외부에서 배열을 받아야 함.

```tsx
  calculateTotal(cartItems: Item[]) {
    return cartItems.reduce((total, item) => {
      let subTotal = item.price * item.quantity;
      if (item.quantity > 10) {
        subTotal *= 0.9;
      }
      return total + subTotal;
    }, 0);
  }
```

subTotal은 또 머임? subTotal 계산 로직을 함수로 분리해야 함.

```tsx
calculateTotal(cartItems: Item[]) {
    return cartItems.reduce((total, item) => {
      let totalPrice = item.price * item.quantity;
      return total + (totalPrice * applyDiscount(item));
    }, 0);
  }

  const applyDiscount = (item: Item ) => {
  return item.quantity > 10 ? 0.9 : 1;
}
```

이렇게만 하면 함수 내부를 보기 전까지는 얼마나 discount하는지 모르기 때문에 외부에서 명시적으로 볼 수 있게 applyDiscount를 변경한다. totalPrice는 이제 변경되는 값이 아니므로 상수로 변경한다.

```tsx
calculateTotalCart(items: Item[]) {
  return items.reduce((total, item) => {
    const pricingBaseQuantity = item.price * item.quantity;
    const discount = applyDiscount({ item, discountQuantity: 10, discountRate: 0.9 })
    return total + (pricingBaseQuantity * discount );
  }, 0);
}

// utils.ts
interface ApplyDiscountParams {
	item: T;
	discountQuantity: number;
	discountRate: number;
}

export const applyDiscount = ({ item, discountQuantity, discountRate  }: ApplyDiscountParams) => {
	return item.quantity > discountQuantity ? discountRate : 1;
}
```

<aside>
💡

cartItems 변수 이름을 items로 바꾸는 것에 대해.

</aside>

<aside>
💡

저자는 할인율 0.9를 상수로 추출(DISCOUNT_RATE)

</aside>

<aside>
💡

무조건 함수형 프로그래밍 언어를 쓰는 것이 좋지 않을 수도 있다.

대용량 데이터 세트를 다룰 때 반복문을 대체하는 것.

→ 어떠한 경우인지?

</aside>

```tsx
calculateTotal() {
  return this.items.reduce((total, item) => {
    const subTotal = item.price * item.quantity;
    return total + applyDiscountIfEligible(item, subTotal);
  }, 0);
}


const DISCOUNT_RATE = 0.9;

function isDiscountEligible(item: Item) {
  return item.quantity > 10;
}

export function applyDiscountIfEligible(
  item: Item,
  subTotal: number,
) {
  return isDiscountEligible(item)
    ? subTotal * DISCOUNT_RATE
    : subTotal;
}
```

개인적으로 이 경우, IfEligible이 어떠한 경우인지 함수를 직접 타고 들어가야 되기 때문에 딥한 추상화 같음.

# Ch.7 TDD

레드-그린-리팩터 루프

- 기본 TDD: 단위 테스트에 집중
- ATDD(Acceptance Test Driven Development): 이해관계자와 협업해 사용자 관점에서 완료 상태 케이스를 정의.
- BDD(Behavior Driven Development): 기대하는 결과값 테스트가 아닌 예상하는대로 동작하는지 테스트. (먼 차이임..?)
  - Cucumber 프레임워크를 사용한다
    - 자연어로 단계별 정의하면 cypress 코드로 변환된다 한다…! 미칀…!

### 태스킹: 스토리나 기능을 작은 태스크로 나눈다

1. 사용자 스토리와 요구사항 검토
2. 논리적인 구성요소 식별: 스토리를 도메인 개념, 비즈니스 규칙, 사용자 작업 단위 로직 단위로 나눈다
3. 목록 작성: 15 ~ 30분안에 구현이 가능해야 함
4. 태스크 순서 배열: 정상 흐름 → 예외 케이스로 오류 처리
5. 태스크를 테스트 케이스 식별

## 7.3 ~ 직접 만들어 보면서 확인해보자
