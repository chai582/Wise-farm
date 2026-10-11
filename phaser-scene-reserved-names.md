---
name: Phaser Scene reserved property names
description: Phaser.Scene has built-in properties that silently shadow custom class fields if you reuse their names.
---

Phaser.Scene exposes several built-in properties on `this`. If a subclass declares a private field with the same name, TypeScript does NOT error — the field silently shadows the built-in at runtime, breaking Phaser internals.

Known reserved names that are easy to accidentally reuse:
- `this.events` — the scene's EventEmitter (`Phaser.Events.EventEmitter`). Very commonly confused with a custom event system.
- `this.input` — Phaser's InputPlugin
- `this.cameras` — CameraManager
- `this.add` — GameObjectFactory
- `this.tweens` — TweenManager
- `this.time` — TimePlugin

**Why:** Phaser extends the scene with these as instance properties, not class methods. TypeScript's structural typing lets a `private` field declaration coexist without error, but at runtime the private field wins and the Phaser built-in is inaccessible.

**How to apply:** Always prefix custom systems with a domain word: `eventSystem`, `farmingSystem`, `saveSystem` etc. Never use bare nouns that match Phaser's plugin names.
