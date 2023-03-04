# React State Management

A quick reference guide/cheatsheet for state management in React with or without state management libraries.

> The goal here is to go through the steps of creating the application with different libraries quickly. If you want to dive deep into concepts, please refer to the docs in each section. Also, if you have a simple application, you don't need to do all the optimizations suggested here. You will only notice the advantages when your app grows bigger.
## Contents
### [The Application](#the-application)  
### [State management without libraries](#state-management-without-libraries)  
  - [1. useState hook](#1-usestate-hook)   
    - [1.1 Declare local values with useState](#11-declare-local-values-with-usestate)    
    - [1.2 Memoize derived values with `useMemo`](#12-memoize-derived-values-with-usememo)    
    - [1.3 Memoize functions with `useCallback`](#13-memoize-functions-with-usecallback)    
    - [1.4 Pass values to components as props](#14-pass-values-to-components-as-props)    
    - [1.5 Use React.memo to memoize components](#15-use-reactmemo-to-memoize-components)    
    - [1.6 CodeSandbox link for useState](#16-codesandbox-link-for-usestate)    
  - [2. useReducer hook](#2-usereducer-hook)   
    - [2.1 Declare use reducer](#21-declare-use-reducer)    
    - [2.2 define initial state](#22-define-initial-state)    
    - [2.3 actions and reducer](#23-actions-and-reducer)    
    - [2.4 convert event hadlers to use actions](#24-convert-event-hadlers-to-use-actions)    
    - [2.5 CodeSandbox link for useReducer](#25-codesandbox-link-for-usereducer)    
  - [3. useContext hook](#3-usecontext-hook)    
    - [3.1 Declare context](#31-declare-context)    
    - [3.2 Declare provider](#32-declare-provider)    
    - [3.3 Wrap Parent component around provider](#33-wrap-parent-component-around-provider)    
    - [3.4 Usage of useContext in individual components](#34-usage-of-usecontext-in-individual-components)    
    - [3.5 CodeSandbox link for useContext](#35-codesandbox-link-for-usecontext)   
 
### [State management with libraries](#state-management-with-libraries)    
  - [1. Redux (Redux-toolkit)](#1-redux-redux-toolkit)    
    - [1.0 Install redux and redux-toolkit](#10-install-redux-and-redux-toolkit)    
    - [1.1 creating a slice](#11-creating-a-slice)    
    - [1.2 create main store and linking the slice](#12-create-main-store-and-linking-the-slice)    
    - [1.3 wrap provier](#13-wrap-provier)    
    - [1.4 create selectors to get only related data](#14-create-selectors-to-get-only--related-data)    
    - [1.5 Use inside a component](#15-use-inside-a-component)    
    - [1.6 CodeSandbox link for Redux](#16-codesandbox-link-for-redux)    
  - [2 Recoil](#2-recoil)    
    - [2.0 Installation](#20-installation)    
    - [2.1 Declare store and reducers](#21-declare-store-and-reducers)    
    - [2.2 Wrap parent component with Recoil Root](#22-wrap-parent-component-with-recoil-root)    
    - [2.3 Use inside a component](#23-use-inside-a-component)    
    - [2.4 CodeSandbox link for Recoil](#24-codesandbox-link-for-recoil)    
  - [3 Zustand](#3-zustand)    
    - [3.0 Installation](#30-installation)    
    - [3.1 Declare store and selectors](#31-declare-store-and-selectors)    
    - [3.2 Use inside a component](#32-use-inside-a-component)    
    - [3.3 Codesandbox link for Zustand](#33-codesandbox-link-for-zustand)    

## The Application
![application](https://res.cloudinary.com/ds574fco0/image/upload/v1677931743/github/state-management_fto8b4.png)
A simple application to demonstrate common features among all state management libraries. Here is what our application will do:

1. The input field should update while the user types in.
2. The error message should be hidden if the user has not typed anything in the input field. After the user starts typing, if the input is empty, the error message should be shown.
3. The user can increase or decrease the value by clicking on the `+` or `-` buttons respectively.
4. The sign box will show the sign of the counter value. It will be empty if the counter is zero.   

Also, we will try to do the following optimizations whenever possible:

1. When the user types in the input field, none of the counter components should re-render.
2. The error message should only re-render when the empty status of the input field changes. It should not re-render every time the user keys in a key.
3. When the counter value changes, the input or the error message should not re-render.
4. The sign will only change when the counter value transitions from one sign to another. So it should not change every time the user clicks on the `+` or `-` button.  


## State management without libraries
### 1. useState hook
#### 1.1 Declare local values with useState
```tsx
const [inputValue, setInputValue] = useState("");
const [counterValue, setCounterValue] = useState(0);
const [isInputTouched, setIsInputTouched] = useState(false);
```
#### 1.2 Memoize derived values with `useMemo`
We can also derive the `errorMessage` and `sign` value from the above state values. We will use `useMemo` here so that the derived value will only be calculated if a related component was changed.
```tsx
const sign = useMemo(() => {
    return getSign(counterValue);
  }, [counterValue]);

const errorMessage = useMemo(() => {
if (isInputTouched && !inputValue) {
    return "Text can't be empty";
}
return "";
}, [isInputTouched, inputValue]);
```
#### 1.3 Memoize functions with `useCallback`
```tsx
const onInputChange = useCallback(
    (e: React.ChangeEvent<HTMLInputElement>) => {
      setInputValue(e.target.value);
      setIsInputTouched(true);
    },
    []
  );

const onCounterChange = useCallback((diff: number) => {
setCounterValue((current) => current + diff);
}, []);
``` 
#### 1.4 Pass values to components as props
```tsx
<Input onChange={onInputChange} value={inputValue} />
<ErrorMessage text={errorMessage} />
<div >
    <CounterSign text={sign} />
    <Counter value={counterValue} onChange={onCounterChange} />
</div>
```
#### 1.5 Use `React.memo` to memoize components
We can use React.memo to make sure our componets only render when their props change.  
```tsx
import { memo } from "react";
type Props = {
  text: string;
};

function ErrorMessage({ text }: Props) {
  return <div >{text}</div>;
}
export default memo(ErrorMessage);
``` 
#### 1.6 [CodeSandbox link for useState](https://codesandbox.io/p/sandbox/usestate-react-state-management-keyg2m)

  
### 2. useReducer hook   
   
#### 2.1 Declare use reducer 
```tsx
const [{ inputValue, counterValue, isInputTouched }, dispatch] = useReducer(
    reducer,
    initialState
);
```   
   
#### 2.2 define initial state
```tsx
const initialState: State = {
  inputValue: "",
  counterValue: 0,
  isInputTouched: false,
};
```    
   
#### 2.3 actions and reducer
```tsx
function reducer(state: State, action: Action) {
  switch (action.type) {
    case INPUT_CHANGE: {
      return {
        ...state,
        inputValue: action.value,
        isInputTouched: true,
      };
    }
    case COUNTER_CHANGE: {
      return {
        ...state,
        counterValue: state.counterValue + action.diff,
      };
    }
  }
  throw Error("Unknown action");
}
```    
   
#### 2.4 convert event hadlers to use actions
```tsx
const onInputChange = useCallback(
    (e: React.ChangeEvent<HTMLInputElement>) => {
      dispatch({ type: INPUT_CHANGE, value: e.target.value });
    },
    []
);
```
No change in how to pass the values to components or using `React.memo` to avoid rerenders.   
    
#### 2.5 [CodeSandbox link for useReducer](https://codesandbox.io/p/sandbox/usereducer-react-state-management-ur3i2l)  
    
### 3. useContext hook
You can  use useContext so that you can avoid props drilling
#### 3.1 Declare context
```tsx
const MainContext = createContext<State & Funcs & Derived>({
  ...initialState,
  ...initialFuncs,
  ...initialDerived,
});
```    
    
#### 3.2 Declare provider
```tsx
export function MainContextProvider({ children }: MainContextProviderProps) {
  const [{ inputValue, counterValue, isInputTouched }, dispatch] = useReducer(
    reducer,
    initialState
  );

 /*
        .... test of the code from original main component
 */

  return (
    <MainContext.Provider
      value={{
        inputValue,
        counterValue,
        isInputTouched,
        onInputChange,
        onCounterChange,
        sign,
        errorMessage,
      }}
    >
      {children}
    </MainContext.Provider>
  );
} 
```     
     
#### 3.3 Wrap Parent component around provider
```tsx
import { MainContextProvider } from "components/MainContext";

export default function Main() {
  return (
    <Dialog storeType="useState">
      <MainContextProvider>
       //...
      </MainContextProvider>
    </Dialog>
  );
}
```     
    
#### 3.4 Usage of `useContext` in individual components
```tsx
import MainContext from "components/MainContext";
import { useContext } from "react";

export default function Input() {
  const { inputValue, onInputChange } = useContext(MainContext);
 //...
}
```
> We can't use react.memo to prevent re-rendering when using context because react.memo only works on props.     
#### 3.5 [CodeSandbox link for useContext](https://codesandbox.io/p/sandbox/usecontext-react-state-management-78zdvf)

## State management with libraries
### 1. Redux (Redux-toolkit)
[Docs](https://redux-toolkit.js.org)     
Redux library [ofically suggests](https://redux.js.org/introduction/getting-started#redux-toolkit) using Redux toolkit so we will use it here.
#### 1.0 Install redux and redux-toolkit
```
npm install react-redux @reduxjs/toolkit 
```
#### 1.1 creating a slice
```tsx
import { createSelector, createSlice } from "@reduxjs/toolkit";
import type { PayloadAction } from "@reduxjs/toolkit";
const initialState: MainState = {
  inputValue: "",
  counterValue: 0,
  isInputTouched: false,
};

export const mainSlice = createSlice({
  name: "main",
  initialState,
  reducers: {
    onInputChange: (state, action: PayloadAction<string>) => {
      state.inputValue = action.payload;
      state.isInputTouched = true;
    },
    onCounterChange: (state, action: PayloadAction<number>) => {
      state.counterValue = state.counterValue + action.payload;
    },
  },
});
export const { onInputChange, onCounterChange } = mainSlice.actions;

export default mainSlice.reducer;
```     
     
#### 1.2 create main store and linking the slice
```tsx
import { configureStore } from "@reduxjs/toolkit";
import { TypedUseSelectorHook, useDispatch, useSelector } from "react-redux";
import main from "reduxStore/reducers/main";

const store = configureStore({
  reducer: {
    main,
  },
});

// Infer the `RootState` and `AppDispatch` types from the store itself
export type RootState = ReturnType<typeof store.getState>;
// Inferred type: {posts: PostsState, comments: CommentsState, users: UsersState}
export type AppDispatch = typeof store.dispatch;

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;

export default store;
```
#### 1.3 wrap provier
```tsx
import { Provider } from "react-redux";

export default function Main() {
  return (
    <Dialog storeType="useState">
      <Provider store={store}>
       //...
      </Provider>
    </Dialog>
  );
}
 
```    
     
#### 1.4 create selectors to get only  related data
```tsx
export const selectInputValue = (state: { main: MainState }) =>
  state.main.inputValue;
export const selectIsInputTouched = (state: { main: MainState }) =>
  state.main.isInputTouched;
export const selectCounterValue = (state: { main: MainState }) =>
  state.main.counterValue;

export const selectErrorMessage = createSelector(
  selectInputValue,
  selectIsInputTouched,
  (value, isTouched) => (isTouched && !value ? "Text field can't be empty" : "")
);

export const selectAbsValue = createSelector(selectCounterValue, (value) =>
  Math.abs(value)
);
export const selectSign = createSelector(selectCounterValue, (value) => {
  if (value > 0) {
    return "+";
  }
  if (value < 0) {
    return "-";
  }
  return "";
});
```

#### 1.5 Use inside a component
    
```tsx
import { useCallback} from "react";
import { useAppDispatch, useAppSelector } from "reduxStore";
import { onCounterChange, selectAbsValue } from "reduxStore/reducers/main";
export default function Counter() {
  const counterValue = useAppSelector(selectAbsValue);
  const dispatch = useAppDispatch();

  const onIncrement = useCallback(() => {
    dispatch(onCounterChange(1));
  }, [dispatch]);

  const onDecrement = useCallback(() => {
    dispatch(onCounterChange(-1));
  }, [dispatch]);

  console.log("Counter rendered");
  return (
    <>
      <div >
        <div > Value </div>
        <div >
          {Math.abs(counterValue)}
        </div>
      </div>
      <Button text="+" onClick={onIncrement} />
      <Button text="-" onClick={onDecrement} />
    </>
  );
}
```   
#### 1.6 [CodeSandbox link for Redux](https://codesandbox.io/p/sandbox/redux-react-state-management-7h9xiy)
### 2 Recoil
[Docs](https://recoiljs.org)
#### 2.0 Installation
```
npm install recoil
```   
#### 2.1 Declare store and reducers
```tsx
import { atom, selector } from "recoil";

export const inputState = atom({
  key: "inputState",
  default: { value: "", isTouched: false },
});

export const counterState = atom({
  key: "counterState",
  default: 0,
});

export const errorMessageState = selector({
  key: "errorMessageState",
  get: ({ get }) => {
    const { value, isTouched } = get(inputState);

    return isTouched && !value ? "Text can't be empty" : "";
  },
});

export const signState = selector({
  key: "signState",
  get: ({ get }) => {
    const value = get(counterState);

    if (value > 0) {
      return "+";
    }
    if (value < 0) {
      return "-";
    }
    return "";
  },
});
```   
   
#### 2.2 Wrap parent component with `Recoil Root`
```tsx
import { RecoilRoot } from "recoil";

export default function Main() {
  return (
    <Dialog storeType="useState">
      <RecoilRoot>
        <Input />
        <ErrorMessage />
        <div className="flex items-end space-x-2">
          <CounterSign />
          <Counter />
        </div>
      </RecoilRoot>
    </Dialog>
  );
}
```

#### 2.3 Use inside a component  

```tsx
import { useRecoilState } from "recoil";
import { inputState } from "recoilStore";

export default function Input() {
  const [input, setInput] = useRecoilState(inputState);

  const onChange = useCallback(
    (e: React.ChangeEvent<HTMLInputElement>) => {
      setInput({
        value: e.target.value,
        isTouched: true,
      });
    },
    [setInput]
  );
  //...
}
```   
   
```tsx
import { useRecoilValue } from "recoil";
import { errorMessageState } from "recoilStore";

export default function ErrorMessage() {
  const errorMessage = useRecoilValue(errorMessageState);
  return <div>{errorMessage}</div>;
}
```

#### 2.4 [CodeSandbox link for Recoil](https://codesandbox.io/p/sandbox/recoil-react-state-management-162e8m)

### 3 Zustand
[Docs](https://docs.pmnd.rs/zustand/getting-started/introduction)
#### 3.0 Installation
```
npm install zustand
```
#### 3.1 Declare store and selectors
```tsx
import { create } from "zustand";

interface InputState {
  inputState: {
    inputValue: string;
    isInputTouched: boolean;
  };
  onInputChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
}

export const useInputState = create<InputState>((set) => ({
  inputState: { inputValue: "", isInputTouched: false },
  onInputChange: (e) =>
    set(() => ({
      inputState: { inputValue: e.target.value, isInputTouched: true },
    })),
}));

interface CounterState {
  value: number;
  onIncrement: () => void;
  onDecrement: () => void;
}

export const useCounterState = create<CounterState>((set) => ({
  value: 0,
  onIncrement: () => set((state) => ({ value: state.value + 1 })),
  onDecrement: () => set((state) => ({ value: state.value - 1 })),
}));

export const getErrorMessage = (state: InputState) => {
  const { inputValue, isInputTouched } = state.inputState;
  return isInputTouched && !inputValue ? "Text can't be empty" : "";
};

export const getSign = (state: CounterState) => {
  const { value } = state;
  if (value > 0) {
    return "+";
  }
  if (value < 0) {
    return "-";
  }
  return "";
};

```
#### 3.2 Use inside a component
```tsx
import { useCounterState } from "zustandStore";

export default function Counter() {
  const counterValue = useCounterState((state) => state.value);
  const onIncrement = useCounterState((state) => state.onIncrement);
  const onDecrement = useCounterState((state) => state.onDecrement);

  console.log("Counter rendered");
  return (
    <>
      <div >
        <div >
          Value
        </div>
        <div >
          {Math.abs(counterValue)}
        </div>
      </div>
      <Button text="+" onClick={onIncrement} />
      <Button text="-" onClick={onDecrement} />
    </>
  );
}
```  
```tsx
import { getSign, useCounterState } from "zustandStore";

export default function CounterSign() {
  const sign = useCounterState(getSign);
  console.log("Counter sign rendered");
  return (
    <div >
      <h4 >
        Sign
      </h4>
      <div >
        {sign}
      </div>
    </div>
  );
}
```  
#### 3.3 [Codesandbox link for Zustand](https://codesandbox.io/p/sandbox/zustand-react-state-management-kl9j9g)

