---
tags: React Native, Typescript, React Navigation
---
<!--
	title: "SERIES: React Native (Step by Step) - Strongly Typed Navigation with React Navigation 5.x and Typescript"
	description: "This time we will integrate React Navigation v5, a very flexible and robust navigation/routing solution and the primary recommendation made by the expo team too."
	author: "Konrad Abe (AllBitsEqual)"
	published_at: 2021-04-12 08:00:00
	header_image: "https://i.imgur.com/7PSC0Tj.jpg"
	categories: "react-native typescript react-navigation best-practices series"
	canonical_url: "https://allbitsequal.medium.com/series-react-native-step-by-step-strongly-typed-navigation-with-react-navigation-5-x-a46f2daaf64c"
    series: "React Native (Step by Step)"
	language: en
-->
# SERIES: React Native (Step by Step) - Strongly-Typed Navigation with React Navigation 5.x and Typescript

Last time we added redux to our app. This time we will integrate React Navigation (v5.x at the time of writing), a very flexible and robust navigation/routing solution and the primary recommendation made by the expo team too.

With the v5 update, React Navigation got a major overhaul. They split up their old repository into several smaller ones and introduced many breaking changes but the current version is in a very good state right now and worth the effort.

In our example today, I will show you how to set up your basic routes and how to implement navigation actions and conditional navigation and in the follow-up to this part, we will look at a very common authentification flow, nested sub navigations and a few tricks to make your life easier.

As always, you can find a link to the finished code on GitHub at the end of the article.

