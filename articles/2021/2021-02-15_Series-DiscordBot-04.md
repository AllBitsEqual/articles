---
tags: Discord, Javascript, DiscordJS, Chatbot, NodeJS, Config
---
<!--
	title: "Build A Bot (DiscordJS) - Better Logging And A Persistent Bot Config"
	description: "Today we will spend a little time on a prettier logger and then allow our bot to read and write his own config file on the server."
	author: "Konrad Abe (AllBitsEqual)"
	published_at: 2021-02-15 08:00:00
	header_image: "https://i.imgur.com/446hSEo.jpg"
	categories: "javascript discord discordjs bot chatbot nodejs config series"
	canonical_url: "https://allbitsequal.medium.com/build-a-bot-discordjs-better-logging-and-a-persistent-bot-config-6e3cd07bc91a"
	series: "Build A Bot (DiscordJS)"
	language: en
-->
# Build A Bot (DiscordJS) - Better Logging And A Persistent Bot Config

Last time we left off, we had turned our simple bot into a bot factory, allowing us to spawn multiple bots with different configs. Those configs though were still pretty simple and not persistent. The user could not make any changes unless he made them directly to the config files.

**Today we will spend a little bit of time on a prettier logger and then allow our bot to read and write his own config file on the server.**

As always, the link to the finished code in GitHub is at the end of the article.

*Credits: Today's session will include code influenced and partly taken from the [Liora Bot Project](https://jackw01.github.io/liora/index). Feel free to look at their code for more inspiration.*

