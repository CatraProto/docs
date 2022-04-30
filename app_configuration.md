# App Configuration
The Telegram API **requires** every app to have its own combination of _api_id_ and _api_hash_ and provides DC configuration for every app.
## What are API credentials?
When using the API, the server uses these values in order to **identify what app is being used** by the user to connect to Telegram.

You may think that using official API credentials, retrieved by decompiling official apps, may allow you no to be restricted as often by Telegram. **This is a wrong assumption**. The server can detect such cases and **ban the abusing account**. Keep in mind that this is also **against** the [TOS](https://core.telegram.org/api/terms).

## How to retrieve API credentials
Log in to my.telegram.org and click on _API development tools_. If this is your first time, the website will ask you to create your app.

![image](https://user-images.githubusercontent.com/81624781/166121199-2278c9a4-cdb3-45d7-9700-d0c7d6899308.png)

After filling in the form and having created your application, the website will show you your app's configuration.

For **no reason** you should share your API credentials. If someone uses your credentials to **abuse** the API or **violate** the [Telegram TOS](https://telegram.org/tos) or [API TOS](https://core.telegram.org/api/terms) your account(s) **may get banned**.

## Available MTProto servers
In this same webpage, you have surely noticed that there is a dedicated section containing information (a pair of IP and _Public Key_) regarding the default DCs assigned to your app. RSA Keys are built-in into CatraProto, so you **don't have to care about them**.

It is a good idea to **use the provided IP(s)**. Keep in mind that when using test DCs, this **must** be specified in the _ConnectionInfo_ constructor, as follows:
```cs
var connectionSettings = new ConnectionInfo(IPAddress.Parse(IP), IS_TEST, 443, DC_ID);
```
