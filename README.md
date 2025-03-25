## Suspense
React에서 비동기 작업을 보다 쉽게 처리할 수 있게 해주는 기능이다. 주로 데이터 패칭이나 코드 분할을 수행하는 동안 로딩 표시와 같은 대체 컨텐츠를 렌더링하는데 사용된다. 이를 통해 앱의 로딩 상태를 세련되게 관리할 수 있다.
Suspense는 주로 두 가지 주요 경우에 사용된다: 데이터 패칭(Data Fetching)과 코드 분할 및 지연 로딩(Code Splitting & Lazy Loading)

#### 1.데이터 패칭(Data Fetching)
React 18의 Suspense는 컴포넌트가 데이터 패칭을 기다리는 동안 "fallback" 컨텐츠를 표시할 수 있도록 해준다. 
이는 컴포넌트가 필요로 하는 데이터가 준비될 때까지 로딩 스피너나 다른 UI 요소를 렌더링 할 수 있음을 의미한다.

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

#### 2.코드 분할 및 지연 로딩(Code Splitting & Lazy Loading)
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


## Error Boundary
Error Boundary는 React 컴포넌트 트리에서 발생하는 JavaScript 오류를 잡아내어 UI의 특정 부분이 오류로 인해 깨지는 것을 방지한다. Error Boundary는 컴포넌트의 자식에서 발생하는 오류를 잡아내고, 대체 UI를 렌더링할 수 있다. 이는 애플리케이션이 예기치 않은 오류로 인해 전체적으로 중단되는 것을 방지하고, 사용자에게 보다 안정적인 경험을 제공한다.

```javascript
import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error("Error caught by ErrorBoundary:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}

function BuggyComponent() {
  throw new Error("I crashed!");
}

function App() {
  return (
    <ErrorBoundary>
      <BuggyComponent />
    </ErrorBoundary>
  );
}
```

## Code Splitting
Code Splitting은 애플리케이션의 성능을 최적화하기 위해 코드의 일부를 분할하여 필요할 때만 로드하는 기법이다. React에서는 React.lazy와 Suspense를 사용하여 코드 분할을 쉽게 구현할 수 있다. 이 방법은 초기 로딩 시간을 줄이고, 사용자가 특정 기능을 요청할 때만 관련 코드를 로드하여 네트워크 사용량을 최적화할 수 있다. Code Splitting은 특히 대규모 애플리케이션에서 유용하며, 사용자가 필요로 하지 않는 코드를 미리 로드하지 않도록 하여 성능을 향상시킨다.

```javascript
import React, { Suspense } from 'react';
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <h1>Welcome to My App</h1>
      {/* Suspense를 사용하여 로딩 중에 표시할 UI를 정의 */}
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}

export default MyComponent;
```

#### 1.API Fetch 오류 처리
Error Boundary는 렌더링 중 발생하는 오류를 처리하지만, 비동기 작업(예: API 요청)에서 발생하는 오류는 처리하지 않는다. API 요청에서 발생하는 오류는 명령형 코드에서 발생하므로, try/catch 또는 .catch() 메서드를 사용하여 처리해야 한다.

#### 2.클래스형 컴포넌트로 구현해야 하는 이유
Error Boundary는 componentDidCatch와 같은 라이프사이클 메서드를 필요로 한다. 이 메서드는 컴포넌트의 자식에서 발생하는 오류를 감지하고 처리하는 데 사용된다. 현재 React에서는 이러한 라이프사이클 메서드를 함수형 컴포넌트에서 직접 사용할 수 없기 때문에, Error Boundary는 클래스형 컴포넌트로 구현해야 한다.

#### 3.react-error-boundary
react-error-boundary 라이브러리는 Error Boundary를 더욱 쉽게 설정하고 사용할 수 있도록 도와주는 유틸리티 라이브러이다. 이 라이브러리를 사용하면 함수형 컴포넌트에서도 Error Boundary의 기능을 쉽게 구현할 수 있다.

