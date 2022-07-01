---
layout: default
title: Library initialization
nav_order: 4
parent: Using the library
---
## Initializing the library
Now that the settings were configured properly and the logger is created we can initialize the client.

## Instanciating the client
The `TelegramClient` constructor takes two arguments, the first takes a ClientSettings [instance](../configuration/library_configuration.md) and the second takes a [logger](../configuration/logging.md).\
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

## Disposing the client
In order to properly shutdown, the instance must be disposed.\
You can do this, by declaring the instance with `await using`:
```cs
await using var client = new TelegramClient(settings, logger);
```
Or by manually calling `client.DiposeAsync()`
```cs
await client.DisposeAsync();
```
<div class="warning">
<p><b>Warning:</b> Before disposing the client it is recommended to save the session in order not to lose the current updates state and other important information.</p>
