<!--
	title: "Build A Bot (DiscordJS) - A scalable setup with command modules"
	description: "Today we will clean up our central index.js file, make it more readable and scaleable and move all our existing commands (and a new one) to a separate folder for import."
	author: "Konrad Abe (AllBitsEqual)"
	published_at: 2021-01-25 08:00:00
	header_image: "https://i.imgur.com/LdxxqzS.jpg"
	categories: "javascript discord discordjs bot chatbot series"
	canonical_url: "https://allbitsequal.medium.com/build-a-bot-discordjs-a-scalable-setup-with-command-modules-7fecdda27b40"
	series: "Build A Bot (DiscordJS)"
	language: en
-->
# React Bootstrapping - Quick Start with Typescript, Linting, Redux, Router

**Setting up a new project can be a daunting task. So many packets that need to work in concert and so many things to keep track of, that could go wrong or be incompatible in certain versions.**

**Add typescript to the mix and you are in for a fun journey with untyped packages, missing return types and complex interfaces.**

I've recently set up a new project base for one of our customers with a well planned and fully functional stack and I'm about to offer the finished project (minus some customer-specific features) as a starter template for you to save some time. Or maybe you have your own but want to see how to set up some packages in combination so lean back and enjoy the show. All code is online on GitHub including separate commits per feature so either code along or copy what you need.

This is a two-sessions tutorial, this week we will take care of the business logic and next week we will add styled-components, storybook and testing.


## Kick off

To start, we will use create-React-app to start with a fresh typescript template using the typescript template code

```bash 
create-react-app yourProjectName --template typescript
```

This gives us a preconfigured react app using typescript with all of the configs taken care of and webpack fully set up, loaders and all.

