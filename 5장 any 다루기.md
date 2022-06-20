# Effective Typescript

## 5장 any 다루기

### 아이템 38. any 타입은 가능한 한 좁은 범위에서만 사용하기

```typescript
function f1() {
  const x: any = expressionReturningFoo();
  processBar(x);
}

function f2() {
  const x = expressionReturningFoo();
  processBar(x as any);
}
```

- f1보다 f2를 권장한다.
  - any 타입이 processBar 함수의 매개변수에서만 사용된 표현식이므로 다른 코드에는 영향을 미치지 않기 때문이다.

```typescript
function f1() {
  const x: any = expressionReturningFoo();
  processBar(x);
  return x;
}

function g() {
  const foo = f1();
  foo.fooMethod(); // error
}
```

- g 함수 내에서 f1이 사용되므로 f1의 반환 타입인 any 타입이 foo의 타입에 영향을 미친다.
  - 함수에서 any를 반환하면 그 영향력은 프로젝트 전반에 퍼지게 된다.
  - 만약 f2처럼 사용 범위를 좁게 제한하면 함수 바깥으로 영향을 미치지 않는다.
- 타입스크립트가 함수의 반환 타입을 추론할 수 있는 경우에도 함수의 반환 타입을 명시하는 것이 좋다.

```typescript
function f1() {
  const x = expressionReturningFoo();
  // @ts-ignore
  processBar(x);
  return x;
}
```

- @ts-ignore를 사용한 다음 줄의 오류가 무시된다.
  - 그러나 근본적인 원인을 해결하는 것이 아니기 때문에 다른 곳에서 더 큰 문제가 발생할 수도 있다.

```typescript
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value as any,
  },
};
```

- 객체 전체를 any로 단언하는 것 보다, 최소한의 범위에만 any를 사용하는 것이 좋다.

#### 요약

- 의도치 않은 타입 안전성의 손실을 피하기 위해 any의 사용 범위를 최소한으로 좁혀야 한다.
- 함수의 반환 타입이 any인 경우 타입 안정성이 나빠진다. 따라서 any 타입을 반환하면 절대 안된다.
- 강제로 타입 오류를 제거하려면 any 대신 @ts-ignore를 사용하는 것이 좋다.

### 아이템 39. any를 구체적으로 변형해서 사용하기

- any를 자바스크립트에서 표현할 수 있는 모든 값을 아우르는 매우 큰 범위의 타입이다.
- 일반적인 상황에서는 any보다 더 구체적으로 표현할 수 있는 타입이 존재할 가능성이 높기 때문에 더 구체적인 타입을 찾아 타입 안전성을 높이도록 해야 한다.

```typescript
function getLengthBad(array: any) {
  return array.length;
}

function getLengthGood(array: any[]) {
  return array.length;
}
```

- 위 예시에서 any[]를 사용하는 함수가 더 좋다.
  - 함수 내의 array.length 타입이 체크된다.
  - 함수의 반환 타입이 any 대신 number로 추론된다.
  - 함수 호출될 때 매개변수가 배열인지 체크된다.
- 함수의 매개변수가 객체이긴 하지만 값을 알 수 없다면 `{[key: string]: any}`처럼 선언하면 된다.
  - object 타입은 객체의 키를 열거할 수는 있지만 속성에 접근할 수 없다는 점에서 약간 다르다.

#### 요약

- any를 사용할 때는 정말로 모든 값이 허용되어야만 하는지 면밀히 검토해야 한다.
- any보다 더 정확하게 모델링할 수 있도록 any[] 또는 `{[id: string]: any}` 또는 `() => any`처럼 구체적인 형태를 사용해야 한다.

### 아이템 40. 함수 안으로 타입 단언문 감추기

- 함수의 모든 부분을 안전한 타입으로 구현하는 것이 이상적이지만, 불필요한 예외 상황까지 고려해 가며 타입 정보를 힘드렉 구성할 필요는 없다.
- 함수 내부에는 타입 단언을 사용하고 함수 외부로 드러나는 타입 정의를 정확히 명시하는 정도로 끝내는 게 낫다.
- 프로젝트 전반에 위험한 타입 단언문이 드러나 있는 것보다, 제대로 타입이 정의된 함수 안으로 타입 단언문을 감추는 것이 더 좋은 설계이다.

#### 요약

- 타입 선언문은 일반적으로 타입을 위험하게 만들지만 상황에 따라 필요하기도 하고 현실적인 해결책이 되기도 한다. 불가피하게 사용해야 한다면, 정확한 정의를 가지는 함수 안으로 숨기도록 한다.

### 아이템 41. any의 진화를 이해하기

- 타입스크립트에서 일반적으로 변수의 타입은 변수를 선언할 떄 결정된다.

```typescript
function range(start: number, limit: number) {
  const out = []; // any[]
  for (let i = start; i < limit; i++) {
    out.push(i); // any[]
  }
  return out; // number[]
}
```

