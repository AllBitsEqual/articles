<!--
	title: "SERIES: React Native (Step by Step) - React Redux with Typescript"
	description: "Setting up React Redux (ducks) with Typescript in Expo "
	author: "Konrad Abe (AllBitsEqual)"
	published_at: 2021-01-11 08:00:00
	header_image: "XXX"
	categories: "react-native typescript redux reducks series"
	canonical_url: "XXX"
    series: "React Native (Step by Step)"
	language: en
-->
# SERIES: React Native (Step by Step) - React Redux with Typescript

**In our last session, we've set up a new and clean React Native Project with Typescript and Linting Support. Today we will add React Redux to the mix, a predictable state container that will allow us to manage global state in one centralised place.**

_As always, you will find the finished code linked on GitHub at the end of the article._

![](https://i.imgur.com/egaWeWg.jpg)


## Redux: WHY
To get started, I’m assuming that you have basic knowledge of React Native and Redux. [If you need a small refresher on Redux, I've got just the article for you!](https://medium.com/@allbitsequal/straightforward-redux-no-strings-attached-e1b5f111bf00) As we continue with the setup from our last session, we are still working with Expo to manage our React Native Build and bundling.


Now, I know that there’s currently a great hype to remove Redux and other dependencies from all projects but I’m still a firm believer in the ideas behind Redux and especially with the newest version of React Navigation (5.x at the time of writing), they improved the integration of Redux in the Navigator and we’ll make good use of that in later sessions when we add React Navigation into the mix.


### Redux Toolkit, WHY
In the past, I had decided to not work with Redux Toolkit. I used to prefer the hands-on way to do it as Redux is not scary and definitely not complicated. I couldn't argue against the boilerplate argument there but then again it's not that bad and you do not spend THAT much time writing redux code in my experience.

That was "past me". "Present-day me" gave the toolset another go, made some prototypes with it, used it on a larger project and is now chuckling when thinking of "past me" vehemently NOT wanting to use it. You still write the same logic, you still write the same code more or less but you do write substantially of it and the files are more compact and way easier to read.


## Getting started

Install the packages from NPM for your production dependencies and additionally the react-redux types (which are missing from the react-redux package) for development.

```sh
# prod packages
npm install redux react-redux @reduxjs/toolkit
# dev package
npm install -D @types/react-redux
```

The first thing we have to do is creating a store to house all our precious data. Create a new index.ts under src/redux/ and let's start coding.

We need to configure a store with the handy function of the toolkit, add our rootReducer (where we gather all out reducers in one centralised place) and export the store we created.

```js=
import { configureStore } from '@reduxjs/toolkit'
import rootReducer from './rootReducer'

export type RootState = ReturnType<typeof rootReducer>

const store = configureStore({
    reducer: rootReducer,
})

export default store
```

Before we move on to write the missing rootReducer, let's first recreate THE tutorial reducer everyone is using... but with less code, thanks to Redux Toolkit. I'm throwing this under src/redux/demo because I will delete it later but for now, let's make a counter.ts file.

Thanks to the toolkit, we can let them do the heavy lifting of creating all the code for actions, reducers, types and such. The redux-toolkit comes with multiple ways to go about this (including `createAction` and `createReducer`) but by far the easiest way to do this is using `createSlice`, which will be enough for most regular cases excluding maybe async stuff.

```js=
import { createSlice, PayloadAction } from '@reduxjs/toolkit'

const counterSlice = createSlice({
    name: 'counter',
    initialState: 0,
    reducers: {
        increment: (state, action: PayloadAction<number>) => state + action.payload,
        decrement: (state, action: PayloadAction<number>) => state - action.payload,
    },
})

export const { increment, decrement } = counterSlice.actions
export default counterSlice.reducer
```

What we have done here is giving a name to our reducer (counter), defining its initial state and writing our two needed reducers to increase and decrease the state of our counter by the payload we send with the dispatch.

The createSlice function returns an object including (but not limited to) the actions and reducer we need to export to use them throughout our app. With this in place, we can now finally add the missing rootReducer where we register all our reducers.

```js=
import { combineReducers } from '@reduxjs/toolkit'
import counterReducer from './demo/counter'

const rootReducer = combineReducers({
    counter: counterReducer,
})

export default rootReducer
```

Not unlike regular redux, we bunch up all our reducers into our state by using a combineReducers function and handing it an object with all our reducers and their desired names.

With this in place, we have a functional but useless redux store. Everything is typed so far because most types can be inferred by their usage and both redux and the toolkit come with all functions properly typed. Congratulations.

To add some functionality to our app, we have to create a component to consume the stored state and dispatch actions to change the state. In the past we would have wrapped our new component in the connect() HOC to gain access to the store and being able to dispatch actions but thanks to React Hooks, we can now simply rely on useSelector() and useDispatch() to interact with redux via react-redux.

Create a new component file Counter.tsx under src/components/demo (again, we will get rid of this demo code at a later step in our series) and get the basic stuff in

```js=
import { Button, Text } from 'react-native'
import React from 'react'

const Counter = (): React.ReactElement => {
    return (
        <>
            <Text>0</Text>
            <Button title="increment" onPress={() => {}}>
                +1
            </Button>
            <Button title="decrement" onPress={() => {}}>
                -1
            </Button>
        </>
    )
}

export default Counter
```

To use the new hooks with typescript, we need to touch out redux store again so let's move back to the src/redux/index.ts for a moment.

We will create an alias for both useDispatch and useSelector, add typing to them and then export those aliases so that we do not need to do this every time we want to use them. I chose to name them useReduxDispatch and useReduxSelector respectively to make them distinguishable but you can use any name you want.

```js=
import { configureStore } from '@reduxjs/toolkit'
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux'
import rootReducer from './rootReducer'

export type RootState = ReturnType<typeof rootReducer>

const store = configureStore({
    reducer: rootReducer,
})

export type AppDispatch = typeof store.dispatch
export const useReduxDispatch = (): AppDispatch => useDispatch<AppDispatch>()
export const useReduxSelector: TypedUseSelectorHook<RootState> = useSelector
export default store

```

We've added line 12 - 14 where we define our AppDispatch type and create our typed aliases. With those in place, let's finish up the Counter.tsx file.

We import the new useReduxDispatch and useReduxSelector and use them to dispatch actions on button click and update the counter with the newest counter state.

```jsx=
import { Button, Text } from 'react-native'
import React from 'react'
import { decrement, increment } from '../../redux/demo/counter'
import { useReduxDispatch, useReduxSelector } from '../../redux'

const Counter = (): React.ReactElement => {
    const value = useReduxSelector(state => state.counter)
    const dispatch = useReduxDispatch()

    return (
        <>
            <Text>{value}</Text>
            <Button title="increment" onPress={() => dispatch(increment(1))}>
                +1
            </Button>
            <Button title="decrement" onPress={() => dispatch(decrement(1))}>
                -1
            </Button>
        </>
    )
}

export default Counter
```

Now everything is in place. Head back to the App.tsx we haven't touched since the last tutorial and add our new Counter component and as usual, our Provider component to wrap our app and provide access to our store.

```jsx=
import { StatusBar } from 'expo-status-bar'
import React from 'react'
import { StyleSheet, View } from 'react-native'
import { Provider } from 'react-redux'
import store from './src/redux'
import Counter from './src/components/demo/Counter'

export default function App(): React.ReactElement {
    return (
        <Provider store={store}>
            <View style={styles.container}>
                <Counter />
                <StatusBar style="auto" />
            </View>
        </Provider>
    )
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#fff',
        alignItems: 'center',
        justifyContent: 'center',
    },
})
```

That's it. If you run the app with `npm start` and click the buttons, the counter should go up and down.

Make sure to run `npm run fix` to take care of any minor code style errors and make sure we did everything according to our linting and rules and it's a wrap.


## Conclusion
We've continued with our React Native Typescript project from last time and added a fully functioning and fully typed Redux Store in about 30 lines of redux code (imports excluded) and a few lines of tsx. That said, I should maybe rewrite my original redux article and the updated version will be a lot shorter...

Small side note, if you want to test your redux config with ease, use the web browser version from the Expo Web Interface and open your dev tools. With the right extensions installed (react + redux) you can look at all components and the store/state with an easy to use interface.

![](https://i.imgur.com/5my0HKg.png)

It ain't much and it ain't pretty but it works and it is typed and scaleable.

In our next sessions, we will have a look at how to make our redux store persistent, some basic solutions for navigation and project file structure.

Here is the promised [link to the (Pre-)Release tag on Github](https://github.com/AllBitsEqual/expo-ts-starter/tree/v0.2.0).
