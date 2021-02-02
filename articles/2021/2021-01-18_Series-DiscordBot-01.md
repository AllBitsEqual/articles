<!--
	title: "Build A Bot (DiscordJS) - Javascript Chatbots made easy"
	description: "Discord bots can be written in Javascript or Python and getting your first bot up and running is a great way to practice your Vanilla JS skills and have some fun. We will cover the whole process of registering the bot with Discord, a basic setup and how to bring it to your server."
	author: "Konrad Abe (AllBitsEqual)"
	published_at: 2021-01-18 08:00:00
	header_image: "https://i.imgur.com/bYXbzEg.jpg"
	categories: "javascript discord discordjs bot chatbot series"
	canonical_url: "https://allbitsequal.medium.com/build-a-bot-discordjs-javascript-chatbots-made-easy-bb31f6738a85"
	series: "Build A Bot (DiscordJS)"
	language: en
-->
# Build A Bot (DiscordJS) - Javascript Chatbots made easy

Discord bots can be written in Javascript or Python and getting your first bot up and running is a great way to practice your Vanilla JS skills and have some fun. We will cover the whole process of registering the bot with Discord, a basic setup and how to bring it to your server.

![](https://i.imgur.com/bYXbzEg.jpg)

## Prerequisites
There is not much that you need to start writing a bot for Discord using Javascript and while you could theoretically compose everything with copy and paste code snippets written by other people, I highly suggest some prior Javascript knowledge. Here is what you need apart from that.
* A Discord Account & (desktop or web) Client
* A Discord Server with "manage server" permission
* Basic Terminal KnowHow
* NodeJS version 10 or higher

## First Steps - Discord Developer Portal
To write and use a bot, you need to register it as a new application/bot user with your Discord account. Go to the [Discord Developers Portal](https://discordapp.com/developers/applications/) (sign in) and create a "New Application".

![](https://i.imgur.com/ufyc2FP.png)

Pick a name of your liking and continue to create the application. For my server AllBitsEqual, I will go with the wonderful pun name "AllBotsEqual" ... don't judge me!

![](https://i.imgur.com/ASp4fMP.png)

On the following page, you can add a short description, avatar image and see your ID, Key and Secret. Don't forget to save your changes, when you are done.

![](https://i.imgur.com/3GXNkyG.png)

Use the left sidebar navigation to go to the "Bot" section and click "Add Bot" to assign a bot user to your newly created application.

![](https://i.imgur.com/33p9pLt.png)

Ok, this was about the hardest part... we now have a Bot with a user ID, can grab the Token for later and define the basic permissions.

![](https://i.imgur.com/KkUV1mv.png)

To continue with the permissions, head over to the OAuth2 section, again using the left sidebar navigation.

From the first box, select the "bot" option. This will open up the second box below with the bot permissions where you can pick and choose what the bot should be able/allowed to do. For this tutorial you will need at least "Send Messages" and "Read Message History" but in later parts, we will add more functionalities including some moderator functionality.

![](https://i.imgur.com/CdYlmcG.png)

Copy the URL that has been generated with our bot ID and permissions when you are done selecting them. Your selection is part of the URL, encoded as the number after the permissions attribute.

When you enter this URL in your web browser of choice and are logged in with your discord user, you can pick the server you want to add the bot to (where you have the "manage server" permission) and "Authorise" it.

![](https://i.imgur.com/BhkWtBU.png)

You will see the list of permissions you just created and need to confirm it. When you are done confirming all confirmations, you should end up at this screen and be done with it.

![](https://i.imgur.com/Geuj5GV.png)

If you check your selected server now, you should see a message that your bot just joined the server.

![](https://i.imgur.com/QntLb1I.png)

## Project Setup
To get you started, I have prepared [a small setup with a few simple commands and the basics on GitHub](https://github.com/AllBitsEqual/allbotsequal/releases/tag/v0.0.1) to start your development with the most useful default tools. You can grab the code from my repository and put it in a new folder for your own project.

This project includes DiscordJS, the library we will be using for most of our actions and functionality on Discord, as well as a basic linter/prettier setup because who does not like clean, formatted and checked code.

As you need to store your super secure and private token somewhere, I also included the dotenv package that allows you to store and use untracked/unversioned environmental variables within your project. This will be the first thing to do after copying the repository above.

To install the included packages, run `npm install` at the root of your new project. Then add a .env file at the root level of your project (which is on the ignore list of our .gitignore file) and add the following line using the token you got from the Discord Developer Portal on the Bot section to replace "7074lly-n07-my-70k3n".

```bash
TOKEN=7074lly-n07-my-70k3n
```

## The initial code, diving into DiscordJS

Without changing a single line of code, you could now start the bot by either calling `node src/index.js` to execute the file or run the script from the package.json file `npm start` which basically does the same.

You will now see the bot as online on your server and your console should show this line with your bot's name and ID number.

![](https://i.imgur.com/Gp6LmgM.png)

> *A short side note: If you plan to configure and test the bot on a regular server with other users, it is advised to create an admin/mod only chat and to add the bot directly via channel permissions. That way your testing of commands will not annoy regular users.*

Let's break down the file src/index.js to guide you through be basics.

```javascript
require('dotenv').config()
const Discord = require('discord.js')
const config = require('../config.json')

const { TOKEN } = process.env
const { prefix, name } = config

const bot = new Discord.Client()
```

We are requiring the discord js and dotenv packages and import our config.json file. After getting a few values via destructuring of the .env and config.json files, we initialise a new bot object.


```javascript
bot.login(TOKEN)

bot.once('ready', () => {
    console.info(`Logged in as ${bot.user.tag}!`) // eslint-disable-line no-console
})
```

After handing our token to the login function on our bot object, we add a special "once" event listener for the ready event to notify us when the bot successfully launched and logged in. Our linter doesn't like the last line but it will have to endure this with blissful ignorance due to our line-disable comment.

The next thing to do is to tell the bot what he is supposed to do with messages he "reads" in channels he has access to. For this, we added another event listener waiting for events of the type "message".

> We're currently using several if/else statements. This is not the optimal way but enough for today. In our next session, I will explain the concept of a command handler in greater detail.

```javascript
bot.on('message', message => {
    // ping command without a prefix (exact match)
    if (message.content === 'ping') {
        const delay = Date.now() - message.createdAt
        message.reply(`**pong** *(delay: ${delay}ms)*`)
        return
    }

    // ignore all other messages without our prefix
    if (!message.content.startsWith(prefix)) return

    // let the bot introduce itself (exact match)
    if (message.content === `${prefix}who`) {
        message.channel.send(`My name is ${name} and I was created to serve!`)
        return
    }

    // user info, either call with valid user name or default to info about message author
    if (message.content.startsWith(`${prefix}whois`)) {
        // if the message contains any mentions, pick the first as the target
        if (message.mentions.users.size) {
            const taggedUser = message.mentions.users.first()
            message.channel.send(
                `User Info: ${
                    taggedUser.username
                } (account created: ${taggedUser.createdAt.toUTCString()})`,
            )
        } else {
            // default to sender if no user is mentioned
            const { author } = message
            message.reply(
                `User Self Info: ${
                    author.username
                } (account created: ${author.createdAt.toUTCString()})`,
            )
        }
    }
})
```

This might be a bit more to digest but I've tried to add a few really basic scenarios to give you a broad understanding of what we have access to. Let's go through those four scenarios one by one again.

### 1) ping
```javascript
if (message.content === 'ping') {
    const delay = Date.now() - message.createdAt
    message.reply(`**pong** *(delay: ${delay}ms)*`)
    return
}
```

The first part listens to all messages that are exactly "ping" with nothing more and nothing less. The bot reacts to those by sending a reply to the message author by using the reply function. Fir this it calculates the time passed between the "message sent" timestamp (createdAt) and the current time in milliseconds and posts this in his reply as a pong.
By using `return` here, we skip all the other code since our condition is met already. Time's a wastin'.

![](https://i.imgur.com/VgjZek5.png)

### 2) check the prefix
```javascript
if (!message.content.startsWith(prefix)) return
```
The next line simply checks all other messages for the prefix we've defined in the config.json, which is currently set to "!". All messages that don't have our prefix (or were "ping") can be ignored.

### 3) !who am I
```javascript
if (message.content === `${prefix}who`) {
    message.channel.send(`My name is ${name} and I was created to serve!`)
    return
}
```
If the bot encounters a message matching (exactly) `!who`, he will answer with a short message containing his own name (again from the config) and a flair text we've written.

![](https://i.imgur.com/PS8Lqfq.png)

### 4) !whois asking?
```javascript
if (message.content.startsWith(`${prefix}whois`)) {
    // if the message contains any mentions, pick the first as the target
    if (message.mentions.users.size) {
        const taggedUser = message.mentions.users.first()
        message.channel.send(
            `User Info: ${
                taggedUser.username
            } (account created: ${taggedUser.createdAt.toUTCString()})`,
        )
    } else {
        // default to sender if no user is mentioned
        const { author } = message
        message.reply(
            `User Self Info: ${
                author.username
            } (account created: ${author.createdAt.toUTCString()})`,
        )
    }
}
```
The last command I've included is a bit more sophisticated. We are checking for messages starting with `!whois` and check the rest of the message for a user mention (@username). If a user is found, the bot will answer with a short message containing the user name and date of the user creation. If no text is entered after the command or no user is mentioned, the bot will do the same for the message author.

![](https://i.imgur.com/fW0AhxE.png)

## Wrapping up
I think we've covered a lot of ground here today and you learned a few basic commands and ways to interact with user messages in addition to the setup process using the discord developer portal.

In the following sessions, we will replace those if/else statements with a scaleable and more flexible command module structure, look at setups allowing multiple bots from one project and dabble with administration commands including warning, kicking and otherwise managing users.
