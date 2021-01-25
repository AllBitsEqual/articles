<!--
	title: "Build A Bot (DiscordJS) - A scalable setup with command modules"
	description: "Today we will clean up our central index.js file, make it more readable and scaleable and move all our existing commands (and a new one) to a separate folder for import."
	author: "Konrad Abe (AllBitsEqual)"
	published_at: 2021-01-25 08:00:00
	header_image: "https://i.imgur.com/LdxxqzS.jpg"
	categories: "javascript discord discordjs bot chatbot series"
	canonical_url: "XXXXXXXX"
	series: "Build A Bot (DiscordJS)"
	language: en
-->
# Build A Bot (DiscordJS) - A scalebale setup with command modules

## Last week on "Build A Bot"
In our last session, we have created a functional discord bot with some basic commands, a small config and linked everything to our discord application/bot setup in the discord developer portal using a generated token.

Today we will clean up our central index.js file, make it more readable and scaleable and move all our existing commands to a separate folder for import. When all else is done, we will also start expanding the functionality of our bot by adding a more complex command to play with on our test server and give you a better understanding of the wide range of functionality, tools and commands possible with discord bots.

![](https://i.imgur.com/LdxxqzS.jpg)

If you want to grab or compare with the code from the last session, here's the [GitHub link to the respective tag](https://github.com/AllBitsEqual/allbotsequal/releases/tag/v0.0.1).

## Cleaning up
First of all, we will replace our simple bot client instance with a more elaborate bot object. Within this new object, we will mirror our discord.Client() as the client and as we are planning to expand our logging in the future, we are hiding our interim console.log behind bot.log with the comment to disable eslint for the no-console rule as before. That way we can use this for our logging and when we later introduce a better logger, we can do it right there.

```javascript
// File: src/index.js
require('dotenv').config()
const discord = require('discord.js')
const config = require('../config.json')

const { TOKEN } = process.env
const { prefix, name } = config

// Define the bot
const bot = {
    client: new discord.Client(),
    log: console.log, // eslint-disable-line no-console
}
```
For comparison, I've included the diff to our old file. At the end of each step, you will find a GitHub link to the commit/changes to compare with your own code.

![](https://i.imgur.com/SJ3xaf5.png)

Next thing on our list is to add some functions that will be triggered by the event handlers as be the backbone of our bot. Right now this might seem to be "overkill" or premature optimisation but if we do this now, the code will be easier to read AND easier to extend and build on.

This is basically nothing new, it's just our load() function and "on ready" event listener from last week, using our new structure.

```javascript
/*
 * Define all the core functions for the bot lifecycle
 */

// Load the bot
bot.load = function load() {
    this.log('Connecting...')
    this.client.login(TOKEN)
}

// Fired on successful login
bot.onConnect = async function onConnect() {
    this.log(`Logged in as: ${this.client.user.tag} (id: ${this.client.user.id})`)
}
```

We will do the same with our "on message" event listener code. Right now we won't change a single line of code within this section but we will wrap it in a function before we bind it to the actual event listeners.


![](https://i.imgur.com/7Lj4wlJ.png)


```javascript
// Check and react to messages
bot.onMessage = async function onMessage(message) {
    /*
     * THIS IS WHERE OUR OLD CODE REMAINS
     * => if ping
     * => if no prefix
     * => if who
     * => if whois with/without mention
     */
}

/*
 * Register event listeners
 */
 
bot.client.on('ready', bot.onConnect.bind(bot))
bot.client.on('error', err => {
    bot.log(`Client error: ${err.message}`)
})
bot.client.on('reconnecting', () => {
    bot.log('Reconnecting...')
})
bot.client.on('disconnect', evt => {
    bot.log(`Disconnected: ${evt.reason} (${evt.code})`)
})
bot.client.on('message', bot.onMessage.bind(bot))

// start the bot
bot.load()
```

As you see we are using simple log calls for all sorts of error states and issues while we bind our onConnect and onMessage function to their respective event handlers.

The last line is really important as that is the line that actually calls our bot once everything else is defined and set up.

For a cleaner separation in our file we now have the following order:
- imports
- setup
- functions
- event handlers
- the call to the load function

Running `npm start` on the command line will boot our bot like it did last time. So far so good.

![](https://i.imgur.com/jP5ehOf.png)

[GitHub Commit](https://github.com/AllBitsEqual/allbotsequal/commit/2bfb4a8a7f7dbc148a2a48d1e414381332fa83d7)

## Extracting our command logic
As you see, even with the basic setup, our index file is already close to 100 lines long and we should try to keep our files both as short as possible AND as focused as possible. With every new command that we add to the bot, this file would get more and more verbose so let's move all those existing commands to a new folder and import them from there.

Under src/ create a new folder called "commands" and add new, empty files for our commands and a central index.js file.

```
yourProject/
    src/
        commands/
            index.js
            ping.js
            who.js
            whois.js
        index.js
...
```

The ping is, again, the easiest case. Simply create a module.exports object with name, description and the execution of our command.
```javascript
// File: src/commands/ping.js
module.exports = {
    name: 'ping',
    description: 'Ping! Pong?',
    execute(message) {
        const delay = Date.now() - message.createdAt
        message.reply(`**pong** *(delay: ${delay}ms)*`)
    },
}
```

Moving on to our "who" command, we run into the first issue. We need to import the config again to have access to the name variable.

```javascript
// File: src/commands/who.js
const { name } = require('../../config.json')

module.exports = {
    name: 'who',
    description: 'Who is this helpful bot?!',
    execute(message) {
        message.channel.send(`My name is ${name} and I was created to serve!`)
    },
}
```
### Importing to export
Repeat the same process for the "whois" command and then open the new src/commands/index.js file. We need to import all our modules and combine them in one object that we will use in our main bot code.

```javascript
// File: src/commands/index.js
const ping = require('./ping')
const who = require('./who')
const whois = require('./whois')

module.exports = {
    ping,
    who,
    whois,
}
```

With this in place, we can now import all commands in our main file and add them to our bot. To do so, we will create a new collection from via `new discord.Collection()`.

```javascript
// File: src/index.js
require('dotenv').config()
const discord = require('discord.js')
const config = require('../config.json')
const botCommands = require('./commands') // <-- this is new

const { TOKEN } = process.env
const { prefix } = config

// Define the bot
const bot = {
    client: new discord.Client(),
    log: console.log, // eslint-disable-line no-console
    commands: new discord.Collection(),   // <-- this is new
}
```

In our bot.load function we will add a new step before logging our bot into the discord servers and create a new set in our collection for each command we have.

```javascript
// Load the bot
bot.load = function load() {
    this.log('Loading commands...')
    Object.keys(botCommands).forEach(key => {
        this.commands.set(botCommands[key].name, botCommands[key])
    })
    this.log('Connecting...')
    this.client.login(TOKEN)
}
```

The last thing to do in this step is to replace the old commands in our onMessage function and add our new and shiny collection to it. There is a minor caveat (or change) right now but I'll explain it after you had a look at the code.

```javascript
// Check and react to messages
bot.onMessage = async function onMessage(message) {
    // ignore all other messages without our prefix
    if (!message.content.startsWith(prefix)) return

    const args = message.content.split(/ +/)
    // get the first word (lowercase) and remove the prefix
    const command = args.shift().toLowerCase().slice(1)

    if (!this.commands.has(command)) return

    try {
        this.commands.get(command).execute(message, args)
    } catch (error) {
        this.log(error)
        message.reply('there was an error trying to execute that command!')
    }
}
```

What is all this code, you might ask? Well, let's see. First of all, we still check for our prefix. Then we split the message into an array and store that as our args. This will be handy later on when we build commands such as `!tag add <tag name> <tag message>`.

Then we shift() the first part out of that array as our command (mutating our args array), strip it from the prefix. If we can't find the command in our command list, we can exit directly. Otherwise, we can attempt to execute the command from the collection and to be extra safe here, we wrap that in a try/catch.

> When writing this part of the tutorial I ran into the issue of the missing "name" for the !who command and luckily the try/catch error directly helped me identify the issue and still keep the bot running. I would otherwise have seen a very angry node error message about an unhandled exception.

### What was the caveat?
Our ping will now also require the prefix. There would have been multiple possible solutions for this issue but none of them felt clean and as I do not have this bot deployed anywhere yet, I can simply change this right now. `ping` is now `!ping`... 

## Adding a default config

Previously, when we added the ping and who/whois commands, we only use the message parameter. We've just added the "args" array too but for allowing our functions to be more flexible and have better integration with discord, let's add our bot object to the command handler as well.

**Why?** Because we can define stuff like our default colours for user feedback (success, error etc.), variables like the bot "name" field we were missing earlier and much more in a config attribute and access those values where we need them. This will help us make adjustments later and prevent redundant code and settings by keeping those values in a central place.

So let's make another change to the src/index.js by adding default colours to the bot settings and adjusting our command execution call to pass in the bot object as well.

```javascript
// File: src/index.js line 7 ff
const { prefix, name } = config // add the name again

// Config
const configSchema = {
    name,
    defaultColors: {
        success: '#41b95f',
        neutral: '#287db4',
        warning: '#ff7100',
        error: '#c63737',
    },
}

// Define the bot
const bot = {
    client: new discord.Client(),
    log: console.log, // eslint-disable-line no-console
    commands: new discord.Collection(),
    config: configSchema, // add the new config to our bot object
}
```

With this done, simply add the bot to the command handler execution.

```javascript
// File: src/index.js line 57 ff
    try {
        this.commands.get(command).execute(message, args, bot) // added bot here
    } catch (error) {
        this.log(error)
        message.reply('there was an error trying to execute that command!')
    }
```

## Finally, a new command - roll the dice
As a fun exercise, we will add a `!dice` command that will let the user choose a number and type of dice and have the bot roll them.

I've previously written a dice function called `getDiceResult()` as an exercise. I've included and adjusted it to generate the results and texts we need to send a nice and well-formatted message into the chat. For reference, here is the schema of the return value of said function.

```javascript
const { 
  type,         // (string) "success" | "error"
  title,        // (string) title of the embedded message
  fieldName,    // (string) description of the result or error
  fieldContent, // (string) roll result or error message
  rest          // (array, optional) the rest of the message bits from args
} = getDiceResult(args)
```

The really interesting part in the new command is the embedded message provided by discordJS. There is a lot of stuff you can add to an embed and there are even multiple ways to achieve the same result when defining the fields ([read the official docs](https://discordjs.guide/popular-topics/embeds.html#using-the-richembedmessageembed-constructor)) but for now, we will restrict ourselves to the title, colour and content fields.

```javascript
// File: src/commands/dice.js
const discord = require('discord.js')

const getDiceResult = args => {...} // my dice function, hidden for readability

module.exports = {
    name: 'dice',
    description: 
        `Roll a number of dice, either with no argument for 1 d6, ` +
        `one argument for a number of dice between 1 and 10 or with 2 arguments ` +
        `to define the dices' sides. (2, 3, 4, 6, 8, 10, 12, 20, 100)`,
    async execute(message, args, bot) {
        // run user input through dice function to get formatted results and feedback
        const { type, title, fieldName, fieldContent, rest } = getDiceResult(args)
        // create the embedded message
        const embed = new discord.MessageEmbed()
            .setTitle(title) // The title of the discord embedded message
            .setColor(bot.config.defaultColors[type]) // either "success" or "error"
            .addField(fieldName, fieldContent) // our dice results or error message
        // all additional/optional text the user entered after the params
        if (rest && rest.length) {
            embed.addField(`You added the following: `, rest.join(' '))
        }

        message.channel.send({ embed })
    },
}
```

This command allows the user to use different combinations of the command and arguments. The following 4 patterns are valid:
- !dice
- !dice [1-10]
- !dice [1-10]d[2, 3, 4, 6, 8, 10, 12, 20, 100]
- !dice [1-10]d[2, 3, 4, 6, 8, 10, 12, 20, 100] "optional message"

Let's look at the getDiceResult function in detail. We pass in the args and receive an object with strings but what happens inside?
If you read the comments below, you will see that we try to get the count of "rolls" and type of "sides" of the command with some defaults, check them for our ruleset and then calculate the result.

If the user passes in an invalid argument, we generate an error response and cancel the execution.

```javascript
const getDiceResult = args => {
    // get the param or default to "1d6"
    const [diceParam = '1d6', ...rest] = args
    // split rolls and sides when applicable with fallback
    const [rolls = 1, sides = 6] = diceParam.split('d')

    // check if rolls and sides are integer
    const intRolls = Number.isNaN(parseInt(rolls, 10)) ? 1 : parseInt(rolls, 10)
    const intSides = Number.isNaN(parseInt(sides, 10)) ? 6 : parseInt(sides, 10)

    // check if rolls and sides are within predefined rules
    const safeRolls = intRolls >= 1 && intRolls <= 10 ? intRolls : 1
    const safeSides = [2, 3, 4, 6, 8, 10, 12, 20, 100].includes(intSides) ? intSides : 6

    // check if the calculated params match the original params of the user
    if (parseInt(rolls, 10) !== safeRolls || parseInt(sides, 10) !== safeSides)
        return {
            type: 'error',
            title: 'Invalid Parameter',
            fieldName:
                'Please specify either no parameter or add a dice count such as 1d6 or 3d12.',
            fieldContent: 'Please see "!help dice" for additional information.',
        }

    // roll the dice
    const results = []
    for (let i = 0; i < safeRolls; i++) results.push(Math.ceil(Math.random() * safeSides))

    // format the response
    return {
        type: 'success',
        title: 'Dice Roll Result',
        fieldName: `You rolled ${safeRolls}d${safeSides}`,
        fieldContent: `[ ${results.sort((a, b) => a - b).join(', ')} ]`,
        rest,
    }
}
```

To check if our bot handles all cases as expected, here are a few variations and their results.



![](https://i.imgur.com/qg1aY4C.jpg)



### Tracing back our steps

With this we are done with the new command (I know, we skipped the !help part today) but with the new config we made for the last part, we can return once again to the `!who` command file and make ONE final edit, getting rid of the additional import and instead using the bot param from the execution call.

```javascript
module.exports = {
    name: 'who',
    description: 'Who is this helpful bot?!',
    execute(message, args, bot) {
        message.channel.send(`My name is ${bot.config.name} and I was created to serve!`)
    },
}
```

## Wrapping up

We've cleaned up our central index file, created a clear separation of code sections based on their intent and introduced a command collection to handle all user input based on a set of imported commands from separate files. Furthermore, we've added a new config and prepared our user messages in a way that allows us to easily scan for keywords and parameters.

Next time I will guide you through the process of writing a scaleable and self-updating help command as well as adding our first user management/administration commands to make the bot a bit more useful.
