# Getting Started

Velt is one of the best plugins for running JS code on Spigot. Velt tries to give you as many utilities for running Spigot code as possible, but it doesn't provide unnecessary modules that aren't useful for creating concise and simple scripts.

As Velt provides you almost the same modules as NodeJS does, many of your same npm scripts should be able to run using Velt, and you can install npm modules on Velt. 
Velt is public and you can download it from its Github page here.

You can find the latest releases to download [here](https://github.com/Veltjs/Velt/releases).

## Scripts

Just as a note, scripts work a little bit differently then other plugins, like Skript.

In your `Velt/scripts` folder, there are two types of scripts you can add:

- **Single-file scripts** are scripts which are only part of one file. They're usually good for testing out things, or for simple scripts. Generally though, you'll want to split your scripts into more complex bundles.
- **Folder scripts** are scripts which are part of an entire folder. They only run the main file of the folder (by default, `index.js`) but that's configurable if you add a `package.json` to it.

# Modules

Velt offers modules to help make your scripting easier.

## Info

Velt offers the same require syntax which is used in Node to import modules. Velt also offers many node modules which you can use, as one of Velt's goals is for you to be able to run node scripts and modules in Velt just fine.
The two main modules in Velt are `velt` and `velt/helpers`.

## The velt module

The `velt` module is the main module for you to use, offering variables that you can utilize to make powerful scripts. Below is a list of variables the module offers.

| Variable | Description                                                                                                                                                                                          | 
|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------| 
| events   | The `events` variable allows you to listen for events asynchronously or not. |
| commands | The `commands` variable allows you to handle commands, subcommands, permissions, etc. |
| server   | The `server` variable contains helpful functions to do specific tasks like getting an array of worlds, broadcasting a message, etc | 
| script   | The `script` variable contains a method especially useful for storing data no matter the server. |
| cast     | Allows for casting variables to specific types. |

Below is an example of using the `commands` and `events` variable of velt.

```ts
const { commands, events } = require('velt');

commands.create('abc', {
    playerOnly: 'Only players can run this'
}, async sender => {
    sender.sendMessage('Tell me "hello"!');
    while (true) {
        //manager.waitFor allows asynchronously for us to wait for an event that matches the condition.
        let event = await events.once('AsyncPlayerChatEvent', event => event.getPlayer() === sender);
        if (event.getMessage() == 'hello') {
            sender.sendMessage('Good job.');
            event.setCancelled(true);
            return;
        } else {
            sender.sendMessage(`I said tell me hello, not ${event.getMessage()}`);
            event.setCancelled(true);
        }
    }  
});
```

## The velt/helpers module

The `velt/helpers` module offers classes which can help make your code easier to read and more concise. Below is an example of using the Gui class of `velt/helpers`.

```ts
const { commands } = require('velt');
const { Gui } = require('velt/helpers');

commands.create('showgui', sender => {
    if (manager.isConsole(sender)) {
        sender.sendMessage('Only players can run this');
        return;
    }
    new Gui('My Gui', 36)
        .format(0, 'wooden sword', () => {
            sender.sendMessage('You clicked me');
        })
        .show(sender);
});
```

## Node modules

Many NodeJS core modules are supported, but particularly, the following aren't supported:

- http
- https
- ws
- vm

The fs module also is only partially supported.