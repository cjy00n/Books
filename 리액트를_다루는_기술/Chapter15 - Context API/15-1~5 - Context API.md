# Context API란?

## Context

- 먼저, 리액트의 context는 React 컴포넌트 트리 안에서 전역적이라고 볼 수 있는 데이터를 공유할 수 있도록 고안된 방법이다.
- 꼭 전역적일 필요는 없으며, 단순히 "리액트 컴포넌트에서 props가 아닌 또 다른 방식으로 컴포넌트 간에 값을 전달하는 방법"이라고 접근하는 것이 좋다.

## Context API

- Context API는 리액트 앱에서 전역적으로 사용할 데이터가 있을 때 사용하는 기능으로, 리덕스, 리액트 라우터, styled-components 등의 라이브러리는 Context API를 기반으로 구현되어 있다.
- 리액트 버전 16부터 사용 가능하다.

# Context API를 사용한 전역 상태 관리 흐름

- 기본적으로, 리액트는 데이터를 부모에서 자식으로 전달하는 단방향 데이터 흐름을 지향한다.
- 하지만, 사용자 로그인 정보, 환경 설정, 테마 등 종종 공유데는 데이터 관리가 필요할 때가 있다. => Context 사용
- state만 사용해서 데이터를 전달할 경우 많은 컴포넌트를 거쳐 데이터를 전달받게 된다. => props drilling
- props drilling 이 되면 유지 보수성이 낮아지고 비효율적인 코드가 된다.
- 단 한번에 원하는 데이터를 전달받기 위해서 리덕스, MobX 같은 상태 관리 라이브러리를 사용하거나, 리액트에 내장된 Context API를 사용하여 상태 관리를 할 수 있다.  
  ![](https://blog.kakaocdn.net/dn/bHKc0i/btrpkwggvMy/HN8THmEmfXwzWw3eGqcDdK/img.png)

# Context API 사용하기

## createContext

- 새 Context를 만들 때는 createContext 함수를 사용한다.
- 파라미터에는 해당 Context의 기본 상태를 지정한다.

```
// src/contexts/color.js

import { creactContext } from "react";

const ColorContext = creactContext({ color: "black" }); // 기본값 black

export default ColorContext;
```

## Consumer

- ColorBox라는 컴포넌트를 만들어서 ColorContext 안에 들어있는 색상값을 불러온다.
- 이때 상태를 props로 받아오는 것이 아니라 ColorContext 안에 들어있는 Consumer라는 컴포넌트를 통해 색상을 조회할 수 있다.

```ts
// components/ColorBox.tsx
import ColorContext from "../contexts/color";

const ColorBox = () => {
  return (
    <ColorContext.Consumer>
      {(value) => (
        <>
          <div
            style={{ background: value.color, width: "200px", height: "200px" }}
          ></div>
          <span>{`color is ${value.color}`}</span>
        </>
      )}
    </ColorContext.Consumer>
  );
};

export default ColorBox;
```

```
//App.tsx
import ColorBox from "./components/ColorBox";

function App() {
  return (
    <div>
      <ColorBox />
    </div>
  );
}

export default App;
```

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbfdmZl%2FbtsBKgGNN3h%2Fql6BEenXNEGsuxlmUDyOg1%2Fimg.png" width="500">

## Provider

- provider를 사용하면 Context의 value를 변경할 수 있다.

```
//App.tsx

function App() {
return (
<ColorContext.Provider value={{ color: "red" }}>

</ColorContext.Provider>
);
}

export default App;
```

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbRtRl0%2FbtsBVBozz54%2FyFdDpUvcwAGkaPucGPmt5k%2Fimg.png" width="500">

# 동적 Context 사용하기

- Context의 value에는 상태 값뿐 아니라 함수를 전달할 수도 있다.
- Provider의 value에는 상태로 state, 업데이트 함수는 actions로 묶어서 전달한다.
- createContext의 기본 값은 실제 Provider의 value에 넣는 객체의 형태와 일치시켜 주는 것이 좋다.

## Context 수정하기

```ts
// src/contexts/color.tsx

const ColorContext = createContext({
  state: { color: "black", subcolor: "red" },
  actions: {
    setColor: (color: string) => {},
    setSubcolor: (subcolor: string) => {},
  },
});

const ColorProvider: React.FC<{ children: React.ReactNode }> = ({
  children,
}) => {
  const [color, setColor] = useState("black");
  const [subcolor, setSubcolor] = useState("red");

  const value = {
    state: { color, subcolor },
    actions: { setColor, setSubcolor },
  };
  return (
    <ColorContext.Provider value={value}>{children}</ColorContext.Provider>
  );
};

const { Consumer: ColorConsumer } = ColorContext;
export { ColorProvider, ColorConsumer };

export default ColorContext;
```

- 기존 ColorContext.Provider를 ColorProvider로 대체한다.

```ts
// src/App.js
function App() {
  return (
    <ColorProvider>
      <div>
        <ColorBox />
      </div>
    </ColorProvider>
  );
}

export default App;
```

- 바뀐 value 형식에 맞춰 ColorBox도 수정한다.

```ts
// src/components/ColorBox.tsx
const ColorBox = () => {
  return (
    <ColorContext.Consumer>
      {({ state }) => (
        <>
          <div
            style={{ background: state.color, width: "200px", height: "200px" }}
          ></div>
          <div
            style={{
              background: state.subcolor,
              width: "100px",
              height: "100px",
            }}
          ></div>
        </>
      )}
    </ColorContext.Consumer>
  );
};

export default ColorBox;
```

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbnWY2J%2FbtsBXNpzh2F%2FaEOHf5gwHP6Aqrf7YZlfBK%2Fimg.png" width="500">

## 색상 선택 컴포넌트 만들기

- Context의 actions에 넣어준 함수를 호출하는 컴포넌트를 만들고, App.tsx에서 렌더링한다.

```ts
// src/components/SelectColors

import { ColorConsumer } from "../contexts/color";

const colors = [
  "red",
  "orange",
  "yellow",
  "green",
  "blue",
  "navy",
  "purple",
  "black",
];

const SelectColors = () => {
  return (
    <div>
      <h2>색상을 선택하세요.</h2>
      <ColorConsumer>
        {({ actions }) => (
          <div style={{ display: "flex" }}>
            {colors.map((color) => (
              <div
                key={color}
                style={{
                  background: color,
                  width: "50px",
                  height: "50px",
                  cursor: "poiner",
                }}
                onClick={() => actions.setColor(color)}
                onContextMenu={(e) => {
                  e.preventDefault();
                  actions.setSubcolor(color);
                }}
              ></div>
            ))}
          </div>
        )}
      </ColorConsumer>

      <hr />
    </div>
  );
};

export default SelectColors;
```

- 각 색상 div를 마우스로 클릭하면 큰 div의 배경 색이 해당 색으로 바뀌게되고, 오른쪽 마우스로 클릭하면 작은 div의 배경 색이 해당 색으로 바뀐다.
- 마우스 오른쪽 버튼 클릭 이벤트는 onContextMenu를 사용한다.(원래 브라우저 메뉴가 나타나는데, 이를 숨기기 위해 e.preventDefault()를 호출한다.)

<img src="https://blog.kakaocdn.net/dn/Olumu/btsBXFFpHai/4XTs6T8EZUgoREsfURr5N0/img.gif" >

# useContext Hook 사용하기

- 리액트에 내장되어 있는 Hooks 중에서 useContext라는 Hook을 사용하면, 함수 컴포넌트에서 Context를 아주 편하게 사용할 수 있다.

```ts
// src/components/ColorBox.tsx

import { useContext } from "react";
import ColorContext from "../contexts/color";

const ColorBox = () => {
  const { state } = useContext(ColorContext);

  return (
    <>
      <div
        style={{ background: state.color, width: "200px", height: "200px" }}
      ></div>
      <div
        style={{
          background: state.subcolor,
          width: "100px",
          height: "100px",
        }}
      ></div>
    </>
  );
};

export default ColorBox;
```

## 정리

- 프로젝트의 컴포넌트 구조가 간단하고 다루는 상태의 종류가 많지 않다면, 굳이 Context를 사용할 필요는 없다.
- 하지만 전역적으로 여기저기서 사용되는 상태가 있고 컴포넌트의 개수가 많은 상황이라면, Context API를 사용하는 것을 권장한다.
