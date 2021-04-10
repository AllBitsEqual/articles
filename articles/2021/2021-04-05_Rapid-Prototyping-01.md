---
tags: Slice of Work, Prototyping, Testing, Game Development
---
<!--
	title: "Rapid Prototyping, test often and fail early"
	description: "As the creator of games, it is often hard to see or judge if the core idea is fun to play and within the realms of reasonable complexity. Prototypes and playtests will allow you to dodge bullets and maybe even pull the plug before it is too late."
	author: "Konrad Abe (AllBitsEqual)"
	published_at: 2021-04-05 08:00:00
	header_image: "https://i.imgur.com/CP7szZ4.jpg"
	categories: "Slice of Work, Prototyping, Testing, Game Development, Proof of Concept"
	canonical_url: "https://allbitsequal.medium.com/rapid-prototyping-test-often-and-fail-early-31cebb025082"
	language: en
-->
# Rapid Prototyping, test often and fail early

**As the creator of games, it is often hard to see or judge if the core idea is fun to play and within the realms of reasonable complexity. Prototypes and playtests will allow you to dodge bullets and maybe even pull the plug before it is too late.**

![](https://i.imgur.com/CP7szZ4.jpg)


## Why Prototypes and 'Proofs of Concept' (PoCs) matter?
I'm coming from a Web Dev background. When you are building a regular website and you are familiar with your tools and languages, this is a pretty straight forward task. Having spent my recent years working on a complex Product Information Management (PIM) Web App, I've seen both ends of the spectrum of complexity and over the years, simple PoCs and Prototypes have become a staple tool in my workflow.

### A Proof of Concept with new toys
When deciding on the choice of your web/app stack, it can provide valuable insights and experience to start a small project with the new framework or library, learn the ropes, talk to the community, see the common questions on StackOverflow and run into a few dead ends yourself. 

> We recently evaluated VueJS for our company projects and no amount of reading reviews and documentation would have given us the same level of insight as simply building a small mini web app with it and finding out how to solve the same problems we know with a new tool.

Most libraries and frameworks have easy to follow tutorials that will allow you to create something simple in no time and expand on what you have learned by adding more features after finishing the original lessons.

Not sure if two things work well together? Maybe don't try to shoehorn both into an existing codebase at the same time and instead build a fresh app/tool/program using both on a clean slate. A good example of this? Try setting up a React Project using typescript exclusively with redux, routing, local storage and full linting for your workflow.

**Bonus Points:** By using clean slate projects, it is easy to share some or all of your code online when asking for help from the community, which is a big factor and really appreciated by those kind souls taking on your beginner questions. No need to delete sensitive client data or in-house code.


### A Proof of Concept with new challenges
When a customer comes to you (or your project lead) and requires a new solution for something you have not done before, it might be a good idea to hack together a rough mini prototype as part of the feasibility check.

Create a dirty mockup in any tool you are familiar with and develop the known user journey required for the new feature. This will allow you to identify UX/UI issues early on, give you something to share between devs, art/concept and customer to develop the final solution and prevent communication issues when description and understanding of the tasks at hand do not match.

### A/B Testing
Whether you are working on new features or making changes to existing ones, it is often a good approach to make a small test and run it by your peers or a small test group to see how it performs. In regards to user experience, performance and conversion rates are often not logical on a surface level unless you fully understand all the emotional and psychological elements of your change and their effect on the user.

Running a quick A/B test with a PoC of your change before you develop the final assets, start translations and run your full QA can save you a lot of time and in some cases even money.

![](https://i.imgur.com/Mr6D8mr.jpg)

## How does this translate into game development?
I will go out on a limb here and claim that a large percentage of people trying to build games do not have the slightest grasp on what makes something fun.

This is an issue I've seen a lot with both young novel authors and inexperienced game developers. Their ideas, concepts and characters sound great in their heads but when presenting them to critics or consumers, those ideas often fall short for a variety of reasons.

Oftentimes, the problem is that taking two or more ideas from different games and simply glueing them together might not be as fun as it seems on paper. Even a full-on copy of a successful game can fall short for a variety of reasons.

### What is fun
I'm not sure if you are familiar with the book **"A Theory Of Fun"** by Raph Koster. If not and you are planning to make entertaining games, I can only recommend you to buy it and read it asap... Fun is a very interesting term because it will be interpreted differently by almost everyone you talk to.

Take me for example. I'm a person that has fun writing spreadsheets with costs, values and production time to maximise the output of a little fantasy store on my smartphone. The game itself might be fun but the metagame of finding more efficient ways to do something is fun for me all the more while the next person might see this as tedious and cumbersome or downright boring.

When you are trying to find a "fun" concept for a game, you should test this basic concept as soon as possible and in the most simple way possible.

### Keep It Simple
If you are working on a card game (analogue or digital), make a paper prototype. If it is a board game, guess what, make a paper prototype. Got a digital tool that allows you to easily define cards and rules and such, use that.

Try to find the most basic, simple and easy to use tool and level of abstraction and see if it works. For games that don't depend on 3d graphics, platformer action or anything like that to work, try to completely ignore the design part and make a text-based version. This works well for most RPGs, strategy games, idle games, sim/management games and their likes.

If you don't plan to make your prototype public in any way and plan to use it solely for internal tests but your game absolutely needs graphics/assets, it's even ok in my book to use copyrighted or unlicensed material/assets from other games. **Just make sure to never use any of those in any form in a published piece of work, be it commercial, open-source or freeware.**

I happen to know that some board game developers used other existing board games during their early planning and testing phase and simply stuck small slips of paper to the miniatures with labels or values.

![](https://i.imgur.com/EVUs12z.jpg)

### Play, Watch and Ask
Ask your friends and family to play with you or, even better, **for** you. Explain the rules and let them play. Listen to their questions, make notes of their reactions, roadblocks and issues with the "gameplay" try to interrupt them as little as possible until they are done. Wait for them to finish or until you got enough notes and ask them what was fun about it and what was not.

Having other people play your game comes with two benefits. Feedback, as previously mentioned, and also the bonus of feeling good about your work when they sincerely enjoyed it. This can be really rewarding as a developer/game maker.

## What should you test?
Again, there are two things to keep in mind:
1) Only because you like the concept or maybe even enjoy the execution, this does not need to hold true for a larger audience.
2) A feature that is fun in one game can fall short in another. Context, context, context.

While a crafting system might be really engaging in one game, it might not be solemnly the crafting itself but the setting, the environment, how it is tied into the game, the lore and the interaction with other players or other aspects of the game itself.

When you try a variation of an existing concept or, even better, a very rare and completely original idea, it might take you countless iterations until you "feel" the idea and enjoy it. I can only recommend to you to search the web for some Game Post Mortem articles or videos and watch closely.

Changing and finding the right theme can make a feature "click" with the player. Switching the setting, premise or end goal (even if it is only affecting the visuals, wording etc. and not touching any of the game mechanics) can make or break a game.

![](https://i.imgur.com/ngT1DIT.jpg)

### Is the basic idea fun?
Find out who your target audience is and try to see if you can match the core mechanic of your game with their expectations. Create a basic game loop (not in the "engine" sense, more like the flow of the game's core mechanic) and see if it is enjoyable to perform.

### Is the basic idea fun repeatedly?
Can you replay (or continue to play) the game again and again without losing interest in it? Are there settings that change over time and are players well enough aware of these or does it feel repetitive for them? Are you using randomisation or map generation?

### Does the fun evolve vertically?
Does your idea allow for any type of "power scaling" and linear progression? Is it fun to be more powerful and progress along the vertical line? Is it maybe even the main goal?

### Does the fun evolve horizontally?
Does your idea allow for spreading/branching concepts that increase complexity while bringing new engagement into play?


## What if early prototyping is hard to do
There are many cases where your concept is based on a working game with a twist. Building it from the ground up might not be feasible for an early prototype.

In some of those cases, modding is a good way to bridge this gap. Take an existing game with similar gameplay that is moddable and try to add your twist/idea/feature to it. Keep in mind that if you publish the mod, your own brainchild is out there and issues with copyright and ownership can follow (always a hot topic with the big game publishers) but as long as you only use it in-house... no harm's done, eh?



## What if early prototyping is impossible
There will be some cases where the core concept is so special and complex, that you simply can't mock it. I'm sorry but there isn't much I could say to you that would help you out here. You will probably have to rely on market research, polls, interviews with potential players and such to gauge the value and viability of your idea before you start development. 

Just keep in mind that prototypes and Proofs of Concept are not limited to early testing. You should definitely use them later on as well when the core mechanics of your project are already implemented. In many cases, small variations can make a large difference in the feel of the game and respectively its user experience.


## Wrapping Up
With the full spectrum of possible game types, genres and all it would be impossible to touch on all aspects you might run into but I hope you got the global idea here.

The mantra of "fail fast, fail often" might be a terribly overused and beaten-to-death buzzword term but my advice is to "**try fast and try often**". You will most likely fail with some of those attempts but the earlier you find out via a prototype that an idea does not work, the faster you can pull the plug or make critical changes and change your path.

If a prototype fails or reaches the end of its life cycle, salvage it, take notes of your learning and discard the rest.


Images:
- Photo by <a href="https://unsplash.com/@christopherphigh?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Christopher Paul High</a> on <a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
- Photo by <a href="https://unsplash.com/@alvarordesign?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Alvaro Reyes</a> on <a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
- Photo by <a href="https://unsplash.com/@jacielmelnik?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Jaciel Melnik</a> on <a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
- Photo by <a href="https://unsplash.com/@nika_benedictova?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Nika Benedictova</a> on <a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
  