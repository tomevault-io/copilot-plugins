## telecode

> tips for tools to use:

tips for tools to use:

## grep

use `grep -rin <WORD>` when trying to find files with a word

Example: `grep -rin bot server/` (search all files in server/ for 'bot', print
line numbers)

- NEVER grep node_modules

### output

```bash
$ grep -rin bot server/
server/routes/tlg.ts:19:   * X-Telegram-Bot-Api-Secret-Token header of the request, or undefined if
server/routes/tlg.ts:35:   * X-Telegram-Bot-Api-Secret-Token headers
server/routes/tlg.ts:54:    header: getHeader(event, 'X-Telegram-Bot-Api-Secret-Token') || undefined,
server/routes/tlg.ts:76:  return await webhookCallback(event.context.bot.bot, nitroModule)(event);
server/utils/telegram.ts:8:import { Bot, Context, session, SessionFlavor } from 'grammy';
server/utils/telegram.ts:156:export class TelegramBot {
server/utils/telegram.ts:157:  public bot: Bot<MyContext>;
...
```

## ls

use `ls -lh <ARG>` to see list of files. includes creation time and size in
human readable form

Example: `ls -lh .` (lists files in current directory with sizes/times)

Example: `ls -lh server/utils/` (see all files in server/utils/ folder)

### output

```bash
$ ls -lh .
total 624K
drwxr-xr-x 1 meyer meyer 7.4K Jan 12 01:11 node_modules
drwxr-xr-x 1 meyer meyer   46 Jan 11 15:56 server
-rw-r--r-- 1 meyer meyer  201 Jan 11 14:02 drizzle.config.ts
-rw-r--r-- 1 meyer meyer  480 Jan 11 23:45 eslint.config.mjs
-rw-r--r-- 1 meyer meyer  409 Jan 11 21:27 nitro.config.ts
...
```

## find

use `find <PATH> -type f` to search for files.

Example: `find server/ -type f` (finds all files in the server/ directory) you
can use find with .

use `find <PATH> -type d` to search for directories.

Example: `find server/ -type d` (finds all directories in the server/ directory)
you can use find with .

### output

```bash
$ find . -type f
./package.json
./pnpm-lock.yaml
./README.md
./server/db/schema.ts
./server/utils/ai.ts

$ find server/ -type d
server/db
server/utils
```

## find | grep

you can use find with grep to filter file names `find <PATH> -type f | grep
<PATTERN>`.

Example: `find server/ -type f | grep goal` (shows files under server/ with
"goal" in their path).

### output

```bash
$ find server/ -type f | grep goal
server/utils/goal.ts:100:    if (!startWord) {
server/utils/goal.ts:101:      await ctx.reply('Invalid input. Please try /start again.');
server/utils/goal.ts:102:      return;
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/meyer1994) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
