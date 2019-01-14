Protocol CS1/CS2
==

Current version: 1.0

If anything is changed, the version number will be incremented. Please write this version number down, so you know whether your server/client supports everything. All changes will be noted within the changelog:

    NO CHANGELOG

If anything is unclear about the protocol, feel free to ask me, or reach out to me via email (j.r.vanbommel@student.utwente.nl).


General information
===


**In the protocol session we assumed there would only be two players playing a game, this is (unfortunately) not true. The player can choose to play with up to 4 persons. As a result of this the commands have been changed accordingly.**

If there are arguments which are defined between `<>`, these might be possible arguments. Anything between `()` is added to quickly clarify what arguments are, this should **not** be included in your protocol.

**EXAMPLE**: 
| Command | Arguments |  
| -------- | -------- |
`CONNECT`|`(supported:) <core,chat,challenge,leaderboard,security>`
The line above means that the client could send one of the below:
 - `CONNECT`
 - `CONNECT core`
 -  `CONNECT core,chat`
 - `CONNECT core,chat,challenge`
 - `CONNECT core,chat,challenge,leaderboard`
 - `CONNECT core,chat,challenge,leaderboard,security`


Between CONNECT and the first argument is a space character (ASCII 0x20).  The arguments that follow are comma seperated. Between different parameters is never a space. (NO `core, chat,     challenge,  aaaa`)

Color and rotation encoding
===
![encoding](encoding.png)

Core Protocol: Client
===
These are messages which are sent by the client to the server. The server should handle these incoming messages.
| Command | Arguments | Description | 
| -------- | -------- | -------- |
|`CONNECT`|`(supported:) <core,chat,challenge,leaderboard,security>` | The client sends this to the server to initiate a connection. The arguments let the server know what extra functions are implemented in the client. Everyone should at least send `CONNECT core`, as everyone has the core protocol implemented. Upon receiving this command the server should respond with the `CONNECTED` command.|
|`SETNAME`|`name` | The name that the player wants the server to store and use. This name is then re-used in the `STARTGAME` command send by the server, to let the opponent know against which player the player is playing. The server should respond by sending `HELLO` or `ERROR`. |
|`LOOKINGFORGAME`| `(amount of players:) <2,3,4>` | This is sent once the player wants to play a game, the client announces that the player is looking to play a game. The argument describes the amount of players the game should have, which can be an integer from 2 to 4. The server will then select opponent(s) which are also looking to play a game, and match them. The server should respond by either telling the client to `WAIT` or that a game was found (`STARTGAME`).|
|`MAKEMOVE`| `(tile colors:) RGB, (rotation:) <0,120,240>, (fieldindex:) <0-35>` | The player has made a move in the game and announces this to the server. The first argument (`RGB`) describes which tile was placed. The second argument is the rotation of the tile. The third argument is a number from 0 to 35, which describes on which fieldindex the tile was placed. The server acknowledges this move by sending the `MOVEMADE` command to both the player and the opponents. If the move is invalid an `ERROR` should be send (see errors table).  |
|`CHANGETILE`| `(tile colors:) RGB` | The player was not able to make a move and can choose between missing a turn or exchanging one of the tiles for a new one and passing the turn. This command tells the server that the player decided to exchange one of their tiles. The argument describes the colors of the tiles that were exchanged. The server responds by sending a `PLAYERTILES` to the player and the opponents.  |
|`SKIPTURN`|  | The player was not able to make a move and chose to miss a turn. The server should respond by sending a `PLAYERTURN` command. |
|`DISCONNECT`|  | The player has disconnected from the server, this should also be sent if the application was killed for example by ALT-F4. If the player was playing a game, the server should tell the opponents the disconnect happened with an `ERROR`. |





