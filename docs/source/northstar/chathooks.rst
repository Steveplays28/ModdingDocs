Chathooks
=========

In ``v1.6.0`` we are introducing a completely new chathook API.
In this document, we provide the usage of this API.
For an example of chathooks in use, check out EmmaM's `OwOfier mod <https://github.com/emma-miler/OwOfier/>`_

Client chat API
---------------

Callbacks
^^^^^^^^^

The client chat callbacks allow you to intercept chat messages and modify or block them.

.. _clclient_messagestruct:

``struct ClClient_MessageStruct { ... }``
"""""""""""""""""""""""""""""""""""""""""

Contains details on a chat message to be displayed. You can receive one of these by adding a chat callback with
:ref:`AddCallback_OnReceivedSayTextMessage <addcallback_onreceivedsaytextmessage>`.

**Fields:**

 - ``string message`` - the text sent by the player.
 - ``entity player`` - the player who sent the chat.
 - ``bool isTeam`` - whether this chat has a ``[TEAM]`` tag.
 - ``bool isDead`` - whether this chat has a ``[DEAD]`` tag.
 - ``bool isWhisper`` - whether this chat has a ``[WHISPER]`` tag.
 - ``bool shouldBlock`` - if true, this chat will not be displayed.

.. _addcallback_onreceivedsaytextmessage:

.. cpp:function:: _void AddCallback_OnReceivedSayTextMessage(callbackFunc)

    Adds a callback that will be run when a chat message is received from the server. This will only be triggered for
    messages from players, not server messages.

    The provided function should accept a :ref:`ClClient_MessageStruct <clclient_messagestruct>` and return an optionally modified copy of it. When a
    chat message is received, each registered callback is run in sequence to modify or block the message.

    **Example:**

    .. code-block:: javascript

        ClClient_MessageStruct function MyChatFilter(ClClient_MessageStruct message)
        {
            if (message.message.find("nft") != null)
            {
                message.shouldBlock = true
            }
            
            message.message = StringReplace(message.message, "yes", "no", true, true)
            
            return message
        }

        void function MyModInit()
        {
            AddCallback_OnReceivedSayTextMessage(MyChatFilter)
        }


Writing to Chat
^^^^^^^^^^^^^^^

A handful of functions are provided to write messages in local chat windows. These do not send messages to other
players, they only display them locally.

.. cpp:function:: void Chat_GameWriteLine(string text)

    Writes a line of text in local game chat panels. Supports :ref:`ANSI escape codes <ansi_escape>`.

    **Example:**

    .. code-block:: javascript

        void function OnGameStarted()
        {
            Chat_GameWriteLine("You got this, " + GetLocalClientPlayer().GetPlayerName() + "!")
        }


.. cpp:function::  void Chat_GameWrite(string text)

    Appends text to local game chat panels. Supports :ref:`ANSI escape codes <ansi_escape>`.

    **Example:**

    .. code-block:: javascript
        
        void function InitialiseHEVSuit()
        {
            Chat_GameWriteLine("SENSOR ARRAYS-")
            ActivateSensorArrays()
            Chat_GameWrite("ACTIVATED")
            wait 1
            Chat_GameWriteLine("BIOMETRIC MONITORING SYSTEMS-")
            ActivateBiometricMonitoringSystems()

.. cpp:function:: void Chat_NetworkWriteLine(string text)

    Writes a line of text in local network chat panels. Supports :ref:`ANSI escape codes <ansi_escape>`.

    **Example:**

    .. code-block:: javascript

        void function MyModInit()
        {
            Chat_NetworkWriteLine("MyMod v1.0.0 is good to go!")
        }


.. cpp:function:: void Chat_NetworkWrite(string text)

    Appends text to local network chat panels. Supports :ref:`ANSI escape codes <ansi_escape>`.

    **Example:**

    .. code-block:: javascript

        void function OnButtonPressed()
        {
            Chat_NetworkWrite("Connecting in 3...")
            wait 1
            Chat_NetworkWrite("2...")
            wait 1
            Chat_NetworkWrite("1...")
            wait 1
            Chat_NetworkWrite("0")
            Connect()
        }



Server Chat API
---------------

Callbacks
^^^^^^^^^

The server chat callbacks allow you to intercept incoming chat messages and modify or block them.

.. _clserver_messagestruct:

``struct ClServer_MessageStruct { ... }``
"""""""""""""""""""""""""""""""""""""""""

Contains details on an incoming chat message. You can receive one of these by adding a chat callback with
:ref:`AddCallback_OnReceivedSayTextMessage <addcallback_onreceivedsaytextmessage_server>`.

**Fields:**
 - ``string message`` - the text sent by the player.
 - ``entity player`` - the player who sent the chat.
 - ``bool isTeam`` - whether this chat is only sent to the player's team.
 - ``bool shouldBlock`` - if true, this chat will not be sent.

.. _addcallback_onreceivedsaytextmessage_server:

