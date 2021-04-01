## Utilities

Some utilities which aren't necessarily as essential as the core parts of `velt`, or some of the more useful parts of `velt/helpers`, but can definitely still be very useful depending on what you're using them for.

## Creating Custom Mobs

Firstly, a clarification. Velt does not allow you to create entirely new mobs with new sound effects and textures, but it does allow you to remix existing mobs to what you want, like a zombie with 50 health and runs twice as fast.

Firstly, let's take a look at the following example:

```ts
const { server } = require('velt');
const { equip, effect } = require('velt/helpers');

//Assume loc is already defined, and spawn a zombie with some armor and speed.
const zombie = server.summon('zombie', loc);
equip(zombie, {
    helmet: 'leather helmet',
    chestplate: 'chainmail chestplate',
    leggings: 'leather leggings',
    boots: 'leather boots'
});
effect(zombie, 'speed', { amplifier: 1, duration: 10000 });
```

However, that may be inconvenient and somewhat ugly, especially if you want to reuse it.

```ts
const summonCustom = loc => {
    const zombie = server.summon('zombie', loc);
    equip(zombie, {
        helmet: 'leather helmet',
        chestplate: 'chainmail chestplate',
        leggings: 'leather leggings',
        boots: 'leather boots'
    });
    effect(zombie, 'speed', { amplifier: 1, duration: 10000 });
};
```

CustomMob offers a much nicer alternative.

```ts
const zombie = new CustomMob({
    type: 'zombie',
    equipment: {
        helmet: 'leather helmet',
        chestplate: 'chainmail chestplate',
        leggings: 'leather leggings',
        boots: 'leather boots'
    },
    effects: [
        { type: 'speed', amplifier: 1, duration: 10000 }
    ]
});

zombie.summon(loc);
```

When creating a CustomMob, you can also add custom mechanics. For example, if you wanted to make a zombie which shoots a fireball every second, CustomMob has support for that with its cycle option.

```ts
const zombie = new CustomMob({
    type: 'zombie',
    cycle() {
        server.schedule({ seconds: 1 }, () => {
            this.shoot('fireball');
        });
    }
});

zombie.summon(loc);
```

In short, Velt's CustomMob utility allows for adding custom mechanics to mobs much easier.

## Creating Custom Items

Firstly, just to clarify, the custom items you can make in Velt aren't new items with new textures and sounds, but just adding in new features to existing items with specific names and properties.
To start, what if you wanted to implement a stick with a special name that lets you shoot a fireball when you right click it? Velt custom items would be the way to go.

```ts
const { c } = require('velt');
const { Item, shoot } = require('velt/helpers');

new Item({
    name: c`Fireball Shooter`,
    interact(event) {
        event.setCancelled(true);
        const player = event.getPlayer();
        shoot(player, 'fireball');
    }
});
```

You can also make it so you can be given a custom item with Item.give

```ts
const snowballShooter = new Item({
    name: c`Snowball Shooter`,
    interact(event) {
        event.setCancelled(true);
        const player = event.getPlayer();
        shoot(player, 'snowball');
    }
});

commands.create('shooter', sender => {
    snowballShooter.give(sender);
});
```

The Item utility for creating custom items isn't finalized, but it's quite useful.