![](https://i.imgur.com/7PSC0Tj.jpg)

## Preparations
If you want to code along and did not follow the previous episodes, simply [go to my GitHub Repository here and grab the code from the v0.2 tag](https://github.com/AllBitsEqual/expo-ts-starter/releases/tag/v0.2.0).

We will start by installing the core library for react native via npm.

```bash
npm install @react-navigation/native
```

There is a list of packages we're going to need but as we are using the expo managed workflow, we can simply use the expo install command to install compatible versions of these libraries.

* react-native-gesture-handler
* react-native-reanimated
* react-native-screens
* react-native-safe-area-context
* @react-native-community/masked-view

A small note, we won't be using all of these in this part of the series but I'm including them here because they are part of my regular setup anyway.

```bash
expo install react-native-gesture-handler react-native-reanimated react-native-screens react-native-safe-area-context @react-native-community/masked-view
```

We will need 4 simple placeholder screens to showcase the navigation. You can simply create them under src/screens and add their names in a <Text> tag in a basic react component. That way we won't get any compiler warnings for missing module exports and can reference the correct files when we set up our routing.

![](https://i.imgur.com/WDmcknP.png)


## Defining The Basic Routes
React Navigations can be nested and combined but for our main navigation, the one that will make whole page transitions, we are going to start with a simple stack.

Install the stack library in addition to the ones we grabbed earlier because we're going with a navigation stack for now.

```bash
expo install @react-navigation/stack
```

A stack navigation works like a stack of card and comes with basic page transition animations based on your current device/os. Each time you navigate in a stack, you put a new "card" on top of the stack and when you navigate back, you remove the top-most card again. Last in, first out. Using a stack navigation will give you an app-like feel of responsiveness right out of the box.

Create a new file called routes.ts under src/routing and import the createStackNavigator function from react-navigation/stack. We will also export an enum of all the screens we register in our routing setup, which will allow us to refer to screens via MainRoutes.Home instead of using a regular string, something that will be helpful when working with typescript and will also be picked up by your IDEs autocompletion.

```javascript
// File: src/routing/routes.ts
import { createStackNavigator } from '@react-navigation/stack'

export enum MainRoutes {
    Splash = 'Splash',
    Loading = 'Loading',
    Home = 'Home',
    Settings = 'Settings',
}

export type MainStackParamList = {
    [MainRoutes.Splash]: undefined
    [MainRoutes.Loading]: undefined
    [MainRoutes.Home]: { update: boolean } | undefined // just an example, "update" will later be used for version checks
    [MainRoutes.Settings]: undefined
}

export const MainStack = createStackNavigator<MainStackParamList>()
```

The second thing you see in this file is the type for all our screens. React navigation allows us to call screens with params. Defining them as undefined means, that this route does not expect any params on load while the "Home" route can be called with an optional boolean flag for the "update" attribute.

We then export our MainStack created with the createStackNavigator and our list of possible stack params.

---

A common example for route params would be something like an userID for a profile page or a sorting param in a list.

```javascript
// Example - don't add this to our project
type ExampleStackParamList = {
  Home: undefined
  Profile: { userId: string }
  Feed: { sort: 'latest' | 'top' } | undefined
}
```

---

## Building Our Main Navigation
To build our MainNavigation component we are using a NavigationContainer from react navigation and map our Screen components with the navigation routes we defined. After importing our MainStack from './routes' we can use its Navigator and Screen components to organise our different screens/pages.

I'm using the attribute `headerMode="none"` on the Navigator to disable the default topBar header that comes out of the box with the component.

```javascript
// File: src/routing/MainNavigation.tsx 
import React from 'react'
import { NavigationContainer } from '@react-navigation/native'

import { MainStack, MainRoutes } from './routes'

import SplashScreen from '../screens/SplashScreen'
import AppLoadingScreen from '../screens/AppLoadingScreen'
import HomeScreen from '../screens/HomeScreen'
import SettingsScreen from '../screens/SettingsScreen'

const MainNavigation = (): React.ReactElement => {

    return (
        <NavigationContainer>
            <MainStack.Navigator headerMode="none">
                <MainStack.Screen name={MainRoutes.Splash} component={SplashScreen} />
                <MainStack.Screen name={MainRoutes.Loading} component={AppLoadingScreen} />
                <MainStack.Screen name={MainRoutes.Home} component={HomeScreen} />
                <MainStack.Screen name={MainRoutes.Settings} component={SettingsScreen} />
            </MainStack.Navigator>
        </NavigationContainer>
    )
}
export default MainNavigation
```

## Implementing The Main Navigation
To use the new main navigation we just have to put it in our root project file, the App.tsx and throw out all content other than the redux provider.

```javascript
// File: App.tsx
import { StatusBar } from 'expo-status-bar'
import React from 'react'
import { Provider } from 'react-redux'
import store from './src/redux'
import MainNavigation from './src/routing/MainNavigation'

export default function App(): React.ReactElement {
    return (
        <Provider store={store}>
            <StatusBar hidden />
            <MainNavigation />
        </Provider>
    )
}
```

When you restart your app now, everything should work again for a limited amount of "work". What we have right now is one big stack of screens. The user will start on the first screen in the stack, our splash screen, and has no means to navigate around.

## About Navigation Patterns
One of the most common navigation patterns is the so-called AuthFlow, where a different stack is used for the authentication of the user and the navigation of the actual app. I will show you a more complex AuthFlow in my next article but to show you the basics, we will mock a login with redux and use it to explain the principle.

---

While we are at it, go to your src/redux folder and rename the "demo/" folder to "ducks/", something I missed in my last article.  

The idea behind ducks is to NOT have an actions.ts, middleware.ts and reducer.ts file for all your parts of the redux store but instead have one file with all parts of one slice combined.

If you want to know more about ducks and redux/reducks, I'll leave you with [an article about the basic idea behind using ducks](https://www.freecodecamp.org/news/scaling-your-redux-app-with-ducks-6115955638be/) and [the ducks proposal by Erik Rasmussen](https://github.com/erikras/ducks-modular-redux).

---

Create a new duck for our mock user and login code. For now, we will only include a setLogin action for toggling the login flag and a selector to access the stored value via a custom useState hook. Don't forget to register the new reducer in our rootReducer.ts and we are good to go.

```javascript
// File: src/redux/ducks/user.ts
/* eslint-disable no-param-reassign */
import { createAction, createReducer } from '@reduxjs/toolkit'
import { RootState } from '../index'

type UserState = {
    login: boolean
}

const initialState: UserState = {
    login: false,
}

export const setLogin = createAction('[USER] Set Login', (isLoggedIn: boolean) => ({
    payload: {
        isLoggedIn,
    },
}))

export const selectLogin = (state: RootState): boolean => state.user.login

const userReducer = createReducer(initialState, builder => {
    builder.addCase(setLogin, (state, action) => {
        state.login = action.payload.isLoggedIn
    })
})

export default userReducer
```

With the new selectLogin selector, we can now adjust our navigation stack a bit.

```javascript
// File: src/routing/MainNavigation.tsx
import React from 'react'
import { NavigationContainer } from '@react-navigation/native'

import { MainStack, MainRoutes } from './routes'
/* new */ import { useReduxSelector } from '../redux'
/* new */ import { selectLogin } from '../redux/ducks/user'

import SplashScreen from '../screens/SplashScreen'
import AppLoadingScreen from '../screens/AppLoadingScreen'
import HomeScreen from '../screens/HomeScreen'
import SettingsScreen from '../screens/SettingsScreen'

const MainNavigation = (): React.ReactElement => {
    const isLoggedIn = useReduxSelector(selectLogin)

    return (
        <NavigationContainer>
            <MainStack.Navigator headerMode="none">
                {isLoggedIn ? (
                    <>
                        <MainStack.Screen name={MainRoutes.Loading} component={AppLoadingScreen} />
                        <MainStack.Screen name={MainRoutes.Home} component={HomeScreen} />
                        <MainStack.Screen name={MainRoutes.Settings} component={SettingsScreen} />
                    </>
                ) : (
                    <MainStack.Screen name={MainRoutes.Splash} component={SplashScreen} />
                )}
            </MainStack.Navigator>
        </NavigationContainer>
    )
}
export default MainNavigation
```

As you can see, we are now using the selectLogin selector to set our isLoggedIn flag and use it in our Navigator to load a different stack of screens based on the login state. If the user is on the "Splash" screen and triggers a login, he will automatically be switched over to the first screen in the other stack and end up on the "Loading" screen.

## Moving On

In our SplashScreen.tsx file we can now set up a simple clickable screen that triggers the fake login.

```javascript
// File: src/screens/SplashScreen.tsx
import React from 'react'
import { Text, View, TouchableWithoutFeedback, StyleSheet } from 'react-native'
import { useReduxDispatch } from '../redux'
import { setLogin } from '../redux/ducks/user'

const SplashScreen = (): React.ReactElement => {
    const dispatch = useReduxDispatch()

    const handleClick = (): void => {
        dispatch(setLogin(true))
    }

    return (
        <TouchableWithoutFeedback onPress={() => handleClick()}>
            <View style={styles.page}>
                <View style={styles.titleBox}>
                    <Text>ALL BITS EQUAL</Text>
                    <Text>presents</Text>
                    <Text>The Expo Starter Kit</Text>
                </View>
                <View style={styles.contentBox}>
                    <Text>Touch Screen to start!</Text>
                </View>
                <View style={styles.footer}>
                    <Text>written by Konrad Abe</Text>
                </View>
            </View>
        </TouchableWithoutFeedback>
    )
}

const styles = StyleSheet.create({
    page: {
        flex: 1,
        backgroundColor: '#fff',
        alignItems: 'center',
        justifyContent: 'center',
    },
    titleBox: {
        width: '100%',
        height: '50%',
        alignItems: 'center',
        justifyContent: 'center',
    },
    contentBox: {
        width: '100%',
        height: '45%',
        alignItems: 'center',
        justifyContent: 'center',
    },
    footer: {
        width: '100%',
        height: '10%',
        paddingRight: '3%',
        alignItems: 'flex-end',
        justifyContent: 'center',
    },
})

export default SplashScreen
```

The important thing to note here is that we are basically NOT triggering a navigation event with this (we'll come to that in a moment) but switching to a separate navigation stack. For this reason, we don't need to access the navigation params of our Navigator at all.

As I said, you can read more about this AuthFlow navigation pattern in the next article. Moving on...

## Navigating Our Routes
When using React Navigation within our app, all screens that are direct children of the navigator have access to the "navigation" object. Using this with strong typing can be a bit tricky because the type for the navigation prop takes 2 generics, the param list object we defined earlier, and the name of the current route. 

We don't want to rebuild this in every screen component that needs to access the navigation prop so I will add a types.ts file to the routing/ directory and import both the MainRoutes enum and the MainStackParamList that holds all our defined routes and their annotations.

```javascript
// File: src/routing/types.ts
import { StackNavigationProp } from '@react-navigation/stack'
import { MainRoutes, MainStackParamList } from './routes'

export type MainNavigationProp<
    RouteName extends keyof MainStackParamList = MainRoutes
> = StackNavigationProp<MainStackParamList, RouteName>
```

We can now use this to annotate the navigation prop in our components. For our first screen after the login/stack switch, we will define a small delay before navigating to the home screen and display a simple "loading..." text.

This would usually be the place to check the version of your app against your servers, load user data and stuff like that. This process would then replace the setTimeout() we used here to fake a real app flow.

```javascript
// File: src/screens/AppLoadingScreen.tsx
import React, { useEffect } from 'react'
import { Text, View, StyleSheet } from 'react-native'
import { MainNavigationProp } from '../routing/types'
import { MainRoutes } from '../routing/routes'

type AppLoadingScreenProps = {
    navigation: MainNavigationProp<MainRoutes.Loading>
}

const AppLoadingScreen = ({ navigation }: AppLoadingScreenProps): React.ReactElement => {
    useEffect(() => {
        setTimeout(() => {
            navigation.navigate(MainRoutes.Home)
        }, 1500)
    }, [navigation])

    return (
        <View style={styles.page}>
            <Text>loading...</Text>
        </View>
    )
}

const styles = StyleSheet.create({
    page: {
        flex: 1,
        backgroundColor: '#fff',
        alignItems: 'center',
        justifyContent: 'center',
    },
})

export default AppLoadingScreen
```

---

On our home screen, we will place a regular navigation button as well as a logout button dispatching a redux action to check if we get redirected to the auth stack when we set the login state to false again.

```javascript
// File: src/screens/HomeScreen.tsx
import React from 'react'
import { Text, View, StyleSheet, Button } from 'react-native'
import { MainNavigationProp } from '../routing/types'
import { MainRoutes } from '../routing/routes'
import { useReduxDispatch } from '../redux'
import { setLogin } from '../redux/ducks/user'

type HomeScreenProps = {
    navigation: MainNavigationProp<MainRoutes.Home>
}
const HomeScreen = ({ navigation }: HomeScreenProps): React.ReactElement => {
    const dispatch = useReduxDispatch()
    const logoutHandler = () => dispatch(setLogin(false))

    return (
        <View style={styles.page}>
            <Text>HOME</Text>
            <Button title="logout" onPress={() => logoutHandler()} />
            <Button title="settings" onPress={() => navigation.navigate(MainRoutes.Settings)} />
        </View>
    )
}

const styles = StyleSheet.create({
    page: {
        flex: 1,
        backgroundColor: '#fff',
        alignItems: 'center',
        justifyContent: 'center',
    },
})

export default HomeScreen
```

---

To wrap up this episode, the last thing I want to show you today is the goBack() function. On our SettingsScreen we will put a button that does not navigate to a specific screen but to the last screen on the stack prior to the SettingsScreen. This means that you could link to the settings from any page in the stack and when clicking the backlink, you would be taken back to the page you were before.

```javascript
// File: src/screens/SettingsScreen.tsx
import React from 'react'
import { Text, View, StyleSheet, Button } from 'react-native'
import { MainNavigationProp } from '../routing/types'
import { MainRoutes } from '../routing/routes'

type SettingsScreenProps = {
    navigation: MainNavigationProp<MainRoutes.Settings>
}

const SettingsScreen = ({ navigation }: SettingsScreenProps): React.ReactElement => (
    <View style={styles.page}>
        <Text>SETTINGS</Text>
        <Button title="back" onPress={() => navigation.goBack()} />
    </View>
)

const styles = StyleSheet.create({
    page: {
        flex: 1,
        backgroundColor: '#fff',
        alignItems: 'center',
        justifyContent: 'center',
    },
})

export default SettingsScreen
```

![](https://i.imgur.com/oXTEeum.png)


## Wrapping Up
Today we learned how to annotate both our routes and our navigation prop. 

As promised, here is [the finished code on GitHub at the v3 release tag](https://github.com/AllBitsEqual/expo-ts-starter/releases/tag/v0.3.0).

Next time I will show you two more complex scenarios with react navigation including the useNavigation custom hook and how to trigger navigation events from redux middleware. We will build a better AuthFlow that mimics a real application (including some extra features), we will validate navigation events before they occur and I will show you how to implement a nested navigation to control parts of a page or modal window without moving away from the current screen.