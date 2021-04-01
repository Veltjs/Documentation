# The Basics

Handling the implementations of basic features like commands in Velt.

# Commands

As you saw in the modules section, the `velt` module can be used to create commands. The velt module offers a few helpful variables. The one we'll mainly be looking at is `commands`, as it offers direct control over handling commands.
The other function we'll be needing is `color`, which has the alias of `c`. This turns a string such as `&6Abcd` to a colored string with the text `Abcd` in gold. The following import will allow you to get both the server object and the color function. 

```ts
const { commands, c } = require('velt');
```

`commands.create` has multiple usages, shown below:

```ts
commands.create(name);
commands.create(options);
commands.create(callback); //Takes command name from callback function name.
commands.create(name, options);
commands.create(name, callback);
commands.create(name, options, callback);
```

The following options can be used:

```ts
{
    description: string,
    usage: string,
    aliases: Array<string>,
    label: string,
    callback: (sender, ...args) => any,
    tabComplete: (sender, ...args) => any,
    name: string,
    permission: string,
    permissionMessage: string,
    playerOnly: boolean | string, //Allows for blocking the console to run the command.
    argParser: str => Array<string> //Allows for custom parsing of arguments, eg with quotes
}
```

However, you can also chain callback and tabComplete if needed. This allows for the following to be valid.

```ts
commands.create('name').run((sender, ...args) => {
    sender.sendMessage('Hello!');
});
```

Although that is generally not the recommended approach. A better approach would be this:

```ts
commands.create('name', (sender, ...args) => {
    sender.sendMessage('Hello!');
}).tabComplete((sender, ...args) => {
    return ['tabCompleted'];
});
```

However in that case, the recommended approach would be using options for that.
```ts
commands.create('name', {
    run(sender, ...args) {
        sender.sendMessage('Hello!');       
    }, 
    tabComplete(sender, ...args) {
        return ['tabCompleted'];
    }
});
```

This allows for a clear and readable way to show your callback and tab completion.
Now, using commands itself. Let's start with a basic command.

```ts
commands.create('cmd', sender => { //You can ignore specifying arguments if they aren't needed
    sender.sendMessage('Hello world!');
});
```

The above command is great, but it doesn't really do much. Let's try using something that might be more useful.

```ts
commands.create('feed', {
    permission: 'velt.feed'
}, sender => {
    sender.setFoodLevel(20);
    sender.sendMessage(c('&6You have been saturated!'));
});
```

However, this won't work if the sender is a console. There are two ways to account for this. Firstly, you can manually block the console from running the command like so.

```ts
commands.create('feed', {
    permission: 'velt.feed'
}, sender => {
    if (manager.isConsole(sender)) {
        sender.sendMessage(c('&cOnly players can run the feed command.'));
        return;
    }

    sender.setFoodLevel(20);
    sender.sendMessage(c('&6You have been saturated!'));
});
```

The alternative is to specify this in your options, which is generally the preferred way.

```ts
commands.create('feed', {
    permission: 'velt.feed',
    playerOnly: c('&cOnly players can run the feed command')
}, sender => {
    sender.setFoodLevel(20);
    sender.sendMessage(c('&6You have been saturated!'));
});
```

If you want to specify arguments, you can do it like so.

```ts
commands.create('cmd', (sender, ...args) => {
    sender.sendMessage(c(`&6Your arguments are: &c${args.join(', ')}`));
});
```

By default, Velt processes arguments itself so that you can use quotations for arguments. You can disable this by overriding argParser.

```ts
commands.create('cmd', {
    argParser: null //Use minecraft's default arg parsing.
}, (sender, ...args) => {
    sender.sendMessage('This is a command');
});
```

Now, you can use Minecraft's builtin command system without support for arguments with over a space. However, there's a little bit more.

## Sub Commands

With the release of `0.1.0`, Velt has a feature which makes it incredibly easy to create sub-commands. From this:

```ts
commands.create('cmd', (sender, cm) => {
    switch (arg) {
        case 'sub':
            sender.sendMessage('Nice sub-command you got there!');
            break;
        case 'f':
            sender.sendMessage('f');
    }
});
```

To this:

```ts
commands.create('cmd', {
    subs: {
        sub: () => 'Nice sub-command you got there!',
        f: () => 'f'
    }
})
```

Sub-commands are just like commands. They support most of the same options, and even can have their own inner-sub commands!

```ts
commands.create('sub', {
    subs: {
        inner: {
            subs: {
                bro: () => 'I am a sub-command in a sub-command at /sub inner bro'
            }
        }
    }
})
```

You can have them tab-complete, only work with specific permissions, and only for reg players, and more.

## Argument Types

With the new `0.1.1` update, Velt has a new feature for command arguments. Before, where you had to manually check if an arg was a type and specify tab completions depending on arg length, now you can just specify an arg-type.

