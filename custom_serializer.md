---
layout: default
title: Custom session serializers
nav_order: 7
---
# Custom session serializers
CatraProto ships with `FileSerializer` which allows you to serialize the session to a specific file. In case you don't like this behaviour and want to store your session somewhere else (for example a database) CatraProto allows that by implementing the `IAsyncSessionSerializer`.

## What does a session file contain?
Session files are not to be confused with the database.\
The database stores all information **required to keep a local cache of data** (used for example to resolve _access\_hash_(es)).
The sessions stores the **authentication keys** used to interact with the server and to store the **current logged in user** as well as **the current updates state**.

## How are they serialized?
Session files are nothing more than binary blobs serialized with TL. 

## How to implement the `IAsyncSessionSerializer` interface
The `IAsyncSessionSerializer` interface declares two methods
- `Task<byte[]> ReadAsync(ILogger logger, CancellationToken cancellationToken)` - This method is only used once at startup in order to retrieve the session from your storage and store in memory. **If no session file is present, an empty array must be returned**.
- `Task SaveAsync(byte[] data, ILogger logger, CancellationToken token)` - This method is used to save the session file to your storage and can be called **multiple times** during the lifetime of the `TelegramClient` object, **even concurrently**.

Even though `SaveAsync` can be called concurrently, you will always receive a fully constructed session, but you may need to lock to avoid two (or more) concurrent calls overwriting each other.
If you need an example implementation, you can use the [FileSerializer](https://github.com/CatraProto/Client/blob/master/src/CatraProto.Client/MTProto/Session/Deserializers/FileSerializer.cs).