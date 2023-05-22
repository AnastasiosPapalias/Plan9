# Chatting on discord from 9front

Sometimes you work with people who doesn't want to use the same communication tools as you do. A sad example is when your team members wants to use discord, and not something simpler like IRC. Oh well, gotta deal with it.

This is the reason I wrote a "discord bot" (ugh) which can run on your 9front system and allow you to chat on discord anyways.

## Example gif

The example below just opens two chat windows with different channels in each. While it works both in acme and in the terminal, I like having everything together inside acme.

![Example gif](https://github.com/AnastasiosPapalias/Plan9/blob/main/discord/discordgif.gif) [](https://github.com/AnastasiosPapalias/Plan9/blob/main/discord/discordgif.gif)

## The code

The project is made of a few scripts and a go program. The only thing that actually communicates with discord is the go program, and the scripts just makes it easier to send and recieve on the correct channels. The code is available [here]([url](https://git.sr.ht/~pmikkelsen/discordfront)), and there is a precompiled version of the go program included (compiled for 9front amd64).

## Usage

To use the service, you must first create a bot on discord and get the access token. I personally found this step much more confusing then the actual coding itself, which either tells you something about my google skills, or about discord.. Oh, and then you must invite the bot to your discord server..

Next, the server part of the bot must be started:

```
`discordsrv YOURTOKEN`
```

This will post a pipe in /srv/discordfront, where messages can be send and recieved from, but you should not read from it directly. The discordsrv script does this for you as well, and it takes each line and dumps it in a nice readable format into $home/lib/discord/logs/$serverName/$channelName where the server name is the name of a discord server (or guild?), and the channel name is, well, the name of the channel.

Now that the server is started, you can either run discordacme which opens an acme with the file $home/lib/discord/channels open. This file is not handled by any of the scripts, so it is just a handy text file to keep shortcuts to various channels (see the gif for an example). Inside discordacme, the openChat command will open a new chat window, using the simpler discord script.

The discord script simply runs tail -f on the given channel's logfile while it reads messages from standard input and sends them to /srv/discordfront at the same time.

## Using remotely

Since this program does not fetch previous messages, it might be a good idea to run the server part discordsrv on a machine that is always online, so that all messages gets logged. Since this is plan9, it is almost trivial to still use the client on your local machine, just by using rimport to import the relevant file trees from the server.

Here is what I do (in a seperate script which is run at startup):

```
#!/bin/rc
rfork
rimport -ac -p $serverhost /srv
rimport -c -p $serverhost /usr/glenda/lib/discord
discordacme
```

I don't even notice it is not running locally.

## Notes

* Yes, the names of the scripts suck.
* Yes, there is a lot of missing functionality.
* No, this has not been tested very much, it just seems to do the job well enough for now.

## Authors

- [@Peter's Website / Random notes](https://pmikkelsen.com/plan9/discord)
