# Simple Socket Server

This is a simple WebSocket server that supports games between two players.

The server only echoes WebSocket events so the game logic should be delegated to the client.

You have to run `npm install` and `node server.js` to start it.

In order to have the server running for both players, one of you has to start the server with `node server.js` and then, find out his IP in the local network:

```
$ ifconfig
```

For example, if your IP is `192.168.1.17` - this will be the location of the server.

In the client, both players shoud have to point to the IP adress, instead of `localhost`:

```javascript
var socket = new io("http://192.168.1.17:3000");
```

Like this, the server will be shared between the clients.

## HTTP API

The serves exposes the following HTTP methods:

### POST `/createGame`

__The request payload should be in JSON:__

```javascript
{
    "playerName": "RadoRado",
    "socketId": "the_socket_id_here"
}
```

The server responds with JSON with the `gameId`. This id is important!

__Response:__

```javascript
{
    "gameId" : "the_game_id_here"
}
```

### POST `/joinGame`

Once there is a hosted game, the other player can join it.

The player that joins the game should provide the `gameId`!

The request payload is:

```javascript
{
    "playerName": "RadoRado",
    "socketId": "the_socket_id_here",
    "gameId": "the_game_id_here"
}
```

Once a player joins a hosted game, __socket event__ `"start"` is emitted to both players.


### GET `/games`

This is a very simple way to check the current games that are being played right now.

The result is JSON of all games.

## Socket Events

The server emits the following socket events:

### start

Once there is a hosted game and a player joins it, a `start` event is emiited with the following payload:

```javascript
{
    "player1": "Player 1 name here",
    "player2": "Player 2 name here"
}
```

To listen for this event on the client, you can do:

```javascript
socket.on("start", function(data) {
    // data is the payload from the server
});
```

### game_disconnected

If the game has started and one of the players closes his browser, the serves destroys the current game and sends to the ohter player a `"game_disconnected"` message.

This is a handy way not the crash the server or the UI.

### render

The server emits a `render` event with the data, that pass passed from one of the clients with the `move` event.

`render` is simply an echo to the players in the given game with the data from `move`

__The server accepts the following events (Emited by the client):__

### move

The clients emits a `move` event.

The data is not specified and it is up to you to decide what kind of data you should transmit to the server.

The idea behind the `move` event is to send the server your coordinates in order to update the game for the other player.

In the `move` event, the client *must* provide the *gameId*!


When a `move` event is fired, both clients receive `render` from the server with the data, that was passed to the `move` event.

You have to decide the format of the data.

For example:

```javascript
// from player1
socket.emit("move" {
    player1Snake: snake.getSections()
})l
```

### Host vs. Partner

Since the server has no game logic in it, to avoid desynchronization between the clients, they should play different roles!

One of the clients should be host - he does all the game logic and calculations and has a game loop.

The partner only listens to socket events and draws the things accordingly.

This will make you think more about the client's code architecture and design!