```ts
commands.create('add (number) (number)', (sender, x, y) => `Result: ${x + y}`);
//Alternatively
commands.create('add', {
    args: [ 'number', 'number' ],
    run: (sender, x, y) => `Result: ${x + y}`
});
```

Arg-types are backwards compatible with any commands without them, but they're an incredibly useful tool. You can also have optional args:

```ts
commands.create('tp (player) (player?)', (sender, player, other) => {
    if (other) {
        player.teleport(other);
        return `You have teleported ${player} to ${other}`;
    } else {
        sender.teleport(player);
        return `You have teleported to ${player}`;
    }
});
```

You can also have `spread` args:

```ts
commands.create('tphere (...player)', (sender, ...players) => {
    for (const player of players) {
        player.teleport(sender);
    }
});
```

Arg-types also work with sub-commands:

```ts
commands.create('math', {
    subs: {
        'add (number) (number)': (sender, x, y) => `Result: ${x + y}`,
        'mul (number) (number)': (sender, x, y) => `Result: ${x * y}`
    }
});
```

You can also make your own arg-types. These are the default ones supported:

```
text
integer
number
boolean
player
```

Here's how to make your own:

```ts
commands.createListType('snack', [ 'chips', 'cookies', 'donuts' ]); //Simple list types
commands.createType('positive', {
    match(sender, arg) {
        if (arg == null) return;
        const parsed = parseInt(arg);
        if (isNaN(parsed)) return;
        if (arg <= 0) return;
        return parsed;
    }
})
```

# Events

As you know, the `events` variable handles everything to do with events in Velt. Events are essential to scripting powerful scripts in Minecraft, since they allow you to handle everything that happens in your server, from a player chatting, to every single mob that spawns, to when a player right clicks their mouse, events are for handling it.

The main way for you to handle your events is using the manager.on function, which takes in the event name, and a callback which handles the event.

```ts
const { events } = require('velt');

events.on('AsyncPlayerChatEvent', event => {
    event.setFormat(`${event.getPlayer().getDisplayName()} > ${event.getMessage()}`);
});
```

Above is a basic example of how to use events in Velt. You can handle any Spigot event and even events from other plugins.

```ts
events.on('PlayerJoinEvent', event => {
    event.setJoinMessage(`[+] ${event.getPlayer()}`);
});
```

Something to note is that there can be multiple event handlers listening for a callback, and that you can close event handlers.

```ts
let handler = events.on('PlayerJoinEvent', event => {
    event.setJoinMessage(`${event.getPlayer()} has joined.`);
    handler.close();
});
```

Another way to listen for events is events.once, which will listen for a single event.

```ts
events.on('PlayerJoinEvent', event => {
    event.setJoinMessage(`${event.getPlayer()} has joined.`);
});
```

However, one of the best ways to listen for events is using events.once, which will return a promise that resolves when the event that matches the condition is detected. This can be extremely useful for detecting events within your code, and can simplify your scripts heavily, especially when combined with JavaScript's async-await syntax.

```ts
commands.create('abc', async sender => {
    sender.sendMessage('Tell me hello.');
    while (true) {
        let event = await events.once('AsyncPlayerChatEvent', event => event.getPlayer() === sender);
        if (event.getMessage() == 'hello') {
            sender.sendMessage('Good job.');
            event.setCancelled(true);
            return;
        } else {
            sender.sendMessage('You did not tell me hello, so tell me hello!');
            event.setCancelled(true)
        }
    }
});
```

# Scheduling Tasks

To start, you'll need to import the `server` variable from the `velt` module.

## Normal Tasks

Velt allows you to schedule both normal tasks and looping tasks. Normal tasks can be scheduled with a promise.

```ts
server.after(5, () => {
    //Run after 5 ticks
});
```

```ts
server.after({ seconds: 1 }, () => {
    //Run after 1 second
});
```

You can also asynchronously schedule tasks.

```ts
(async () => {
    await server.after(5);
    console.log('Run after 5 ticks');
    await server.after({ seconds: 1 });
    console.log('Run 1 second after the 5 ticks were waited');
})();
```

## Looping Tasks

Periodic tasks, or looping tasks, can also be scheduled in a similar way, except using a single callback and not a promise.

```ts
server.schedule({ seconds: 1 }, () => {
    console.log('This is called every second');
});
```

You can also asynchronously use `server.after` to simulate the above example.

```ts
(async () => {
    while (true) {
        await server.after({ seconds: 1 });
        console.log('This is called every second');
    }
})();
```

However, you can also use server.schedule as a scheduler object to simulate the above example asynchronously much more easily and much better for performance.

```ts
(async () => {
    const scheduler = server.schedule({ seconds: 1 });
    while (true) {
        await scheduler.next();
        console.log('This is called every second');
    }
})();
```

