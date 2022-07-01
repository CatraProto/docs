---
layout: default
title: Library Configuration
nav_order: 2
parent: Initial configuration
---
# Configuring CatraProto
CatraProto **requires** some user-provided configuration data such as [API credentials](app_configuration.md) to work.
## API Settings
The _ApiSettings_ constructor takes configuration data to be **transmitted to Telegram** regarding the app.
- `apiId` and `apiHash`, which can be retrieved by following these [instructions](app_configuration.md).
- `langPack` is the [language pack](https://telegram.org/blog/translations-iv2) used by the client. Allowed values are: _desktop_, _android_, _ios_, _macos_ and *android_x*.
- `deviceModel`, `appVersion` and `systemVersion` are custom parameters shown to the user in its devices tab. They do not follow any standard.
- `langCode` and `systemLangCode` should follow the ISO 639-1 standard.

Example:
```cs 
var apiSettings = new ApiSettings(45654, "63h4ofskqp045jsl204h", "Raspberry Pi", "1.0", "en", "tdesktop", "en", "1.0");
```
## Session Settings
Session settings are used by CatraProto to know where to store both the session and data received by Telegram.
- The `sessionSerializer` parameter takes an instance of [IAsyncSessionSerializer](https://github.com/CatraProto/Client/blob/master/src/CatraProto.Client/MTProto/Session/Interfaces/IAsyncSessionSerializer.cs) and uses it to serialize the session.

  CatraProto provides a [FileSerializer](https://github.com/CatraProto/Client/blob/master/src/CatraProto.Client/MTProto/Session/Deserializers/FileSerializer.cs).
  It only requires a parameter:
  - `filePath` where the session will be saved.
  You can also implement your own IAsyncSessionSerializer.
- The `databaseSettings` parameter takes an instance of [DatabaseSettings](https://github.com/CatraProto/Client/blob/master/src/CatraProto.Client/MTProto/Settings/DatabaseSettings.cs). 
  
    DatabaseSettings' constructor requires two parameters:
  - `path` which specifies a path where the persistent cache will be created.
  - `peerCacheSize` which is the maximum number of elements for the in-memory peer cache.
- The `sessionName` parameters takes a string specifying the name of the session which will be used to write logs.

To not create confusion, it is recommended to give the session a _.catra_ extension and a _.db_ extension to the database.

Example:
```cs 
var sessionSettings = new SessionSettings(new FileSerializer("data/accountSession.catra"), new DatabaseSettings($"data/accountData.db", 50), "Private account");
```
## Connection Settings
Connection settings are used to specify the default MTProto server, PFS key duration and other connection settings.
- The `defaultDatacenter` parameter takes an instance of [ConnectionInfo](https://github.com/CatraProto/Client/blob/master/src/CatraProto.Client/Connections/ConnectionInfo.cs).   
  ConnectionInfo takes the following parameters:
  - `IPAddress` takes an instance of [IPAddress](https://docs.microsoft.com/en-us/dotnet/api/system.net.ipaddress). It is used to specify the IP of the **default datacenter**. You can get information about your default datacenter by following these [instructions](app_configuration.md).
  - `port` specifies the port to be used in order to connect to the datacenter. You can find it in your app's configuration. You can find it right after the ":" in the IP address.
  - `dcId` used to specify which DC we are connecting to. You can find what DC has been assigned to your app just under the IP address. **If this value is incorrect the library won't be able to perform many operations such as logging in correctly**.
  - `test` specifies whether the DC we are connecting to is a test DC. It is **very important** to make sure this parameter is correct otherwise CatraProto **won't** be able to communicate with the API. 
- The `pfsKeyDuration` parameter specifies in seconds how long the PFS Key should last. When expired, the library will automatically generated a new one. This **does not mean** you will lose access to your session. **One usually doesn't need to change this parameter**. 
- The `connectionTimeout` parameter specifies in seconds how much time must elapse before reconnecting if the server does not reply.
- The `connectionRetry` parameter specifies in seconds how much the library must wait before trying to connect again after a failed attempt. 
- The `ipv6Allowed` parameter specifies whether IPv6 is allowed or not.
- The `ipv6Only` parameter specifies whether only IPv6 must be used or not. This works only for connections that are not the default connection as Telegram only provides IPv4s.

Example:
```cs 
var connectionInfo = new ConnectionInfo(IPAddress.Parse("149.154.167.50"), 443, 2, false);
var connectionSettings = new ConnectionSettings(connectionInfo, 86400, 25, 15, false, false);
```

## Updates settings
**This is an optional parameter, you can skip this part if you're not interested.**\
Currently, the only property exposed by the UpdatesSettings class is `queueUpdates` and when false it disables [this](/usage/receiving_updates.md#how-updates-are-delivered) behaviour.

Example:
```cs 
var updatesSettings = new UpdatesSettings(false);
```

## Combining every setting together
After having specified our configuration data and behaviour preferences we must put everything together. 

To do this, we use the [ClientSettings](https://github.com/CatraProto/Client/blob/master/src/CatraProto.Client/MTProto/Settings/ClientSettings.cs) class. It simply takes the settings we have just created as parameters in the following order: `SessionSettings`, `ApiSettings`, `ConnectionSettings`. Optionally, [UpdatesSettings](#updates-settings) are also accepted as last parameter.

Example:
```cs 
var clientSettings = new ClientSettings(sessionSettings, apiSettings, connectionSettings);
```