- out의 타입은 any[]로 선언되었지만 number 타입의 값을 넣는 순간부터 타입은 number[]로 진화(evolve)한다.
- 배열에 다양한 타입의 요소를 넣으면 배열의 타입이 확장되며 진화한다.
- 조건문에서는 분기에 따라 타입이 변할 수도 있다.
- 변수의 초깃값이 null인 경우도 any의 진화가 일어난다.
- any 타입의 진화는 noImplicitAny가 설정된 상태에서 변수의 타입이 암시적 any인 경우에만 일어난다.
  - 그러나 명시적으로 any를 선언하면 타입이 그대로 유지된다.
- any 타입의 진화는 암시적 any 타입에 어떤 ㄱ밧을 할당할 때만 발생한다.
- 어떤 변수가 암시적 any 상태일 때 값을 읽으려고 하면 오류가 발생한다.
- 암시적 any 타입은 함수 호출을 거쳐도 진화하지 않는다.
- 타입을 안전하게 지키기 위해서는 암시적 any를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 더 좋은 설계이다.

#### 요약

- 일반적인 타입들은 정제되기만 하는 반면, 암시적 any와 any[] 타입은 진화할 수 있다. 이러한 동작이 발생하는 코드를 인지하고 이해할 수 있어야 한다.
- any를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 안전한 타입을 유지하는 방법이다.

### 아이템 42. 모르는 타입의 값에는 any 대신 unknown을 사용하기

```typescript
function safeParseYAML(yaml: string): unknown {
  return parseYAML(yaml);
}

const book = safeParseYAML(`
  name: The Tenantof Wildfell Hall
  author: Anne Bronte
`);

alert(book.title); // error, unknown
book('read'); // error, unknown
```

- 어떠한 타입이든 any 타입에 할당 가능하다.
- any 타입은 어떠한 타입으로도 할당 가능하다.
- 타입 체커는 집합 기반이기 때문에 any를 사용하면 타입 체커가 무용지물이 된다.
- unknown은 any 대신 쓸 수 있는 타입 시스템에 부합하는 타입이다.
  - never 타입은 unknown과 정반대이다.
- unknown 타입인 채로 값을 사용하면 오류가 발생한다.
- 함수의 반환 타입인 unknown 그대로 값을 사용할 수 없기 때문에 Book으로 타입 단언을 해야 한다.
- 어떠한 값이 있지만 그 타입을 모르는 경우에 unknown을 사용한다.
- instanceof를 체크한 후 unknown에서 원하는 타입으로 변환할 수 있다.
- 제너릭을 사용한 스타일은 타입 단언문과 달라 보이지만 기능적으로는 동일하다.
  - 제너릭보다는 unknown을 반환하고 사용자가 직접 단언문을 사용하거나 원하는 대로 타입을 좁히도록 강제하는 것이 좋다.
- unknown의 경우는 분리되는 즉시 오류를 발생하게 되므로 더 안전하다.
- {} 타입은 null과 undefined를 제외한 모든 값을 포함한다.
- object 타입은 모든 비기본형(non-primitive) 타입으로 이루어진다.

#### 요약

- unknwon은 any 대신 사용할 수 있는 안전한 타입이다. 어떠한 값이 있지만 그 타입을 알지 못하는 경우라면 unknown을 사용하면 된다.
- 사용자가 타입 단언문이나 타입 체크를 사용하도록 강제하려면 unknown을 사용하면 된다.
- {}, object, unknown의 차이점을 이해해야 한다.

### 아이템 43. 몽키 패치보다는 안전한 타입을 사용하기

- 전역 변수를 사용하면 은연중에 프로그램 내에서 서로 멀리 떨어진 부분들 간에 의존성을 만들게 된다.
  - 함수를 호출할 때마다 부작용(side effect)을 고려해야 한다.
- interface 보강을 사용하면 any보다 나은 점
  - 타입이 더 안전하다. 타입 체커는 오타나 잘못된 타입의 할당을 오류로 표시한다.
  - 속성에 주석을 붙일 수 있다.
  - 속성에 자동완성을 사용할 수 있다.
  - 몽키 패치가 어떤 부분에 적용되었는지 정확한 기록이 남는다.

#### 요약

- 전역 변수나 DOM에 데이터를 저장하지 말고, 데이터를 분리하여 사용해야 한다.
- 내장 타입에 데이터를 저장해야 하는 경우, 안전한 타입 접근법 중 하나를 사용해야 한다.
- 보강의 모듈 영역 문제를 이해해야 한다.

### 아이템 44. 타입 커버리지를 추적하여 타입 안전성 유지하기

- any 타입이 존재할 수 있는 경우
  - 명시적 any 타입
  - 서드파티 타입 선언
- any 타입은 타입 안전성과 생산성에 부정적 영향을 미칠 수 있으므로 프로젝트에서 any의 개수를 추적하는 것이 좋다.
- npm의 type-coverage 패키지를 활용하여 any를 추적할 수 있다.

#### 요약

- noImplicitAny가 설정되어 있어도, 명시적 any 또는 서드파티 타입 선언(@types)을 통해 any 타입은 코드 내에 여전히 존재할 수 있다는 점을 주의해야 한다.
- 작성한 프로그램의 타입이 얼마나 잘 선언되었는지 추적해야 한다. 추적함으로써 any의 사용을 줄여 나갈 수 있고 타입 안전성을 꾸준히 높일 수도 있다.