# Creating Configs

You'll need to import `Storage` from velt/helpers.

```ts
const { Storage } = require('velt/storage');
```

Velt can save configurations to JSON and YAML, which can either be manually specified, or defaults to the file extension.

```ts
//Create a config from the ./config.yml file.
let config = Storage.createConfig('./config.yml');
//Create a config from the ./config.yml file with a default object for a value if the path doesn't exist.
let config = Storage.createConfig('./config.yml', {});
​
//Update config properties with an object
config.set({
    a: 1,
    b: 2,
    c: 3
});
​
//Get properties via destructuring
const { a, b, c } = config.get();
​
//Utilize fields for getting and setting
config.field('a').set(1);
config.field('b').get();
​
//Save the config;
config.save();
//Save the config as a specific type.
config.save({type: 'json'}); 
config.save({type: 'yaml'});
//Save the config at a specific path
config.save('config.json');
//Save the config at a specific path with a specific type
config.save('config.yml', {type: 'json'});
```

# Creating GUIs

With Velt, you can make powerful, effective GUIs with very little code using Velt's utility module, `velt/helpers`. For GUI creation, you need to import the `Gui` class.

```ts
const { Gui } = require('velt/helpers');
```

Next, create your GUI by initializing it with the name, and number of rows.

```ts
const gui = new Gui('My GUI', 6);
```

Now, you can format items onto the GUI with `gui.format` (or alternatively with the alias `gui.set`)

```ts
gui.format(0, 'diamond sword');
gui.format(1, 'iron sword');
```

You can also make it so when you click on an item, a callback is run.

```ts
gui.format(2, 'stone sword', () => server.broadcast('The stone sword was clicked!'));
//Alternative option
gui.format(2, 'stone sword', {
    run() { server.broadcast('The stone sword was clicked!'); }
});
```

GUIs can be shown to players with the `gui.show` method (or alternatively, using `gui.open`).

```ts
gui.show(player);
gui.show(playerOne, playerTwo);
gui.show(...arrayOfPlayers);
```

Most of the time, you should use method-chaining with GUIs, since it's simple and much easier to use.

```ts
new Gui('My GUI', 6)
    .format(0, 'diamond sword')
    .format(1, 'iron sword')
    .format(2, 'stone sword', () => server.broadcast('I got clicked!'))
    .show(player);
```

When formatting an item, a lot of the time you want to close the GUI after clicking it. You can do that like so.

```ts
gui.format(0, 'stone block', {
    close: true, //Make it close after clicking it
    run() {
        server.broadcast('I was clicked!');
    }
});
```

You can also make a GUI item movable, so you can move it across the GUI to other slots.

```ts
gui.format(1, 'iron ingot', {
    movable: true
});
```

Finally, one extra thing to note is that you can specify to get the event when listening for a callback with a GUI. This can be useful for doing different things depending on the click type, as demonstrated below:

```ts
gui.format(0, 'stone block', {
  movable: false, // InventoryInteractEvent is cancelled
  close: true, //Make it close after clicking it
  run(event) {
    const clickType = event.getClick().name();
    switch(clickType) {
        case 'DROP':
            server.broadcast('This item has been dropped!');
            break;
        case 'LEFT':
        case 'RIGHT':
            server.broadcast('This has been left/right clicked.');
            break;
        case default:
            server.broadcast('Something else happened, huh.');
    }
  }
});
```

# Utilizing Scoreboards

Scoreboards is yet another helper class from the velt/helpers module, so to use scoreboards you will need to import Scoreboard from the velt/helpers module like so.

```ts
const { Scoreboard } = require('velt/helpers');
```

Scoreboards can be initialized in several ways, using a list, or using an object with key-value pairs, or using method chaining.

```ts
//Explicit scoreboard setting example
​
let scoreboard = new Scoreboard();
​
scoreboard.setName('My Scoreboard');
scoreboard.set(1, 'First Value');
scoreboard.set(0, 'Second Value');
​
scoreboard.show(player);
​
//Using an object
​
let scoreboard = new Scoreboard({
    name: 'My Scoreboard',
    scores: {
        1: 'First Value',
        0: 'Second Value'
    }
}).show(player);
​
//Using an array instead of numbers (numbers auto-filled in)
​
let scoreboard = new Scoreboard({
    name: 'My Scoreboard',
    scores: [
        'First Value',
        'Second Value'
    ]
}).show(player);
​
//Combining objects and lists with a non-method chaining solution.
​
let scoreboard = new Scoreboard();
​
scoreboard.setName('My Scoreboard');
​
//Use an object
​
scoreboard.set({
    1: 'First Value',
    0: 'Second Value'
});
​
//Use an array
​
scoreboard.set([
    'First Value',
    'Second Value'
]);
​
//Method chaining
​
let scoreboard = new Scoreboard()
    .setName('My Scoreboard')
    .set(1, 'First Value')
    .set(0, 'Second Value')
    .show(player);
```