[GitHub Commit](https://github.com/AllBitsEqual/react-ts-starter/commit/512b61c198a38d5dbdbb4d57a268bcba135b719d) (not worth looking as it's just the boilerplate from create-react-app)

## Check the controls

The next step in every project I work on is setting up eslint, prettier and the .editorcofig file.

If you run this command in your terminal, it will install all dependencies, plugins and presets needed for this setup.

```bash
npm i -S eslint eslint-config-airbnb eslint-config-prettier eslint-plugin-prettier eslint-plugin-react eslint-plugin-react-hooks @typescript-eslint/eslint-plugin @typescript-eslint/parser prettier
```

In this guide, I'll keep it short and point you to my files, but I've recently written a longer article about linting in typescript projects so if you would like more information on this topic, head over to [my other article about linting in react native projects](https://allbitsequal.medium.com/series-react-native-step-by-step-working-with-typescript-and-linting-3961c4226793). Even though this article is for React Native / Expo, the linting is basically the same so I recommend to take a look at it if you want to know more.

To continue with our current step, go to this repository and copy the following files and put them into your project's root:
- .editorcofig
- eslintrc
- prettierrc

Then head over to your package.json and add the following 2 lines in your scripts object.

```javascript
"lint": "tsc --noEmit && eslint --ext .js,.jsx,.ts,.tsx ./ || true",
"fix": "tsc --noEmit && eslint --fix --ext .js,.jsx,.ts,.tsx ./ || true",
```

If you run `npm run lint` in your terminal, you will get the linting output and running `npm run fix` will try to fix and prettify all your files according to your linting rules.

[GitHub Commit A](https://github.com/AllBitsEqual/react-ts-starter/commit/bee5fc48e55613cc57ed06b75bcd3402bfe8b292) Changes

[GitHub Commit B](https://github.com/AllBitsEqual/react-ts-starter/commit/c1624f7e2fedbce449b0ca4844afa4f4f2177da8) Applied Linting

## Keep track of your state

Next step on our fast-paced journey is adding redux, using the redux toolkit (rtk).

Again, grab the necessary packages from npm and we can continue.

```bash
npm i -S react-redux @reduxjs/toolkit react-router react-router-dom connected-react-router @types/react-redux @types/react-router @types/react-router-dom history@4.10.1
```

> Mind the version number. At the time of writing (01/02/2021), using a v5 of history will break stuff because of incompatibilities. If you run into other issues, feel free to copy my package-lock.json to get the exact working setup I'm using.

With this in place, we need a file to export our store and another for our rootReducer where we register all the reducers we are going to write. We will add them under src/redux/.

Again, you can grab them including a demo file using the created react hooks from my repo.

- src
    - redux
        - demo/
        - index.ts
        - rootReducer.ts

```javascript
// File: src/redux/index.ts
import { configureStore } from '@reduxjs/toolkit'
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux'
import { createBrowserHistory } from 'history'
import rootReducer from './rootReducer'

export const history = createBrowserHistory()

const store = configureStore({
    reducer: rootReducer(history),
    // middleware: getDefaultMiddleware => getDefaultMiddleware(), // .prepend(middleware)
})

export type RootState = ReturnType<typeof store.getState>

export type AppDispatch = typeof store.dispatch
export const useReduxDispatch = (): AppDispatch => useDispatch<AppDispatch>()
export const useReduxSelector: TypedUseSelectorHook<RootState> = useSelector
export default store
```

What's special about this? We are using the default react hooks for useSelector and useDispatch but we wrap them in our own variations including all the typing needed to satisfy typescript and export them again as `useTypedDispatch` and `useTypedSelector`.
We don't have middleware yet so this line is commented out but I've left it there for when I write my middleware in the future.

If you look at the rootReducer, you can see how we hooked up the demo counter reducer and our route reducer. I've added a TODO marker to keep track of the fixed history package version here as a reminder to check for updates when going through my TODOs.

```javascript
// File: src/redux/rootReducer.ts
import { combineReducers } from '@reduxjs/toolkit'
import { connectRouter } from 'connected-react-router'
import { History } from 'history' // TODO: check for updates to switch to more recent version of history
import counterReducer from './demo/counter'

const rootReducer = (history: History) =>
    combineReducers({
        counter: counterReducer,
        router: connectRouter(history),
    })

export default rootReducer
```

Last but not least, this is the counterReducer, small and readable thanks to Redux Toolkit.

```javascript
// File: src/redux/demo/counter.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit'

const initialState = 0

const counterSlice = createSlice({
    name: '[DEMO] counter',
    initialState,
    reducers: {
        increment: (state, action: PayloadAction<number>) => state + action.payload,
        decrement: (state, action: PayloadAction<number>) => state - action.payload,
    },
})

export const { increment, decrement } = counterSlice.actions
export default counterSlice.reducer
```

Next stop is our router. In the past, it was seen as an anti-pattern to pair routing and state/redux but over the last few years, this has become a proven setup that allows us to control user navigations and state in a more fine-grained and state-checked manner. To make this work, we will ad React-router and connected-React-router for easy integration of both.

To check if Redux and Routing work, we will add a demo/counter example and set up some basic routing.

Create or copy the following files from my repostitory:
- src/
    - components/demo/Counter.tsx
    - routes/index.tsx

In the Counter component, you can see the typed redux hooks at work. It's your well known basic counter example, just a bit shorter.

```javascript
// File: src/components/demo/Counter.tsx
import React from 'react'
import { decrement, increment } from '../../redux/demo/counter'
import { useTypedDispatch, useTypedSelector } from '../../redux'

const Counter = (): React.ReactElement => {
    const value = useTypedSelector(state => state.counter)
    const dispatch = useTypedDispatch()

    return (
        <>
            <input type="text" disabled value={value} />
            <button type="button" title="increment" onClick={() => dispatch(increment(1))}>
                +1
            </button>
            <button type="button" title="increment" onClick={() => dispatch(decrement(1))}>
                -1
            </button>
        </>
    )
}

export default Counter
```

Our demo version of the routes is similarly simple but it should be enough for you to understand the general idea and expand on it. We're creating a reusable NavBar Component here to simulate a real navigation and a couple of stand-in pseudo page components for home, hello and our counter page as well as a generic 404 for broken/unmatched URLs.


```javascript
// File: src/routes/index.tsx
import React from 'react'
import { Route, Switch } from 'react-router'
import { Link } from 'react-router-dom'
import Counter from '../components/demo/Counter'

const NavBar = () => (
    <>
        <Link to="/"><button type="button">Home</button></Link>
        <Link to="/hello"><button type="button">Hello</button></Link>
        <Link to="/counter"><button type="button">Counter</button></Link>
    </>
)

const Home = () => (<><NavBar /><h1>home</h1></>)
const Hello = () => (<><NavBar /><h1>Hello</h1></>)
const NoMatch = () => (<><NavBar /><h1>404</h1></>)

const DemoCounter = () => (
    <>
        <NavBar />
        <h1>Counter</h1>
        <Counter />
    </>
)

const Routes = (): React.ReactElement => (
    <div>
        <Switch>
            <Route exact path="/" component={Home} />
            <Route path="/hello" component={Hello} />
            <Route path="/counter" component={DemoCounter} />
            <Route component={NoMatch} />
        </Switch>
    </div>
)

export default Routes
```

By adding the counter.tsx component to the project in one of the routes, we can see that routing and redux work. If you run `npm run lint` again, the linter and ts compiler won't flag any typescript issues either. Another hurdle taken.

![](https://i.imgur.com/RpqxkAF.png)

If you check the redux tools in your browser, you can see that every navigation action triggers an action on our store and that our counter actions are clearly discernable by their `[DEMO]` prefix.


[GitHub Commit (final)](https://github.com/AllBitsEqual/react-ts-starter/commit/930463f4407206c0c9fa6f88316bfa6ed0bd0f81)

## Conclusion
We've covered a lot of ground today, skipping some of the finer details. As mentioned before, the idea is to allow for a quick setup. I will add more articles in the future, looking in-depth into some of those topics I have not covered yet in other articles.

Next week we will add Storybook and styled-components. Having a dynamic and interactive UI library as part of a project can be a real asset. Storybook has proven it's worth for us many times over, allowing us to show UI elements on their own with adjustable attributes and toggles in an intuitive web UI. Your QA and Design/Concept teams will love it.