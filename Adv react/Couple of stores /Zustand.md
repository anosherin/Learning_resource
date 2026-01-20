A small, fast, and scalable bearbones state management solution. Zustand has a comfy API based on hooks.

Installation
Zustand is available as a package on NPM for use:

# NPM
npm install zustand
# Or, use any package manager of your choice.

First create a store
Your store is a hook! You can put anything in it: primitives, objects, functions. The set function merges state.
import { create } from 'zustand'

const useBear = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 }),
  updateBears: (newBears) => set({ bears: newBears }),
}))

Then bind your components, and that's it!
You can use the hook anywhere, without the need of providers. Select your state and the consuming component will re-render when that state changes.

function BearCounter() {
  const bears = useBear((state) => state.bears)
  return <h1>{bears} bears around here...</h1>
}

function Controls() {
  const increasePopulation = useBear((state) => state.increasePopulation)
  return <button onClick={increasePopulation}>one up</button>
}

Comparison
How Zustand stacks up against similar libraries

Zustand is one of many state management libraries for React. On this page we will discuss Zustand in comparison to some of these libraries, including Redux, Valtio, Jotai, and Recoil.

Each library has its own strengths and weaknesses, and we will compare key differences and similarities between each.

Redux
State Model (vs Redux)
Conceptually, Zustand and Redux are quite similar, both are based on an immutable state model. However, Redux requires your app to be wrapped in context providers; Zustand does not.

Zustand

import { create } from 'zustand'

type State = {
  count: number
}

type Actions = {
  increment: (qty: number) => void
  decrement: (qty: number) => void
}

const useCountStore = create<State & Actions>((set) => ({
  count: 0,
  increment: (qty: number) => set((state) => ({ count: state.count + qty })),
  decrement: (qty: number) => set((state) => ({ count: state.count - qty })),
}))

import { create } from 'zustand'

type State = {
  count: number
}

type Action = {
  type: 'increment' | 'decrement'
  qty: number
}

type Actions = {
  dispatch: (action: Action) => void
}

const countReducer = (state: State, action: Action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + action.qty }
    case 'decrement':
      return { count: state.count - action.qty }
    default:
      return state
  }
}

const useCountStore = create<State & Actions>((set) => ({
  count: 0,
  dispatch: (action: Action) => set((state) => countReducer(state, action)),
}))

Redux

import { createStore } from 'redux'
import { useSelector, useDispatch } from 'react-redux'

type State = {
  count: number
}

type Action = {
  type: 'increment' | 'decrement'
  qty: number
}

const countReducer = (state: State, action: Action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + action.qty }
    case 'decrement':
      return { count: state.count - action.qty }
    default:
      return state
  }
}

const countStore = createStore(countReducer)

import { createSlice, configureStore } from '@reduxjs/toolkit'

const countSlice = createSlice({
  name: 'count',
  initialState: { value: 0 },
  reducers: {
    incremented: (state, qty: number) => {
      // Redux Toolkit does not mutate the state, it uses the Immer library
      // behind scenes, allowing us to have something called "draft state".
      state.value += qty
    },
    decremented: (state, qty: number) => {
      state.value -= qty
    },
  },
})

const countStore = configureStore({ reducer: countSlice.reducer })

Render Optimization (vs Redux)
When it comes to render optimizations within your app, there are no major differences in approach between Zustand and Redux. In both libraries it is recommended that you manually apply render optimizations by using selectors.

Zustand

import { create } from 'zustand'

type State = {
  count: number
}

type Actions = {
  increment: (qty: number) => void
  decrement: (qty: number) => void
}

const useCountStore = create<State & Actions>((set) => ({
  count: 0,
  increment: (qty: number) => set((state) => ({ count: state.count + qty })),
  decrement: (qty: number) => set((state) => ({ count: state.count - qty })),
}))

const Component = () => {
  const count = useCountStore((state) => state.count)
  const increment = useCountStore((state) => state.increment)
  const decrement = useCountStore((state) => state.decrement)
  // ...
}

Redux

import { createStore } from 'redux'
import { useSelector, useDispatch } from 'react-redux'

type State = {
  count: number
}

type Action = {
  type: 'increment' | 'decrement'
  qty: number
}

const countReducer = (state: State, action: Action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + action.qty }
    case 'decrement':
      return { count: state.count - action.qty }
    default:
      return state
  }
}

const countStore = createStore(countReducer)

const Component = () => {
  const count = useSelector((state) => state.count)
  const dispatch = useDispatch()
  // ...
}

import { useSelector } from 'react-redux'
import type { TypedUseSelectorHook } from 'react-redux'
import { createSlice, configureStore } from '@reduxjs/toolkit'

const countSlice = createSlice({
  name: 'count',
  initialState: { value: 0 },
  reducers: {
    incremented: (state, qty: number) => {
      // Redux Toolkit does not mutate the state, it uses the Immer library
      // behind scenes, allowing us to have something called "draft state".
      state.value += qty
    },
    decremented: (state, qty: number) => {
      state.value -= qty
    },
  },
})

const countStore = configureStore({ reducer: countSlice.reducer })

const useAppSelector: TypedUseSelectorHook<typeof countStore.getState> =
  useSelector

const useAppDispatch: () => typeof countStore.dispatch = useDispatch

const Component = () => {
  const count = useAppSelector((state) => state.count.value)
  const dispatch = useAppDispatch()
  // ...
}

