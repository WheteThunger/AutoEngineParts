## Features

- Automatically adds engine parts to engine modules when they spawn
  - This includes when a player adds a module to a car at a car lift
- Locks engine modules that were given free parts, preventing players from removing and scrapping the parts
- Allows moving and removing locked engine modules, as if they don't contain engine parts
- Prevents damage to engine parts in locked modules (since players won't be able to repair them)
- Allows configuring engine part quality based on the permissions of the car owner

## Installation

1. Add the plugin to the `oxide/plugins` directory of your Rust server installation
2. Update the config with the desired `DefaultEnginePartsTier` (use `3` for high quality parts)
3. Reload the plugin

Engine parts will be automatically added to all existing engine modules. Any engine parts that were already present in engine modules will be dropped to the ground because players may have worked hard to obtain them.

If desired, you can make this plugin only apply to cars owned by players with permissions, and/or make such cars have higher quality parts than the default. Be sure to reload the plugin after granting permissions to automatically add engine parts to existing cars.

## Configuration

Default configuration (no engine parts are added):

```json
{
  "DefaultEnginePartsTier": 0
}
```

- `DefaultEnginePartsTier` (`0` - `3`) -- The quality of engine parts to automatically add to all engine modules. Players with permission can have higher quality parts than this applied to cars they own (see permissions section for details).
  - 0 = No parts are added (existing parts are untouched), engine containers remain unlocked
  - 1 = Low quality parts, engine containers are locked
  - 2 = Medium quality parts, engine containers are locked
  - 3 = High quality parts, engine containers are locked

## Permissions

*You can ignore this section if you simply want all cars to have the same quality of engine parts.*

Cars **owned** by players with the following permissions will have their engine modules filled with the corresponding quality of engine parts.

- `autoengineparts.tier1` - Low quality parts
- `autoengineparts.tier2` - Medium quality parts
- `autoengineparts.tier3` - High quality parts

Note: Granting permissions or changing a user's group ownership will not automatically update the engine parts of existing cars they own. However, simply removing and re-adding the engine modules on their existing cars will update their engine parts. Existing cars are also updated on plugin reload or server restart.

### How ownership works

**There is no such thing as car ownership in the vanilla Rust.** You will need another plugin to assign ownership to cars in order for the permissions in this plugin to be effective.

Car ownership is determined by the `OwnerID` property of the car, which is usually a player's Steam ID, or `0` for no owner. Most plugins that spawn cars for a player (such as [Craft Car Chassis](https://umod.org/plugins/craft-car-chassis) and [Spawn Modular Car](https://umod.org/plugins/spawn-modular-car)) will assign that player as the owner. For cars spawned by the vanilla game, it's recommended to use [Claim Vehicle](https://umod.org/plugins/claim-vehicle) to allow players to claim them with a command on cooldown.

## Plugin compatibility

This plugin is aggressive. It will replace engine parts in every car unless another plugin blocks it with a hook.

- [Car Spawn Settings](https://umod.org/plugins/car-spawn-settings) -- Compatible. Unless you are using only the permissions feature of Auto Engine Parts, it's recommended to disable engine part features in Car Spawn Settings because Auto Engine Parts will override them anyway.
- [Spawn Modular Car](https://umod.org/plugins/spawn-modular-car) -- Compatible. Recommended to disable engine part features in that plugin because Auto Engine Parts will override them anyway.
- [Bomb Trucks](https://umod.org/plugins/bomb-trucks) -- Compatible. Bomb trucks are specifically blocked from being affected by Auto Engine Parts.
- [Claim Vehicle](https://umod.org/plugins/claim-vehicle) -- Compatible.
  - When a player claims an unowned car, the engine parts will be automatically updated to match that player's permission if applicable.
    - If the car already contains unlocked engine parts, they will be dropped next to the car because the player may have worked hard to obtain them.
  - When a player unclaims a car that has locked engines, the engine parts will be reset to the defaults configured in Auto Engine Parts.
- [Engine Parts Durability](https://umod.org/plugins/engine-parts-durability) -- Compatible. That plugin will not affect engine modules locked by Auto Engine Parts. Whenever Auto Engine Parts unlocks an engine module, Engine Parts Durability will resume control.
- [No Engine Parts](https://umod.org/plugins/no-engine-parts) -- Compatible. If custom engine stats in that plugin are higher than what is provided by the engine parts in the module, then the engine parts provide no benefit other than to inform players that the car is ready to drive.

## Uninstallation

Simply remove the plugin. Free engine parts will be automatically removed, and locked engine modules will be unlocked.

## Developer Hooks

#### OnEngineStorageFill

- Called right before this plugin adds or removes parts from an engine storage container
- Returning `false` will prevent this plugin from adding or removing engine parts
- Returning `null` will allow this plugin to add or remove engine parts

```csharp
object OnEngineStorageFill(EngineStorage engineStorage, int enginePartsTier)
```

Note: The `enginePartsTier` argument may be `0` - `3`.
