# What is CatraProto?
CatraProto is a fully-asynchronous library that implements the MTProto protocol and the Telegram API. 
This means you can interact with the Telegram API (**as a regular user** as well) **without having any knowledge of the encryption and communication protocol**, it also implements the API so **you won't have to take care of any database implementation or updates handling.**

# Getting started
## Retrieving API credentials
In order to interact with the API, you will need a combination of api_id and api_hash which you can retrieve by logging-in in your account on my.telegram.org.
**Never share these credentials with anyone.**
## Adding CatraProto to your project
After retrieving your API credentials you must add CatraProto to your project. You can do this by referencing CatraProto.Client from NuGet or by downloading the nupkg package from GitHub's Releases page.
