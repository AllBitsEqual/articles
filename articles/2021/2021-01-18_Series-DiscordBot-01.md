<!--
	title: "Build A Bot (DiscordJS) - Chat bots made easy"
	description: "Discord bots can be written in Javascript or Python and getting your first bot up and running is a great way to practice your Vanilla JS skills and have some fun. We will cover the whole process of registering the bot with Discord, a basic setup and how to bring it to your server."
	author: "Konrad Abe (AllBitsEqual)"
	published_at: 2021-01-18 08:00:00
	header_image: ""
	categories: "javascript discord bot series"
	canonical_url: ""
    series: "Build A Bot (DiscordJS)"
	language: en
-->
# Build A Bot (DiscordJS) - Chat bots made easy

Discord bots can be written in Javascript or Python and getting your first bot up and running is a great way to practice your Vanilla JS skills and have some fun. We will cover the whole process of registering the bot with Discord, a basic setup and how to bring it to your server.

## Prerequisites
There is not much that you need to start writing a bot for Discord using Javascript and while you could theoretically compose everything with copy and paste code snippets written by other people, I highly suggest some prior Javascript Knowledge. Here is what you need apart from that.
* A Discord Account / (desktop or web) Client
* A Discord Server with "manage server" permission
* Basic Terminal KnowHow
* NodeJS version 10 or higher

## First Steps - Discord Developer Portal
To write and use bot, you need to register it with your Discord account. Go to the [Discord Developers Portal](https://discordapp.com/developers/applications/) (sign in) and create a "New Application".

![](https://i.imgur.com/ufyc2FP.png)

Pick a name of your liking and continue to create the application. For my own server AllBitsEqual I will go with the wonderful pun name "AllBotsEqual" ... don't judge me!

![](https://i.imgur.com/ASp4fMP.png)

On the following page you can add a short description, avatar image and check your ID, Key and Secret. Don't forget to save your changes, when you are done.

![](https://i.imgur.com/3GXNkyG.png)

Use the left sidebar navigation to go to the "Bot" section and click "Add Bot" to add a bot to your newly created application.

![](https://i.imgur.com/33p9pLt.png)

Ok, this was about the hardest part... we now have a Bot with a user ID, can grab the Token for later and define the basic permissions.

![](https://i.imgur.com/KkUV1mv.png)

To continue with the permissions, head over to the OAuth2 secion, again using the left sidebar navigation.

From the first box, select the [ ] bot option. This will open up the second box with the Bot Permissions where you can pick and choose what the bot should be able to do. For this tutorial you will need at least "Send Messages" and "Read Message History" but in later parts we will add more functionalities including some moderator functionality.

![](https://i.imgur.com/CdYlmcG.png)

Copy the URL that has been generated with our bot ID and permissions when you are done selecting them. Your selection is part of the URL as the number after the permissions attribute.

When you enter this URL in your web browser of choice, you can pick the server you want to add the bot to (where you have the "manage server" permission) and "Authorize" it.

![](https://i.imgur.com/BhkWtBU.png)

You will see the list of permissions you just created and need to confirm it. When you are done confirming all confirmations, you should end up at this screen and be done with it.

![](https://i.imgur.com/Geuj5GV.png)

If you check your selected server now, you should see a message that your bot just joined the server.

![](https://i.imgur.com/QntLb1I.png)