```javascript 
import React, { useState } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>Something went wrong:</p>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function BuggyComponent({ resetKey }) {
  if (resetKey === 1) {
    throw new Error("I crashed!");
  }
  return <div>Component is working fine.</div>;
}

function App() {
  const [resetKey, setResetKey] = useState(0);

  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => {
        // Reset the state of your app so the error doesn't happen again
        setResetKey(prevKey => prevKey + 1);
      }}
      resetKeys={[resetKey]}
    >
      <BuggyComponent resetKey={resetKey} />
    </ErrorBoundary>
  );
}

export default App;
```


## Automatic Batching
React 17과 React 18은 상태 업데이트 방식에서 주요 차이점이 있다. React 17 이전의 버전에서는 이벤트 핸들러 내에서 발생한 여러 상태 업데이트가 자동으로 일괄 처리(batch)되었다. 
그러나, 비동기 코드 또는 프로미스 내에서의 상태 업데이트는 일괄 처리되지 않는다. 반면, React 18에서는 이벤트 핸들러 내의 상태 업데이트뿐만 아니라 비동기 코드 내의 상태 업데이트도 일괄 처리한다.

#### 1.이벤트 핸들러 내부에서 여러개 상태 업데이트 

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

#### 2.이벤트 핸들러 내부 비동기 코드에서 여러개 상태 업데이트

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

* 동기 코드에서의 상태 업데이트: React는 이벤트 핸들러와 같은 동기 코드 내에서 발생하는 상태 업데이트를 일괄 처리하여 성능을 최적화한다. 여러 상태 업데이트가 있을 때, React는 이를 일괄 처리하여 한 번의 리렌더링만 수행한다.
* 비동기 코드에서의 상태 업데이트: 비동기 콜백(예: setTimeout, Promise.then 등)은 이벤트 루프의 '마이크로태스크' 또는 '매크로태스크' 큐에 들어가 다음 업데이트 사이클로 넘어간다. 

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
* React 18: React 17과 동일하게 동작한다. 비동기 콜백 내에서의 상태 업데이트는 여전히 자동으로 일괄 처리되지 않는다.

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
React 18에서 도입된 "Concurrent Rendering"은 사용자 경험을 개선하고 애플리케이션의 렌더링 성능을 향상시키기 위한 기능아다. 이 기능은 애플리케이션 UI의 반응성을 향상시키기 위해 작업을 백그라운드에서 수행할 수 있도록 한다. 
즉, 브라우저가 다른 중요한 작업들(예: 사용자 입력 같은 것)을 처리하는 동안 React는 렌더링 작업을 백그라운드에서 진행할 수 있다.

Concurrent Rendering의 주요 장점은 다음과 같다:

* 1.작업 분할: 렌더링 작업을 작은 청크로 나누고, 브라우저가 다른 작업을 수행할 시간을 만든다. 이렇게 함으로써 애플리케이션이 더 반응적으로 느껴진다.
* 2.우선 순위 설정: React는 어떤 업데이트가 더 중요한지 파악하고, 중요한 업데이트를 먼저 수행한다. 예를 들어, 사용자 입력과 관련된 업데이트는 데이터 패칭과 관련된 업데이트보다 먼저 처리될 수 있다.
* 3.중단 가능한 렌더링: React는 현재 진행 중인 렌더링 작업을 중단하고, 더 중요한 작업을 먼저 처리할 수 있다. 이후에 중단된 작업을 다시 시작할 수 있다.

React 18에서는 startTransition을 사용하여 상태 변경을 '전환(transition)'으로 표시할 수 있다. 이를 통해 React는 이러한 상태 변경을 우선 순위가 더 낮은 백그라운드 작업으로 처리할 수 있다.
이 예시에서 startTransition은 setData를 비동기적으로 처리하여, 만약 다른 높은 우선순위 작업(예: 사용자 입력)이 있다면, setData 상태 변경을 지연시킬 수 있게 한다. 이는 애플리케이션이 사용자와의 상호작용에 더 민첩하게 반응할 수 있게 돕는다.

