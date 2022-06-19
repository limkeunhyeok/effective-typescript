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
