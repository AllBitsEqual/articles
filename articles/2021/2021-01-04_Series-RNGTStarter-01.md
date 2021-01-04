<!--
	title: "SERIES: React Native (Step by Step) - Working with Typescript"
	description: "In the first part of our React Native (Step by Step) series, we will look at how to start a new project with Expo and Typescript, configure our linter and talk a bit about the how and why. Grab your coffee, relax and strap in for a fascinating journey."
	author: "Konrad Abe (/)AllBitsEqual)"
	published_at: 2021-01-04 08:00:00
	header_image: "https://i.imgur.com/UYSyTVN.jpg"
	categories: "react-native typescript best-practise series"
	canonical_url: "https://allbitsequal.medium.com/series-react-native-step-by-step-working-with-typescript-and-linting-3961c4226793"
    series: "React Native (Step by Step)"
	language: en
-->
# SERIES: React Native (Step by Step) - Working with Typescript and Linting

**In the first part of our *React Native (Step by Step)* series, we will look at how to start a new project with Expo and Typescript, configure our linter and talk a bit about the how and why. Grab your coffee, relax and strap in for a fascinating journey.**

![](https://i.imgur.com/UYSyTVN.jpg)

## Typescript - WHY?
There are a lot of strong sentiments and feelings around typescript that go in both directions. The die-hard fans swear by the fact that thanks to typescript, their code is now less buggy, more stable, easier to reason about and easier to extend, share or collaborate on. On the other side of the ring, we have those who deem typescript a nuisance, something that creates a lot of unnecessary overhead and in some cases even duplication while not actually fixing any issues at runtime.

I will not start a long-winded discussion about the pro's and con's and keep it short. I've decided to work with typescript in my private projects for a lot of reasons.
* I am working with typescript in my 9 to 5 job on a regular basis
* I like the additional information, my IDE can draw from strongly typed code
* Many of my reusable code bits will at some point be put into separated repositories and distributed via NPM

I, too, have had my fair share of "Duh" moments when running into issues with stuff my compiler/linter did not like because I've been too vague or simply had not typed something at all because it looked too complex and had hindered me at the time I was writing it. I've adopted a routine that allows me to `// @ts-ignore` something for the time being and running a check before I commit my changes, making sure I think twice before allowing an unchecked piece of code into my repositories.

There are moments when you might puzzle about WHY the typescript compiler does not want to allow a certain piece of code or accept a type that looks like it should be ok and I've come to the realisation that in most cases I was about to do something that COULD go wrong at some point in the future and bite me in my backside. *Yeah, sure. I, as the developer, know fully well that I will never call this function at a point where my props are still undefined... or do I?* Adding null checks (and other safeguards) has by now become a habit I am happy to live with. I've seen enough times in enterprise projects where something breaks because, for some unforeseen reason, a server response was empty or had a missing or wrong type. It was not our code that was wrong but putting too much trust in the contracts of APIs and not making sure on both ends that things are as expected can lead to some nasty and sometimes hard to track down bugs.

**Long story short:** I've learned to stop worrying and love the compiler. I've learned a lot by trying to understand WHAT typescript was trying to tell me and I think it made me a better programmer by NOT TRUSTING the Typescript Compiler to prevent everything.



## Typescript - HOW?
### Getting Started with React Native and Typescript
Starting a new project could not be easier. As in earlier projects, we will again start with Expo and we will stay in the managed workflow to keep things nice and easy.

I assume you have worked with NPM and GitHub in the past and at least know the basics. To get started with React Native using EXPO you need to install a command-line tool called expo-cli and it is recommended to install it globally using the -g flag.
At the time of writing, I'm using expo-cli@4.0.17 but there won't be any breaking changes for later versions of 4.x when you are reading this in the future.

```bash
npm install -g expo-cli
```

Using Expo to initialise our new project, we have to typescript templates to choose from. For our case, we will build everything from scratch step by step so we will choose the empty typescript template from the [managed workflow](https://docs.expo.io/introduction/managed-vs-bare/). The following command, when entered from your desired location via command line, will then create a new folder by the name you provide in the init and start populating it with the template files you choose from the following menu.

```bash
expo init yourProjectName
```

![](https://i.imgur.com/ZY4lyX1.png "test")

It might take a moment for the expo-cli to download and install all dependencies from npm but once it's done, you can simply `cd yourProjectName` into the new folder and start working.

That's it, you did it. You successfully started and configured a React Native project via Expo that runs with typescript.


### GitHub - sync your progress
While it might not feel like we did a lot so far, I can only recommend syncing your work to a versioned repository service like GitHub, GitLab or whatever floats your boat. I use GitLab at work and GitHub for both open source and private projects.

[This article on docs.github.com](https://docs.github.com/en/free-pro-team@latest/github/importing-your-projects-to-github/adding-an-existing-project-to-github-using-the-command-line) shows the whole process in a very clean and easy to follow explanation so I won't repeat that.


### Did it work?
If you want to make sure, everything so far is working as intended, simply enter the following line into your command-line tool while located at the project root and you should see the following page opened in your default browser. To check the actual mobile app, you can start an emulator from the left-hand menu, for mac users with Xcode, the iOS simulator should work best.

```bash
npm start
```

Expo Web Interface 
![](https://i.imgur.com/jcM8df5.png)



## Typescript - Additional Setup
Of course, you might want to customise your own project a bit more. The basic setup is nice and all but maybe we can import a few sensible presets, add a few more rules to our linting and throw in an editor config file. 


### Editor Config
This is usually the first thing I copy from my other projects because it makes working in most IDEs so much easier. A `.editorconfig` file at the root level of your project folder can be read by most IDEs by default while others can be "enabled" by installing a small plugin. The IDE will then automatically help you with correct indentations, mark the max line length and more.

[Go to this site](https://editorconfig.org/), if you want to know more or check your own IDE for plugins. Don't be put off by the 90's comic style website. Their content is state of the art.

Here's my current config, feel free to add it to your projects too.

**File:** /.editorconfig

```json
root = true

[*]
charset = utf-8
end_of_line = lf
indent_size = 4
indent_style = space
insert_final_newline = true
max_line_length = 100
trim_trailing_whitespace = true

[*.{json,yml,*rc}]
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

### Linting and Prettier
To get started with linting, we’ll need to install both eslint and a bunch of smart presets that we’re gonna use for development.  
We will also be using Prettier to automatically fix the easier formatting stuff so that we do not need to adjust all indentations and semicolons in copied code snippets from the web.

#### eslint
We are going to use the Airbnb ESLint setup and there are multiple necessary packages. Fortunately, eslint-config-airbnb exists and makes things easy. You only need to run:

```bash
npx install-peerdeps --dev eslint-config-airbnb
```

This will install the following packages to your projects dev dependencies: 
+ eslint@7.2.0
+ eslint-config-airbnb@18.2.1
+ eslint-plugin-react@7.22.0
+ eslint-plugin-import@2.22.1
+ eslint-plugin-react-hooks@4.0.0
+ eslint-plugin-jsx-a11y@6.4.1

#### linting typescript
To work with Typescript, we need to install the libraries that will enable us to lint Typescript code as well.

```bash
npm i --save-dev @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

#### prettier linting
The installation of Prettier is similarly easy to that of Typescript. We need a config and a plugin.

```bash
npm i --save-dev prettier eslint-config-prettier eslint-plugin-prettier
```


### Configuration


To run you through the basics here, eslint is the actual linter we will be using while eslint plugin import is used to extend the linter with additional functionality. The Typescript parser is needed to parse our typescript code so eslint can do its job and prettier will allow us to automatically fix/change some of the code issues according to our predefined rules.


To get a basic setup, you could simply install eslint and run `eslint --init` to start a guided init process but we will do that ourselves and add our own .eslintrc config file to the project, again at root level.

Let's have a look at our finished config file.

**File:** /.eslintrc

```json
{
  "extends": [
    "airbnb",
    "airbnb/hooks",
    "plugin:@typescript-eslint/recommended",
    "prettier",
    "prettier/react",
    "prettier/@typescript-eslint",
    "plugin:prettier/recommended"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaFeatures": {
      "jsx": true
    },
    "ecmaVersion": 2018,
    "sourceType": "module",
    "project": "./tsconfig.json"
  },
  "plugins": ["@typescript-eslint", "react", "prettier"],
  "rules": {
    "func-names": 0,
    "import/extensions": ["error", "never"],
    "import/no-unresolved": 0,
    "import/prefer-default-export": 0,
    "jsx-a11y/accessible-emoji": 0,
    "prettier/prettier": [
      "error",
      {
        "singleQuote": true,
        "trailingComma": "all",
        "arrowParens": "avoid",
        "endOfLine": "auto"
      }
    ],
    "react/jsx-filename-extension": [1, {
      "extensions": [
        ".ts",
        ".tsx"
      ]
    }],
    "react/jsx-one-expression-per-line": "warn",
    "react/prop-types": 0,
    "react/require-default-props": 0,
    "react/style-prop-object": 0,
    "semi": ["error", "never"],
    "no-plusplus": ["error", {"allowForLoopAfterthoughts": true}],
    "no-shadow": "off",
    "no-use-before-define": "off",
    "@typescript-eslint/ban-ts-comment": 0,
    "@typescript-eslint/explicit-function-return-type": 0,
    "@typescript-eslint/explicit-member-accessibility": 0,
    "@typescript-eslint/no-shadow": ["error"],
    "@typescript-eslint/no-use-before-define": ["error", {
      "functions": true,
      "classes": true,
      "variables": false
    }],
    "@typescript-eslint/no-var-requires": 0
  }
}
```

Let's have a quick look at the different parts of our config file:
* **"extends"** => our presets of rules
* **"parser"** => points at our typescript parser
* **"parserOptions"** => as the name implies
* **"plugins"** => some plugins for our convenience
* **"rules"** => this is where we add our own rules and overwrite presets from the **"extends"** section that we disagree with

The last thing we need to add is our .prettierrc config file. There are only a few minor adjustments we need to make here because most of the things we want to be fixed are already covered by our eslint rules. Simply add a new .prettierrc file at root level with these 5 lines.


**File:** /.prettierrc

```json
{
  "singleQuote": true,
  "trailingComma": "es5",
  "semi": false
}
```

If you want to be adamant about clean code in your repository, you could go one step further and run the linter before accepting new code as a commit. I'm not working with a pre-commit hook that completely prevents pushing code with linting errors into the repository. This might be the right thing for you and I can only recommend to you to have a look at husky for this but as I'm doing a lot of prototyping in my projects and my workflow includes code reviews before I merge anything into my development or main branches, I'm not restricting this.

To run our linter and make use of the prettier auto code corrections, we need to add two new scripts to our package.json.

```json
"scripts": {
  ...
  "lint": "tsc --noEmit && eslint --ext .js,.jsx,.ts,.tsx ./ || true",
  "fix": "tsc --noEmit && eslint --fix --ext .js,.jsx,.ts,.tsx ./ || true",
  ...
}
```

Both commands run the typescript compiler without making changes to the code (tsc --noEmit) and then run eslint on all matching files starting at root level. 

One word of warning/advice on my scripts here. I'm piping all erroneous output away (|| true) because the script will return an error when running and finding linting errors. This is the expected behaviour and very useful when you chain commands. Do not add "|| true" at the end, if you plan to use these scripts in combination with others.  
I personally do not like to see 10 lines of useless "npm ERR!" every time I run my linter. I want to look at my linting errors in my terminal and determine where I need to fix stuff so this console output has no value to me in this scenario.

![](https://i.imgur.com/8alvUed.png)


## Wrapping Up
When you now finally run the linter scripts, you will find 2 things.  
Running `npm run lint` will show you a long list of minor issues. This is not because the basic template has errors but our custom set of rules has diverging paradigms (like my preference to not use semicolons, to enforce trailing comma and to use an indentation of 4).

![](https://i.imgur.com/QzKirSj.png)

Running `npm run fix` will make most of those errors magically disappear. Feels good, doesn't it?

![](https://i.imgur.com/d4lTTfv.png)

The remaining error is a missing return type so let's fix this real quick and go home. App() is returning a React Element so we can simply tap into React.ReactElement and add this as the return type of the App Component.

![](https://i.imgur.com/1sVdJ9B.png)


## Conclusion
We've started a new project with a typescript template, added linting and eslint typescript support and adjusted our rules as we seemed fit. We also included Prettier's auto fixing ability and reached the goal of today's journey.

In our next sessions, we will have a look at some basic solutions for navigation, state management and project file structure.
