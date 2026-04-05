<h1 align="center">Redux Notes</h1>

- [Installation:](#installation)
- [Introduction:](#introduction)
  - [What is Redux:](#what-is-redux)
  - [What is Redux Toolkit:](#what-is-redux-toolkit)
  - [Redux vs Redux Toolkit:](#redux-vs-redux-toolkit)
- [Different ways to manage state in react:](#different-ways-to-manage-state-in-react)
  - [Using Props drilling:](#using-props-drilling)
    - [When to use:](#when-to-use)
    - [Problem:](#problem)
  - [Using Context API:](#using-context-api)
    - [When to use:](#when-to-use-1)
    - [Problem:](#problem-1)
  - [Using Redux:](#using-redux)
    - [When to use:](#when-to-use-2)

# Installation:

```bash
npm install @reduxjs/toolkit react-redux
```

# Introduction: 

## What is Redux:
Redux is a JS library for predictable and maintainable global state management. Means redux is a central place (store) where all of our app’s state lives, and we can update it in a predictable way. 

## What is Redux Toolkit:
Redux Toolkit (RTK) is the official modern way to use Redux. It Built by the Redux team to fix all the problems by using raw redux.

## Redux vs Redux Toolkit:

| Feature        | Redux (Old) 😓 | Redux Toolkit 🚀 |
| -------------- | ------------- | --------------- |
| Boilerplate    | High          | Very low        |
| Learning curve | Hard          | Easier          |
| Setup          | Manual        | Automatic       |
| Best practice  | Optional      | Enforced        |
| Performance    | Good          | Better          |


# Different ways to manage state in react: 
## Using Props drilling: 

```js
// src/main.jsx

import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client';
import './index.css'
import { createBrowserRouter, RouterProvider } from 'react-router';
import App from './App';


const router = createBrowserRouter([
  {
    path: '/',
    Component: App,
  },
])

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <RouterProvider router={router}></RouterProvider>
  </StrictMode>,
)
```

```js
// src/App.jsx

import React, { useState } from "react";
import Parent from "./Parent";

export default function App() {
  const [likes, setLikes] = useState(0);

  return (
    <Parent likes={likes} setLikes={setLikes} />
  );
}
```

```js
// src/Parent.jsx

import Child from "./Child";

export default function Parent({ likes, setLikes }) {
    return (
        <Child likes={likes} setLikes={setLikes} />
    );
}
```

```js
// src/Child.jsx

export default function Child({ likes, setLikes }) {
    return (
        <div>
            <h2>Likes: {likes}</h2>
            <button className="btn" onClick={() => setLikes(likes + 1)}>Like</button>
            <button className="btn" onClick={() => setLikes(likes - 1)}>Dislike</button>
            <button className="btn" onClick={() => setLikes(0)}>Reset</button>
        </div>
    );
}
```

### When to use:
- Small apps
- if state is LOCAL and mostly used by 1–3 levels deep only
### Problem: 
- Becomes messy when deeply nested
- we also need to drilling the props for all components to get the child even intermediate components don’t need it

## Using Context API: 


```js
// src/Context.jsx

import { createContext } from "react";

export const LikeContext = createContext();
```

```js
// src/LikeContextProvider.jsx

import { useState } from "react";
import { LikeContext } from "./Context";


export default function LikeContextProvider({ children }) {
    const [likes, setLikes] = useState(0);

    return (
        <LikeContext.Provider value={{ likes, setLikes }}>
            {children}
        </LikeContext.Provider>
    );
}
```

```js
// src/main.jsx

import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client';
import './index.css'
import { createBrowserRouter, RouterProvider } from 'react-router';
import App from './App';
import LikeContextProvider from './LikeContextProvider';


const router = createBrowserRouter([
  {
    path: '/',
    Component: App,
  },
])

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <LikeContextProvider>
      <RouterProvider router={router}></RouterProvider>
    </LikeContextProvider>
  </StrictMode>,
)
```

```js
// src/App.jsx

import React from "react";
import Parent from "./Parent";

export default function App() {

  return (
    <Parent />
  );
}
```

```js
// src/Parent.jsx

import Child from "./Child";

export default function Parent() {
    return (
        <Child />
    );
}
```

```js
// src/Child.jsx

import { useContext } from "react";
import { LikeContext } from "./Context";

export default function Child() {
    const { likes, setLikes } = useContext(LikeContext)
    return (
        <div>
            <h2>Likes: {likes}</h2>
            <button className="btn" onClick={() => setLikes(likes + 1)}>Like</button>
            <button className="btn" onClick={() => setLikes(likes - 1)}>Dislike</button>
            <button className="btn" onClick={() => setLikes(0)}>Reset</button>
        </div>
    );
}
```

### When to use:
- Want to Avoid props drilling
- Manage Simple Global state (theme, auth, etc.)

### Problem:
- Can cause unnecessary re-renders
- Not great for complex logic

## Using Redux:

```js
// src/stores.js

import { configureStore } from "@reduxjs/toolkit";
import likeReducer from "./likeSlice";

export const store = configureStore({
    reducer: {
        likes: likeReducer,
    },
});
```

```js
// src/likeSlice.js

import { createSlice } from "@reduxjs/toolkit";

const likeSlice = createSlice({
    name: "likes",
    initialState: { value: 0 },
    reducers: {
        increment: (state) => {
            state.value += 1;
        },
        decrement: (state) => {
            state.value -= 1;
        },
        reset: (state) => {
            state.value = 0;
        },
    },
});

export const { increment, decrement, reset } = likeSlice.actions;
export default likeSlice.reducer;
```

```js
// src/main.jsx

import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client';
import './index.css'
import { createBrowserRouter, RouterProvider } from 'react-router';
import App from './App';
import { store } from './store';
import { Provider } from 'react-redux';


const router = createBrowserRouter([
  {
    path: '/',
    Component: App,
  },
])

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <Provider store={store}>
      <RouterProvider router={router}></RouterProvider>
    </Provider>
  </StrictMode>,
)
```

```js
// src/App.jsx

import React from "react";
import Parent from "./Parent";

export default function App() {

  return (
    <Parent />
  );
}
```

```js
// src/Parent.jsx

import Child from "./Child";

export default function Parent() {
    return (
        <Child />
    );
}
```

```js
// src/Child.jsx

import { useDispatch, useSelector } from "react-redux";
import { decrement, increment, reset } from "./likeSlice";

export default function Child() {
    const likes = useSelector((state) => state.likes.value);
    const dispatch = useDispatch();
    return (
        <div>
            <h2>Likes: {likes}</h2>
            <button className="btn" onClick={() => dispatch(increment())}>Like</button>
            <button className="btn" onClick={() => dispatch(decrement())}>Dislike</button>
            <button className="btn" onClick={() => dispatch(reset())}>Reset</button>
        </div>
    );
}
```

### When to use:
- Want to Avoid props drilling
- Manage Complex Global state that needs to implement on multiple features