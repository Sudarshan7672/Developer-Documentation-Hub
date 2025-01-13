# Redux Toolkit
Redux is a library used for managing the state of an application through centralized state management.

## Key Concepts in Redux Toolkit

### 1. Store
A centralized store holds the entire state tree of your application. It is the single source of truth for your app.

### 2. Reducers
Reducers are functions that take the current state and an action as arguments, and return a new state. In other words:
```javascript
(state, action) => newState
```

### 3. Action
An action is a plain JavaScript object that has a `type` field. It describes what you want to do and may optionally include additional data (payload).

### 4. Slice
A slice is a collection of Redux reducer logic and actions for a single feature, organized together in one file.

---

# Getting Started

### 1. Set Up Your Vite Project
Run the following command to create a new Vite project:
```bash
npm create vite@latest my-redux-app
```
Select the desired framework and language when prompted.

Change to the project directory:
```bash
cd my-redux-app
```

Install dependencies:
```bash
npm install
```

### 2. Add Redux Toolkit and React-Redux
Install Redux Toolkit and React-Redux:
```bash
npm install @reduxjs/toolkit react-redux
```

### 3. Create the Store

#### Steps:
1. Inside the `src` folder, create an `app` directory.
2. Create a file named `store.js` in the `app` folder.

Here is the complete code for setting up the store:
```javascript
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from '../features/counter/counterSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});

// Infer the `RootState` and `AppDispatch` types from the store itself
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### 4. Create a Feature Slice

1. Inside the `src` folder, create a `features` directory.
2. Inside `features`, create a folder for the feature (e.g., `counter`).
3. Create a file named `counterSlice.js` inside the `counter` folder.

Here is an example of a counter slice:
```javascript
import { createSlice } from '@reduxjs/toolkit';

const initialState = {
  value: 0,
};

const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;
```

### 5. Integrate Store with React

1. Open `main.jsx` (or `index.js`) in the `src` folder.
2. Wrap your application with the `Provider` component from React-Redux.

```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import { Provider } from 'react-redux';
import { store } from './app/store';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
```

### 6. Using Redux in Components

1. Use `useSelector` to access state.
2. Use `useDispatch` to dispatch actions.

Here is an example component:
```javascript
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement, incrementByAmount } from './features/counter/counterSlice';

const Counter = () => {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    <div>
      <h1>Counter: {count}</h1>
      <button onClick={() => dispatch(increment())}>Increment</button>
      <button onClick={() => dispatch(decrement())}>Decrement</button>
      <button onClick={() => dispatch(incrementByAmount(5))}>Increment by 5</button>
    </div>
  );
};

export default Counter;
```

### 7. Test Your Application
Run the development server:
```bash
npm run dev
```

Navigate to the displayed URL and test the counter functionality.

---

By following this guide, you can set up and implement Redux Toolkit in your application quickly and efficiently.

