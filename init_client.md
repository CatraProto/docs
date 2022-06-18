---
layout: default
title: Using the library
nav_order: 5
---
## Using the library
Now that the settings were configured properly and the logger is created we can initialize the client.

## Instanciating the client
The `TelegramClient` constructor takes two arguments, the first takes a ClientSettings [instance](library_configuration.md) and the second takes a [logger](logging.md).

Example:
```cs
var client = new TelegramClient(settings, logger);
```

## Setting the event handler
After having declared our event handler we must set it by calling `client.SetEventHandler()`. If this method is called when an event handler is already set it will throw an `InvalidOperationException`.

Example:
```cs
client.SetEventHandler(new EventHandler(client));
```


## Initialize the client
After creating the instance, we can use the `client.InitClientAsync()` method to initialize and start up the library.

This method returns a ClientState enum which represents the following states:
- `Working`, when the session file was read successfully.
- `Corrupted`, this means the session file is corrupted. The session must be recreated and the login operation must be performed again.

Example:
```cs
var state = await client.InitClientAsync();
```

## Saving information
At any time, the session can be manually saved by calling `client.ForceSaveAsync();`. 

To not lose login information, it is also recommended to **call this method after logging in**, and to avoid re-fetching older updates, **this method must be called upon disposal**.

## Disposing the client
In order to properly shutdown, the instance must be disposed.

You can do this, by declaring the instance with `await using`
```cs
await using var client = new TelegramClient(settings, logger);
```

Or by manually calling `client.DiposeAsync()`
```cs
await client.DisposeAsync();
```
In both cases **don't forget** to call `client.ForceSaveAsync()` before disposal.
