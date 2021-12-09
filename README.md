# Renting Apartments Application
#### Running on nextjs react framework ultimate solution for house renting

This repository contains the backend and frontend parts

<details>
<summary>How do I dropdown?</summary>
<br>
This is how you dropdown.
</details>

## Technologies

- Languages:
    - Server: NodeJS (Typescript supersetted)
    - Client: JS (Typescript supersetted)
- Frameworks:
    - Server: ExpressJS + Firebase
    - Client: NextJS + ReactJS

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
```
CONTENTS OF THIS FILE
---------------------

 * Introduction
 * Installation
 * Starting the Bot
 * Starting the Bot with Drush
 * Starting the Bot with bot_start.php
 * Using the Bot
 * IRC Message Hooks
 * Design Decisions
 * Technongies


INTRODUCTION
------------

Current Maintainer: Morbus Iff <morbus@disobey.com>

Druplicon is an IRC bot that has been servicing #drupal, #drupal-support,
and many other IRC channels since 2005, proving itself an invaluable resource.
Originally a Perl Bot::BasicBot::Pluggable application coded by Morbus Iff,
he always wanted to make the official #drupal bot an actual Drupal module.

This is the fruit of these labors. Whilst the needs of Druplicon are driving
the future and design of the module, this is intended as a generic framework
for IRC bots within Drupal, and usage outside of Druplicon is encouraged.


INSTALLATION
------------

The bot.module is not like other Drupal modules and requires a bit more
effort than normal to get going. Unlike a regular Drupal page load, an
IRC bot has to run forever and, for reasons best explained elsewhere, this
entails running the bot through a shell, NOT through web browser access.

1. This module REQUIRES Net_SmartIRC, a PHP class available from PEAR.
   In most cases, you can simply run "pear install Net_SmartIRC".

2. Copy this bot/ directory to your sites/SITENAME/modules directory.

3. Enable the module(s) and then configure them at admin/config/bot.


STARTING THE BOT
----------------

If you have Drush installed, the following commands are available:

  drush bot-start
  drush bot-status
  drush bot-status-reset
  drush bot-stop

Note, however, that bot.module does use your site's URL in some commands, but
Drush doesn't ever know your URL by default. To ensure proper reporting by the
bot, you'll need to either set your $base_url in settings.php, or tell Drush
your URL with "--uri=http://example.com/". If you don't, the bot will report
all URLs as "http://default/". I prefer setting the URL in settings.php.

To start the bot as a background process, use:

  nohup drush bot-start &

Stopping the bot is accomplished with:

  drush bot-stop

If the bot crashes, is killed, Ctrl-C'd, or otherwise improperly interrupted,
the internal connection status will be stuck in a "connected" state, and
Drush will refuse to "bot-start". You can force a status reset with "drush
bot-status-reset".

IF YOU DO NOT HAVE DRUSH INSTALLED, scripts/bot_start.php is a simple wrapper
around Drupal and the IRC network libraries. To run the bot, move to the
scripts directory and issue the following command:

  php bot_start.php --root /path/to/drupal --url http://www.example.com

--root refers to the full path to your Drupal installation directory and
allows you to execute bot_start.php without moving it to the root directory.
--url is required (and is equivalent to Drupal's base URL) to trick Drupal
into thinking that it is being run as through a web browser. It sets all
the required Drupal environment variables.

If you want to run the bot as a background process, try:

  nohup php bot_start.php --root /path/to/drupal --url http://www.example.com &


USING THE BOT
-------------

Your bot is now started and is trying to connect.

Once your bot is connected, you can query it for more information:

   <Morbus> YOUR_BOTNAME: help

   <YOUR_BOTNAME>: Detailed information is available by asking for
     "help <feature>" where <feature> is one of: Botagotchi, Drupal URLs,
     Factoids, Seen.

   <Morbus> YOUR_BOTNAME: help Seen

   <YOUR_BOTNAME> If someone asks "seen Morbus", the bot will report the
     last time they've been seen, where, and what their last known message
     was. Directly addressing the bot will also allow the more complex
     syntax of "seen Morbus? seen d8uv?", "have you seen sbp?" and similar
     forms. * can be used as a wildcard, but only with a minimum of three
     other characters. A maximum of three results are displayed for any
     one request.

You can also go to http://www.SITENAME.com/bot/ for a web-based version
of all the feature help available through the IRC syntax above.


IRC MESSAGE HOOKS
-----------------

The following message types are supported by Net_SmartIRC:

  UNKNOWN     CHANNEL  QUERY     CTCP         NOTICE        WHO
  JOIN        INVITE   ACTION    TOPICCHANGE  NICKCHANGE    KICK
  QUIT        LOGIN    INFO      LIST         NAME          MOTD
  MODECHANGE  PART     ERROR     BANLIST      TOPIC         NONRELEVANT
  WHOIS       WHOWAS   USERMODE  CHANNELMODE  CTCP_REQUEST  CTCP_REPLY

A module may create a function in the form of:

  MODULENAME_irc_msg_MESSAGETYPE

such that a module named "bot_example" could respond or act upon all channel
messages with a function called bot_example_irc_msg_channel(). Passed to
this function is $data, an object reference that contains the message, who
said it, in what channel, and more.

Modules can respond to the user or channel with bot_message($to, $message),
where $to is either a channel name ("#drupal") or user ("Morbus"). Other
IRC actions are demonstrated in the modules shipped with this package. In a
worse case scenario (ie., there's no helper function in bot.module that will
accomplish your desired tasks), you can use "global $irc;" to get the actual
Net_SmartIRC object that represents the IRC connection. Under the most ideal
conditions, you'd contribute back a patch to bot.module that'd let you
accomplish your needs without using the $irc global. Generally speaking,
try not to use the $irc global.

There is another hook available called irc_bot_reply_message (such that, in
our above example, it'd be bot_example_irc_bot_reply_message()). This function
allows you to act whenever the bot sends a message. This was added primarily
to allow us to log bot responses in bot_log.module. If you use this, be sure
NOT to use bot_message() within your implementation, else you'll cause an
infinite loop. Another hook, irc_bot_reply_action, does the same for actions.


DESIGN DECISIONS
----------------

 * We do not enforce command prefixes such as !.

 * We do not enforce direct addressing like "botname: <command>".

Since the entire raw IRC message is passed off to each module, you are more
than welcome to enforce either of the above in your own code. Note that you
WILL have to insure that "botname: <command>" and simply "<command>" both do
as you'd expect - we do not remove "botname:" from the start of messages (and
thus a simple test of "^command$" will fail if the bot is addressed). Most
of our shipped modules cater to these two possibilities.

This is an IRC bot... in PHP. PHP is not especially awesome with regards to
memory management, and it certainly wasn't intended to run a script for any
respectable period of time (like, say, longer than the default 30 seconds).
Likewise, there's no way to uninclude a file, so any change to the loaded
modules (either codewise or enabled/disabled) will require the bot to be
restarted entirely.

Love the limitations, and craziness, of this project.
       
TECHNOLOGIES
---------------

- Languages:
    - Server: NodeJS (Typescript supersetted)
    - Client: JS (Typescript supersetted)
- Frameworks:
    - Server: ExpressJS + Firebase
    - Client: NextJS + ReactJS


