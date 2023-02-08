![Nextsnake logo](docs/_static/logo.svg)
# Nextcore
A low level Discord API wrapper

## Features
### Speed
We try to make the library as fast as possible, without compromising on readability of the code or features.  
<video src="https://user-images.githubusercontent.com/35035079/172221406-b8d618e6-75fd-45d4-a470-62aeeab5bc0a.mp4" />

### Modularity
All the components can easily be swapped out with your own.

### Control
Nextcore offers fine-grained control over things most libraries don't support.  
  
This currently includes:  
- Setting priority for individual requests
- Swapping out components

## Examples
### Ping pong
```py
import asyncio
from os import environ
from typing import cast

from discord_typings import MessageData

from nextcore.gateway import ShardManager
from nextcore.http import BotAuthentication, HTTPClient

# Constants
AUTHENTICATION = BotAuthentication(environ["TOKEN"])

# Intents are a way to select what intents Discord should send to you.
# For a list of intents see https://discord.dev/topics/gateway#gateway-intents
INTENTS = 1 << 9 | 1 << 15  # Guild messages and message content intents.


# Create a HTTPClient and a ShardManager.
# A ShardManager is just a neat wrapper around Shard objects.
http_client = HTTPClient()
shard_manager = ShardManager(AUTHENTICATION, INTENTS, http_client)


@shard_manager.event_dispatcher.listen("MESSAGE_CREATE")
async def on_message(message: MessageData):
    # This function will be called every time a message is sent.
    if message["content"] == "ping":
        # Send a pong message to respond.
        await http_client.create_message(AUTHENTICATION, message["channel_id"], content="pong")


async def main():
    await http_client.setup()

    # This should return once all shards have started to connect.
    # This does not mean they are connected.
    await shard_manager.connect()

    # Raise a error and exit whenever a critical error occurs
    error = await shard_manager.dispatcher.wait_for(lambda: True, "critical")

    raise cast(Exception, error)


asyncio.run(main())
```

More can be seen in the [examples](examples/) directory

## Contributing
Want to help us out? Please read our [contributing](https://nextcore.readthedocs.io/en/latest/contributing/getting_started.html) docs.
