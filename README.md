## Automatic Batchin
React 17과 React 18은 상태 업데이트 방식에서 주요 차이점이 있다. React 17 이전의 버전에서는 이벤트 핸들러 내에서 발생한 여러 상태 업데이트가 자동으로 일괄 처리(batch)되었다. 
그러나, 비동기 코드 또는 프로미스 내에서의 상태 업데이트는 일괄 처리되지 않는다. 반면, React 18에서는 이벤트 핸들러 내의 상태 업데이트뿐만 아니라 비동기 코드 내의 상태 업데이트도 일괄 처리한다.

### 이벤트 핸들러 내부에서 여러개 상태 업데이트 

```javascript
import React, { useState } from 'react';

function MultipleStateUpdates() {
  const [counter, setCounter] = useState(0);
  const [flag, setFlag] = useState(false);

  const handleClick = () => {
    setCounter(prev => prev + 1);
    setCounter(prev => prev + 1);
    setFlag(prev => !prev);
  };

  return (
    <div>
      <p>Counter: {counter}</p>
      <p>Flag: {flag.toString()}</p>
      <button onClick={handleClick}>Update state</button>
    </div>
  );
}
```
* React 17: 여러 상태 업데이트가 자동으로 일괄 처리되어, DOM 렌더링은 한 번만 발생한다.
* React 18: React 17과 동일하게 동작한다.

### 이벤트 핸들러 내부 비동기 코드에서 여러개 상태 업데이트

```javascript
import React, { useState } from 'react';

function AsyncStateUpdates() {
  const [counter, setCounter] = useState(0);

  const handleClick = () => {
    new Promise(resolve => setTimeout(resolve, 1000))
      .then(() => {
        setCounter(prev => prev + 1);
        setCounter(prev => prev + 1);
      });
  };

  return (
    <div>
      <p>Counter: {counter}</p>
      <button onClick={handleClick}>Update state asynchronously</button>
    </div>
  );
}
```
* React 17: 비동기 코드 내의 상태 업데이트가 일괄 처리되지 않아, 각 상태 업데이트마다 별도의 렌더링이 발생한다.
* React 18: 새로운 자동 일괄 처리 메커니즘 덕분에 비동기 코드 내의 상태 업데이트도 일괄 처리되어, 한 번의 렌더링만 발생한다.


### 이벤트 핸들러 내부에서 일반적인 상태 업데이트 + 콜백 상태 업데이트

동기 코드와 비동기 코드에서 별도로 리렌더링이 발생하는 이유는 React의 상태 업데이트 메커니즘과 자바스크립트 이벤트 루프의 특성 때문이다.

동기 코드에서의 상태 업데이트: React는 이벤트 핸들러와 같은 동기 코드 내에서 발생하는 상태 업데이트를 일괄 처리하여 성능을 최적화한다. 여러 상태 업데이트가 있을 때, React는 이를 일괄 처리하여 한 번의 리렌더링만 수행한다.
비동기 코드에서의 상태 업데이트: 비동기 콜백(예: setTimeout, Promise.then 등)은 이벤트 루프의 '마이크로태스크' 또는 '매크로태스크' 큐에 들어가 다음 업데이트 사이클로 넘어간다. 

이 시점에서 React의 일괄 처리 범위 밖에 있으므로, 이러한 비동기 상태 업데이트는 별도의 리렌더링을 유발한다.
React 18에서도 이러한 동작은 유지되며, 비동기 코드 내의 상태 업데이트가 여전히 일괄 처리 범위 밖에 있기 때문에 별도로 리렌더링이 발생한다.

```javascript
import React, { useState } from 'react';

function CallbackStateUpdates() {
  const [counter, setCounter] = useState(0);

  const handleClick = () => {
    setCounter(prev => prev + 1);
    setTimeout(() => {
      setCounter(prev => prev + 1);
    }, 1000);
  };

  return (
    <div>
      <p>Counter: {counter}</p>
      <button onClick={handleClick}>Update state with callback</button>
    </div>
  );
}
```
* React 17: setTimeout 내의 상태 업데이트는 일괄 처리되지 않아 별도의 렌더링이 발생한다.
* React 18: React 17과 동일하게 동작합니다. 비동기 콜백 내에서의 상태 업데이트는 여전히 자동으로 일괄 처리되지 않는다.

