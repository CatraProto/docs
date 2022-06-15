# Logging-in
CatraProto sends updates regarding the current session to the `OnSessionUpdateAsync` method of the `IEventHandler` interface. This means you will not just receive updates when the authorization flow changes, but also if the session is invalid. In the examples below, login is implemented in this exact method, but you are free to redirect them wherever you want.

**Note:** This method is invoked in a sequential-manner, this means that you won't receive a new update until you finished processing the new one.

## How it works  
The way login works is pretty easy. Some methods, either return `Task<RpcError?>` or `SomeValue?`. In the first case, null is returned when no error has occured, in the second case null is returned when the operation could not be performed (check the logs).\
The app advances state (i.e from asking for the phone number to asking the sms-code) when the according update is received. 

If `receivedState >= LoginState.LoggedOut` the app must close the instance, delete the session file and create a new instance, because the session is now invalid.

## Handling errors
Some errors are recoverable, which means you can just re-try the query with different parameters, while others are not and the app will receive a new state.\
The only recoverable errors are:
- `PhoneCodeIncorrectError` which is returned when the provided phone code was entered incorrectly. 
- `PasswordIncorrectError` which is returned when the provided password was entered incorrectly.

All other errors are supposed to be shown to the user, even if they are `UnknownError`.

## Example of user login
```cs
public async Task OnSessionUpdateAsync(LoginState loginState)
{
    Task<RpcError?>? task = null;
    if (loginState is LoginState.AwaitingLogin)
    {
        char finalChoice;
        while (true)
        {
            Console.Write("Welcome! Would you like to login as a bot or as a user? (b/u): ");
            var readLine = Console.ReadLine();
            if (readLine is null || readLine.Length != 1 || (readLine[0] != 'u' && readLine[0] != 'b'))
            {
                Console.WriteLine("I'm sorry but this option is invalid");
                continue;
            }

            finalChoice = readLine[0];
            break;
        }

        if (finalChoice == 'u')
        {
            Console.Write("You've selected login as user. Please enter the phone number: ");
            task = _client.LoginManager.UsePhoneNumberAsync(Console.ReadLine() ?? "", new CodeSettings());
        }
        else
        {
            Console.Write("You've selected login as bot. Please enter the bot token: ");
            task = _client.LoginManager.UseBotTokenAsync(Console.ReadLine() ?? "");
        }

        var result = await task;
        if (result is BotTokenIncorrectError)
        {
            Console.WriteLine("I'm sorry, but this token does not work. Make sure you're copying it correctly.");
            return;
        }
        else if (result is PhoneNumberIncorrectError)
        {
            Console.WriteLine("I'm sorry, but this phone number does not work. Make sure you're typing every digit correctly.");
            return;
        }
    }
    else if (loginState is LoginState.AwaitingCode)
    {
        Console.Write("Please insert the login code or r to resend it: ");
        while (true)
        {
            var read = Console.ReadLine();
            if (read is null)
            {
                continue;
            }

            if (read == "r")
            {
                task = _client.LoginManager.ResendCodeAsync();
                var result = await task;
                if (result is null)
                {
                    var codeType = _client.LoginManager.GetCodeTypes()!.Value.CodeType;
                    Console.WriteLine($"The code was resent successfully. Code type {codeType}");
                }
            }
            else
            {
                task = _client.LoginManager.UseLoginCodeAsync(read);
                var result = await task;
                if (result is not null)
                {
                    if (result is PhoneCodeIncorrectError)
                    {
                        Console.WriteLine("The given phone code was invalid, try again or press r to resend it: ");
                        continue;
                    }
                }
                break;
            }
        }
    }
    else if (loginState is LoginState.AwaitingTermsAcceptance)
    {
        Console.WriteLine(_client.LoginManager.GetTermsOfService()!.Text);
        Console.Write("Great! Please accept the terms of service (y/n): ");
        _client.LoginManager.SetTerms(Console.ReadKey().KeyChar == 'y');
        Console.WriteLine();
    }
    else if (loginState is LoginState.AwaitingRegistration)
    {
        Console.WriteLine("It looks like this phone number is not registered.");
        var name = string.Empty;
        while (string.IsNullOrEmpty(name) || string.IsNullOrWhiteSpace(name))
        {
            Console.Write("What's your first name? ");
            name = Console.ReadLine();
        }

        Console.Write($"Ok {name}. What about your last name (Press enter to leave blank)? ");
        var lastName = Console.ReadLine() ?? "";
        task = _client.LoginManager.UseProfileDataAsync(name, lastName);
    }
    else if (loginState is LoginState.AwaitingPassword)
    {
        Console.Write("Yay! Please type your 2FA password here: ");
        while (true)
        {
            task = _client.LoginManager.UsePasswordAsync(Console.ReadLine() ?? "");
            var error = await task;
            if (error is not null)
            {
                if (error is PasswordIncorrectError)
                {
                    Console.WriteLine("Oops! The password is incorrect. Please try again.");
                    continue;
                }

                break;
            }
        }
    }
    else if (loginState is LoginState.LoggedIn)
    {
        var getSelf = await _client.Api.CloudChatsApi.Users.GetSelfAsync();
        if (getSelf.RpcCallFailed)
        {
            Console.WriteLine($"Something went wrong and I could not fetch the bot's profile. Error {getSelf.Error}");
        }
        Console.WriteLine($"Wow! We are logged in! We are: {getSelf.Response}");
    }
    else if (loginState >= LoginState.LoggedOut)
    {
        Console.WriteLine($"Hey! The session is dead. I received the following state: {loginState}");
    }

    if (task is not null)
    {
        var result = await task;
        if (result is not null)
        {
            Console.WriteLine($"I'm sorry, the following error occurred while logging in {result}")
        }
    }
}
```

