# Discord Bot - TBD


## Better logging
This time we will implement a prettier solution for our console logs using winston for the logging and chalk for the colours.

```bash
npm i -S winston chalk
```



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

Wire it up

![](https://i.imgur.com/hqCPgv9.png)

Change all occurances of the old logger.

![](https://i.imgur.com/0kpp0iR.png)


So, your console logging should now look like this.

![](https://i.imgur.com/8roRFxI.png)

One thing that we can now get rid off is putting the tag everywhere by hand. We can let winston handle that for us.

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

The difference in the visual output is minimal but in our code we just removed a lot of redundancy.

![](https://i.imgur.com/xfzPbkt.png)

Before we move on to the config, we still need to clean up a bit. There are still useless tags scattered throughout our code.

![](https://i.imgur.com/zYoIgtw.png)


## Read & Write Configs

Some of the tools we're going to use for our config are prebaked in Node but in addition to those we will need a way to work with json files, a way to create directories and to open files.

```bash
npm i -S jsonfile mkdirp opn
```


Let's start by adding our new tools to the imports and defining a useful sanitise function to radically clean up user input.

```javascript
// File: src/index.js
const os = require('os')
const path = require('path')
const fs = require('fs')
const opn = require('opn')
const mkdirp = require('mkdirp')
const jsonfile = require('jsonfile')


const sanitise = str => str.replace(/[^a-z0-9_-]/gi, '')
```

As we are going to implement proper configs now, let's put some work in here and define a more detailed config schema. We can replace our old configSchema with this.

```javascript
// File: src/index.js

// Config
const configSchema = {
    discordToken: { type: 'string', default: 'Paste your bot token here.' },
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

Every value given here is a valid value. Anything not in this list or of a wrong type will be discarded from the user input or old copies of the bot's config. This way we can make sure that the current config is always compatible.

You should also add 2 lines to the rules in out .eslintrc file because we will need them soon to not get bugged by the linter about stuff that is working as intended / we want it to be.

```json=
// File: .eslintrc
    "no-param-reassign": ["error", { "props": false }],
    "valid-typeof": 0
```


### set the config directory


```javascript
// File: src/index.js

    // Set the config directory to use
    bot.setConfigDirectory = function setConfigDirectory(configDir) {
        this.configDir = configDir
        this.configFile = path.join(configDir, 'config.json')
    }
```


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

```javascript
// File: src/index.js
    // Set default config directory
    bot.setConfigDirectory(path.join(os.homedir(), `.discord-${sanitise(initialConfig.name)}-bot`))
```



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







```javascript
// File: src/index.js
    bot.loadConfig = function loadConfig(config, callback) {
        const configExists = fs.existsSync(this.configFile)

        bot.log.info(`Checking for config file...`)
        // If file does not exist, create it
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

        // Load the created file, even if it is empty
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

        this.configIterator(this.config, configSchema)

        // Write the checked config data and open it again
        fs.writeFileSync(this.configFile, JSON.stringify(this.config, null, 4))
        if (!configExists) {
            this.log.warn('Config file created for the first time.')
            this.log.warn('Paste your Discord bot token into the botToken field and restart Liora.')
            this.openConfigFile()
        } else {
            jsonfile.readFile(this.configFile, (err, obj) => {
                if (err) {
                    bot.log.error(chalk.red.bold(`Unable to load config.json: ${err.message}`))
                    throw err
                } else {
                    bot.config = obj
                }
            })
        }

        this.log.info(`Reading config...`)
        try {
            if (!config || !has(config, 'token')) {
                throw Error(`Config or token are missing.`)
            }
            this.config = {
                ...configSchema,
                ...config,
            }
            callback()
        } catch (err) {
            this.log.error(`Error loading config: ${err.message}`)
            this.log.error('Please fix the config error and retry.')
        }
    }
```