```javascript
import { useState } from 'react';
import { startTransition } from 'react';

function MyComponent() {
  const [count, setCount] = useState(0);
  const [data, setData] = useState(null);

  function handleIncrement() {
    // 버튼 클릭 이벤트: 우선순위 처리
    setCount(c => c + 1);

    // 데이터 패칭 후 UI 업데이트: startTransition 사용
    fetchData().then(response => {
      startTransition(() => {
        setData(response);
      });
    });
  }

  async function fetchData() {
    const response = await axios.get();
    return response;
  }

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleIncrement}>Increment</button>
      <div>Data: {data}</div>
    </div>
  );
}

```
startTransition은 React가 UI 업데이트의 우선순위를 관리할 수 있게 해준다. 긴급하지 않은 업데이트(예: 데이터 패칭 후의 UI 업데이트)는 startTransition 내부에서 처리되어, 긴급한 업데이트(예: 키 입력, 버튼 클릭)가 더 빠르게 처리된다.


### 비동시성 모드 vs 동시성 모드
비동기적 모드 (전통적인 React 렌더링):<br/>
* 렌더링 작업이 시작되면 중단할 수 없으며, 작업이 완료될 때까지 다른 작업(사용자 입력 처리 등)을 수행할 수 없다.
* 큰 컴포넌트 트리가 있을 때 UI가 "멈춤" 상태가 될 수 있으며, 이는 사용자 경험에 부정적인 영향을 미칠 수 있다.

동시성 모드 (React 18+):<br/>
* 렌더링 작업을 작은 단위로 나눌 수 있으며, 필요에 따라 작업을 중단하고 재개할 수 있다.
* 이를 통해 애플리케이션은 항상 반응적이며, 사용자 입력과 같은 중요한 작업을 즉시 처리할 수 있다.
* 더 나은 사용자 경험을 제공하면서도 컴포넌트의 업데이트를 더 효율적으로 관리할 수 있다.

## React Server Components
React Server Components는 React 18 버전에서 도입된 실험적인 기능이다. 이 기능은 서버 사이드에서 React 컴포넌트를 렌더링하고 클라이언트로 전송하는 것을 가능하게 해준다.

### 특징:
* 즉시 렌더링: 서버 컴포넌트는 서버에서 렌더링되고 결과 HTML이 클라이언트로 전송된다. 이를 통해 초기 페이지 로딩 시간이 단축된다.
* 데이터 페칭 간소화: 서버 컴포넌트는 데이터를 미리 가져와서 클라이언트로 전송할 수 있기 때문에, 클라이언트에서 복잡한 데이터 페칭 로직을 걱정할 필요가 없어진다.
* 클라이언트 번들 크기 감소: 서버 컴포넌트는 클라이언트로 전송되지 않기 때문에 클라이언트 사이드의 번들 크기를 줄일 수 있다.

### 컴포넌트 타입:
* 서버 컴포넌트(.server.js): 서버에서만 실행되며 결과가 HTML로 직렬화되어 클라이언트로 전송된다.
* 클라이언트 컴포넌트(.client.js): 클라이언트에서만 실행된다. 동적 인터랙션을 처리하는 데 사용된다.
* 공유 컴포넌트(.js): 서버와 클라이언트 양쪽에서 사용할 수 있다.

```javascript
// UserData.js
import { Pool } from 'pg'; 

const pool = new Pool({
  user: 'dbuser',
  host: 'database.server.com',
  database: 'mydatabase',
  password: 'mypassword',
  port: 5432,
});

export async function fetchUserData(userId) {
  const { rows } = await pool.query('SELECT * FROM users WHERE id = $1', [userId]);
  return rows[0];
}
```