**Please note that if you use `Console.ReadLine` to prevent the app from quitting, you'll need to do it in another way or to call it only when the login is finished. The latter is done in the [full example](https://github.com/CatraProto/Client/example/ConsoleLogin) using a `TaskCompletionSource`.**

### Code explaination
Even if at first sight this might look very complex and hard to understand, this explaination will try to make it as simple as possible.
Remember that each time an API call is made, it is assigned to the `task` variable so that at the end of the method, it will be awaited and the error, if present, will be shown to the user.

- When the state is _AwaitingLogin_, the app keeps asking the user whether they want to login as a user or as a bot until a a valid answer is received.\
If they choose to login as a user, the app will ask for a phone number, if they choose to login as a bot the app will ask for a bot token.\
The app awaits the API call to see if an error occurred, and if it is one of the known login-related errors it will show a better message to the user.

- When the state is _AwaitingPhoneCode_, the app sent the phone code to the user and keeps asking the user whether they want to try a code or to resend it, until no error is returned or another error (such as flood wait) is received.

- When the state is _AwaitingPassword_, the account is protected by 2FA (Two-Factor Authentication) and the app keeps asking the user for a valid password until login succeeds or an error different from `PasswordIncorrectError` from is returned.

- When the state is _AwaitingTermsAcceptance_ the app shows the user the terms of service and asks the user whether they accept them or not.

- When the state is _AwaitingRegistration_ the app asks the user for a first name and a last name (optionally) to register the user to Telegram.

- When the state is _LoggedIn_, the app calls `GetSelfAsync` to retrieve information about the bot and logs it in json-format. The app also checks whether the API call received an error, because the session could have been terminated while we were still handling this update.

- When the state is >= _LoggedOut_, the app will simply let the user know that it has changed (i.e. the session was terminated and _SESSION_REVOKED_ was received).

## Example of bot login
This is what handling a bot session from a simple console app looks like:
```cs
public async Task OnSessionUpdateAsync(LoginState loginState)
{
    if (loginState is LoginState.AwaitingLogin)
    {
        var r = await _client.LoginManager.UseBotTokenAsync("5525446665:AAH95cataglrak0Mpro9Awto4gud0zbUM");
        if (r is not null)
        {
            Console.WriteLine($"Could not login, the following error occurred: {r}");
        }
    }
    else if (loginState is LoginState.LoggedIn)
    {
        var getSelf = await _client.Api.CloudChatsApi.Users.GetSelfAsync();
        if (getSelf.RpcCallFailed)
        {
            Console.WriteLine($"Something went wrong and I could not fetch the bot's profile. Error {getSelf.Error}");
        }

        Console.WriteLine($"We are logged-in as {r.Response.ToJson()}");
    }
    else if (loginState >= LoginState.LoggedOut)
    {
        Console.WriteLine($"Received state {loginState}");
    }
}
```
### Code explaination
The code is pretty simple, each time the state changes `OnSessionUpdateAsync` is invoked and the app checks whether the state is _AwaitingLogin_, _LoggedIn_ or something else.

- When the state is _AwaitingLogin_, the app tries to login using the bot token provided in the first parameter of the UseBotTokenAsync method, if an error is returned, it is logged to the user otherwise the app waits until it receives a new state.
- When the state is _LoggedIn_, the app calls `GetSelfAsync` to retrieve information about the bot and logs it in json-format. The app also checks whether the API call received an error, because the session could have been terminated while we were still handling this update.
- When the state is >= _LoggedOut_, the app will simply let the user know that it has changed (i.e. the session was terminated and _SESSION_REVOKED_ was received).
