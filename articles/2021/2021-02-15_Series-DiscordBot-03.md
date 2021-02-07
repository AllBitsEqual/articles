---
tags: Discord, Javascript, DiscordJS, Chatbot, Modules
---
<!--
	title: "Build A Bot (DiscordJS) - A Bot Factory and Revealing Module Design Pattern"
	description: "This time we will turn our simple bot into a bot factory, allowing us to use it to spawn multiple bots with different congigs, using the well established Factory and Revealing Module design patterns."
	author: "Konrad Abe (AllBitsEqual)"
	published_at: 2021-02-8 08:00:00
	header_image: "https://i.imgur.com/BebCIWs.jpg"
	categories: "javascript discord discordjs bot chatbot series"
	canonical_url: ""
	series: "Build A Bot (DiscordJS)"
	language: en
-->
# Build A Bot (DiscordJS) - A Bot Factory and Revealing Module Design Pattern

Last time we made our commands more flexible and improved the structure of our code To be more readable and scalable. This time we will turn our simple bot into a bot factory, allowing us to use it to spawn multiple bots with different congigs, using the well established Factory and Revealing Module design patterns.

![](https://i.imgur.com/BebCIWs.jpg)

Things you should know for this part include scope and closure of Javascript functions, es6 basics (const, let and fat arrow functions) 



First of all, let's clean up a bit. We're going to throw out a lot of our old code as the config, token and such will be supplied when the factory function is used to create a new bot instance.

![](https://i.imgur.com/XWzULX6.png)

Then we will wrap all of our remaining code in our factory function and call it `createBot`. This means everything within this function will be bundled together if we return the bot object. We don't want that, do we? 

To make sure we control what is visible and accessible from the outside, we won't return the bot object but only the functions that need to be usable. In our case right now this is only the `bot.load()` function.

Add a return {} to the createBot function and only define one attribue as `start()` that will call the load function.

This is often referred to as the Reveal Module Pattern

![](https://i.imgur.com/fHXD7DH.png)

The last thing to add now is the module.export with our createBot function.

![](https://i.imgur.com/auieR0W.png)

> You might have noticed that I did not use the correct indentation here. To show you a better screenshot, I simply kept the indentation even though I wrapped the code in a function, so that the diff would show exactly what I changed.
>
> I will correct this before we move on.


As we get our config handed in now, we need to make a few adjustments. First of all we need to rewrite our `bot.load()` function as follows.

The new `load()` will expect a config object with mandatory (token, name) and optional (prefix and later others) attributes and attempt to merge them with our configSchema in `loadConfig()`. Our old cold will be passed into `loadConfig()` as a callback here.

```javascript
    // Load the bot
    bot.load = function load(config) {
        // Set up some properties
        this.config = {}

        // Load config, load modules, and login
        this.loadConfig(config, () => {
            this.log('Loading commands...')
            Object.keys(botCommands).forEach(key => {
                this.commands.set(botCommands[key].name, botCommands[key])
            })
            this.log('Connecting...')
            this.client.login(this.config.token)
        })
    }
```

In `loadConfig()` we will check if our initial config is there and contains a token. If either check fails, we will throw an error, otherwise we will merge the initial config with our configSchema and attach it to our bot before we execute the callback code.

```javascript=
    // little helper to keep the code clean
    const has = (obj, prop) => Object.prototype.hasOwnProperty.call(obj, prop)

    bot.loadConfig = function loadConfig(config, callback) {
        this.log('Loading config...')
        try {
            if (!config || !has(config, 'token')) {
                throw Error('Config or token are missing.')
            }
            this.config = {
                ...configSchema,
                ...config,
            }
            callback()
        } catch (err) {
            this.log(`Error loading config: ${err.message}`)
            this.log('Please fix the config error and retry.')
        }
    }
```

We need to make a small adjustment to grab our prefix from the new config and then we're done here.

![](https://i.imgur.com/WvEzArX.png)

---

With our factory in place, it's time to try it out. For this part to work properly, you would need to head over to my first installment of this series and create a second bot but you can as well just make an array of 1 bot and go with it pretending to make better use of it.

Create a new index.js at the root of the project. This is where we will import our factory, load our .env variables containing our tokens and add the configs for our bots.

```javascript
// File: index.js
require('dotenv').config()
const BotFactory = require('./src/index')

const { TOKEN } = process.env

const abe = BotFactory.createBot({
    token: TOKEN,
    name: 'AllBotsEqual',
    prefix: '!',
})

abe.start()

```

If you have only one bot, it's super easy to start it. Create the bot using the factory with a short config and run the `start()` that is publicaly available.

You are done!

If you want to define a group of bots to run from your project, you can simply do so in the config.


```javascript
require('dotenv').config()
const config = require('./config.json')
const BotFactory = require('./src/index')

const { bots } = config

bots.forEach(botConfig => {
    const { name, token, prefix} = botConfig
    const bot = BotFactory.createBot({
        token: process.env[token],
        name,
        prefix,
    })

    bot.start()
})
```

If you name your tokens in your .env file accordingly, you can map them in your config file like this.

```json
{
  "bots": [
    {
      "name": "AllBotsEqual",
      "token": "TOKEN_ABE",
      "prefix": "!"
    },
    {
      "name": "Scout",
      "token": "TOKEN_SCOUT",
      "prefix": "$"
    }
  ]
}
```

![](https://i.imgur.com/X0Vpu42.png)

![](https://i.imgur.com/exOjx8Y.png)


With one final adjustment to our package.json to switch to the new index.js file we are now done and can spawn as many bots as we like (and have registered with Discord)-

![](https://i.imgur.com/hc5VQHz.png)

> One word of advice though. 
> 
> If you are running bots on your own server, this way of creating multiple bots is ok. If you plan on hosting this bot on many servers to be used by many people, grouping more than one bot together is not adviseable because it can lead to performance issues. 
> 
> In that case it would be better to reuse the code for the factory (for example by submitting it to and loading/importing it from npm).