### ReactDOM.unstable_batchedUpdates[참고]
ReactDOM.unstable_batchedUpdates는 React에서 제공하는 API로, 이를 사용하면 수동으로 여러 상태 업데이트를 하나의 배치(batch)로 그룹화하여 성능을 최적화할 수 있다. 
이름에 "unstable_"이 붙어 있지만, 많은 라이브러리에서 널리 사용되고 있으며 안정적으로 작동한다. 하지만 이 API는 내부용으로 고려되어 있기 때문에 향후 React 버전에서 변경될 수 있다.

기본적으로 React는 DOM 이벤트 핸들러 내부의 상태 업데이트를 자동으로 배치 처리한다. 그러나 비동기 작업 또는 DOM 이벤트 핸들러 외부에서 상태를 업데이트할 때는 이러한 최적화가 적용되지 않는다. 
이런 경우에 ReactDOM.unstable_batchedUpdates를 사용하면 여러 상태 업데이트를 강제로 하나의 배치로 그룹화할 수 있다.
```javascript
import React, { useState } from 'react';
import ReactDOM from 'react-dom';

function MyComponent() {
  const [counter, setCounter] = useState(0);

  const handleAsyncUpdate = () => {
    setTimeout(() => {
      ReactDOM.unstable_batchedUpdates(() => {
        setCounter(prev => prev + 1);
        setCounter(prev => prev + 1);  // 두 번째 업데이트
      });
    }, 1000);
  };

  return (
    <div>
      <p>Counter: {counter}</p>
      <button onClick={handleAsyncUpdate}>Increment Counter Async</button>
    </div>
  );
}
```

## Concurrent Rendering
React 18에서 도입된 "Concurrent Rendering"은 사용자 경험을 개선하고 애플리케이션의 렌더링 성능을 향상시키기 위한 기능입니다. 이 기능은 애플리케이션 UI의 반응성을 향상시키기 위해 작업을 백그라운드에서 수행할 수 있도록 합니다. 
즉, 브라우저가 다른 중요한 작업들(예: 사용자 입력 같은 것)을 처리하는 동안 React는 렌더링 작업을 백그라운드에서 진행할 수 있습니다.

Concurrent Rendering의 주요 장점은 다음과 같습니다:

* 1.작업 분할: 렌더링 작업을 작은 청크로 나누고, 브라우저가 다른 작업을 수행할 시간을 만듭니다. 이렇게 함으로써 애플리케이션이 더 반응적으로 느껴집니다.
* 2.우선 순위 설정: React는 어떤 업데이트가 더 중요한지 파악하고, 중요한 업데이트를 먼저 수행합니다. 예를 들어, 사용자 입력과 관련된 업데이트는 데이터 패칭과 관련된 업데이트보다 먼저 처리될 수 있습니다.
* 3.중단 가능한 렌더링: React는 현재 진행 중인 렌더링 작업을 중단하고, 더 중요한 작업을 먼저 처리할 수 있습니다. 이후에 중단된 작업을 다시 시작할 수 있습니다.

React 18에서는 startTransition을 사용하여 상태 변경을 '전환(transition)'으로 표시할 수 있습니다. 이를 통해 React는 이러한 상태 변경을 우선 순위가 더 낮은 백그라운드 작업으로 처리할 수 있습니다.
이 예시에서 startTransition은 setCount를 비동기적으로 만들어, 만약 다른 높은 우선순위 작업이 있다면, 이 상태 변경을 지연시킬 수 있게 합니다. 이는 애플리케이션이 사용자와의 상호작용에 더 민첩하게 반응할 수 있게 돕습니다.