Core Protocol: Server
===
These are messages which are sent by the server to the client. Your client should handle these incoming messages.
| Command | Arguments | Description |
| -------- | -------- | -------- |
|`CONNECTED`|`(supported by server:)<core,chat,challenge,leaderboard,security>`| A player has succesfully connected to the server. If anything happens, which would mean that a connection would not be succesful (e.g. the client does not tell the server that it understands the core protocol) the server should send an `ERROR`. |
|`ERROR`|`error_code`| If an error happens the server should send this to the client. The error codes are described below. |
|`HELLO`|`name ` |A player has set their name succesfully. We acknowledge this by sending `HELLO` followed by the name. Make sure to check for any invalid characters in names, such as a comma (which will introduce bugs in commands where the player name is used. *-thanks to Jurre for this addition*)|
|`WAIT`||Once a player has announced that they are looking for a game, the server tells the client that they should wait for a game to be available. Once a game is found, the `STARTGAME` command is sent by the server.|
|`STARTGAME`|`(opponent names: ) player1,<player2,player3> ` |A game is found. The arguments are the names of the opponents. Which can be one name, or multiple.|
|`PLAYERTILES`|`(player name:) player, (tiles:) RGB,<RGB,RGB,RGB>` |The tiles that the player posesses. This can be 4 tiles or less: at the end of the game there are no more tiles to grab from the bag.|
|`YOURTURN`| |The player that receives this commands may make the next move. It is their turn to play.|
|`MOVEMADE`|`(tile colors:) RGB, (rotation:) <0,120,240>, (fieldindex:) <0-35>, (player name:) name` |Acknowledges that a move was made succesfully and broadcasts that to other players in the game. The name of the player that made the move is appended to the end.|
|`GAMEOVER`|`(name:) name,(points:) integer, (name:) name, (points:) integer, <..>` |The game has ended. The name of each player is sent, after that a comma and then the amount of points that player has gotten. This is not sorted. |

Example of a game:

![GameExample](examplegamesd.png)

Error codes
---------
```
0  Username is invalid or in use
1  Connection is refused (for some reason)
2  You tried to make a move, but it wasn't your turn yet
3  The move you made was invalid, for example because the field is already used.
4  Opponent disconnected during the game.
BONUS:
5 Invalid chat message, not correctly encoded.
6 Incorrect password.
```


Chat Protocol
===

The following are messages which are sent by the **server to the client**. Your client should handle these incoming messages if you wish to implement the Chat Box extension.
| Command | Arguments | Description |
| -------- | -------- | -------- |
|`INCOMINGMESSAGE`|`(message:)dGVzdA==,(from:)name`|A message has been sent to the player, this message is Base64 encoded. (Example: `test` -> `dGVzdA==`). The server checks whether this message was correctly encoded by decoding it once (and seeing if that results in errors). If it is incorrect it should give an `ERROR`. The name of the player that sent the message is appended to the end.|

The following are messages which are sent by the **client to the server**. 
| Command | Arguments | Description |
| -------- | -------- | -------- |
|`SENDMESSAGE`|`(message:)dGVzdA==,<(to:)name>`|A player wants to send a chat message to all players that are currently in the game. If this is the case, no name has to be specified. If the player wants to send a message specifically to one person, the name of that player has to be specified.   |

WIP: Challenge Protocol
===

The following are messages which are sent by the **server to the client**. Your client should handle these incoming messages if you wish to implement the Challenge extension.
|Command|Arguments|Description|
|--------|--------|--------|
|`LOBBYPLAYERS`|`<name,name,name,...>`|The names of the players that are in the lobby.|
|`CHALLENGED`|`(challenger:)name, <(player1:)name,(player2:)name>`|The player has been challenged by the challenger. If there is more than 1 person being challenged, their names are the second and third argument.|
|`DENIED`|`(player1:)name, <(player2:)name,(player3:)name>`|The arguments should be in the same order as used in the `CHALLENGE` command, below.|
|`REVOKED`|`(challenger:)name, (player1:)name, <(player2:)name,(player3:)name>`|The challenger has revoked the challenge. A player can thus no longer accept or deny a challenge.|


