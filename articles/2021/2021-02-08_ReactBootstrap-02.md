# Bootstrapping React 2 - Styles, Stories and Tests

## Now again, with styles

Having a dynamic and interactive UI library as part of a project can be a real asset. Storybook has proven it's worth for us many times over, allowing us to show UI elements on their own with adjustable attributes and toggles in an intuitive web UI. Your QA and Design/Concept teams will love it.

To install Storybook, you can simply run their auto installer with the following line.

```bash
npx sb init
```

This will install all required packages, add 2 new scripts to our package.json and create a folder with demo stories and assets. 
We will immediately delete all those components, stories and css in src/stories and only keep the assets for now. With our linting rules, those files would do more harm than good.

> If you wanted to see how those demo files look like in Storybook, you could add a lint disabling comment like `/* eslint-disable */` in all those new .tsx files and then run `npm run storybook` to verify that everything is in order. You should see the following demo page, when you open the URL shown in your command log.

![](https://i.imgur.com/IYJY5Cl.png)
*Storybook Demo with the prepared files*

After getting rid of the prepared demo files, let's create a new button for ourselves and use styled-components to make them 'pretty'.

```bash
npm i -S styled-components @types/styled-components
```

We will add a theme and a ThemeProvider to our project and store the css for our components on component level but in a separate file. This means we need to hop through an additional hoop but we can simply separate our styles from our components and have a clearer separation of concerns. Additionally, we'll use the atomic design principle where we gather all our ui components in folders for atoms, molecules, organisms, templates and pages. This is a bit deeper in the file structure than I would prefer but we've got all our files in one place, sorted and orderly. For the scalability of our setup, this way has proven itself for our teams. 