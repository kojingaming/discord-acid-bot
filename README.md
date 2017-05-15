# Acid-Bot
My personal Discord chat bot, written in Python 3

`acid-bot` runs on **Python 3.4+** (tested only on 3.5) and requires [gTTS](https://github.com/pndurette/gTTS) (Python TTS library), [discord.py](https://github.com/Rapptz/discord.py/) (Discord Bot API wrapper for Python) and `requests`. It also requires my postfix calculator implementation, but this is included (`postfix.py`).

The `log_server.py` Flask server requires `flask` to be installed. It is a separate program which gives users access to view the logs that the bot keeps.

On start-up, the Discord API token must be read from file `secrettoken` in the same directory.

## Currently implemented commands

	\help                This helpful message.
	\rr [subreddit]      Get a random image from [subreddit]
	\rrtop [subreddit]   Get a random image from the all time top posts of [subreddit]
	\calc [query]        Postfix calculator
	\whoami              User stats
	\whois [username]    ^ Ditto
	\ping                Pong.
	\define [word]       Lookup the definition of [word]
	\ud [word]           Lookup the urban definition of [word]
	\50/50               You feeling lucky?
	\flip                Flip a coin
	\tell [name] [msg]   Send [msg] to [name] next time the bot sees them. [name] can be a @mention or a username.

	\imitate [username] (length) (tts)  Imitate [username] (Markov Chains!).
	\markovusers         List users' markov ratings (higher number means better \imitate)
	\markovsave          Save markov data to disk

	\reactionadd [name] [url]  Add a new link to the [name] collection
	\reactions                 Lists all the reaction collections
	\\[name]                   Random link from [name] collection (TWO backslashes)

	\voice               Connect/disconnect from voice channel
	\tts                 Say something with the tts
	\chlang              Changes the tts language (from https://pastebin.com/QxdGXShe)

	\scores                List math scores
	\problems              Start a short 10 question basic facts test
	\ans [ans1] [ans2] ... Answer the basic facts test

	Debug (Admin) Commands:
	\markovload \markovclear [username] \markovfeed [username] [url]
	\rename [newname] \setgame [playing] \reactiondel [name] [num]

Everything is functional. Improvements are needed on the math-quiz commands and the markov bot imitation fails to produce correct output.

### Additional features/notes

 - The bot logs all messages, edits, and deletions into an sqlite3 database `logs.db`. This is to counter censorship.
 - The bot saves all reactions to a `reactions.json`. This file can be edited directly, but the bot must be restarted to take the changes.
 - Word-pair data for the markov bot is stored in `markov.pickle`. This data is **not** saved unless the `\markovsave` command is run.
 - Math-quiz score data is storred in `mathscores.json`.
 - The `[url]` supplied to `\markovfeed` must point to a resource with `Content-Type: text/plain`.
 - Permissions are defined per-command hardcoded into the bot. Change them to your liking. Use IDs given by `\whoami` and `\whois` commands.
 - You do not need to type the whole username for `\whois` to work. The first few letters will be sufficient.

### How does it work!?!!

There really needs to be some documentation on how to fully run this program. I'm going to assume you've installed all the required dependencies.

The program requires a file called `secrettoken` which contains the bot token [generated by Discord](https://discordapp.com/developers/applications/me). Put the `token` (NOT the client secret) in this file. It will be read when `discordbot.py` starts.

When you start `discordbot.py` a bunch of files will be loaded and settings will be set. First, it opens an sqlite3 database. This database must already exist. Creation of this table is trivial, just run this line of SQL:

`CREATE TABLE logs (time real, channel text, id integer, name text, displayname text, messageid integer, deleted integer, edited integer, message text)`

And save the file as `logs.db`. Put the file in the same directory as `discordbot.py`. The `time` field is the unix epoch of the event in UTC, `id` is the ID of the author, and `messageid` is the unique ID for that message.

Next, the `Markov` manager is started. It will load a Python pickle file, `markov.pickle`, containing the markov data (if it exists). To save markov data, you **must** run `\markovsave` command. This will save all past conversation data to the disk. The bot will not do this automatically.

Next, the `VoiceWrapper` is created. It has no settings. After that, `MathRunner` is created. This will load users' math game scores from `mathscores.json` (if it exists).

The reaction list is then loaded from `reactions.json`. The tells (see `\tell`) are loaded from `tells.json`. The banned IDs are hardcoded. IDs in this list cannot execute commands. The settings for the logserver are loaded from `logserver_config.json`. This file MUST exist, even if you do not intend to use the logserver.

**The log server**

`log_server.py` contains a Python 3 Flask program which serves up the information contained in `logs.db`. It must run in the same directory as the `logs.db` file. To access the logs, a token must be generated by accessing `/gentoken?password=password` where the password is stored in `logserver_config.json` under `token_generator_password`. Only the server and the bot can know this password.

When a successful GET request is made to `/gentoken` the response body will be, in plaintext, a 12 character token, such as `43247b7fe87F`. Then by accessing `/logs/?t=43247b7fe87F` a user will be granted access to view the logs. The token lasts 1 hour by default. This can be modified in `logserver_config.json` in the field `token_lifetime`.

**The commands**

 - `\rr` and `\rrtop` read JSON data from reddit.com.
 - `\calc` runs on the RPN calculator implementation: `postfix.py`.
 - `\define` makes a GET request to dictionary.com and retrieves the first definition only. See `dictionarycom.py`.
 - `\ud` makes an API request to Urban Dictionary. See `dictionarycom.py`.
 - `\50/50` makes a request to [reddit.com/r/fiftyfifty](https://reddit.com/r/fiftyfifty) and reformats the title.
 - `\tell` will let you send messages to users. The messages will only be sent the next time they speak.
 - `\voice` will join/leave the voice channel. It currently joins the first voice channel it finds.
 - `\tts` will send text to Google, and Google will return a wav file saved to `voice.wav`. The bot will use ffmpeg to play the wav file on voice chat. The bot hangs while it downloads from Google, so really long tts texts will cause the bot to time out from Discord.
 - `\logs` will generate a link and token for a user to access the logs that the bot keeps.