# Making Use of Storage

Storage is by far one of the most useful features in Velt, as a lot of the time you need to store data across restarts and variables just won't cut it. Storage allows for getting and setting individual fields of JSON files, YAML files (to come), and databases entirely at some point.

Firstly, you need to import the Storage class from `velt/storage`.

```ts
const { Storage } = require('velt/storage');
```

Storage can be created from a file as its path.

```ts
const storage = new Storage('./data.json');
```

The storage can be manually used with the .get and .set methods to handle data.

```ts
storage.get(); //Gets the value of the storage
storage.set({ hello: 'world' }); //Sets the value of the storage
```

Here's an example of a spawn command, and a setspawn command.

```ts
commands.create('spawn', sender => {
    storage.get()['spawn'].teleport(sender);
});
commands.create('setspawn', sender => {
    storage.set({ ...storage.get(), spawn: sender.getLocation() });
});
```

Here's an example of a GUI which allows you to increment a number and keeps your progress across restarts.

```ts
function Gui(player) {
    let num = storage.get()['value'] ? storage.get()['value'] : 0;
    new Gui(`Number: ${num}`, 6)
        .set(0, {
            material: 'iron ingot',
            name: c`&6Increase counter by 1`
        }, () => {
            storage.set({ ...storage.get(), value: num + 1 });
            Gui(player);
        });
}
```

However, both of these are not very readable as you have to get the entire storage value and then get a specific field, and when setting it, you have to update the entire storage and keep everything but one value. For more complex cases like kits and warps, this would be even more complex.

This is where the field comes in, it can simplify using the storage dramatically by only making you set one field at a time.
Here's our spawn example, but using a field.

```ts
const spawn = storage.field('spawn');

commands.create('spawn', sender => {
    spawn.get().teleport(sender);
});

commands.create('setspawn', sender => {
    spawn.set(sender.getLocation());
});
```

And here's our GUI counter using a value field.

```ts
const value = storage.field('value', { default: 0 });

function Gui(player) {
    new Gui(`Number: ${value.get()}`, 6)
        .set(0, {
            material: 'iron ingot',
            name: c`&6Increase counter by 1`
        }, () => {
            value.add(1);
            Gui(player);
        });
}
```

The benefits of using fields are especially useful when you need to use arrays or objects to store things like warps and kits.

The following is an example of a warp system using storage fields.

```ts
const warps = storage.field('warps', { default: {} });

function setWarp(name, loc) {
    warps.field(name).set(cast.asLocation(loc));
}
function gotoWarp(player, name) {
    player.teleport(warps.field(name).get());
}
function removeWarp(name) {
    warps.pop(name);
}
```

The following is an example of a kits system using storage fields.

```ts
const kits = storage.field('kits', { default: {} });

function createKit(player, name) {
    kits.field(name).set(new Inventory(player));
}
function giveKit(player, name) {
    kits.field(name).get().give(player);
}
function removeKit(name) {
    kits.pop(name);
}
```

# Importing Java Classes

With Velt, you have plenty of features built-in and easily accessible, like making commands, listening for events, equipping players with armor, saving inventories to a `JSON` file with `velt/storage`, but it doesn't support everything. Not every part of Spigot is supported with Velt, and Velt doesn't have any support for other plugins directly.

Fortunately, Velt does have support for importing Java classes. Infact, it uses it directly in the source code of the core of Velt, which you can view [here](https://github.com/Veltjs/Velt/blob/main/src/main/resources/velt.js).

There are two ways to import a Java class in Velt. The recommended approach is to use `Java.pkg` (or alternatively, `Java.package`), like so:

```ts
const { Player } = Java.pkg('org.bukkit.entity');
```

There are two reasons why you should generally use `Java.pkg` over `Java.type`:
- It makes it so you don't need to repeat the class name twice.
- It lets you easily import multiple classes at once.

The other approach, `Java.type`, lets you import a class more directly, like so:

```ts
const Player = Java.type('org.bukkit.entity.Player');
```

Both work well, but `Java.pkg` is generally prefered.

# Casting

Normally, you don't need to do this directly, since Velt automatically casts for you in most cases, but sometimes you might find yourself wanting to cast objects, like perhaps an object with an `x`, `y`, `z`, and `world` to a location to teleport an entity to. To do that, you could make some complex function to handle it, or let Velt do it for you.

```ts
const loc = cast.asLocation({ x: 1, y: 2, z: 3, world: 'world' });
```

One of the most important use-cases of this would be to convert some text to a player, like so:

```ts
const player = cast.asPlayer('Cormans');
```

Casting is an important part of Velt, and it's a feature that when necessary, you should make use of.