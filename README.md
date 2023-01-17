# useEffect와 useEffect 실행 순서

# useEffect 훅이란?

컴포넌트가 렌더링 될 때 특정 작업을 실행할 수 있도록 하는 Hook이다.

일반적으로 함수형 컴포넌트 내부에서 발생하는 **Side Effect**를 처리할 때 **react** 모듈에서 제공하는 `useEffect`함수를 사용한다.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/3dcba8cd-4f4f-4429-9cd2-2670d5e11378/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230117%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230117T104830Z&X-Amz-Expires=86400&X-Amz-Signature=07e62ecc9f29f3b314487d2a2097ade3a289b980de812254464830cb012ef1cd&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

react 공식문서에 따르면, `useEffect` 는 렌더링 이후에 어떤 일을 수행하는 지를 말해준다고 한다.

## 언제 사용할까?

`useEffect` Hook은 `dependency` 배열 내에 지정된 값의 변화가 일어났을 때 이펙트 함수가 실행된다. 이러한 특성으로 인해 컴포넌트가 마운트될 때 API를 통해 데이터를 가져오거나, `state`  값 또는 `props`  값이 변경될 때 특정 함수를 실행시키는 등의 작업을 하는 데 사용된다.

## ****useEffect 의 dependency array****

### **매 렌더링마다 실행되는 것 방지**

이펙트가 매 렌더링마다 실행되는 것을 방지하기 위해 두 번째 인자에 값을 넘길 수 있다. 이 배열은 이펙트가 실행되기 위한 의존성 배열이다. 배열 내에 작성된 값 중 하나가 변경된 경우 이펙트가 실행된다.

```jsx
const [value, setValue] = useState(0);
 
React.useEffect(() => {
  // 이펙트에서 'value' 값을 사용 중입니다.
  // dependency 배열에 'value' 값을 추가해야 합니다.
  console.log(value);
}, [value]);  // 'value' 값이 변경될 때 이펙트를 실행합니다.
```

### **마운트시에 한 번만 실행**

컴포넌트가 마운트될 때 한 번만 실행하고, 언마운트될 때 정리시키기 위한 방법으로 `useEffect`의 `dependency` 배열에 빈 값(`[]`)을 전달할 수 있다.

```jsx
React.useEffect(() => {
  console.log('component mounted');
 
  return () => console.log('component unmounting');
}, []);
```

컴포넌트는 최초 렌더링 이후 '`component mounted`' 텍스트를 출력할 것이고, 컴포넌트가 언마운트되는 시점에 '`component unmounting`' 텍스트가 출력된다.

이펙트 함수 내에서 사용하는 값이 있음에도 의존성 배열을 비워놓는 경우 의도치 않은 버그가 생길 수 있음에 유의해야 한다.

## useEffect 실행순서

`useEffect`는 기본적으로 컴포넌트가 렌더링 된 이후에 실행된다.
부모 컴포넌트 내부에 자식 컴포넌트가 존재한다면 자식 컴포넌트의 `useEffect`가 먼저 실행 된 후 부모 컴포넌트의 `useEffect`가 실행된다.
부모 컴포넌트가 완전히 렌더링 될려면 자식 컴포넌트가 먼저 렌더링이 완료 되어야 가능한 것이라고 생각하면 된다.

다음의 코드는 어떻게 실행이 될까?

```jsx
function App() {
  useEffect(() => {
    console.log(1);
  }, []);

  return <OuterBox/>
}

const OuterBox: FC = () => {
  useEffect(() => {
    console.log(2);
  }, []);

  return <InnerBox/>
};

const InnerBox: FC = () => {
  useEffect(() => {
    console.log(3);
  }, []);

  return ...
};
```

하위에 있는 컴포넌트 먼저 실행된다.

```jsx
//실행결과
3
2
1
```

`useEffect`는 컴포넌트가 렌더링이 된 후에 실행되는 것이다. 

`App`이 `render`되기 위해서 `OuterBox`가 먼저 렌더링이 되어야 되고, `OuterBox`가 완전히 렌더링 되기 위해서는 `InnerBox`가 렌더링이 되어야 한다.

### return 문이 useEffect 보다 먼저 읽힌다.

아래의 코드 결과는 `return` -> `useEffect` 순이다!

```jsx
import { useEffect, useState } from 'react';

function Detail() {

  useEffect(() => {
    console.log(`useEffect`);
  }, []);

  return (
    <div>
      {
        console.log(`return`)
      }
    </div>
  )
}
export default Detail;
```

그 것은 왜일까?

`useEffect` 자체가 원래 `return` 후에 실행되기 때문이다.

또한, `A` 페이지 컴포넌트와 `B` 페이지 컴포넌트가 있고, `A`에서 `B`로 라우터 이동을 한다고 가정할 때, 아래와 같은 순서로 실행이 된다.

1. `A`컴포넌트 렌더링

2. `A`컴포넌트의 `useEffect` 실행

3. `A`>> `B`로 페이지 이동 하면

4. `B`컴포넌트 렌더링

5. `A`컴포넌트 `useEffect return` 실행

6. `B`컴포넌트 `useEffect` 실행

## ****useEffect의 cleanup 함수****

****`useEffect  hook` 에서 `return`시 사용하는 함수**

- `cleanup` 함수를 사용하게 되면 렌더링이 될 때, 이전에 남은 함수가 실행되어 메모리 누수가 일어나는 일을 막을 수 있다.
- useEffect를 사용하여 컴포넌트에서 side effect를 수행하는데, 이때 렌더링이 발생시 메모리 누수가 일어날 수 있다.🍠 데이터 가져오기, 구독 설정, 수동으로 리액트 컴포넌트 DOM을 수정하는 것이 **side effects**이다.
- 특히 라우터와 useEffect 훅을 함께 사용한다면 이를 고려해야한다.

`useEffect`의 이펙트 함수에서 반환되는 `cleanup` 함수는 컴포넌트가 `unmount`되는 시점에만 실행되지는 않는다. `cleanup` 함수는 이펙트가 실행되기 전에 매번 호출되며, 컴포넌트가 `unmount`될 때 역시 실행될 때 정리된다. 필요에 따라 모든 렌더링 전후에 사이드 이펙트를 발생시킬 수 있으므로 `componentWillUnmount`보다 더 자주 호출된다.

### useEffect 리턴함수 동작 순서

1. `props`나 `state`가 업데이트

2. 컴포넌트 **리렌더링**

3. 이전 이펙트 **클린업**

4. 새로운 이펙트 실행

`props.id`가 10에서 `props.id`가 20으로 업데이트 된 상황

1. `props.id` = 20으로 변경

2. 컴포넌트 리렌더링

3. 이전 이펙트 클린업 (이 이펙트 함수는 **`props.id = 10`**을 바라보고 있다.)

4. 새로운 이펙트 실행 (이 이펙트 함수는 **`props.id = 20`**을 바라보고 있다.)

컴포넌트를 리렌더링 하기 전에 클린업 되는게 아니라 **리렌더링이 되고 난 다음에 클린업**
되고 그 다음 이펙트가 새로 실행이 된다.

### 정리

리렌더링 -> 이전 이펙트 클린업 -> 이펙트 실행

1. `useEffect`함수의 `SideEffect`함수가 실행되기 전에는 작동하지 않는다.
2. 모든 `SideEffect`함수가 실행되기 전 작동한다.