.. cpp:function:: void AddCallback_OnReceivedSayTextMessage(callbackFunc)

    Adds a callback that will be run when a chat message is received from a player. This will not be fired for custom
    messages sent by server mods.

    The provided function should accept a :ref:`ClServer_MessageStruct <clserver_messagestruct>` and return an
    optionally modified copy of it. When a chat message is received, each registered callback is run in sequence to modify
    or block the message.

    **Example:**

    .. code-block:: javascript

        ClServer_MessageStruct function MyChatFilter(ClServer_MessageStruct message)
        {
            if (message.message.find("nft") != null)
            {
                message.shouldBlock = true
            }
            
            message.message = StringReplace(message.message, "yes", "no", true, true)
            
            return message
        }
        void function MyModInit()
        {
            AddCallback_OnReceivedSayTextMessage(MyChatFilter)
        }


Custom Messages
^^^^^^^^^^^^^^^

With custom messages you can send chat messages at any time, to all players or to specific players.

.. cpp:function:: void Chat_Impersonate(entity player, string text, bool isTeamChat)

    Displays a chat message as if the player sent it. Only use this when the player has performed a clear action to send a
    chat message.

    **Parameters:**

    - ``entity player`` - the player that the chat message will appear to be from.
    - ``string text`` - the contents of the chat message. Supports :ref:`ANSI escape codes <ansi_escape>` for colors.
    - ``bool isTeamChat`` - whether this chat is only sent to the player's team.

    **Example:**

    .. code-block:: javascript

        void function OnSayRedCommand(entity player, string text)
        {
            Chat_Impersonate(player, "red text -> \x1b[31m" + text)
        }


.. cpp:function:: void Chat_PrivateMessage(entity fromPlayer, entity toPlayer, string text, bool whisper)

    Sends a private chat message from a player that is only displayed to one other player. Only use this when the player has
    performed a clear action to send a chat message.

    **Parameters:**

    - ``entity fromPlayer`` - the player the message will be from.
    - ``entity toPlayer`` - the player that the message will be shown to.
    - ``string text`` - the contents of the chat message. Supports :ref:`ANSI escape codes <ansi_escape>` for colors.
    - ``bool whisper`` - if true, ``[WHISPER]`` will be displayed before the message to indicate the message is private.

    **Example:**

    .. code-block:: javascript

        void function OnSendToFriendsCommand(entity fromPlayer, string text)
        {
            array<entity> friends = GetPlayerFriends(fromPlayer)
            foreach (friend in friends)
            {
                Chat_PrivateMessage(fromPlayer, friend, text, true)
            }
        }


.. cpp:function:: void Chat_ServerBroadcast(string text)

    Displays a server message to all players in the chat.

    **Parameters:**

    - ``string text`` - the contents of the chat message. Supports :ref:`ANSI escape codes <ansi_escape>` for colors.

    **Example:**

    .. code-block:: javascript

        void function RestartServerThread()
        {
            // wait one hour
            wait 3600
            Chat_ServerBroadcast("Server will be shut down in \x1b[93m5 seconds")
            wait 1
            Chat_ServerBroadcast("Server will be shut down in \x1b[93m4 seconds")
            wait 1
            Chat_ServerBroadcast("Server will be shut down in \x1b[93m3 seconds")
            wait 1
            Chat_ServerBroadcast("Server will be shut down in \x1b[93m2 seconds")
            wait 1
            Chat_ServerBroadcast("Server will be shut down in \x1b[93m1 second")
            wait 1
            StopServer()
        }


.. cpp:function:: void Chat_ServerPrivateMessage(entity toPlayer, string text, bool whisper)

    Sends a server message to a specific player in the chat.

    **Parameters:**

    - ``entity toPlayer`` - the player that the message will be shown to.
    - ``string text`` - the contents of the chat message. Supports :ref:`ANSI escape codes <ansi_escape>` for colors.
    - ``bool whisper`` - if true, ``[WHISPER]`` will be displayed before the message to indicate the message is private.

    **Example:**

    .. code-block:: javascript
        void function OnBanCommand(entity player, array<string> args)
        {
            if (!PlayerIsModerator(player))
            {
                Chat_ServerPrivateMessage(player, "You do not have the permissions to perform this command.", true)
                return
            }
            
            BanPlayerByName(args[0])
        }


.. _ansi_escape:

ANSI Escape Codes
-----------------

All messages support ANSI escape codes for customising text color. These are commands in strings that have special
meaning. For example, the string:

.. code-block::

    Hello world, \x1b[31mthis text is red\x1b[0m. And \x1b[34mthis text is blue\x1b[0m.


``\x1b`` is a special character that Squirrel (and other languages) replace with a reserved ASCII character. For future
reference this will be referred to with ``ESC`` (e.g. setting red text is ``ESC[31m``).

The following commands are available:

 - ``ESC[0m`` and ``ESC[39m`` - reset text formatting
 - ``ESC[30-37m``, ``ESC[90-97m`` - set to one of `the available color presets <https://en.wikipedia.org/wiki/ANSI_escape_code#3-bit_and_4-bit>`_.
 - ``ESC[38;5;Xm`` - set to one of `the available 8-bit colors <https://en.wikipedia.org/wiki/ANSI_escape_code#8-bit>`_.
 - ``ESC[38;2;R;G;Bm`` - set to an RGB color, with ``R``, ``G`` and ``B`` in the range 0-255.
 - ``ESC[110m`` - set to chat text color
 - ``ESC[111m`` - set to friendly player name color
 - ``ESC[112m`` - set to enemy player name color
 - ``ESC[113m`` - set to network name color