```javascript
// UserDetails.server.js
import { fetchUserData } from './UserData';

export default function UserDetails({ userId }) {
  const userData = fetchUserData(userId);
  return (
    <div>
      <h1>{userData.name}</h1>
      <p>{userData.description}</p>
    </div>
  );
}
```
```javascript
// LikeButton.client.js
import { useState } from 'react';

export default function LikeButton() {
  const [liked, setLiked] = useState(false);
  
  return (
    <button onClick={() => setLiked(!liked)}>
      {liked ? 'Liked' : 'Like'}
    </button>
  );
}
```
```javascript
// App.js
import UserDetails from './UserDetails.server';
import LikeButton from './LikeButton.client';

export default function App({ userId }) {
  return (
    <div>
      <UserDetails userId={userId} />
      <LikeButton />
    </div>
  );
}
```
위의 예시에서 UserDetails는 서버 컴포넌트로서 사용자 정보를 서버에서 가져온 후 렌더링한다. LikeButton은 클라이언트 컴포넌트로서 사용자의 인터랙션을 처리한다. App 컴포넌트는 이 두 컴포넌트를 결합하여 전체 페이지를 구성한다.

## New Root API
React 18에서 도입된 New Root API는 React의 새로운 동시성(concurrency) 모델을 사용하기 위한 핵심 메커니즘이다. 동시성 모델을 통해 여러 개의 작업을 동시에 수행하면서, UI의 반응성을 향상시키는 것이 주요 목적이다. 이를 위해 두 가지 주요 함수, createRoot와 createBlockingRoot가 도입되었다.

### 1.createRoot
이 함수는 React 18의 동시성 모드를 활성화한다. 동시성 모드는 React의 작업을 비동기적으로 처리하여 애플리케이션의 반응성을 향상시킨다. Suspense, Concurrent Rendering 등의 새로운 기능과 함께 작동한다.

```javascript
import { createRoot } from 'react-dom';
import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```
위 코드에서 createRoot는 DOM 노드를 인자로 받아 React root를 생성하고, root.render를 사용하여 App 컴포넌트를 렌더링한다. 이를 통해 애플리케이션에서 React의 동시성 기능을 활성화하게 된다.

### 2.createBlockingRoot
이 함수는 동시성 모드를 비활성화하고, 레거시 모드와 비슷한 동작을 하게 한다. 그러나 React 18의 일부 성능 향상과 새로운 기능을 사용할 수 있다.

```javascript
import { createBlockingRoot } from 'react-dom';
import App from './App';

const root = createBlockingRoot(document.getElementById('root'));
root.render(<App />);
```
 함수는 애플리케이션에서 최신 비동기 기능을 사용하지 않고도 React 18의 일부 새로운 기능과 성능 개선을 제공한다.

## New Hook

### 1.useLayoutEffect
useLayoutEffect는 React에서 DOM 변형 후에 동기적으로 실행되는 Hook이다. 이 Hook은 DOM에서 레이아웃을 읽고 동기적으로 다시 렌더링해야 할 때 사용된다. 모든 DOM 변경이 완료된 후에 실행되며, 브라우저가 화면을 그리기 전에 실행된다. 이는 useEffect와의 주요 차이점이다.

```javascript
import React, { useState, useRef, useLayoutEffect } from 'react';

function LayoutEffectExample() {
  const [height, setHeight] = useState(0);
  const divRef = useRef(null);

  useLayoutEffect(() => {
    // DOM 요소의 높이를 측정
    const measuredHeight = divRef.current.getBoundingClientRect().height;
    setHeight(measuredHeight);
  }, []); 

  return (
    <div>
      <div ref={divRef} style={{ backgroundColor: 'lightblue', padding: '20px' }}>
        <p>This is a paragraph inside a div.</p>
      </div>
      <p>The height of the above div is: {height}px</p>
    </div>
  );
}

export default LayoutEffectExample;
```

#### useEffect 차이점

