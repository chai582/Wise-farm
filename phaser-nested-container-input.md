---
name: Phaser 4 container input bug
description: Interactive GameObjects inside ANY container (even a single-level one with scrollFactor=0) don't receive pointer events reliably in Phaser 4. Definitive fix is manual hit-testing.
---

In Phaser 4, `setInteractive()` on children inside a Container with `setScrollFactor(0)` does NOT reliably fire pointer events — this applies even to single-level containers (not just nested ones).

**Why:** The Phaser 4 input plugin processes objects in the scene's display list. Container children with a scrollFactor=0 parent have their hit areas computed in world/camera space, but the transform chain between container→child breaks down for fixed-to-camera containers. The result: buttons render correctly but clicks never fire.

**Definitive fix pattern (farm-game shop):**
1. Remove ALL `setInteractive()` and `.on('pointerdown', ...)` from container children
2. Store a `shopHitAreas` array on the scene: `{lx, ly, w, h, tabIdx, action, rect, normal, hover}[]`
   - `lx/ly`: button position in **container-local space** (container is at width/2, height/2)
   - `tabIdx`: which tab this belongs to (-1 = always visible)
3. In the existing scene-level `this.input.on('pointerdown', ...)`:
   ```ts
   if (this.inventoryOpen && this.tryShopClick(ptr.x, ptr.y)) return;
   ```
4. `tryShopClick(px, py)`: converts `px - width/2, py - height/2` → container-local, tests hit areas
5. `updateShopHover(px, py)`: called from `pointermove` when shop is open, updates rect fill color

**How to apply:** Any Phaser 4 UI that needs button clicks in a scrollFactor=0 container must use this manual hit-testing approach. Never rely on `setInteractive()` inside containers.
