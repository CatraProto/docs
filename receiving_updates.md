---
title: Receiving Messages (Updates)
nav_order: 5
---
# Receiving Updates
Receiving messages means in fact **receiving an update**. Updates **do not only communicate whether a new message was sent, edited or received but many, many other things** such as a user typing or **in the case of bots** when a user's permissions in a chat have changed.

**CatraProto takes care of the correct handling of updates so you don't have to make sure there are no gaps or duplicate updates**.

## Creating our EventHandler
To handle updates a class extending the [IEventHandler](https://github.com/CatraProto/Client/blob/master/src/CatraProto.Client/Updates/Interfaces/IEventHandler.cs) interface must be defined.

Example:
```cs
public class EventHandler : IEventHandler
{
    private readonly TelegramClient _client;

    public EventHandler(TelegramClient client)
    {
        _client = client;
    }

    public async Task OnUpdateAsync(UpdateBase update)
    {
        if (update is UpdateNewMessage { Message: Message { Out: false } message })
        {
            var asPeerId = PeerId.FromPeer(message.PeerId);
            if (asPeerId.Type is not PeerType.User)
            {
                // We only want to reply to messages sent in private chat.
                return;
            }

            await _client.Api.CloudChatsApi.Messages.SendMessageAsync(asPeerId, "Hello user. Thank you for contacting me and trying CatraProto!");
        }
    }
}
```

The following code checks whether the update is an instance of `UpdateNewMessage` and that the Message inside it is an instance of `Message` with the `Out` property set to false. If the check is successfull, it makes sure the message was received inside a private chat and then replies to the user.

## Avoiding older messages
When first logging in, all updates from when the client was created are fetched. This may lead to undersired behaviour as you may not want your bot to start replying to older messages. To mitigate this, you can check the message's date.
Example:
```cs
if (message.Date - StartTime < 0)
{
   _logger.Information("Skipping message {Id} because it's old.", message.Id);
   return;
}
```
Where `StartTime` is the unix timestamp value of when you started the script. You can get it by calling `DateTimeOffset.UtcNow.ToUnixTimeSeconds()` at startup and save it in a static field.

## How updates are delivered
In order to **avoid flooding** the event handler some internal queues are used. There is a queue for each peer and **if an update is not bound to a peer it is added to a common queue**. This means that **each update is separated based on which chat it was sent in** and **it will wait** to trigger your event handler **if an updated from the same chat is still being processed.**