| Feature | useEffect | useLayoutEffect |
|--------------------|------------------------------------------------|------------------------------------------------|
| 실행 시점 | 브라우저가 화면을 그린 후 비동기적으로 실행 | DOM 변형 후, 브라우저가 화면을 그리기 전에 동기적으로 실행 |
| 주 사용 사례 | 비동기 데이터 패칭, 이벤트 리스너 등록 등 | DOM 측정 및 동기적 업데이트 필요 시 |
| 성능 영향 | 화면 그리기 후 실행되어 성능에 덜 민감 | 화면 그리기 전에 실행되어 성능에 민감 |


### 2.useTransition
useTransition는 연산 중에 일시적으로 UI 업데이트를 "지연"시키는 데 사용된다. 이를 통해 React는 사용자에게 빠른 응답성을 제공할 수 있다.
useTransition는 두 가지 값을 반환한다: startTransition 함수와 isPending 상태.

```javascript
import { useTransition } from 'react';

function App() {
  const [data, setData] = useState(null);
  const [startTransition, isPending] = useTransition();

  function handleClick() {
    startTransition(() => {
      // 비동기 작업, 예: 데이터 페칭
      fetchSomething().then(newData => setData(newData));
    });
  }

  return (
    <div>
      <button onClick={handleClick}>
        Load Data
      </button>
      {isPending ? "Loading..." : data}
    </div>
  );
}
```
* startTransition:  함수 내부에서 비동기 작업을 시작한다.
* isPending은 해당 트랜지션이 진행 중인지 여부를 나타낸다.

### 3.useDeferredValue
useDeferredValue는 값의 업데이트를 "지연"시키는 데 사용된다. 이는 useTransition과 유사한 목적으로 사용되며, 주로 사용자 입력과 같은 연속적인 업데이트에 유용하다.

```javascript
import { useState, useDeferredValue } from 'react';

function SearchComponent() {
  const [inputValue, setInputValue] = useState('');
  const deferredInputValue = useDeferredValue(inputValue, { timeoutMs: 200 });

  function handleInputChange(event) {
    setInputValue(event.target.value);
  }

  return (
    <div>
      <input value={inputValue} onChange={handleInputChange} />
      {/* deferredInputValue는 지연된 값을 사용하여 비동기 작업을 수행 */}
      <AsyncSearchResults query={deferredInputValue} />
    </div>
  );
}
```
* useDeferredValue: 입력 값의 업데이트를 지연시켜 주어, 사용자가 빠르게 타이핑할 때 모든 중간 값에 대해 검색 요청을 보내는 것을 방지한다.

### 4.useImperativeHandle
React에서 ref를 사용자 정의할 수 있게 해주는 Hook이다. 이 Hook은 주로 부모 컴포넌트가 자식 컴포넌트의 인스턴스에 대해 명령형 접근을 해야 할 때 사용된다. forwardRef와 함께 사용되며, 외부에서 접근 가능한 메서드나 속성을 정의할 수 있다.
이 Hook은 주로 제3자 라이브러리와의 통합이나, 특정 DOM 조작이 필요한 경우에 유용하다. useImperativeHandle을 사용하면 컴포넌트의 내부 구현을 숨기면서도 외부에서 필요한 기능을 노출할 수 있다.

```javascript
import React, { useRef, useImperativeHandle, forwardRef } from 'react';

const CustomInput = forwardRef((props, ref) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    },
    clear: () => {
      inputRef.current.value = '';
    }
  }));

  return <input ref={inputRef} {...props} />;
});


function ParentComponent() {
  const inputRef = useRef();

  const handleFocus = () => {
    inputRef.current.focus();
  };

  const handleClear = () => {
    inputRef.current.clear();
  };

  return (
    <div>
      <CustomInput ref={inputRef} />
      <button onClick={handleFocus}>Focus Input</button>
      <button onClick={handleClear}>Clear Input</button>
    </div>
  );
}

export default ParentComponent;
```


