<!--
	title: "Build A Bot (DiscordJS) - Writing command modules and a scalebale structure"
	description: "XXXXXXXX"
	author: "Konrad Abe (AllBitsEqual)"
	published_at: 2021-01-25 08:00:00
	header_image: "XXXXXXXX"
	categories: "javascript discord discordjs bot chatbot series"
	canonical_url: "XXXXXXXX"
	series: "Build A Bot (DiscordJS)"
	language: en
-->
# Build A Bot (DiscordJS) - a scalebale structure with command modules

## Last week on "Build A Bot"
When we wrapped up last time, we had created a functional discord bot with some basic commands, a small config and linked everything to our discord application/bot setup in the developer portal using a generated token.

Today we will clean up our central index.js file, make it more readable and scaleable and move all our existing commands to a separate folder for import. When all else is done, we will also add a few more usefull commands to play with on our test server and give you a better understanding of the wide range of functions and commands possible with discord bots.

## Cleaning up
First of all we will replace our simple bot client instance with a more elaborate bot object. Within this new object, we will have a client for the discord.Client() and as we are planning to expand our logging later on, we are hiding our temporary console.log behind bot.log with the eslint-disable for no-console. That way we can use this for our logging and when we later introduce a better logger, we can do it right there.

```javascript
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
For comparison I've included the diff to our old file. 

![](https://i.imgur.com/SJ3xaf5.png)

Next thing on our list is to add the functions that will be the backbone of our bot in the future. Right now this might seem to be "overkill" or premature optimisation but if we do this now, the code will be easier to read AND easier to extend and build on.

This is nothing new, it's just our load function and "on ready" event listener from last week, using our new structure.

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

We will do the same with our "on message" event listener code. Right now we won't change a single line of code within this section but we will wrap it a function before we bind it to the actual event listener and then we set up our event listeners.

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

![](https://i.imgur.com/jP5ehOf.png)

Running `npm start` on the command line will boot our bot like it did last time. So far so good.

## Extracting our command logic
As you see, even with the basic setup, out index file is already close to 100 lines long and we should try to keep our files both as short as possible AND as focused as possible. With every new command we add to the bot, this file would get more and more verbose so let's move all those existing commands to a new folder and import them from there.

Under src/ create a new folder called "commands" and add new files for our commands and a central index.js file.
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
```
const { name } = require('../../config.json')

module.exports = {
    name: 'who',
    description: 'Who is this helpful bot?!',
    execute(message) {
        message.channel.send(`My name is ${name} and I was created to serve!`)
    },
}

```

Repeat the same process for the "whois" command and then open the new src/commands/index.js file. We need to import all our modules and combine them in one object that we will use in our main bot code.
```javascript
const ping = require('./ping')
const who = require('./who')
const whois = require('./whois')

module.exports = {
    ping,
    who,
    whois,
}
```


With this in place we can now import all commands in our main file and ad them to our bot. To do so, we will create a new collection from via `new discord.Collection()`.


```javascript
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

The last thing to do in this step is to replace the old commands in our onMessage function and add our new and shiny collection to it. There is a caveat though and I'll explain it after you had a look at the code.
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

What is all this code, you might ask? Well, let's see. First of all we still check for our prefix. Then we split the message into an array and store that as our args. This will be handy lateron when we build commands such as `!tag add <tag bane> <tag message>`.

Then we shift the first part out of that array as our command, strip it from the prefix. If we can't find the command in out command list, we can exit directly. Otherwise we can attempt to execute the command from the collection and to be extra safe here, we wrap that in a try/catch.

> When writing this part of the tutorial I ran into the issue of the missing "name" for the !who command and luckily the try/catch error directly helped me identifiying the issue and still keep the bot running.

### What was the caveat?
Our ping will now also require the prefix. There would have been multiple possible solutions for this issue but none of them felt clean and as I do not have this bot deployed anywhere yet, I can simply change this right now.

_  

_  

_  

_  

_  

_  

_  

_  

_  

_  