```javascript
import { useState } from 'react';
import { startTransition } from 'react';

function MyComponent() {
  const [count, setCount] = useState(0);

  function handleIncrement() {
    startTransition(() => {
      setCount(c => c + 1);
    });
  }

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleIncrement}>Increment</button>
    </div>
  );
}
```

### 비동시성 모드 vs 동시성 모드
비동기적 모드 (전통적인 React 렌더링):<br/>
* 렌더링 작업이 시작되면 중단할 수 없으며, 작업이 완료될 때까지 다른 작업(사용자 입력 처리 등)을 수행할 수 없다.
* 큰 컴포넌트 트리가 있을 때 UI가 "멈춤" 상태가 될 수 있으며, 이는 사용자 경험에 부정적인 영향을 미칠 수 있다.

동시성 모드 (React 18+):<br/>
* 렌더링 작업을 작은 단위로 나눌 수 있으며, 필요에 따라 작업을 중단하고 재개할 수 있다.
* 이를 통해 애플리케이션은 항상 반응적이며, 사용자 입력과 같은 중요한 작업을 즉시 처리할 수 있다.
* 더 나은 사용자 경험을 제공하면서도 컴포넌트의 업데이트를 더 효율적으로 관리할 수 있다.

동시성 랜더링 도입된 덕분에 리액트18에서는 suspense,서버 랜더링,변이같은 기능이 추가적으로 도입되었다

## Suspense
React에서 비동기 작업을 보다 쉽게 처리할 수 있게 해주는 기능이다. 주로 데이터 패칭이나 코드 분할을 수행하는 동안 로딩 표시와 같은 대체 컨텐츠를 렌더링하는데 사용된다. 이를 통해 앱의 로딩 상태를 세련되게 관리할 수 있다.
Suspense는 주로 두 가지 주요 경우에 사용된다: 데이터 패칭(Data Fetching)과 코드 분할 및 지연 로딩(Code Splitting & Lazy Loading)

### 데이터 패칭(Data Fetching)
React 18의 Suspense는 컴포넌트가 데이터 패칭을 기다리는 동안 "fallback" 컨텐츠를 표시할 수 있도록 해준다. 
이는 컴포넌트가 필요로 하는 데이터가 준비될 때까지 로딩 인디케이터나 다른 UI 요소를 렌더링 할 수 있음을 의미한다.

예를 들어, 데이터를 패칭하는 중인 컴포넌트가 있고, 해당 데이터가 준비될 때까지 로딩 스피너를 보여주고 싶다면, Suspense를 사용하여 이를 수행할 수 있다.
```javascript
import React, { Suspense } from 'react';
import { fetchProfileData } from './fakeApi';

const profileData = fetchProfileData();

function ProfilePage() {
  return (
    <Suspense fallback={<Spinner />}>
      <ProfileDetails />
      <ProfileTimeline />
    </Suspense>
  );
}

function ProfileDetails() {
  const user = profileData.user.read();
  return <h1>{user.name}</h1>;
}

function ProfileTimeline() {
  const posts = profileData.posts.read();
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}

function Spinner() {
  return <div>Loading...</div>;
}
```

### 코드 분할 및 지연 로딩(Code Splitting & Lazy Loading)
Suspense는 또한 애플리케이션의 특정 부분을 지연 로딩하고 코드 분할하는데 사용된다. 
이는 큰 앱의 성능을 향상시키기 위해 특정 섹션 또는 컴포넌트가 필요할 때만 코드를 로드하고자 할 때 유용하다.

React.lazy는 동적 임포트를 사용하여 컴포넌트를 로드한다. 이 컴포넌트들은 초기 로드 시에는 포함되지 않지만, 나중에 필요할 때 로드된다.
Suspense는 이러한 컴포넌트들이 로드되는 동안 표시될 UI를 지정한다.

```javascript
import React, { Suspense } from 'react';
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <OtherComponent />
    </Suspense>
  );
}
```