![](https://i.imgur.com/446hSEo.jpg)

## Better logging
To start today's session, we will implement a prettier solution for our console logs using Winston for the logging and chalk for the pretty colours.

You know the drill, grab what we need from npm and then let's get busy.

```bash
npm i -S winston chalk
```

Winston is working with log levels and colours so let's start by setting up some sensible defaults. Right now we will mostly work with error, warn and info but later on, those other levels will be used too.

```javascript
// File: src/index.js

// add this at the top
const winston = require('winston')
const chalk = require('chalk')

// define log levels
const logLevels = {
    error: 0,
    warn: 1,
    info: 2,
    modules: 3,
    modwarn: 4,
    modinfo: 5,
    debug: 6,
}

// define log colours
winston.addColors({
    error: 'red',
    warn: 'yellow',
    info: 'green',
    modules: 'cyan',
    modwarn: 'yellow',
    modinfo: 'green',
    debug: 'blue',
})
```

Then we create a new logger instance with the basic setup and formating. Within the printf function, we can format our desired logout format. We want a timestamp here along with the log level and of course the logged message.

```javascript
// File: src/index.js

// add the configured new logger using winston.createLogger()
const logger = winston.createLogger({
    levels: logLevels,
    transports: [new winston.transports.Console({ colorize: true, timestamp: true })],
    format: winston.format.combine(
        winston.format.colorize(),
        winston.format.padLevels({ levels: logLevels }),
        winston.format.timestamp(),
        winston.format.printf(info => `${info.timestamp} ${info.level}:${info.message}`),
    ),
    level: 'debug',
})
```

What's left to do now is to wire it up with our bot object, finally getting rid of that `eslint-disable`...

![](https://i.imgur.com/hqCPgv9.png)

... and apply it in the places where we used the old and too simple logger and add our desired log levels and use chalk to paint the message where we see fit.

![](https://i.imgur.com/0kpp0iR.png)

When you are done, your console logging should now look like this. If you want to see my choice of colours, [check out this commit](https://github.com/AllBitsEqual/allbotsequal/commit/bf208704c5a5f56663d3f25dc036d0c8a0289809).

![](https://i.imgur.com/8roRFxI.png)

One thing that we can now get rid of is putting the tag everywhere by hand. We can let Winston handle that for us. Change the line where we assigned the `winston.createLogger()` result and turn it into a fat arrow function that passes in the tag and returns the logger. This way we can include the tag in our printf output via `${tag}`.

```javascript
// File: src/index.js
const logger = tag =>
    winston.createLogger({
        levels: logLevels,
        transports: [new winston.transports.Console({ colorize: true, timestamp: true })],
        format: winston.format.combine(
            winston.format.colorize(),
            winston.format.padLevels({ levels: logLevels }),
            winston.format.timestamp(),
            winston.format.printf(info => `${info.timestamp} ${info.level}: ${tag}${info.message}`),
        ),
        level: 'debug',
    })
```

Now we need to add the tag (including a sensible default) to our log assignment and we're done.

```javascript
// File: src/index.js
// Define the bot
    const bot = {
        client: new discord.Client(),
        log: logger(initialConfig.tag || `[Bot ${initialConfig.index}]`),
        commands: new discord.Collection(),
    }
```

The difference in the visual output is minimal but in our code, we just removed a lot of redundancy.

![](https://i.imgur.com/xfzPbkt.png)

Before we move on to the config, we still need to clean up a bit. There are still useless tags scattered throughout our code.

![](https://i.imgur.com/zYoIgtw.png)

---

## Read & Write Configs

Some of the tools we're going to use for our config are prebaked in Node but in addition to those, we will need a way to work with json files, a way to create directories and to open files.

```bash
npm i -S jsonfile mkdirp opn
```

Let's start by adding our new tools to the imports and defining a useful small sanitise function to radically clean up user input. We'll use this later to create directories for the bots' config files and we don't want any funny characters in those directory names.

```javascript
// File: src/index.js
const os = require('os')     // nodeJS
const path = require('path') // nodeJS
const fs = require('fs')     // nodeJS
const opn = require('opn')
const mkdirp = require('mkdirp')
const jsonfile = require('jsonfile')


const sanitise = str => str.replace(/[^a-z0-9_-]/gi, '')
```

As we are going to implement proper configs now, let's put some work in here and define a more detailed config schema. We can replace our old configSchema with this.

I'm using this schema to define what type of data the config accepts. This way we can run a basic check later to make sure every attribute resembles our requirements and we can include defaults in case the user has not set an attribute. Anything not in this list or of a wrong type will be discarded from the user input or old copies of the bot's config. This way we can make sure that the current config is always compatible.

> One advice, don't put your token into the configSchema by hand. Include it in the initialConfig on bot start, as we had set it up last time. You would not want to hard code your bot's token **(or upload it to a public repository in any case!)** as it better sits in the non-versioned .env file or environment config of your hosted project.

```javascript
// File: src/index.js

// Config
const configSchema = {
    discordToken: { type: 'string', default: 'HERE BE THE TOKEN' },
    owner: { type: 'string', default: '' },
    name: { type: 'string', default: 'BotAnon' },
    defaultGame: { type: 'string', default: '$help for help' },
    prefix: { type: 'string', default: '$' },
    commandAliases: { type: 'object', default: {} },
    defaultColors: {
        type: 'object',
        default: {
            neutral: { type: 'string', default: '#287db4' },
            error: { type: 'string', default: '#c63737' },
            warning: { type: 'string', default: '#ff7100' },
            success: { type: 'string', default: '#41b95f' },
        },
    },
    settings: { type: 'object', default: {} },
}
```

You should also add 2 lines to the rules in out .eslintrc file because we will need them soon to not get bugged by the linter about stuff that is working as intended / we want it to be.

```json
// File: .eslintrc
    "no-param-reassign": ["error", { "props": false }],
    "valid-typeof": 0
```


### 1) Setting the config directory
We will need a way to keep track of config file paths to a certain directory. We simply store those in our bot object.

```javascript
// File: src/index.js

    // Set the config directory to use
    bot.setConfigDirectory = function setConfigDirectory(configDir) {
        this.configDir = configDir
        this.configFile = path.join(configDir, 'config.json')
    }
```

### 2) Run it once initially
Here we are using the sanitise function we defined earlier to take the bot name and use it to create a directory for each bot. If you run the script on your own PC during test and development, the config files will be written to your home/user directory instead of the server's respective directory. Simply check for files starting with `.discord-` followed by your bot's name.

```javascript
// File: src/index.js
    // Set default config directory
    bot.setConfigDirectory(
        path.join(os.homedir(), `.discord-${sanitise(initialConfig.name)}-bot`)
    )
```

### 3) Open generated config files for proofreading
Furthermore, I want to be able to open the files our script has created on the first run so that the user can check if his values have been merged correctly.

For this we will use something node provides us with, `opn` and if one of the bots had his config generated for the first time, we will open the generated file exit the process. On the next run of our script, all bots will connect regularly.

```javascript
// File: src/index.js

    // Open the config file in a text editor
    bot.openConfigFile = function openConfigFile() {
        bot.log.info('Opening config file in a text editor...')
        opn(this.configFile)
            .then(() => {
                bot.log.info('Exiting.')
                process.exit(0)
            })
            .catch(err => {
                this.log.error('Error opening config file.')
                throw err
            })
    }

```

### 4) Check the configSchema
We also need a function to validate the user-supplied config and merge it with our schema to generate the new bot config. We'll go through our schema step by step, compare the existence and type of the respective attribute in the bot config and either delete or overwrite it depending on our checks. For objects, it will call itself recursively layer by layer.

```javascript
// File: src/index.js

    // Recursively iterate over the config to check types and reset properties to default if they are the wrong type
    bot.configIterator = function configIterator(startPoint, startPointInSchema) {
        Object.keys(startPointInSchema).forEach(property => {
            if (!has(startPoint, property)) {
                if (startPointInSchema[property].type !== 'object') {
                    startPoint[property] = startPointInSchema[property].default
                } else {
                    startPoint[property] = {}
                }
            }
            if (startPointInSchema[property].type === 'object') {
                configIterator(startPoint[property], startPointInSchema[property].default)
            }
            if (
                !Array.isArray(startPoint[property]) &&
                typeof startPoint[property] !== startPointInSchema[property].type
            ) {
                startPoint[property] = startPointInSchema[property].default
            }
        })
    }
```

### 5) The big one, loadConfig
This is the place where it all comes together. I broke it down into 5 subsections that we will go through piece by piece.

Our new loadConfig function will do a lot of things so I stripped it down to the shell and some comments to give you the outlines.

First of all, check for the existence of a config file. We will need this in a moment.

```javascript
// File: src/index.js
    bot.loadConfig = function loadConfig(config, callback) {
        bot.log.info(`Checking for config file...`)
        const configExists = fs.existsSync(this.configFile)

        /* [ALPHA]
         *  If the file does not exist, create it
         */
         

        /* [BETA]
         * Load the config file from the directory
         */


        /* [GAMMA]
         * iterate over the given config, check all values and sanitise
         */


        /* [DELTA]
         * write the changed/created config file to the directory
         */
         
         
         /*
          * read the new file from the directory again 
          * - assign it to the bot's config
          * - execute callback() or abort on error
          */
    }
```

#### ALPHA
If no old config is found, we simply create a new config.json in our chosen location using `mkdirp`, a small package resembling the desktop command `mkdir -p`, and prepare it with the most basic and important fields from what we are passing in on project start; discordToken, Prefix and 

```javascript
// File: src/index.js

        /* [ALPHA]
         *  If the file does not exist, create it
         */
        if (!configExists) {
            bot.log.info(`No config file found, generating...`)
            try {
                mkdirp.sync(path.dirname(this.configFile))
                const { token, name, prefix } = initialConfig
                const baseConfig = {
                    discordToken: token,
                    prefix,
                    name,
                }
                fs.writeFileSync(this.configFile, JSON.stringify(baseConfig, null, 4))
            } catch (err) {
                this.log.error(chalk.red.bold(`Unable to create config.json: ${err.message}`))
                throw err
            }
        }
```

#### BETA
Next step, we load the config file, no matter if it's an old one or we just created it.

```javascript
// File: src/index.js

        /* [BETA]
         * Load the config file from the directory
         */
        this.log.info(`Loading config...`)
        try {
            this.config = JSON.parse(fs.readFileSync(this.configFile))
        } catch (err) {
            this.log.error(`Error reading config: ${err.message}`)
            this.log.error(
                'Please fix the config error or delete config.json so it can be regenerated.',
            )
            throw err
        }
```

#### GAMMA
Now call our configIterator with the config we read from the disk and compare it to our schema. As previously written, this makes sure that no old or mismatched values remain in the config once we decide to change the schema in the future.

```javascript
// File: src/index.js

        /* [GAMMA]
         * iterate over the given config, check all values and sanitise
         */
        this.configIterator(this.config, configSchema)
```

#### DELTA
Write the checked and clean config back to the server.

```javascript
// File: src/index.js

        /* [DELTA]
         * write the changed/created config file to the directory
         */
         fs.writeFileSync(this.configFile, JSON.stringify(this.config, null, 4))
```

#### EPSILON
Last but not least, reload the config from the directory and check one last time. If everything is fine, execute the callback to continue and otherwise abort with an error.

```javascript
// File: src/index.js

        /* [EPSILON]
         * read the new file from the directory again
         * - assign it to the bot's config
         * - execute callback() or abort on error
         */
        jsonfile.readFile(this.configFile, (err, obj) => {
            if (err) {
                bot.log.error(chalk.red.bold(`Unable to load config.json: ${err.message}`))
                throw err
            } else {
                bot.config = obj
                callback()
            }
        })
```

If you want to make sure you got everything, have a look at the finished function in all it's glory and complexity.

```javascript
bot.loadConfig = function loadConfig(config, callback) {
        bot.log.info(`Checking for config file...`)
        const configExists = fs.existsSync(this.configFile)

        /*
         *  If the file does not exist, create it
         */
        if (!configExists) {
            bot.log.info(`No config file found, generating...`)
            try {
                mkdirp.sync(path.dirname(this.configFile))
                const { token, name, prefix } = initialConfig
                const baseConfig = {
                    discordToken: token,
                    prefix,
                    name,
                }
                fs.writeFileSync(this.configFile, JSON.stringify(baseConfig, null, 4))
            } catch (err) {
                this.log.error(chalk.red.bold(`Unable to create config.json: ${err.message}`))
                throw err
            }
        }

        /*
         * Load the config file from the directory
         */
        this.log.info(`Loading config...`)
        try {
            this.config = JSON.parse(fs.readFileSync(this.configFile))
        } catch (err) {
            this.log.error(`Error reading config: ${err.message}`)
            this.log.error(
                'Please fix the config error or delete config.json so it can be regenerated.',
            )
            throw err
        }

        /*
         * iterate over the given config, check all values and sanitise
         */
        this.configIterator(this.config, configSchema)

        /*
         * write the changed/created config file to the directory
         */
        fs.writeFileSync(this.configFile, JSON.stringify(this.config, null, 4))

        /*
         * read the new file from the directory again
         * - assign it to the bot's config
         * - execute callback() or abort on error
         */
        jsonfile.readFile(this.configFile, (err, obj) => {
            if (err) {
                bot.log.error(chalk.red.bold(`Unable to load config.json: ${err.message}`))
                throw err
            } else {
                bot.config = obj
                callback()
            }
        })
    }
```

[Link to the finished code / tag v0.0.4 on GitHub](https://github.com/AllBitsEqual/allbotsequal/releases/tag/v0.0.4)

---

## Wrapping up
Using nodeJS for the first time to access and work with files can be a daunting task so depending on where you are/were with your experience, I hope I was able to keep it nice and basic and understandable.

Our Bot(s) can now be started by creating a new or loading an existing config file. Next time we will add some commands that let the users with the right roles and permissions change the config on the fly, add new tags and maybe even access those from a dashboard... stay tuned.