The following are messages which are sent by the **client to the server**. 
| Command | Arguments | Description |
| -------- | -------- | -------- |
|`LEAVELOBBY`||A player wants to leave the lobby. All challenges from the player will be revoked. (it is up to you if you decide this should also  mean that the player is `LOOKINGFORGAME`).  |
|`JOINLOBBY`||A player wants to join a lobby, the player is no longer `LOOKINGFORGAME`. Once the player has joined the server, they will be updated on the players that are also in the lobby. They can then challenge these players.  |
|`CHALLENGE`|`(player1:)name, <(player2:)name,(player3:)name>`|A player wants to challenge another specific player in the lobby to play a game with. (maximum of 3 other people that you can play with) A challenge can be revoked by sending `REVOKE` |
|`ACCEPT`|`(challenger:)name`|The player accepts the challenge to play with the other players. The argument is the name of the player that has sent the challenge request. If the player joins a game, they should be removed from the lobby. |
|`DENY`|`(challenger:)name`|The player denies the challenge to play with the other players. The argument is the name of the player that has sent the challenge request. If multiple players are challenged, and one of them denies, the player |
|`REVOKE`|`(player1:)name, <(player2:)name,(player3:)name>`|The player revokes the challenge to play with the other players. The arguments should be in the same order as used in the `CHALLENGE` command. |

**We should determine what happens if one of the player denies, does that mean that the whole group gets denied, or they could still play with the people that did accept?**

**It would make it easier and cleaner if a person could only have one open challenge, for example when using `REVOKE` and `DENIED`**

WIP: Leaderboard Protocol
===

The following are messages which are sent by the **server to the client**. Your client should handle these incoming messages if you wish to implement the Leaderboard extension.
| Command | Arguments | Description |
| -------- | -------- | -------- |
|`LEADERBOARD`|`BASE64(name,score,datetime),<BASE64(name,score,datetime)>,...`|An entry of the requested leaderboard. Each entry is Base64 encoded. The encoded entries are comma seperated. The datetime is a UNIX timestamp. If the player has requested a daily average, the name may be `Daily Average` and the datetime could be the current datetime, as this information is not relevant.|

The following are messages which are sent by the **client to the server**. 
| Command | Arguments | Description |
| -------- | -------- | -------- |
|`GETLEADERBOARD`|`<[top,000] [above,000,000] [below,000,000] [avgscore] [dailyavg] [dailybest]>`|Request information  from the leaderboard.

 - `top,5` will return the top 5 best scores
 - `above,5000,2` will return the 2 scores above 5000 points
 - `below,1000,3` will return the 3 scores below 1000 points.
 - `avgscore` will return the average score over all games
 - `dailyavg` will return the average score over all games on that day
 - `dailybest` will return the best score over all games of that day
   
**There is also an item for the team results. It was unclear whether this had to be implemented, as there aren't any teams in the game.**

WIP: Security Protocol
===

The following are messages which are sent by the **server to the client**. Your client should handle these incoming messages if you wish to implement the Security extension.
| Command | Arguments | Description |
| -------- | -------- | -------- |
|`LOGINVALID`|`(message:)dGVzdA==,(from:)name`|If the password was correct, otherwise an `ERROR` is sent.|

The following are messages which are sent by the **client to the server**. 
| Command | Arguments | Description |
| -------- | -------- | -------- |
|`LOGIN`|`name, password`|Password is encrypted with SHA256 with UTF8 as the charset. The resulting bytes are then encoded using Base64. This resembles the password.


**I suggest that we revise this protocol a bit during the next session, as it sounds too simple to get all bonus points**
For example we might be able to add a `REGISTER` command, as this is more user friendly than a text file on the server. We could also let the server send a salt/challenge over the network, which the client then encrypts with the SHA256 hash, in order to make sure that the password can't be easily recovered (hash differs every time). We could also make a `GETAUTHENTICATEDUSERS` command, which shows us all authenticated users.
