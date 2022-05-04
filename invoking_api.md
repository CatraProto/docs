---
nav_order: 6
title: Calling API methods
---
# Invoking API Methods
In order to interact with the Telegram API, CatraProto exposes API methods in the `Api` property in the `TelegramClient` class.

## Calling the methods
Methods **follow the same namespacing** as they are defined in the TL scheme provided by Telegram. Of course, **cancellation is supported** by providing a [CancellationToken](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtoken).

For example, calling `messages.sendMessage` looks like this:

```cs
await _client.Api.CloudChatsApi.Messages.SendMessageAsync(...);
```

As you may have noticed, the method not only contains _Async_ at the end of it, but it is actually inside the `CloudChatsApi` property. This is because we need to differenciate between the MtProto API (**not exposed to the user**) and the future SecretChats API.

### Simplified API
**CatraProto tries to simplify the API as much as possible** allowing you not to care about implementing a database and other implementation details. This is done by using the `PeerId` struct, exposing `long` directly for IDs where possible, and making some parameters optional.

### An in-depth look at how CatraProto simplifies the API
For example, for each method that requires a `InputPeer` **an overload is generated to simplify the API** and allow you to use the PeerId struct. You can see it in the EventHandler [example](receiving_updates.md#creating-an-eventhandler).

An example of simplified method is `messages.sendMessage`. This is its TL definition:
```
messages.sendMessage#d9d75a4 flags:# no_webpage:flags.1?true silent:flags.5?true background:flags.6?true clear_draft:flags.7?true noforwards:flags.14?true peer:InputPeer reply_to_msg_id:flags.0?int message:string random_id:long reply_markup:flags.2?ReplyMarkup entities:flags.3?Vector<MessageEntity> schedule_date:flags.10?int send_as:flags.13?InputPeer = Updates;
```

This is the overload automatically generated:
```cs
SendMessageAsync(PeerId peer, string message, bool noWebpage = false, bool silent = false, bool background = false, bool clearDraft = false, bool noforwards = false, int? replyToMsgId = null, long? randomId = null, CatraProto.Client.TL.Schemas.CloudChats.ReplyMarkupBase? replyMarkup = null, List<CatraProto.Client.TL.Schemas.CloudChats.MessageEntityBase>? entities = null, int? scheduleDate = null, PeerId? sendAs = null, CatraProto.Client.Connections.MessageScheduling.MessageSendingOptions? messageSendingOptions = null, CancellationToken cancellationToken = default)
```

**Here's a list of changes**:
- The `flags` parameter was removed as **it is handled by CatraProto**.
- The `InputPeer` type was replaced by `PeerId`. This means that **CatraProto will also automatically fetch all the data it needs** about the peer to send the message. 
- The `random_id` parameter was made nullable. It is handled by CatraProto but the user can still specify a value for it if they want.

### What is MessageSendingOptions?
You may have noticed that almost every method has a `messageSendingOptions` parameter. **As of now, this is useless** but it will be used in the future to define custom settings.

## Retrieving results
Each method returns its result through `RpcResponse<T>`. If a method returns a List (or Vector, in the TL) the return type will be `RpcResponse<RpcVector<T>>` where **RpcVector is just a class extending `List<T>`**.

`RpcResponse` exposes the following properties:
- `RpcCallFailed`, true when an error was returned by the API or the message failed to send.
- `Error`, returns an instance of RpcError describing the error occured.
- `Response`, returns the response by the API.

**Exception are raised** in the following cases:
- When the request **was completed successfully** (meaning that no RpcError was returned) **and the `Error` property is used** an `InvalidOperationException` is thrown.
- When the request **was not completed successfully** and the `Response` property is used a `RpcException` containing an `RpcError` is thrown.

The second is more of a **not reccomendend** feature for people who want to handle rpc errors using exceptions. The use of exceptions is **discouraged** as they are **slow** and make the code **harder to read**.

Example of **correct usage**:
```cs
var apiCall = await _client.Api.CloudChatsApi.Messages.SendMessageAsync(PeerId.FromPeer(message.PeerId), "Hello user! This is a reply to your message", replyToMsgId: message.Id);
if (apiCall.RpcCallFailed)
{
    // We log the error as there isn't much we can do except analyze the problem in a later moment
    _logger.Error("Couldn't reply to the user because the following error occurred: {Error}", apiCall.Error);
    return;
}

// Do what we want with the result or ignore it
var result = apiCall.Response;
```

## Handling errors
To provide a cleaner API, CatraProto parses known errors and provides a simple API to interact with them. If an **error is not known to CatraProto, you will receive an instance of UnknownError**.

An example of such error is `FloodWaitError` which exposes the WaitTime property.

Example:
```cs
if (apiCall.RpcCallFailed)
{
    if (apiCall.Error is FloodWaitError error)
    {
        _logger.Error("Couldn't reply to the user because the API requires a wait time of: {Time} seconds", error.WaitTime.TotalSecodns);
    }
    else
    {
        _logger.Error("Couldn't reply to the user because the following error occurred: {Error}", apiCall.Error);
    }
    
    return;
}
```

RpcError also **overloads** `ToString()`. The error is printed in the following format: `[Code][Message][Description]`.
