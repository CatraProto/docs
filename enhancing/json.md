---
layout: default
title: JSON Serialization
nav_order: 2
parent: Enhancing your experience
---
# Json Serialization
Sometimes you might find it useful to convert TL Objects to human-readble text, this is where the `ToJson()` extension located in the `CatraProto.Client.TL` namespace comes into play.\
It also offers an optional parameter to enable/disable formatting of the resulting text.

# Example
```cs
_logger.Information("Hi! I received the following object but couldn't process it: {Obj}", myMessage.ToJson());
```
<div class="note">
<p><b>Note:</b> As of now, the resulting text does not contain the constructors' name.</p>
</div>

# How does it work?
Each property of every object has a `Newtonsoft.Json.JsonProperty` attribute, where the name is the TL-name.
Example: 
```cs
[Newtonsoft.Json.JsonProperty("menu_button")]
public sealed override CatraProto.Client.TL.Schemas.CloudChats.BotMenuButtonBase MenuButton { get; set; }
```
