This repo is a fork from main repo and will usually have new features bundled faster than main repo (and maybe bundle some bugs, too).



# Nova Facebook Chat API
_@syntaxt0x1c_

[Need help with nova-fca? Reach out on GitHub.](https://github.com/syntaxt0x1c)



# Added features:
* Once the error is detected it will automatically relogin the appstate.
* If the appstate itself is logged out it will automatically logged out and you can resubmit it again, But aside of that, if it has an automated behavior, it will relogin then it will dismiss automatically and refreshes the login

![Image](https://i.imgur.com/rqYXgBn.jpeg)

* Added a feature where if the account is locked/suspended, it will stop the login process and shows the information and why it was locked/suspended.

![Image](https://i.imgur.com/R0lzR6R.jpeg)
![Image](https://i.imgur.com/PPE3fB5.jpeg)
No need to put on listenMqtt.

## Also some other features:
* Added api.nova.relogin()
* Added api.stopListenMqtt()
* Added api.getRegion()
* Added api.setProfileGuard()
* Added api.addFunctions()
* Fixed api functions including api.follow()
* Changed npmlog to normal console.log due to render log issues
* Added a detection if it's locked or suspended (will show the information about the lock/suspension)
* Tested on Mirai/Autobot (*try on Xavia or Goat*)


Facebook now has an official API for chat bots [here](https://developers.facebook.com/docs/messenger-platform).



This API is the only way to automate chat functionalities on a user account. We do this by emulating the browser. This means doing the exact same GET/POST requests and tricking Facebook into thinking we're accessing the website normally. Because we're doing it this way, this API won't work with an auth token but requires the credentials of a Facebook account.

_Disclaimer_: We are not responsible if your account gets banned for spammy activities such as sending lots of messages to people you don't know, sending messages very quickly, sending spammy looking URLs, logging in and out very quickly... Be responsible Facebook citizens.

## Install
If you just want to use nova-fca, you should use this command:
```bash
npm install nova-fca@latest
```
It will download `nova-fca` from NPM repositories

## Example Usage
```javascript
const login = require("nova-fca");

// Create simple echo bot
login({
  appState: []
}, (err, api) => {
    if (err) return console.error(err);
    api.listenMqtt((err, event) => {
        api.sendMessage(event.body, event.threadID);
    });
});
```

Result:

<img width="517" alt="screen shot 2016-11-04 at 14 36 00" src="https://cloud.githubusercontent.com/assets/4534692/20023545/f8c24130-a29d-11e6-9ef7-47568bdbc1f2.png">


## Main Functionality

### Sending a message
#### api.sendMessage(message, threadID[, callback][, messageID])

Various types of message can be sent:
* *Regular:* set field `body` to the desired message as a string.
* *Sticker:* set a field `sticker` to the desired sticker ID.
* *File or image:* Set field `attachment` to a readable stream or an array of readable streams.
* *URL:* set a field `url` to the desired URL.
* *Emoji:* set field `emoji` to the desired emoji as a string and set field `emojiSize` with size of the emoji (`small`, `medium`, `large`)

Note that a message can only be a regular message (which can be empty) and optionally one of the following: a sticker, an attachment or a url.

__Tip__: to find your own ID, you can look inside the cookies. The `userID` is under the name `c_user`.

__Example (Basic Message)__
```js
const login = require("nova-fca");

login({email: "FB_EMAIL", password: "FB_PASSWORD"}, (err, api) => {
    if(err) return console.error(err);

    var yourID = "000000000000000";
    var msg = "Hey!";
    api.sendMessage(msg, yourID);
});
```

__Example (File upload)__
```js
const login = require("nova-fca");

login({email: "FB_EMAIL", password: "FB_PASSWORD"}, (err, api) => {
    if(err) return console.error(err);

    // Note this example uploads an image called image.jpg
    var yourID = "000000000000000";
    var msg = {
        body: "Hey!",
        attachment: fs.createReadStream(__dirname + '/image.jpg')
    }
    api.sendMessage(msg, yourID);
});
```

------------------------------------
### Saving session.

To avoid logging in every time you should save AppState (cookies etc.) to a file, then you can use it without having password in your scripts.

__Example__

```js
const fs = require("fs");
const login = require("nova-fca");

var credentials = {email: "FB_EMAIL", password: "FB_PASSWORD"};

login(credentials, (err, api) => {
    if(err) return console.error(err);

    fs.writeFileSync('appstate.json', JSON.stringify(api.getAppState()));
});
```

Alternative: Use [c3c-fbstate](https://github.com/c3cbot/c3c-fbstate) to get fbstate.json (appstate.json)

------------------------------------

### Listening to a chat
#### api.listenMqtt(callback)

Listen watches for messages sent in a chat. By default this won't receive events (joining/leaving a chat, title change etc…) but it can be activated with `api.setOptions({listenEvents: true})`. This will by default ignore messages sent by the current account, you can enable listening to your own messages with `api.setOptions({selfListen: true})`.

__Example__

```js
const fs = require("fs");
const login = require("nova-fca");

// Simple echo bot. It will repeat everything that you say.
// Will stop when you say '/stop'
login({appState: JSON.parse(fs.readFileSync('appstate.json', 'utf8'))}, (err, api) => {
    if(err) return console.error(err);

    api.setOptions({listenEvents: true});

    var stopListening = api.listenMqtt((err, event) => {
        if( err) return console.error(err);

        api.markAsRead(event.threadID, (err) => {
            if(err) console.error(err);
        });

        switch(event.type) {
            case "message":
                if(event.body === '/stop') {
                    api.sendMessage("Goodbye…", event.threadID);
                    return stopListening();
                }
                api.sendMessage("TEST BOT: " + event.body, event.threadID);
                break;
            case "event":
                console.log(event);
                break;
        }
    });
});
```
