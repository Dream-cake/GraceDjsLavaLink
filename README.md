# GraceDjsLavaLink
A lavalink client for Discord.js, created by mrjacz, edited to work with Grace!

## Documentation (NEW)
[**mrjacz.github.io/discord.js-lavalink**](https://mrjacz.github.io/discord.js-lavalink/)

## Documentation (OLD)
[**mrjacz.github.io/discord.js-lavalink**](https://mrjacz.github.io/discord.js-lavalink/)

## Installation

**For stable**
```bash
# Using yarn
yarn add ssh://github.com/Dream-cake/GraceDjsLavaLink

# Using npm
npm install dream-cake/gracedjslavalink
```

**For Development**
```bash
# Using yarn
yarn add ssh://github.com/Dream-cake/GraceDjsLavaLink

# Using npm
npm install dream-cake/gracedjslavalink
```

## LavaLink configuration
Download from [the CI server](https://ci.fredboat.com/viewLog.html?buildId=lastSuccessful&buildTypeId=Lavalink_Build&tab=artifacts&guest=1)

Put an `application.yml` file in your working directory. [Example](https://github.com/Frederikam/Lavalink/blob/master/LavalinkServer/application.yml.example)

Run with `java -jar Lavalink.jar`

# Implementation
Start by creating a new `PlayerManager` passing an array of nodes and an object with `user` the client's user id and `shards` The total number of shards your bot is operating on.

```js
const { PlayerManager } = require("discord.js-lavalink");

const nodes = [
    { host: "localhost", port: 2333, password: "youshallnotpass" }
];

const manager = new PlayerManager(client, nodes, {
    user: client.user.id, // Client id
    shards: shardCount // Total number of shards your bot is operating on
});
```

Resolving tracks using LavaLink REST API

```js
const fetch = require("node-fetch");
const { URLSearchParams } = require("url");

async function getSongs(search) {
    const node = client.player.nodes.first();

    const params = new URLSearchParams();
    params.append("identifier", search);

    return fetch(`http://${node.host}:${node.port}/loadtracks?${params.toString()}`, { headers: { Authorization: node.password } })
        .then(res => res.json())
        .then(data => data.tracks)
        .catch(err => {
            console.error(err);
            return null;
        });
}

getSongs("ytsearch:30 second song").then(songs => {
    // handle loading of the tracks somehow ¯\_(ツ)_/¯
});
```

Joining and Leaving channels

```js
// Join
manager.join({
    guild: guildId, // Guild id
    channel: channelId, // Channel id
    host: "localhost" // lavalink host, based on array of nodes
}).then(player => {
    player.play(track); // Track is a base64 string we get from Lavalink REST API

    player.once("error", error => console.error(error));
    player.once("end", data => {
        if (data.reason === "REPLACED") return; // Ignore REPLACED reason to prevent skip loops
        // Play next song
    });
});

// Leave voice channel and destory Player
manager.leave(guildId); // Player ID aka guild id
```

For a proper example look at [**example/app.js**](https://github.com/Dream-Cake/GraceDjsLavaLink/blob/master/example/app.js)
