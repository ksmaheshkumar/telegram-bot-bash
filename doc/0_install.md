#### [Home](../README.md)

## Install bashbot

1. Go to the directory you want to install bashbot, e.g.
    * your $HOME directory (install and run with your user-ID)
    * /usr/local if you want to run as service
2. [Download latest release zip from github](https://github.com/topkecleon/telegram-bot-bash/releases) and extract all files. 
3. Change into the directory ```telegram-bot-bash```
4. Acticate the bot example commands  ``cp mycommands.sh.dist mycommands.sh```
5. Run ```./bashbot.sh init``` to setup the environment and enter your Bots token given by botfather.

Edit 'mycommands.sh to your needs.
Now your Bot is ready to start ...

**If you are new to Bot development read [Bots: An introduction for developers](https://core.telegram.org/bots)**

### Install from github

As an alternative to download the zip files, you can clone the github repository to get the latest improvements/fixes.

1. Go to the directory you want to install bashbot, e.g.
    * your $HOME directory (install and run with your user-ID)
    * /usr/local if you want to run as service
2. Run ```git clone https://github.com/topkecleon/telegram-bot-bash.git```
3. Change into the directory ```telegram-bot-bash```
4. Run ``` test/ALL-tests.sh``` and if everything finish OK ...
5. Run ```sudo ./bashbot.sh init``` to setup the environment and enter your Bots token given by botfather.

###  Update bashbot

**Note: all files including 'mycommands.sh' may overwritten, make a backup!**

1. Go to the directory where you had installed bashbot, e.g.
    * your $HOME directory
    * /usr/local
2. [Download latest release zip from github](https://github.com/topkecleon/telegram-bot-bash/releases)
3. Stop all running instances of bashbot
4. Extract all files to your existing bashbot dir 
5. Run ```sudo ./bashbot.sh init``` to setup your environment after the update

If you modified ```commands.sh``` move your changes to ```mycommands.sh```, this avoids overwriting your commands on update.

Now you can restart your bashbot instances.

### Note for BSD and MacOS

**On MacOS** you must install a more recent version of bash, as the default bash is way to old,
see e.g. [Install Bash on Mac](http://macappstore.org/bash/)

**On BSD and MacOS** I recommend to install gnu coreutils and include them in your PATH
environment variable before running bashbot, e.g. the gnu versions of sed, grep, find ...

On BSD and MacOS you must adjust the shebang line of the scripts ```bashbot.sh``` and ```json.sh``` to point to to the correct bash
or use the script: ```examples/bash2env *.sh */*.sh``` to convert them for you.

Bashbot will stay with /bin/bash shebang, as using a fixed path is more secure than the portable /usr/bin/env variant, see
[Security Considerations](../README.md#Security-Considerations)

I considered to make bashbot BSD sed compatible, but much of the bashbot "magic" relies on
(gnu) sed features, e.g. alternation ```|```, non printables ```\n\t\<``` or repeat ```?+``` pattern, not supported by BSD sed.
BSD/MacOS sed compatibility will result in a rewrite of all grep/sed commands with an uncertain outcome,
see [BSD/MacOS vs. GNU sed](https://riptutorial.com/sed/topic/9436/bsd-macos-sed-vs--gnu-sed-vs--the-posix-sed-specification)
to get an impression how different they are.

If you can convert the following examples to work correct with gnu and BSD sed, contact me.

```bash
  # easy start
  sed -n -e '0,/\['"$1"'\]/ s/\['"$1"'\][ \t]\([0-9.,]*\).*/\1/p'
  OUT="$(sed -e ':a;N;$!ba;s/\r\n/ mynewlinestartshere /g' <<<"$1"| iconv -f utf-8 -t utf-8 -c)"

  # more complex
  address="$(sed <<< "${2}" '/myaddressstartshere /!d;s/.*myaddressstartshere //;s/ *my[nkfltab][a-z]\{2,13\}startshere.*//;s/ *mykeyboardendshere.*//')"

  # for experts?
  source <( printf "$1"'=( %s )' "$(sed -E -n -e ':x /"text"\]/ { N; s/([^"])\n/\1\\n/g ; tx }' -e '/\["[-0-9a-zA-Z_,."]+"\]\+*\t/ s/\t/=/gp' -e 's/=(true|false)/="\1"/')" )
```

### Notes per Version

#### Change in storing config values

Up to version 0.94 bashbot stores config values as values in ```token```, ```botadmin``` and ```count```. Since version 0.96 bashbot
uses jsonDB key/value store. Config is stored in ```botconfig.jssh```, counting of users is done in ```count.jssh```.
The acl file ```botacl``` stay as is. On first run of bashbot.sh after an update bashbot converts
the files to the new config format. Afterwards the files ```token```, ```botadmin``` and ```count``` can be deleted.

You may notice the new file ```blocked.jssh```, every telegram user or chat id stored here will be blocked from 
using your bot.

#### removal of TMUX
From version 0.80 on TMUX is no longer needed and the bashbot command 'attach' is deleted. Old function 'inproc'
is replaced by 'send_interactive'. send_interactive does checks if an interactive job is running internally.
Pls check if you make use of inproc and remove it including the old checks, e.g.
```bash
if tmux ls | grep -v send | grep -q "$copname"; then inproc; fi
# or
[ checkprog ] && inproc
```
must be replaced by ```send_interactive "${CHATD[ID]}" "${MESSAGE}"```

### Do not edit commands.sh
From version 0.60 on your commands must be placed in 'mycommands.sh'. If you update from a version with your commands
in 'commands.sh' move all your commands and functions to 'mycommands.sh'.

From version 0.80 on 'commands.sh' will be overwritten on update!

#### Location of var / tmp / data dirs
From version 0.70 on the tmp dir is renamed to 'data-bot-bash' to reflect the fact that not only temporary files are stored. an existing 'tmp-bot-bash' will be automatically renamed after update.

From version 0.50 on the temporary files are no more placed in '/tmp'. instead a dedicated tmp dir is used.

#### Changes to send_keyboard in v0.6
From Version 0.60 on keyboard format for ```send_keyboard``` and ```send_message "mykeyboardstartshere ..."``` was changed.
Keyboards are now defined in JSON Array notation e.g. "[ \\"yes\\" , \\"no\\" ]".
This has the advantage that you can create any type of keyboard supported by Telegram.
The old format is supported for backward compatibility, but may fail for corner cases.

*Example Keyboards*:

- yes no in two rows:
    - OLD format: 'yes' 'no' *(two strings)*
    - NEW format: '[ "yes" ] , [ "no" ]' *(two arrays with a string)*
- new layouts made easy with NEW format:
    - Yes No in one row: '[ "yes" , "no" ]'
    - Yes No plus Maybe in 2.row: '[ "yes" , "no" ] , [ "maybe" ]' 
    - numpad style keyboard: '[ "1" , "2" , "3" ] , [ "4" , "5" , "6" ] , [ "7" , "8" , "9" ] , [ "0" ]'



#### [Next Create Bot](1_firstbot.md)

#### $$VERSION$$ v1.0-0-g99217c4