## Streaming SSR
리액트 18의 Streaming SSR은 서버에서 페이지를 렌더링할 때, 전체 페이지를 렌더링하는 것을 기다리지 않고 조각조각 나누어 스트림으로 전송하는 새로운 방식이다. 이를 통해 사용자는 전체 페이지가 로드될 때까지 기다릴 필요 없이 일부 컨텐츠를 먼저 볼 수 있게 되어 사용자 경험이 크게 개선된다.

### 주요 특징:
* 1.조각조각 나눠 전송: 전체 페이지를 한 번에 렌더링하는 대신, 조각조각 나눠서 렌더링하고 이를 클라이언트에 전송
* 2.응답성 향상: 사용자는 애플리케이션이 더 빠르게 로딩되는 것처럼 느낌
* 3.리소스 효율성: 필요한 데이터만 로드하여 렌더링을 시작하므로 서버 리소스가 효율적으로 사용됨

```javascript
//server code
const express = require('express');
const { renderToNodeStream } = require('react-dom/server');
const App = require('./App');

const app = express();

app.get('/', (req, res) => {
  res.setHeader('Content-Type', 'text/html');
  
  // React 컴포넌트를 스트림으로 렌더링
  const stream = renderToNodeStream(<App />);
  
  // 스트림 시작
  res.write('<!DOCTYPE html><html><head><title>Streaming SSR</title></head><body>');
  stream.pipe(res, { end: false });
  
  stream.on('end', () => {
    res.write('</body></html>');
    res.end();
  });
});

app.listen(3000);
```

```javascript
//client code
import React from 'react';
import ReactDOM from 'react-dom';

function HeaderComponent() {
  return <header>Header</header>;
}

function ContentComponent() {
  return <main>Content</main>;
}

function FooterComponent() {
  return <footer>Footer</footer>;
}

function App() {
  return (
    <div>
      <HeaderComponent />
      <ContentComponent />
      <FooterComponent />
    </div>
  );
}

ReactDOM.hydrate(<App />, document.getElementById('root'));
```
위의 예시에서, 서버는 App 컴포넌트를 스트림으로 렌더링하고, 이를 클라이언트에 직접 스트리밍한다. renderToNodeStream 함수는 React 컴포넌트를 Node 스트림으로 렌더링하는 데 사용된다.
클라이언트 코드의 각 부분(또는 조각)을 더 빨리 스트림 함으로써, 브라우저는 서버에서 보내온 HTML 조각을 더 빨리 렌더링하고 사용자에게 더 빨리 컨텐츠를 보여줄 수 있다.

## forwardRef
forwardRef는 React에서 부모 컴포넌트가 자식 컴포넌트의 DOM 노드나 클래스 인스턴스에 대한 참조를 전달할 수 있도록 하는 고차 컴포넌트이다.
기본적으로, React에서는 ref를 함수형 컴포넌트에 직접 전달할 수 없다. forwardRef를 사용하면 함수형 컴포넌트가 ref를 받아서 특정 DOM 요소나 클래스 인스턴스에 연결할 수 있다.

#### 이 기능은 주로 다음과 같은 경우에 유용
* DOM 조작: 부모 컴포넌트가 자식 컴포넌트의 특정 DOM 요소에 직접 접근해야 할 때.
* 제3자 라이브러리 통합: 외부 라이브러리와의 통합 시, 특정 DOM 요소에 대한 참조가 필요할 때.

```javascript
// 아래는 forwardRef를 사용하여 부모 컴포넌트가 자식 컴포넌트의 입력 필드에 직접 접근할 수 있도록 하는 예시

import React, { useRef, forwardRef } from 'react';

// 자식 컴포넌트
const CustomInput = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

// 부모 컴포넌트
function ParentComponent() {
  const inputRef = useRef();

  const handleFocus = () => {
    inputRef.current.focus();
  };

  return (
    <div>
      <CustomInput ref={inputRef} placeholder="Type here..." />
      <button onClick={handleFocus}>Focus Input</button>
    </div>
  );
}

export default ParentComponent;
```
