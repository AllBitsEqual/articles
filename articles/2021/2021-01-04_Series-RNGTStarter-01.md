<!--
	title: "SERIES: React Native - Step by Step - Working with Typescript"
	description: "In the first part of our React Native - Step by Step series we will look at how to start a new project with Expo and Typescript and talk a bit about the how and why. Grab your coffee, relax and strap in for a fascinating journey."
	author: "Konrad Abe (/)AllBitsEqual)"
	published_at: 2021-01-04 08:00:00
	// header_source: (Link, falls du den Header von einer Seite hast)
	header_image: header.jpg
	categories: "react-native typescript best-practise series"
	canonical_url:
	language: en
-->  
# SERIES: React Native - Step by Step - Working with Typescript

**In the first part of our *React Native - Step by Step* series we will look at how to start a new project with Expo and Typescript and talk a bit about the how and why. Grab your coffee, relax and strap in for a fascinating journey.**

<!-- TODO: HERO IMAGE -->  

## Typescript - WHY?
There are a lot of strong sentiments and feelings around typescript that go in both directions. The die hard fans swear by the fact that, thanks to typescript, their code is now less buggy, more stable, easier to reason about and easier to extend, share or collaborate on. On the other side of the ring we have those who deem typescript a nuisance, something that creates a lot of unnecessary overhead and in some cases even duplication while not actually fixing any issues at runtime.

I will not start a long-winded discussion about the pro's and con's and keep it short. I've decided to work with typescript in my own projects for a lot of reasons.
* I am working with typescript in my 9 to 5 job on a regular basis
* I really like the additional information, my IDE can draw from strongly typed code
* Many of my reusable code bits will at some point be put into separated repositories and distributed via NPM

I, too, have had my fair share of "Du'h" moments when running into issues with stuff my compiler/linter did not like because I've been to vague or simply had not typed something at all because it looked too complex and had hindered me at the time I was writing it. I've adopted a routine that allows me to `// @ts-ignore` something for the time being and running a check before my commit to make sure I think twice before allowing an unchecked piece of code into my repositories.

There are moments when you might puzzle about WHY the typescript compiler does not want to allow a certain piece of code or accept a type that looks like it should be ok and I've come to the realisation that in most cases I was about to do something that COULD go wrong at some point in the future and bite me in my backside. *Yeah, sure. I, as the developer, know fully well that I will never call this function at a point where my props are still undefined... or do I?* Adding null checks and other safeguards has by now become a habit I am happy to live with. I've seen enough times in enterprise projects where something breaks because, for some unforeseen reason, a server response was empty or had a missing or wrong type. It was not our code that was wrong but putting too much trust in the contracts of APIs and not making sure on both ends that things are as expected can lead to some nasty and sometimes hard to track down bugs.

**Long story short:** I've learned to stop worrying and love the compiler. I've learned a lot by trying to understand WHAT typescript was trying to tell me and I think it made me a better programmer by NOT TRUSTING the Typescript Compiler to prevent everything.

## Getting Started with React Native and Typescript
Starting a new project could not be easier. As in earlier projects, we will again start with Expo and we will stay in the managed workflow to keep things nice and easy.

I assume you have worked with NPM and GitHub in the past and at least know the basics. To get started with React Native using EXPO you need to install a command line tool called expo-cli and it is recommended to install it globally using the -g flag.
At the time of writing, I'm using expo-cli@4.0.17 but there won't be any breaking changes for later versions of 4.x when you are reading this in the future.
```
npm install -g expo-cli
```  
Using Expo to initialise our new project, we have to typescript templates to choose from. For our case we will build everything from scratch step by step so we will choose the empty typescript template. The following command, when entered from your desired location via command line, will then create a new folder by the name you provide in the init and start populating it with the template files you choose from the following menu.
```
expo init yourProjectName
```






## TODO:


<!-- TODO: THEME 1 IMAGE -->  
<!-- TODO: THEME 2 IMAGE -->  
<!-- TODO: THEME 3 IMAGE -->  
<!-- TODO: WRAP UP IMAGE -->
