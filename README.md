# Discord bot template

This is a template for a Discord bot written in TypeScript, using the [discord.js](https://discord.js.org/) library.

## Features

- [x] TypeScript
- [x] Slash commands
- [x] Event handlers
- [x] Built-in pagination
- [x] Automatic help command
- [x] Fancy logging
- [x] Easy handling of interaction IDs

## Getting started

To get started, create a new repository from this template, as explained in the [GitHub docs](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-repository-from-a-template).

Add the required environemnt variables, as explained in the [Environment variables](#environment-variables) section.
Then run the following commands to install the dependencies and start the bot:

```sh
npm i
npm run dev
```

## Environment variables

An example `.env` file is provided under [`example.env`](./.env.example). You can copy this file and rename it to `.env` to get started.

Environment variables are encapsulated in the `ENV` object, located in the `src/env.ts` file.
When adding a new environment variable, you should add it to the `ENV` object as well.
This will check that the environment variable is set, and throw an error if it is not.

You should provide at least the following environment variables:

- `BOT_TOKEN`: the token of your bot
- `TEST_GUILD`: the ID of the guild to register slash commands in during development

In case you want to use a separate bot token for development, you can provide it in the `TEST_TOKEN` environment variable.
If no `TEST_TOKEN` is provided, the `BOT_TOKEN` will be used for development as well.

## Commands

Commands are located in the [`src/commands`](./src/commands) folder. The [index.ts](./src/commands/index.ts) file exports all command categories, which is used to register them in the [`src/events/interactionCreate/commands.ts`](./src/events/interactionCreate/commands.ts) file.

Commands are grouped into _categories_, each of which has its own folder. The `index.ts` file in each folder exports the commands in that category and provides some metadata about the category, which includes at least the name of the category.

Each command gets its own file, and consists of a `meta` object, built using a `SlashCommandBuilder` from `discord.js`, and a `exec` function, which is called when the command is executed. This `exec` function is passed a context containing the bot client, a Logger instance (instantiated with the command name; see the [Logging](#logging) section), and the interaction itself.

A command file should look something like this:

```ts
import { SlashCommandBuilder } from 'discord.js';
import { command } from '../../utils';

const meta = new SlashCommandBuilder()
  .setName('example')
  .setDescription('Example command.');

export default command(meta, async ({ interaction }) => {
  return interaction.reply({
    ephemeral: true,
    content: 'Hello world!',
  });
});
```

A `/help` command is already provided in the [`src/commands/general/help.ts`](./src/commands/general/help.ts) file, which automatically generates an embed that allows for navigating through all of the commands and their descriptions, using the pagination functionality described in section [Pagination](#pagination).

## Events

Events are located in the [`src/events`](./src/events) folder. The [index.ts](./src/events/index.ts) file exports all event handlers, which is used to register them in the [`src/index.ts`](./src/index.ts) file, which in turn calls the `registerEvents` function from [`src/utils/event.ts`](./src/utils/event.ts).

All `interactionCreate` events are grouped together in the [`src/events/interactionCreate`](./src/events/interactionCreate) folder, and are exported in the corresponding [`index.ts`](./src/events/interactionCreate/index.ts) file.
When adding new events, it is recommended to group them in a similar way.

Each event handler consists of the event name and a listener function, which is called when the event is triggered. The listener function recieves a context with the bot client and a Logger instance (instantiated with the event name; see the [Logging](#logging) section), as well as a list of arguments specific to the event.

An event handler file should look something like this:

```ts
import { event } from '../utils';

export default event('ready', ({ logger }, client) => {
  logger.system(
    `\x1b[4m${client.user.tag}\x1b[0m\x1b[36m is up and ready to go!`
  );
});
```

## Logging

The [`src/utils/log.ts`](./src/utils/log.ts) file exports a `Logger` class, which can be used to log messages to the console. It is instantiated with a `category` string, which is used to prefix the log messages. The file also exports a `log()` function, with additional functions, such as `log.system()`, for every type of log message.

The `Logger` class is used in the contexts of events and commands to automatically prefix every logged message in specific commands or events with the corresponding command or event name. Anywhere else, you can easily use the `log()` function, or any sub-function, to log messages. This takes the category as the first argument, and the message as the second argument.

The following log message types are currently supported:

- `default`: default log message (white text)
- `error`: error messages (red text)
- `warn`: warning messages (yellow text)
- `debug`: debug messages (gray text)
- `system`: system messages, such as startup messages (cyan text)

## Interaction replies

The [`src/utils/reply.ts`](./src/utils/reply.ts) file exports a `reply` class, which can be used to send replies to interactions, without having to worry about whether you should be using `interaction.reply()` or `interaction.editReply()`, as it automatically uses the correct method.

The function takes the interaction to reply to, the options for the reply (meaning you can provide the same options as you would to `interaction.reply()` or `interaction.editReply()`), and optionally a reply type. The reply will be made ephemeral by default, but it can be overwritten by providing a value for it yourself in the options (second argument). The reply type can be one of the following:

- `default`: send a normal reply
- `error`: send an error reply, which is prepended with an error emoji
- `warn`: send a warning reply, which is prepended with a warning emoji

Similar to the `log()` function (see the [Logging](#logging) section), the `reply()` function provides easy-to-use sub-functions for each type, so currenlty `reply.error()` and `reply.warn()`.

If you want these replies to use embeds by default, this can be easily changed by modifying the `getOptions()` function in the [`src/utils/reply.ts`](./src/utils/reply.ts) file.

## Pagination

There is built-in support for pagination of content using embeds. Currently, this is only used in the `/help` command, but you can create your own pagination by following these steps:

1. Create a paginator in the [`src/utils/paginators.ts`](./src/utils/paginators) folder, satisfying the interface specification in [`src/types/paginators.ts`](./src/types/paginators.ts)
2. Use the `paginationEmbed()` function from the utils to create an embed that can be used to navigate through the pages

The pagination for the `/help` command uses a separate paginator for each category of commands, which are defined in the [`src/utils/paginators/help.ts`](./src/utils/paginators/help.ts) file. The pagination embed for a selected category is created in the [`src/events/interactionCreate/help.ts`](./src/events/interactionCreate/help.ts) file.

The pagination uses embed fields to display the content, and thus the limit of items to show on a single page is 25 (the maximum number of fields allowed in an embed). You can also pass additional components to the embed (such as buttons), which will be added to the embed, but the amount is limited to 3, because of Discord's limit of 5 action rows per embed. Two action rows are already used by the pagination embed, one for the _next_ and _back_ buttons, and another for the page selection menu.

## Interaction IDs

Discord interaction IDs are used to identify interactions, such as slash commands, buttons, and select menus.
This allows the bot to differentiate between multiple interactions of the same type.
For example, if you have a slash command that sends a message with a button, and you click that button, the bot needs to know which button was clicked.
This is done by using the interaction ID.
In this template, **namespaces** are used as the primary identifier of an interaction, and additional arguments, separated by `;`, can be used to pass specific data along.
The currently available namespaces can be found in the [`src/constants/namespaces.ts`](./src/constants/namespaces.ts) file.
The general structure of an interaction ID is thus as follows:

```
<namespace>;<arg1>;<arg2>;<...>
```

The namespaces can be used to filter out other interactions in the `interactionCreate` event handler, and the arguments can be used to pass specific data along with the interaction.
For example, the pagination (see the [Pagination](#pagination) section) uses the `pagination` namespace, and passes the paginator name and current offset as arguments.
The event handler for the pagination buttons can then filter out other interactions by checking the namespace, and use the arguments to determine which paginator to use and what the current offset is.
In particular, it looks something like this:

```ts
if (!interaction.isButton() && !interaction.isStringSelectMenu()) return;
const [namespace] = parseId(interaction.customId);
if (namespace !== NAMESPACES.pagination) return;
```

The first line filters out other types of interactions, such as slash commands, and the second line parses the interaction ID to retrieve the namespace, which is then used to check that the interaction is indeed a pagination interaction.
Notice the use of the `NAMESPACES` object to make sure we always use the correct string identifier for each namespace and allow for easy renaming of namespaces.

To facilitate the easy usage of this configuration of interaction IDs, there are two utility functions available in the [`src/utils/interaction.ts`](./src/utils/interaction.ts) file:

- `createId()`: create an interaction ID from a namespace and a list of arguments
- `parseId()`: parse an interaction ID into a namespace and a list of arguments

## Other utility functions

- [`splitSend`](./src/utils/split.ts): split a a list of lines over multiple embeds, respecting Discord's embed description length limit
- [`chunk`](./src/utils/chunk.ts): split an array into chunks of a given size (jagged array / matrix)

## Database

If you wish to use a database, you can add a `src/client/db.ts` file, which exports a `db` object, and re-export it in [`src/client/index.ts`](./src/client/index.ts).
This `db` object should be imported in [`src/index.ts`](./src/index.ts) and passed to the `registerEvents` function.
In the `registerEvents` function (located in [`src/utils/event.ts`](./src/utils/event.ts)), you can then add it to the context for the event handlers, giving your commands and events access to the database.
Note that passing the database in the context like this is recommended over importing a database object in every file.

For example, the following snippet initializes a database connection using the `firebase-admin` package and would be added to the `src/client/db.ts` file:

```ts
const serviceAccount = JSON.parse(ENV.FIREBASE_SDK);

// Initialize Firebase with the realtime database
admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: process.env.DB_URL,
});

export const db = admin.database();
```

## Server

If you need to run a server alongside the bot, for example to create an API which you can use to control the bot from a dashboard, you can add a `src/server` folder.
Create an `index.ts` file in this folder, and add the following snippet to run an Express server:

```ts
import express from 'express';
import { log } from '../utils';
import { ENV } from '../env';

const app = express();

app.get('/', (req, res) => {
  res.send('Hello world!');
});

app.listen(process.env.PORT || 3000, () => {
  log.system('server', 'Listening on port 3000!');
});
```

In this snippet, you can access the bot client by importing it from [`src/client/index.ts`](./src/client/index.ts).
The process.env.PORT environment variable is usually provided by the hosting service, such as Heroku, and defaults to 3000 if not provided.

You can do whatever you want with this Express server, including adding routes, middleware, etc. (see the [Express documentation](https://expressjs.com/)).
Folder structure is up to you, but you can use the following as a starting point:

```
src
├── server
│   ├── index.ts
│   ├── routers
│   │   ├── index.ts
│   │   └── ...
│   ├── middleware
│   │   ├── index.ts
│   │   └── ...
├── ...
```
