# Adding New Items and Recipes in EaglercraftX

This guide explains how to add new items and custom recipes (crafting table and furnace) to EaglercraftX.

## Prerequisites

- Java 17 or higher
- Gradle build system
- Basic understanding of Java and Minecraft modding

## Adding a New Item

### Step 1: Create the Item Class (Optional)
If your item needs special behavior (e.g., food, tool, armor), create a new class extending `Item` or a subclass.

**File to create:** `src/game/java/net/minecraft/item/ItemCustomFood.java`

```java
package net.minecraft.item; // Import the item package for ItemFood and other item classes

public class ItemCustomFood extends ItemFood { // Extend ItemFood for edible items
    public ItemCustomFood(int healAmount, float saturation, boolean isWolfFood) { // Constructor with healing, saturation, and wolf food parameters
        super(healAmount, saturation, isWolfFood); // Call parent constructor to set food properties
        this.setUnlocalizedName("customFood"); // Set the internal name for localization
        this.setCreativeTab(CreativeTabs.tabFood); // Place item in the Food creative tab
    }
}
```

For a basic item like an ingot:

**File to create:** `src/game/java/net/minecraft/item/ItemCustomIngot.java`

```java
package net.minecraft.item; // Import the item package

public class ItemCustomIngot extends Item { // Extend base Item class for simple items
    public ItemCustomIngot() { // Constructor
        this.setUnlocalizedName("customIngot"); // Set the internal name for localization
        this.setCreativeTab(CreativeTabs.tabMaterials); // Place item in the Materials creative tab
    }
}
```

### Step 2: Register the Item
Add the item registration in the `registerItems()` method.

**File to edit:** `src/game/java/net/minecraft/item/Item.java`

Find the `registerItems()` method (around line 484) and add your registration at the end, before the closing brace. Choose a unique ID (check existing ones, start from 5000+).

```java
registerItem(5000, "custom_food", (new ItemCustomFood(4, 0.6F, false)).setUnlocalizedName("customFood"));
registerItem(5001, "custom_ingot", new ItemCustomIngot());
```

### Step 3: Add Static Reference
Add a static field and initialize it.

**File to edit:** `src/game/java/net/minecraft/init/Items.java`

Add the static declaration with other item declarations:

```java
public static Item custom_food;
public static Item custom_ingot;
```

Then in the `doBootstrap()` method (around line 225), add:

```java
custom_food = getRegisteredItem("custom_food");
custom_ingot = getRegisteredItem("custom_ingot");
```

### Step 4: Add Texture and Localization (Optional)
- **Texture:** Add `custom_food.png` to `assets/minecraft/textures/items/`
- **Texture (Ingot):** Add `custom_ingot.png` to `assets/minecraft/textures/items/`
- **Localization:** Edit language files in `javascript/lang/` (e.g., `en_US.lang`) and add:
  ```
  item.customFood.name=Custom Food
  item.customIngot.name=Custom Ingot
  ```

### Items with Right-Click Behaviors
To make an item perform an action when right-clicked (e.g., shooting a fireball or teleporting), override the `onItemRightClick` method in your item class.

#### Example: Fireball-Shooting Item
**File to create:** `src/game/java/net/minecraft/item/ItemFireballWand.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.entity.projectile.EntitySmallFireball;
import net.minecraft.util.MathHelper;
import net.minecraft.world.World;

public class ItemFireballWand extends Item {
    public ItemFireballWand() {
        this.setUnlocalizedName("fireballWand");
        this.setCreativeTab(CreativeTabs.tabMisc);
        this.setMaxStackSize(1);
    }

    @Override
    public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
        if (!world.isRemote) {
            // Calculate direction
            float f = -MathHelper.sin(player.rotationYaw * 0.017453292F) * MathHelper.cos(player.rotationPitch * 0.017453292F);
            float f1 = -MathHelper.sin(player.rotationPitch * 0.017453292F);
            float f2 = MathHelper.cos(player.rotationYaw * 0.017453292F) * MathHelper.cos(player.rotationPitch * 0.017453292F);

            EntitySmallFireball fireball = new EntitySmallFireball(world, player, f, f1, f2);
            fireball.setPosition(player.posX, player.posY + player.getEyeHeight(), player.posZ);
            world.spawnEntityInWorld(fireball);
        }
        return itemStack;
    }
}
```

Register it as in Step 2, and add static reference.

#### Example: Block Modifier Wand (Throws projectile that destroys or creates blocks in area)
**File to create:** `src/game/java/net/minecraft/item/ItemBlockModifierWand.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.util.MathHelper;
import net.minecraft.world.World;

public class ItemBlockModifierWand extends Item {
    public ItemBlockModifierWand() {
        this.setUnlocalizedName("blockModifierWand");
        this.setCreativeTab(CreativeTabs.tabTools);
        this.setMaxStackSize(1);
    }

    @Override
    public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
        if (!world.isRemote) {
            // Create and throw a custom projectile (you need to create EntityBlockModifierProjectile)
            float f = -MathHelper.sin(player.rotationYaw * 0.017453292F) * MathHelper.cos(player.rotationPitch * 0.017453292F);
            float f1 = -MathHelper.sin(player.rotationPitch * 0.017453292F);
            float f2 = MathHelper.cos(player.rotationYaw * 0.017453292F) * MathHelper.cos(player.rotationPitch * 0.017453292F);

            EntityBlockModifierProjectile projectile = new EntityBlockModifierProjectile(world, player, f, f1, f2);
            projectile.setPosition(player.posX, player.posY + player.getEyeHeight(), player.posZ);
            world.spawnEntityInWorld(projectile);
        }
        return itemStack;
    }
}
```

You also need to create the projectile entity:

**File to create:** `src/game/java/net/minecraft/entity/projectile/EntityBlockModifierProjectile.java`

```java
package net.minecraft.entity.projectile;

import net.minecraft.entity.EntityLivingBase;
import net.minecraft.init.Blocks;
import net.minecraft.util.MovingObjectPosition;
import net.minecraft.world.World;

public class EntityBlockModifierProjectile extends EntityThrowable {
    public EntityBlockModifierProjectile(World world) {
        super(world);
    }

    public EntityBlockModifierProjectile(World world, EntityLivingBase thrower, double x, double y, double z) {
        super(world, thrower);
        this.setThrowableHeading(x, y, z, 1.5F, 1.0F);
    }

    @Override
    protected void onImpact(MovingObjectPosition mop) {
        if (!this.worldObj.isRemote) {
            int range = 3; // Area range
            for (int x = -range; x <= range; x++) {
                for (int y = -range; y <= range; y++) {
                    for (int z = -range; z <= range; z++) {
                        int blockX = (int)this.posX + x;
                        int blockY = (int)this.posY + y;
                        int blockZ = (int)this.posZ + z;
                        if (x*x + y*y + z*z <= range*range) { // Spherical area
                            // Destroy blocks: set to air
                            this.worldObj.setBlockToAir(blockX, blockY, blockZ);
                            // Or create blocks: this.worldObj.setBlock(blockX, blockY, blockZ, Blocks.stone);
                        }
                    }
                }
            }
            this.setDead();
        }
    }
}
```

Register the projectile entity in `EntityList.java` like other projectiles.

#### Example: Teleportation Item
**File to create:** `src/game/java/net/minecraft/item/ItemTeleportCrystal.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.util.ChatComponentText;
import net.minecraft.world.World;

public class ItemTeleportCrystal extends Item {
    public ItemTeleportCrystal() {
        this.setUnlocalizedName("teleportCrystal");
        this.setCreativeTab(CreativeTabs.tabMisc);
        this.setMaxStackSize(16);
    }

    @Override
    public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
        if (!world.isRemote) {
            // Teleport to a random location within 100 blocks
            double newX = player.posX + (world.rand.nextDouble() - 0.5) * 200;
            double newZ = player.posZ + (world.rand.nextDouble() - 0.5) * 200;
            double newY = world.getTopSolidOrLiquidBlock((int)newX, (int)newZ) + 1;
            player.setPositionAndUpdate(newX, newY, newZ);
            player.addChatMessage(new ChatComponentText("Teleported!"));
        }
        return itemStack;
    }
}
```

Register similarly.

## Creating Guns, Bullets, and Grenades

This section covers creating firearms with custom rate of fire, projectiles (bullets), and explosive grenades.

### Guns with Custom Rate of Fire

To create a gun that shoots bullets with a custom delay between shots, create an item class that tracks the last shot time and enforces a cooldown.

#### Example: Pistol with Custom Fire Rate
**File to create:** `src/game/java/net/minecraft/item/ItemPistol.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.util.MathHelper;
import net.minecraft.world.World;

public class ItemPistol extends Item {
    private long lastShotTime = 0; // Track last shot time
    private int fireRateDelay = 500; // Delay in milliseconds (e.g., 500ms for 2 shots per second)

    public ItemPistol() {
        this.setUnlocalizedName("pistol");
        this.setCreativeTab(CreativeTabs.tabCombat);
        this.setMaxStackSize(1);
    }

    @Override
    public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
        long currentTime = System.currentTimeMillis();
        if (currentTime - lastShotTime >= fireRateDelay) {
            if (!world.isRemote) {
                // Calculate direction
                float f = -MathHelper.sin(player.rotationYaw * 0.017453292F) * MathHelper.cos(player.rotationPitch * 0.017453292F);
                float f1 = -MathHelper.sin(player.rotationPitch * 0.017453292F);
                float f2 = MathHelper.cos(player.rotationYaw * 0.017453292F) * MathHelper.cos(player.rotationPitch * 0.017453292F);

                EntityBullet bullet = new EntityBullet(world, player, f, f1, f2);
                bullet.setPosition(player.posX, player.posY + player.getEyeHeight(), player.posZ);
                world.spawnEntityInWorld(bullet);
            }
            lastShotTime = currentTime; // Update last shot time
        }
        return itemStack;
    }
}
```

Register this item as in Step 2, add static reference, texture, and localization.

To customize the fire rate, change the `fireRateDelay` value (lower for faster firing, higher for slower).

#### Example: Sniper Rifle with Zoom
For a sniper that zooms in when aiming (right-click held), override `onItemRightClick` and use client-side FOV changes.

**File to create:** `src/game/java/net/minecraft/item/ItemSniperRifle.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.util.MathHelper;
import net.minecraft.world.World;

public class ItemSniperRifle extends Item {
    private long lastShotTime = 0;
    private int fireRateDelay = 2000; // Slower fire rate

    public ItemSniperRifle() {
        this.setUnlocalizedName("sniperRifle");
        this.setCreativeTab(CreativeTabs.tabCombat);
        this.setMaxStackSize(1);
    }

    @Override
    public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
        if (world.isRemote) {
            // Client-side zoom: reduce FOV
            // Note: This requires accessing Minecraft instance, e.g., Minecraft.getMinecraft().gameSettings.fovSetting = 30.0F;
            // For simplicity, assume you handle zoom in a separate method or event.
        } else {
            long currentTime = System.currentTimeMillis();
            if (currentTime - lastShotTime >= fireRateDelay) {
                float f = -MathHelper.sin(player.rotationYaw * 0.017453292F) * MathHelper.cos(player.rotationPitch * 0.017453292F);
                float f1 = -MathHelper.sin(player.rotationPitch * 0.017453292F);
                float f2 = MathHelper.cos(player.rotationYaw * 0.017453292F) * MathHelper.cos(player.rotationPitch * 0.017453292F);

                EntityBullet bullet = new EntityBullet(world, player, f, f1, f2);
                bullet.setPosition(player.posX, player.posY + player.getEyeHeight(), player.posZ);
                world.spawnEntityInWorld(bullet);
                lastShotTime = currentTime;
            }
        }
        return itemStack;
    }

    // To handle zoom on right-click hold, you might need to use item use duration or client events.
    // For advanced zoom, override onUsingTick or use key bindings.
}
```

For proper zoom, you need client-side code to change FOV when the item is being used. This might require editing `Minecraft.java` or using events.

#### Example: Shotgun
Shoots multiple bullets in a spread.

**File to create:** `src/game/java/net/minecraft/item/ItemShotgun.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.util.MathHelper;
import net.minecraft.world.World;

public class ItemShotgun extends Item {
    private long lastShotTime = 0;
    private int fireRateDelay = 1000;

    public ItemShotgun() {
        this.setUnlocalizedName("shotgun");
        this.setCreativeTab(CreativeTabs.tabCombat);
        this.setMaxStackSize(1);
    }

    @Override
    public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
        long currentTime = System.currentTimeMillis();
        if (currentTime - lastShotTime >= fireRateDelay) {
            if (!world.isRemote) {
                // Shoot 5 bullets in a spread
                for (int i = -2; i <= 2; i++) {
                    float yawOffset = i * 0.1F; // Spread angle
                    float f = -MathHelper.sin((player.rotationYaw + yawOffset) * 0.017453292F) * MathHelper.cos(player.rotationPitch * 0.017453292F);
                    float f1 = -MathHelper.sin(player.rotationPitch * 0.017453292F);
                    float f2 = MathHelper.cos((player.rotationYaw + yawOffset) * 0.017453292F) * MathHelper.cos(player.rotationPitch * 0.017453292F);

                    EntityBullet bullet = new EntityBullet(world, player, f, f1, f2);
                    bullet.setPosition(player.posX, player.posY + player.getEyeHeight(), player.posZ);
                    world.spawnEntityInWorld(bullet);
                }
            }
            lastShotTime = currentTime;
        }
        return itemStack;
    }
}
```

#### Example: Knife (Melee Weapon)
Extend ItemSword for a custom knife with unique damage or effects.

**File to create:** `src/game/java/net/minecraft/item/ItemKnife.java`

```java
package net.minecraft.item;

import net.minecraft.entity.EntityLivingBase;
import net.minecraft.item.ItemSword;
import net.minecraft.item.Item.ToolMaterial;

public class ItemKnife extends ItemSword {
    public ItemKnife() {
        super(ToolMaterial.IRON);
        this.setUnlocalizedName("knife");
        this.setCreativeTab(CreativeTabs.tabCombat);
    }

    @Override
    public boolean onLeftClickEntity(ItemStack stack, EntityPlayer player, Entity entity) {
        if (entity instanceof EntityLivingBase) {
            // Custom effect: bleed or something, but for simplicity, just higher damage
            entity.attackEntityFrom(DamageSource.causePlayerDamage(player), 6.0F); // Custom damage
        }
        return false;
    }
}
```

#### Example: AK-47 Style Assault Rifle
Faster fire rate, perhaps with ammo consumption (but simplified).

**File to create:** `src/game/java/net/minecraft/item/ItemAK47.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.util.MathHelper;
import net.minecraft.world.World;

public class ItemAK47 extends Item {
    private long lastShotTime = 0;
    private int fireRateDelay = 100; // Fast fire rate

    public ItemAK47() {
        this.setUnlocalizedName("ak47");
        this.setCreativeTab(CreativeTabs.tabCombat);
        this.setMaxStackSize(1);
    }

    @Override
    public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
        long currentTime = System.currentTimeMillis();
        if (currentTime - lastShotTime >= fireRateDelay) {
            if (!world.isRemote) {
                float f = -MathHelper.sin(player.rotationYaw * 0.017453292F) * MathHelper.cos(player.rotationPitch * 0.017453292F);
                float f1 = -MathHelper.sin(player.rotationPitch * 0.017453292F);
                float f2 = MathHelper.cos(player.rotationYaw * 0.017453292F) * MathHelper.cos(player.rotationPitch * 0.017453292F);

                EntityBullet bullet = new EntityBullet(world, player, f, f1, f2);
                bullet.setPosition(player.posX, player.posY + player.getEyeHeight(), player.posZ);
                world.spawnEntityInWorld(bullet);
            }
            lastShotTime = currentTime;
        }
        return itemStack;
    }
}
```

#### Example: Rocket Launcher
Shoots explosive rockets.

**File to create:** `src/game/java/net/minecraft/item/ItemRocketLauncher.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.util.MathHelper;
import net.minecraft.world.World;

public class ItemRocketLauncher extends Item {
    private long lastShotTime = 0;
    private int fireRateDelay = 2000;

    public ItemRocketLauncher() {
        this.setUnlocalizedName("rocketLauncher");
        this.setCreativeTab(CreativeTabs.tabCombat);
        this.setMaxStackSize(1);
    }

    @Override
    public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
        long currentTime = System.currentTimeMillis();
        if (currentTime - lastShotTime >= fireRateDelay) {
            if (!world.isRemote) {
                float f = -MathHelper.sin(player.rotationYaw * 0.017453292F) * MathHelper.cos(player.rotationPitch * 0.017453292F);
                float f1 = -MathHelper.sin(player.rotationPitch * 0.017453292F);
                float f2 = MathHelper.cos(player.rotationYaw * 0.017453292F) * MathHelper.cos(player.rotationPitch * 0.017453292F);

                EntityRocket rocket = new EntityRocket(world, player, f, f1, f2);
                rocket.setPosition(player.posX, player.posY + player.getEyeHeight(), player.posZ);
                world.spawnEntityInWorld(rocket);
            }
            lastShotTime = currentTime;
        }
        return itemStack;
    }
}
```

#### Rocket Entity
**File to create:** `src/game/java/net/minecraft/entity/projectile/EntityRocket.java`

```java
package net.minecraft.entity.projectile;

import net.minecraft.entity.EntityLivingBase;
import net.minecraft.util.MovingObjectPosition;
import net.minecraft.world.World;

public class EntityRocket extends EntityThrowable {
    public EntityRocket(World world) {
        super(world);
    }

    public EntityRocket(World world, EntityLivingBase thrower, double x, double y, double z) {
        super(world, thrower);
        this.setThrowableHeading(x, y, z, 2.0F, 1.0F);
    }

    @Override
    protected void onImpact(MovingObjectPosition mop) {
        if (!this.worldObj.isRemote) {
            this.worldObj.createExplosion(this, this.posX, this.posY, this.posZ, 5.0F, true); // Larger explosion
            this.setDead();
        }
    }
}
```

Register `EntityRocket` in `EntityList.java`:

```java
addMapping(EntityRocket.class, "Rocket", 503);
```

Register all these items as in Step 2.

### Bullets (Projectiles)

Bullets are custom projectiles that deal damage on impact.

#### Example: Bullet Entity
**File to create:** `src/game/java/net/minecraft/entity/projectile/EntityBullet.java`

```java
package net.minecraft.entity.projectile;

import net.minecraft.entity.EntityLivingBase;
import net.minecraft.entity.monster.EntityBlaze;
import net.minecraft.util.DamageSource;
import net.minecraft.util.MovingObjectPosition;
import net.minecraft.world.World;

public class EntityBullet extends EntityThrowable {
    public EntityBullet(World world) {
        super(world);
    }

    public EntityBullet(World world, EntityLivingBase thrower, double x, double y, double z) {
        super(world, thrower);
        this.setThrowableHeading(x, y, z, 3.0F, 1.0F); // Speed and inaccuracy
    }

    @Override
    protected void onImpact(MovingObjectPosition mop) {
        if (!this.worldObj.isRemote) {
            if (mop.entityHit != null) {
                // Deal damage to hit entity
                mop.entityHit.attackEntityFrom(DamageSource.causeThrownDamage(this, this.getThrower()), 5.0F); // 5 damage
            }
            this.setDead();
        }
    }
}
```

Register the bullet entity in `EntityList.java` like other projectiles:

```java
addMapping(EntityBullet.class, "Bullet", 501);
```

### Grenades (Explosive Throwables)

Grenades are throwable items that explode after a delay or on impact.

#### Example: Grenade Item
**File to create:** `src/game/java/net/minecraft/item/ItemGrenade.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.world.World;

public class ItemGrenade extends Item {
    public ItemGrenade() {
        this.setUnlocalizedName("grenade");
        this.setCreativeTab(CreativeTabs.tabCombat);
    }

    @Override
    public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
        if (!world.isRemote) {
            EntityGrenade grenade = new EntityGrenade(world, player);
            grenade.setPosition(player.posX, player.posY + player.getEyeHeight(), player.posZ);
            world.spawnEntityInWorld(grenade);
            if (!player.capabilities.isCreativeMode) {
                --itemStack.stackSize; // Consume the grenade
            }
        }
        return itemStack;
    }
}
```

#### Example: Grenade Entity
**File to create:** `src/game/java/net/minecraft/entity/projectile/EntityGrenade.java`

```java
package net.minecraft.entity.projectile;

import net.minecraft.entity.EntityLivingBase;
import net.minecraft.util.MovingObjectPosition;
import net.minecraft.world.World;

public class EntityGrenade extends EntityThrowable {
    private int fuse = 60; // Fuse time in ticks (3 seconds at 20 TPS)

    public EntityGrenade(World world) {
        super(world);
    }

    public EntityGrenade(World world, EntityLivingBase thrower) {
        super(world, thrower);
        this.setThrowableHeading(thrower.getLookVec().xCoord, thrower.getLookVec().yCoord, thrower.getLookVec().zCoord, 1.5F, 1.0F);
    }

    @Override
    public void onUpdate() {
        super.onUpdate();
        if (fuse > 0) {
            fuse--;
        } else {
            this.explode();
        }
    }

    @Override
    protected void onImpact(MovingObjectPosition mop) {
        this.explode();
    }

    private void explode() {
        if (!this.worldObj.isRemote) {
            this.worldObj.createExplosion(this, this.posX, this.posY, this.posZ, 3.0F, true); // Explosion with radius 3
            this.setDead();
        }
    }
}
```

Register the grenade entity in `EntityList.java`:

```java
addMapping(EntityGrenade.class, "Grenade", 502);
```

Register the grenade item as in Step 2, add static reference, texture, and localization.

## Custom Textures, Animations, and Models

In EaglercraftX, items and entities typically use 2D textures, but you can create animated textures and, for more advanced setups, custom 3D models.

### Animated Textures

To create animations for items, entities, or blocks, use a texture with multiple frames stacked vertically. The game will cycle through the frames automatically.

#### For Items (e.g., Animated Gun Texture)
- Create a PNG file with frames stacked vertically (e.g., 16x16 pixels per frame, multiple rows).
- The animation speed is fixed, but you can control it by the number of frames.
- Add the texture to `assets/minecraft/textures/items/` (e.g., `animated_gun.png`).

Minecraft automatically animates item textures if the PNG has a height that's a multiple of the width (e.g., 16x32 for 2 frames).

#### For Entities (e.g., Animated Mob Texture)
- Similar to items, stack frames vertically in the PNG.
- Add to `assets/minecraft/textures/entity/mob_name/` (e.g., `animated_mob.png`).
- The entity will animate as it moves or stands.

#### Custom Animation Speed
For more control, you can override the rendering in the entity's or item's renderer class, but that's advanced and requires custom rendering code.

### Custom Models

Items in vanilla Minecraft 1.8 are 2D sprites, but you can create custom 3D models for entities or blocks using custom renderers.

#### For Entities (Custom 3D Models)
To give entities custom 3D models instead of basic 2D sprites:
1. Create a custom renderer class extending `RenderLiving`.
2. Override the `doRender` method to use a custom model (e.g., from Blockbench or modeled manually).
3. Register the renderer in `RenderManager`.

**Example: Custom Model for a Mob**
**File to create:** `src/game/java/net/minecraft/client/renderer/entity/RenderCustomMob.java`

```java
package net.minecraft.client.renderer.entity;

import net.minecraft.client.model.ModelBase;
import net.minecraft.entity.Entity;
import net.minecraft.util.ResourceLocation;

public class RenderCustomMob extends RenderLiving {
    private static final ResourceLocation texture = new ResourceLocation("textures/entity/custom_mob.png");

    public RenderCustomMob(ModelBase model, float shadowSize) {
        super(model, shadowSize);
    }

    @Override
    protected ResourceLocation getEntityTexture(Entity entity) {
        return texture;
    }
}
```

Then, in `RenderManager.java`, register it:

```java
this.entityRenderMap.put(EntityCustomMob.class, new RenderCustomMob(new ModelCustomMob(), 0.5F));
```

You need to create `ModelCustomMob` extending `ModelBase` with custom box additions for the 3D shape.

For items, to have 3D models, you'd need to create a custom item renderer, which is more complex and involves overriding `renderItem` in `ItemRenderer`.

For basic setups, stick to 2D textures with animations.

#### Custom 3D Models for Items (e.g., Guns in Hand)
To render guns or other items as 3D models in the player's hand instead of 2D sprites, create a custom item renderer.

1. Create a custom model class extending `ModelBase`.
2. Create a custom renderer that handles the 3D rendering in first-person view.
3. Register the custom renderer for the item.

**Example: 3D Gun Model**
**File to create:** `src/game/java/net/minecraft/client/model/ModelGun.java`

```java
package net.minecraft.client.model;

import net.minecraft.client.model.ModelBase;
import net.minecraft.client.model.ModelRenderer;

public class ModelGun extends ModelBase {
    public ModelRenderer gunBody;

    public ModelGun() {
        this.gunBody = new ModelRenderer(this, 0, 0);
        this.gunBody.addBox(-1F, -1F, -1F, 2, 2, 6); // Simple box for gun body
    }

    @Override
    public void render(float scale) {
        this.gunBody.render(scale);
    }
}
```

**File to create:** `src/game/java/net/minecraft/client/renderer/entity/RenderItemCustom.java`

```java
package net.minecraft.client.renderer.entity;

import net.minecraft.client.Minecraft;
import net.minecraft.client.renderer.ItemRenderer;
import net.minecraft.client.renderer.Tessellator;
import net.minecraft.item.ItemStack;
import net.minecraft.util.ResourceLocation;
import org.lwjgl.opengl.GL11;

public class RenderItemCustom extends ItemRenderer {
    private ModelGun modelGun = new ModelGun();
    private static final ResourceLocation gunTexture = new ResourceLocation("textures/items/gun_3d.png");

    @Override
    public void renderItem(EntityPlayer player, ItemStack itemStack, int renderType) {
        if (itemStack.getItem() == Items.pistol) { // Check if it's your gun item
            GL11.glPushMatrix();
            // Position and rotate for first-person view
            GL11.glTranslatef(0.5F, 0.2F, 0.5F);
            GL11.glRotatef(45F, 0F, 1F, 0F);
            GL11.glScalef(0.5F, 0.5F, 0.5F);

            Minecraft.getMinecraft().getTextureManager().bindTexture(gunTexture);
            this.modelGun.render(0.0625F); // Render the 3D model

            GL11.glPopMatrix();
        } else {
            super.renderItem(player, itemStack, renderType); // Fallback to default
        }
    }
}
```

Then, in `Minecraft.java` or your main client class, set the custom item renderer:

```java
this.itemRenderer = new RenderItemCustom(this);
```

This replaces the default item renderer with your custom one that renders 3D models for specific items.

Note: This is a simplified example. Full implementation may require handling different render types (first-person, third-person, inventory) and more precise positioning.

## Items with Effects When Equipped in Hand

To add effects when an item is held in the player's hand (equipped), override the `onUpdate` method in your item class. This method is called every tick while the item is in the player's inventory, and you can check if it's selected (held in hand) using the `isSelected` parameter.

#### Example: Regeneration Amulet (gives regeneration when held)
**File to create:** `src/game/java/net/minecraft/item/ItemRegenerationAmulet.java`

```java
package net.minecraft.item;

import net.minecraft.entity.Entity;
import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.potion.Potion;
import net.minecraft.potion.PotionEffect;
import net.minecraft.world.World;

public class ItemRegenerationAmulet extends Item {
    public ItemRegenerationAmulet() {
        this.setUnlocalizedName("regenerationAmulet");
        this.setCreativeTab(CreativeTabs.tabMisc);
        this.setMaxStackSize(1);
    }

    @Override
    public void onUpdate(ItemStack stack, World world, Entity entity, int slot, boolean isSelected) {
        if (entity instanceof EntityPlayer && isSelected) {
            EntityPlayer player = (EntityPlayer) entity;
            if (!world.isRemote) {
                player.addPotionEffect(new PotionEffect(Potion.regeneration.id, 40, 0)); // Regeneration I for 2 seconds
            }
        }
    }
}
```

Register this item as in Step 2, add static reference, texture, and localization as needed.

#### Example: Dual-Effect Amulet (Different effects in inventory vs hand)
**File to create:** `src/game/java/net/minecraft/item/ItemDualEffectAmulet.java`

```java
package net.minecraft.item;

import net.minecraft.entity.Entity;
import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.potion.Potion;
import net.minecraft.potion.PotionEffect;
import net.minecraft.world.World;

public class ItemDualEffectAmulet extends Item {
    public ItemDualEffectAmulet() {
        this.setUnlocalizedName("dualEffectAmulet");
        this.setCreativeTab(CreativeTabs.tabMisc);
        this.setMaxStackSize(1);
    }

    @Override
    public void onUpdate(ItemStack stack, World world, Entity entity, int slot, boolean isSelected) {
        if (entity instanceof EntityPlayer) {
            EntityPlayer player = (EntityPlayer) entity;
            if (!world.isRemote) {
                if (isSelected) {
                    // Effect when held in hand: Speed
                    player.addPotionEffect(new PotionEffect(Potion.moveSpeed.id, 40, 1)); // Speed II
                } else {
                    // Effect when in inventory (not in hand): Jump Boost
                    player.addPotionEffect(new PotionEffect(Potion.jump.id, 40, 0)); // Jump Boost I
                }
            }
        }
    }
}
```

Register this item as in Step 2, add static reference, texture, and localization as needed.

## Advanced Item Behaviors

Beyond right-click and passive effects, items can have behaviors when used on blocks, entities, or when attacking.

### Items with Effects on Right-Clicking Blocks
Override `onItemUse` to perform actions when right-clicking on blocks.

#### Example: Block Placer Wand
**File to create:** `src/game/java/net/minecraft/item/ItemBlockPlacer.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.init.Blocks;
import net.minecraft.world.World;

public class ItemBlockPlacer extends Item {
    public ItemBlockPlacer() {
        this.setUnlocalizedName("blockPlacer");
        this.setCreativeTab(CreativeTabs.tabTools);
    }

    @Override
    public boolean onItemUse(ItemStack stack, EntityPlayer player, World world, int x, int y, int z, int side, float hitX, float hitY, float hitZ) {
        if (!world.isRemote) {
            // Place a diamond block above the clicked block
            world.setBlock(x, y + 1, z, Blocks.diamond_block);
        }
        return true;
    }
}
```

### Items with Effects on Attacking Mobs
Override `onLeftClickEntity` to apply effects when attacking entities.

#### Example: Flaming Sword
**File to create:** `src/game/java/net/minecraft/item/ItemFlamingSword.java`

```java
package net.minecraft.item; // Import item package

import net.minecraft.entity.Entity; // Import Entity for entity interactions
import net.minecraft.entity.EntityLivingBase; // Import for living entities
import net.minecraft.entity.player.EntityPlayer; // Import for player
import net.minecraft.item.ItemSword; // Import ItemSword for sword functionality
import net.minecraft.item.Item.ToolMaterial; // Import tool material enum

public class ItemFlamingSword extends ItemSword { // Extend ItemSword for sword behavior
    public ItemFlamingSword() { // Constructor
        super(ToolMaterial.IRON); // Call parent with iron material for durability and damage
        this.setUnlocalizedName("flamingSword"); // Set internal name
        this.setCreativeTab(CreativeTabs.tabCombat); // Place in Combat tab
    }

    @Override
    public boolean onLeftClickEntity(ItemStack stack, EntityPlayer player, Entity entity) { // Called when attacking entities
        if (entity instanceof EntityLivingBase) { // Check if entity is living
            entity.setFire(5); // Set the entity on fire for 5 seconds
        }
        return false; // Return false to allow normal attack damage
    }
}
```

### Armor with Effects
For armor, override `onArmorTick` in the armor class to apply effects while worn.

#### Example: Speed Boots
**File to create:** `src/game/java/net/minecraft/item/ItemSpeedBoots.java`

```java
package net.minecraft.item; // Import item package

import net.minecraft.entity.player.EntityPlayer; // Import for player
import net.minecraft.potion.Potion; // Import for potion types
import net.minecraft.potion.PotionEffect; // Import for potion effects
import net.minecraft.world.World; // Import for world

public class ItemSpeedBoots extends ItemArmor { // Extend ItemArmor for armor functionality
    public ItemSpeedBoots(ArmorMaterial material, int renderIndex, int armorType) { // Constructor with material, render index, and armor type
        super(material, renderIndex, armorType); // Call parent constructor
        this.setUnlocalizedName("speedBoots"); // Set internal name
        this.setCreativeTab(CreativeTabs.tabCombat); // Place in Combat tab
    }

    @Override
    public void onArmorTick(World world, EntityPlayer player, ItemStack itemStack) { // Called every tick while armor is worn
        if (!world.isRemote) { // Only run on server side
            player.addPotionEffect(new PotionEffect(Potion.moveSpeed.id, 20, 0)); // Add Speed I for 1 second (20 ticks)
        }
    }
}
```

Register armor similarly to items, but in the armor registration section if exists, or in Item.java.

### Tools with Custom Mining Behavior
For tools that can mine blocks and cause specific drops, extend the appropriate tool class and override methods like `onBlockDestroyed` or modify the block's drop behavior.

#### Example: Diamond-Dropping Pickaxe
**File to create:** `src/game/java/net/minecraft/item/ItemDiamondPickaxe.java`

```java
package net.minecraft.item; // Import item package

import net.minecraft.block.Block; // Import Block for block interactions
import net.minecraft.entity.player.EntityPlayer; // Import for player
import net.minecraft.item.ItemPickaxe; // Import ItemPickaxe for pickaxe functionality
import net.minecraft.item.Item.ToolMaterial; // Import tool material
import net.minecraft.world.World; // Import for world

public class ItemDiamondPickaxe extends ItemPickaxe { // Extend ItemPickaxe for pickaxe behavior
    public ItemDiamondPickaxe() { // Constructor
        super(ToolMaterial.EMERALD); // Use emerald material for high durability (like diamond)
        this.setUnlocalizedName("diamondPickaxe"); // Set internal name
        this.setCreativeTab(CreativeTabs.tabTools); // Place in Tools tab
    }

    @Override
    public boolean onBlockDestroyed(ItemStack stack, World world, Block block, int x, int y, int z, EntityPlayer player) { // Called when a block is destroyed with this tool
        if (!world.isRemote) { // Only on server side
            if (block == net.minecraft.init.Blocks.stone) { // Check if the block is stone
                block.dropBlockAsItem(world, x, y, z, world.getBlockMetadata(x, y, z), 0); // Drop the normal stone item
                world.setBlockToAir(x, y, z); // Remove the block
                // Additionally drop a diamond
                net.minecraft.init.Blocks.diamond_ore.dropBlockAsItem(world, x, y, z, 0, 1); // Drop diamond ore item (or use Items.diamond)
            }
        }
        return super.onBlockDestroyed(stack, world, block, x, y, z, player); // Call parent method
    }
}
```

Register this tool as in Step 2, add static reference, texture, and localization.

## Adding a New Block

### Step 1: Create the Block Class (Optional)
If your block needs special behavior, create a new class extending `Block` or a subclass like `BlockOre` for ores.

**File to create:** `src/game/java/net/minecraft/block/BlockCustomOre.java`

For a basic block:

```java
package net.minecraft.block;

import net.minecraft.block.material.Material;

public class BlockCustomOre extends Block {
    public BlockCustomOre() {
        super(Material.rock);
        this.setHardness(3.0F);
        this.setResistance(5.0F);
        this.setStepSound(soundTypeStone);
        this.setUnlocalizedName("customOre");
        this.setCreativeTab(CreativeTabs.tabBlock);
    }
}
```

For an ore that drops items:

```java
package net.minecraft.block;

import net.minecraft.item.Item;

public class BlockCustomOre extends BlockOre {
    public BlockCustomOre() {
        this.setHardness(3.0F);
        this.setResistance(5.0F);
        this.setUnlocalizedName("customOre");
    }

    @Override
    public Item getItemDropped(IBlockState state, EaglercraftRandom rand, int fortune) {
        return Items.custom_ingot; // Assuming you have a custom ingot item
    }
}
```

### Step 2: Register the Block
Add the block registration in the `registerBlocks()` method.

**File to edit:** `src/game/java/net/minecraft/block/Block.java`

Find the `registerBlocks()` method and add your registration. Choose a unique ID (check existing ones, start from 500+).

```java
registerBlock(500, "custom_ore", new BlockCustomOre());
```

### Step 3: Add Static Reference
Add a static field and initialize it.

**File to edit:** `src/game/java/net/minecraft/init/Blocks.java`

Add the static declaration:

```java
public static Block custom_ore;
```

Then in the `doBootstrap()` method, add:

```java
custom_ore = getRegisteredBlock("custom_ore");
```

### Step 4: Add Texture and Localization (Optional)
- **Texture:** Add `custom_ore.png` to `assets/minecraft/textures/blocks/`
- **Localization:** Edit language files and add:
  ```
  tile.customOre.name=Custom Ore
  ```

### Blocks with Right-Click Behaviors
To make a block perform an action when right-clicked, override the `onBlockActivated` method in your block class.

**Example:** Add to `BlockCustomOre.java`

```java
import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.util.ChatComponentText;
import net.minecraft.world.World;

@Override
public boolean onBlockActivated(World world, int x, int y, int z, EntityPlayer player, int side, float hitX, float hitY, float hitZ) {
    if (!world.isRemote) {
        player.addChatMessage(new ChatComponentText("You activated the custom ore!"));
    }
    return true;
}
```

#### Example: Item Duplicator Block
Create a new block that duplicates the item held when right-clicked.

**File to create:** `src/game/java/net/minecraft/block/BlockDuplicator.java`

```java
package net.minecraft.block;

import net.minecraft.block.material.Material;
import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.item.ItemStack;
import net.minecraft.world.World;

public class BlockDuplicator extends Block {
    public BlockDuplicator() {
        super(Material.rock);
        this.setHardness(3.0F);
        this.setResistance(5.0F);
        this.setUnlocalizedName("duplicator");
        this.setCreativeTab(CreativeTabs.tabBlock);
    }

    @Override
    public boolean onBlockActivated(World world, int x, int y, int z, EntityPlayer player, int side, float hitX, float hitY, float hitZ) {
        if (!world.isRemote) {
            ItemStack heldItem = player.getHeldItem();
            if (heldItem != null) {
                ItemStack duplicate = heldItem.copy();
                if (player.inventory.addItemStackToInventory(duplicate)) {
                    player.addChatMessage(new ChatComponentText("Item duplicated!"));
                } else {
                    // Drop the item if inventory is full
                    player.dropPlayerItemWithRandomChoice(duplicate, false);
                    player.addChatMessage(new ChatComponentText("Item duplicated and dropped!"));
                }
            } else {
                player.addChatMessage(new ChatComponentText("Hold an item to duplicate!"));
            }
        }
        return true;
    }
}
```

Register this block as in Step 2, add static reference, texture, and localization.

## Adding New Entities/Mobs

Adding custom entities (mobs) involves creating an Entity class, registering it, adding spawn conditions, and optionally custom AI behaviors.

### Step 1: Create the Entity Class
Extend `EntityLiving` or a subclass like `EntityMob` for hostile mobs.

**File to create:** `src/game/java/net/minecraft/entity/monster/EntityCustomMob.java`

```java
package net.minecraft.entity.monster;

import net.minecraft.entity.SharedMonsterAttributes;
import net.minecraft.world.World;

public class EntityCustomMob extends EntityMob {
    public EntityCustomMob(World world) {
        super(world);
        this.setSize(0.6F, 1.8F);
        this.tasks.addTask(1, new EntityAIAttackOnCollide(this, EntityPlayer.class, 1.0D, false));
        this.tasks.addTask(2, new EntityAIWander(this, 1.0D));
        this.targetTasks.addTask(1, new EntityAINearestAttackableTarget(this, EntityPlayer.class, 0, true));
    }

    @Override
    protected void applyEntityAttributes() {
        super.applyEntityAttributes();
        this.getEntityAttribute(SharedMonsterAttributes.maxHealth).setBaseValue(20.0D);
        this.getEntityAttribute(SharedMonsterAttributes.movementSpeed).setBaseValue(0.25D);
        this.getEntityAttribute(SharedMonsterAttributes.attackDamage).setBaseValue(3.0D);
    }

    @Override
    protected String getLivingSound() {
        return "mob.zombie.say";
    }

    @Override
    protected String getHurtSound() {
        return "mob.zombie.hurt";
    }

    @Override
    protected String getDeathSound() {
        return "mob.zombie.death";
    }

    @Override
    protected void dropFewItems(boolean wasRecentlyHit, int lootingLevel) {
        this.dropItem(Items.diamond, 1);
    }
}
```

### Step 2: Register the Entity
Add registration in the entity registration method.

**File to edit:** `src/game/java/net/minecraft/entity/EntityList.java`

Find the `addMapping` calls and add:

```java
addMapping(EntityCustomMob.class, "CustomMob", 500);
```

### Step 3: Add Spawn Conditions
**File to edit:** `src/game/java/net/minecraft/world/biome/BiomeGenBase.java` or specific biome files.

In the biome's `getSpawnableList` method, add:

```java
this.spawnableMonsterList.add(new BiomeGenBase.SpawnListEntry(EntityCustomMob.class, 10, 1, 4));
```

### Step 4: Add Texture and Localization (Optional)
- **Texture:** Add `custom_mob.png` to `assets/minecraft/textures/entity/custom_mob/`
- **Localization:** Add `entity.CustomMob.name=Custom Mob` to language files.

### Custom Mob Abilities
To give mobs special abilities, override methods like `onLivingUpdate` for passive effects or `attackEntityAsMob` for attack behaviors.

#### Example: Fire-Breathing Mob
Add to `EntityCustomMob.java`:

```java
@Override
public void onLivingUpdate() {
    super.onLivingUpdate();
    if (!this.worldObj.isRemote && this.rand.nextInt(100) == 0) {
        // Breathe fire occasionally
        EntitySmallFireball fireball = new EntitySmallFireball(this.worldObj, this, 0, 0, 0);
        fireball.setPosition(this.posX, this.posY + this.getEyeHeight(), this.posZ);
        this.worldObj.spawnEntityInWorld(fireball);
    }
}
```

### Rideable Entities (e.g., Cars or Flying Mounts)
To create a rideable entity like a car or flying mount, extend `EntityAnimal` or `EntityLiving`, make it interactable for mounting, and override movement to control speed and direction based on rider input.

#### Example: Rideable Car
**File to create:** `src/game/java/net/minecraft/entity/passive/EntityRideableCar.java`

```java
package net.minecraft.entity.passive;

import net.minecraft.entity.Entity;
import net.minecraft.entity.SharedMonsterAttributes;
import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.util.MathHelper;
import net.minecraft.world.World;

public class EntityRideableCar extends EntityAnimal {
    public EntityRideableCar(World world) {
        super(world);
        this.setSize(1.0F, 0.5F); // Size of the car
    }

    @Override
    protected void applyEntityAttributes() {
        super.applyEntityAttributes();
        this.getEntityAttribute(SharedMonsterAttributes.maxHealth).setBaseValue(20.0D);
        this.getEntityAttribute(SharedMonsterAttributes.movementSpeed).setBaseValue(0.5D); // Base speed, can be customized
    }

    @Override
    public boolean interact(EntityPlayer player) {
        if (!this.worldObj.isRemote && (this.riddenByEntity == null || this.riddenByEntity == player)) {
            player.mountEntity(this);
            return true;
        }
        return super.interact(player);
    }

    @Override
    public void moveEntityWithHeading(float strafe, float forward) {
        if (this.riddenByEntity != null && this.riddenByEntity instanceof EntityPlayer) {
            EntityPlayer rider = (EntityPlayer) this.riddenByEntity;
            this.rotationYaw = rider.rotationYaw;
            this.prevRotationYaw = this.rotationYaw;
            this.rotationPitch = rider.rotationPitch * 0.5F;
            this.setRotation(this.rotationYaw, this.rotationPitch);
            this.renderYawOffset = this.rotationYaw;
            this.rotationYawHead = this.rotationYaw;

            // Custom speed
            float speed = 0.8F; // Adjust for desired speed
            if (rider.isSneaking()) {
                speed *= 0.3F; // Slow down when sneaking
            }
            this.motionX = -MathHelper.sin(this.rotationYaw * (float)Math.PI / 180.0F) * speed * forward;
            this.motionZ = MathHelper.cos(this.rotationYaw * (float)Math.PI / 180.0F) * speed * forward;
            this.motionY = 0; // Ground car, no vertical movement

            if (this.onGround) {
                this.isAirBorne = false;
            }
        } else {
            super.moveEntityWithHeading(strafe, forward);
        }
    }

    @Override
    protected String getLivingSound() {
        return null; // No sound
    }

    @Override
    protected String getHurtSound() {
        return "random.break"; // Sound when hurt
    }

    @Override
    protected String getDeathSound() {
        return "random.break";
    }
}
```

Register this entity similarly to the custom mob (Steps 2-4 above), but use `EntityRideableCar.class` and adjust spawn conditions if desired.

#### Example: Flying Mount
To make it fly, modify the `moveEntityWithHeading` method to allow vertical movement. Add this to the car class or create a separate flying entity:

```java
// In moveEntityWithHeading, after setting motionX and motionZ:
if (rider.moveForward > 0) {
    this.motionY += 0.1F; // Ascend when moving forward
} else if (rider.moveForward < 0) {
    this.motionY -= 0.1F; // Descend when moving backward
}
// Clamp motionY to prevent excessive speed
this.motionY = MathHelper.clamp_float(this.motionY, -0.5F, 0.5F);
```

This allows the entity to fly by controlling vertical movement based on rider input.

## Creating Custom GUIs

GUIs (Graphical User Interfaces) allow players to interact with custom menus, inventories, or settings. In EaglercraftX, GUIs consist of a `GuiScreen` for client-side rendering and a `Container` for server-side logic (especially for inventories).

### Step 1: Create the Container Class (for Inventories)
If your GUI has an inventory, create a Container class to handle slot logic.

**File to create:** `src/game/java/net/minecraft/inventory/ContainerCustom.java`

```java
package net.minecraft.inventory;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.entity.player.InventoryPlayer;
import net.minecraft.inventory.Container;
import net.minecraft.inventory.Slot;
import net.minecraft.item.ItemStack;

public class ContainerCustom extends Container {
    public ContainerCustom(InventoryPlayer playerInv) {
        // Add player inventory slots
        for (int i = 0; i < 3; ++i) {
            for (int j = 0; j < 9; ++j) {
                this.addSlotToContainer(new Slot(playerInv, j + i * 9 + 9, 8 + j * 18, 84 + i * 18));
            }
        }
        for (int k = 0; k < 9; ++k) {
            this.addSlotToContainer(new Slot(playerInv, k, 8 + k * 18, 142));
        }
    }

    @Override
    public boolean canInteractWith(EntityPlayer player) {
        return true;
    }

    @Override
    public ItemStack transferStackInSlot(EntityPlayer player, int index) {
        // Handle shift-clicking
        return null;
    }
}
```

### Step 2: Create the GuiScreen Class
Create the client-side GUI class that renders the interface.

**File to create:** `src/game/java/net/minecraft/client/gui/GuiCustom.java`

```java
package net.minecraft.client.gui;

import net.minecraft.client.gui.inventory.GuiContainer;
import net.minecraft.inventory.ContainerCustom;
import net.minecraft.util.ResourceLocation;
import org.lwjgl.opengl.GL11;

public class GuiCustom extends GuiContainer {
    private static final ResourceLocation texture = new ResourceLocation("textures/gui/container/custom.png");

    public GuiCustom(ContainerCustom container) {
        super(container);
        this.xSize = 176;
        this.ySize = 166;
    }

    @Override
    protected void drawGuiContainerBackgroundLayer(float partialTicks, int mouseX, int mouseY) {
        GL11.glColor4f(1.0F, 1.0F, 1.0F, 1.0F);
        this.mc.getTextureManager().bindTexture(texture);
        int k = (this.width - this.xSize) / 2;
        int l = (this.height - this.ySize) / 2;
        this.drawTexturedModalRect(k, l, 0, 0, this.xSize, this.ySize);
    }

    @Override
    protected void drawGuiContainerForegroundLayer(int mouseX, int mouseY) {
        this.fontRendererObj.drawString("Custom GUI", 8, 6, 4210752);
    }
}
```

### Step 3: Open the GUI
To open the GUI, use `player.openGui()` or for items, in `onItemRightClick`.

**Example: Open GUI from Item Right-Click**
Add to an item's `onItemRightClick`:

```java
if (!world.isRemote) {
    player.openGui(modInstance, guiId, world, (int)player.posX, (int)player.posY, (int)player.posZ);
}
```

For client-side, the GuiHandler is needed, but in vanilla, it's simpler.

In EaglercraftX, you might need to register the GUI handler.

**File to edit:** `src/game/java/net/minecraft/client/Minecraft.java` or create a GuiHandler.

For simplicity, assume using the existing system.

### Step 4: Add GUI Texture
Create `custom.png` in `assets/minecraft/textures/gui/container/`

### Custom GUI with Buttons
For non-inventory GUIs, extend `GuiScreen` directly.

**File to create:** `src/game/java/net/minecraft/client/gui/GuiCustomMenu.java`

```java
package net.minecraft.client.gui;

import net.minecraft.client.gui.GuiButton;
import net.minecraft.client.gui.GuiScreen;

public class GuiCustomMenu extends GuiScreen {
    @Override
    public void initGui() {
        this.buttonList.add(new GuiButton(0, this.width / 2 - 100, this.height / 2 - 24, "Button 1"));
        this.buttonList.add(new GuiButton(1, this.width / 2 - 100, this.height / 2, "Button 2"));
    }

    @Override
    protected void actionPerformed(GuiButton button) {
        switch (button.id) {
            case 0:
                // Action for button 1
                break;
            case 1:
                // Action for button 2
                this.mc.displayGuiScreen(null); // Close GUI
                break;
        }
    }

    @Override
    public void drawScreen(int mouseX, int mouseY, float partialTicks) {
        this.drawDefaultBackground();
        this.drawCenteredString(this.fontRendererObj, "Custom Menu", this.width / 2, 20, 16777215);
        super.drawScreen(mouseX, mouseY, partialTicks);
    }
}
```

## Creating and Activating Effects

Effects (potion effects) are status conditions that can be applied to entities (players, mobs, etc.). They can grant abilities, apply damage, alter appearance, or modify behavior.

### Types of Effects

Effects can be triggered by:
- **Right-clicking an item** (wands, crystals, potions)
- **Drinking a potion**
- **Equipping armor or costume**
- **Being hit by a projectile**
- **Command activation** (`/effect @s speed 300`)
- **Entity collision** (touching a special block or entity)
- **On-tick updates** (continuous effect while holding/wearing item)

### Step 1: Create a Custom Effect Class

Custom effects extend the base `Potion` class to define custom behavior.

**File to create:** `src/game/java/net/minecraft/potion/PotionCustomEffect.java`

```java
package net.minecraft.potion;

import net.minecraft.entity.EntityLivingBase;
import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.potion.Potion;

public class PotionCustomEffect extends Potion {
    public PotionCustomEffect(int id, boolean isBadEffect, int liquidColor) {
        super(id, isBadEffect, liquidColor);
        this.setPotionName("effect.customEffect");
    }

    @Override
    public void performEffect(EntityLivingBase entity, int amplifier) {
        // Called every time the effect ticks (see isReady method)
        if (entity instanceof EntityPlayer) {
            EntityPlayer player = (EntityPlayer) entity;
            // Custom behavior here
            player.addPotionEffect(new PotionEffect(Potion.moveSpeed, 10, amplifier));
        }
    }

    @Override
    public boolean isReady(int duration, int amplifier) {
        // Effect triggers every 10 ticks
        int interval = 10 - amplifier; // Higher level = more frequent
        return duration % Math.max(interval, 1) == 0;
    }
}
```

### Step 2: Register the Custom Effect

**File to edit:** `src/game/java/net/minecraft/potion/Potion.java`

Add to the class definition (around line 150):

```java
// Custom effect ID (choose unused IDs like 50+)
public static Potion customEffect = new PotionCustomEffect(50, false, 0xFF00FF); // Purple color
public static Potion floatEffect = new PotionCustomEffect(51, false, 0x9900FF); // Levitation effect
public static Potion speedEffect = new PotionCustomEffect(52, false, 0x00FF00); // Speed effect
```

### Step 3: Activate Effects on Items

#### Method A: Right-Click Activation

**File to create:** `src/game/java/net/minecraft/item/ItemEffectWand.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.potion.Potion;
import net.minecraft.potion.PotionEffect;
import net.minecraft.world.World;

public class ItemEffectWand extends Item {
    public ItemEffectWand() {
        this.setUnlocalizedName("effectWand");
        this.setCreativeTab(CreativeTabs.tabMagic);
        this.setMaxStackSize(1);
    }

    @Override
    public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
        if (!world.isRemote) {
            // Apply custom effect when right-clicked
            player.addPotionEffect(new PotionEffect(Potion.customEffect, 600, 0));
        }
        return itemStack;
    }
}
```

#### Method B: On-Tick Activation (Continuous Effect While Holding)

**File to create:** `src/game/java/net/minecraft/item/ItemContinuousEffectItem.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.potion.Potion;
import net.minecraft.potion.PotionEffect;
import net.minecraft.world.World;

public class ItemContinuousEffectItem extends Item {
    public ItemContinuousEffectItem() {
        this.setUnlocalizedName("continuousEffectItem");
        this.setCreativeTab(CreativeTabs.tabMagic);
        this.setMaxStackSize(1);
    }

    @Override
    public void onUpdate(ItemStack itemStack, World world, net.minecraft.entity.Entity entity, int slot, boolean isSelected) {
        // Called every tick while in player inventory
        if (!world.isRemote && entity instanceof EntityPlayer && isSelected) {
            EntityPlayer player = (EntityPlayer) entity;
            // Apply effect every tick (or implement interval logic)
            if (world.getTotalWorldTime() % 20 == 0) { // Every second
                player.addPotionEffect(new PotionEffect(Potion.floatEffect, 25, 0));
            }
        }
    }
}
```

#### Method C: Armor Activation (Equip Effect)

**File to create:** `src/game/java/net/minecraft/item/ItemEffectArmor.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.inventory.EntityEquipmentHelper;
import net.minecraft.potion.Potion;
import net.minecraft.potion.PotionEffect;
import net.minecraft.world.World;

public class ItemEffectArmor extends ItemArmor {
    public ItemEffectArmor(ArmorMaterial material, int renderIndex, int armorType) {
        super(material, renderIndex, armorType);
        this.setUnlocalizedName("effectArmor");
    }

    @Override
    public void onArmorTick(World world, EntityPlayer player, ItemStack itemStack) {
        // Called every tick while armor is equipped
        if (!world.isRemote) {
            // Apply effect while wearing armor
            player.addPotionEffect(new PotionEffect(Potion.speedEffect, 10, 0));
        }
    }
}
```

#### Method D: Entity Collision Activation

**File to create:** `src/game/java/net/minecraft/block/BlockEffectTrigger.java`

```java
package net.minecraft.block;

import net.minecraft.entity.Entity;
import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.potion.Potion;
import net.minecraft.potion.PotionEffect;
import net.minecraft.world.World;

public class BlockEffectTrigger extends Block {
    public BlockEffectTrigger() {
        this.setUnlocalizedName("effectTriggerBlock");
        this.setCreativeTab(CreativeTabs.tabMagic);
    }

    @Override
    public void onEntityWalking(World world, int x, int y, int z, Entity entity) {
        // Called when entity walks on the block
        if (!world.isRemote && entity instanceof EntityPlayer) {
            EntityPlayer player = (EntityPlayer) entity;
            player.addPotionEffect(new PotionEffect(Potion.customEffect, 300, 0));
        }
    }

    @Override
    public void onEntityCollidedWithBlock(World world, int x, int y, int z, Entity entity) {
        // Called when entity collides with block
        if (!world.isRemote && entity instanceof EntityPlayer) {
            EntityPlayer player = (EntityPlayer) entity;
            player.addPotionEffect(new PotionEffect(Potion.floatEffect, 100, 0));
        }
    }
}
```

#### Method E: Projectile Activation

**File to create:** `src/game/java/net/minecraft/entity/projectile/EntityEffectProjectile.java`

```java
package net.minecraft.entity.projectile;

import net.minecraft.entity.EntityLivingBase;
import net.minecraft.entity.projectile.EntityThrowable;
import net.minecraft.potion.Potion;
import net.minecraft.potion.PotionEffect;
import net.minecraft.world.World;

public class EntityEffectProjectile extends EntityThrowable {
    public EntityEffectProjectile(World world) {
        super(world);
    }

    public EntityEffectProjectile(World world, EntityLivingBase entity) {
        super(world, entity);
    }

    @Override
    protected void onImpact(RayTraceResult result) {
        if (!this.worldObj.isRemote && result.entityHit instanceof EntityLivingBase) {
            // Apply effect to entity hit by projectile
            EntityLivingBase entity = (EntityLivingBase) result.entityHit;
            entity.addPotionEffect(new PotionEffect(Potion.customEffect, 600, 1));
        }
    }
}
```

### Step 4: Common Built-in Effects Reference

| Effect Name | Potion ID | Color | Uses |
|------------|-----------|-------|------|
| Speed | moveSpeed | Green | Faster movement |
| Slowness | moveSlowdown | Gray | Slower movement |
| Haste | digSpeed | Orange | Faster mining |
| Mining Fatigue | digSlowdown | Gray | Slower mining |
| Strength | damageBoost | Red | Increased damage |
| Instant Health | heal | Red | Heal instantly |
| Instant Damage | harm | Blue | Damage instantly |
| Jump Boost | jump | Green | Higher jumps |
| Regeneration | regeneration | Pink | Health regen |
| Resistance | resistance | Purple | Damage resistance |
| Fire Resistance | fireResistance | Orange | Immune to fire |
| Water Breathing | waterBreathing | Cyan | Breathe underwater |
| Invisibility | invisibility | Gray | Become invisible |
| Blindness | blindness | Black | Can't see |
| Night Vision | nightVision | Blue | See in dark |
| Hunger | hunger | Brown | Faster hunger |
| Weakness | weakness | Blue | Less damage |
| Poison | poison | Green | Damage over time |
| Wither | wither | Black | Wither effect |
| Health Boost | healthBoost | Red | Extra health |
| Absorption | absorption | Yellow | Temporary shield |
| Saturation | saturation | Red | Food saturation |
| Glowing | glowing | White | Glowing outline |
| Levitation | levitation | Purple | Float upward |

### Step 5: Register and Test Effects

After creating your effect classes and items:

1. **Register items** in `Item.java` (see Item Registration section)
2. **Add textures** to `assets/minecraft/textures/items/`
3. **Add localization** to `javascript/lang/en_US.lang`:
   ```
   item.effectWand.name=Effect Wand
   item.continuousEffectItem.name=Magical Staff
   effect.customEffect=Custom Effect
   effect.floatEffect=Float
   effect.speedEffect=Speed Boost
   ```
4. **Compile** with `CompileEPK.sh` and `CompileJS.sh`
5. **Test** by obtaining items with `/give @s [itemname]`

### Step 6: Combining Multiple Effects

Apply multiple effects at once:

```java
@Override
public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
    if (!world.isRemote) {
        // Apply multiple effects
        player.addPotionEffect(new PotionEffect(Potion.moveSpeed, 600, 1));      // Speed level 2
        player.addPotionEffect(new PotionEffect(Potion.jump, 600, 1));            // Jump boost level 2
        player.addPotionEffect(new PotionEffect(Potion.damageBoost, 600, 0));     // Strength level 1
    }
    return itemStack;
}
```

### Effect Parameters

When applying effects:
```java
new PotionEffect(potionType, duration, amplifier, showParticles, showIcon)
```

- **potionType**: The `Potion` enum value
- **duration**: Ticks (20 ticks = 1 second)
- **amplifier**: Level - 1 = 0, Level 2 = 1, etc.
- **showParticles**: Boolean (true = show particles)
- **showIcon**: Boolean (true = show status icon)

### Example: Making the Player Levitate

Here's a complete example of creating a levitation wand that makes the player float when right-clicked.

#### Step 1: Create the Levitation Wand Item

**File to create:** `src/game/java/net/minecraft/item/ItemLevitationWand.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.potion.Potion;
import net.minecraft.potion.PotionEffect;
import net.minecraft.world.World;

public class ItemLevitationWand extends Item {
    public ItemLevitationWand() {
        this.setUnlocalizedName("levitationWand");
        this.setCreativeTab(CreativeTabs.tabMagic);
        this.setMaxStackSize(1);
    }

    @Override
    public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
        if (!world.isRemote) {
            // Apply levitation potion effect
            // Duration: 300 ticks = 15 seconds
            // Amplifier: 0 = Level 1 (gentle float)
            player.addPotionEffect(new PotionEffect(Potion.levitation, 300, 0, true, true));
            
            // Optional: Consume durability or cooldown
            // itemStack.damageItem(1, player);
        }
        return itemStack;
    }
}
```

#### Step 2: Register the Levitation Wand

**File to edit:** `src/game/java/net/minecraft/item/Item.java`

Add to `registerItems()` method:

```java
registerItem(5051, "levitation_wand", new ItemLevitationWand());
```

**File to edit:** `src/game/java/net/minecraft/init/Items.java`

Add static field:
```java
public static Item levitation_wand;
```

Add in `doBootstrap()` method:
```java
levitation_wand = getRegisteredItem("levitation_wand");
```

#### Step 3: Add Textures and Localization

**Texture:** Create `assets/minecraft/textures/items/levitation_wand.png`

**Localization:** Edit `javascript/lang/en_US.lang`
```
item.levitationWand.name=Levitation Wand
```

#### Step 4: Create a Continuous Levitation Armor

For a costume that keeps the player floating while equipped:

**File to create:** `src/game/java/net/minecraft/item/ItemLevitationBoots.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.potion.Potion;
import net.minecraft.potion.PotionEffect;
import net.minecraft.world.World;

public class ItemLevitationBoots extends ItemArmor {
    public ItemLevitationBoots(ArmorMaterial material, int renderIndex, int armorType) {
        super(material, renderIndex, armorType);
        this.setUnlocalizedName("levitationBoots");
    }

    @Override
    public void onArmorTick(World world, EntityPlayer player, ItemStack itemStack) {
        // Apply levitation while boots are equipped
        if (!world.isRemote) {
            // Continuous levitation (10 ticks to keep refreshing)
            player.addPotionEffect(new PotionEffect(Potion.levitation, 10, 0, false, false));
        }
    }
}
```

#### Step 5: Create a Levitation Potion

**File to create:** `src/game/java/net/minecraft/item/ItemLevitationPotion.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.potion.Potion;
import net.minecraft.potion.PotionEffect;
import net.minecraft.world.World;

public class ItemLevitationPotion extends ItemPotion {
    public ItemLevitationPotion() {
        super();
        this.setUnlocalizedName("levitationPotion");
        this.setCreativeTab(CreativeTabs.tabBrewing);
    }

    @Override
    public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
        if (!world.isRemote) {
            // Apply levitation for 20 seconds when drunk
            player.addPotionEffect(new PotionEffect(Potion.levitation, 400, 0));
            
            // Player drinks the potion
            if (!player.capabilities.isCreativeMode) {
                itemStack.stackSize--;
                if (itemStack.stackSize <= 0) {
                    return new ItemStack(Items.glass_bottle);
                }
            }
        }
        return itemStack;
    }
}
```

#### Step 6: Create a Levitation Spell Item (On-Tick)

For a continuous levitation effect while holding the item:

**File to create:** `src/game/java/net/minecraft/item/ItemLevitationStaff.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.potion.Potion;
import net.minecraft.potion.PotionEffect;
import net.minecraft.world.World;

public class ItemLevitationStaff extends Item {
    public ItemLevitationStaff() {
        this.setUnlocalizedName("levitationStaff");
        this.setCreativeTab(CreativeTabs.tabMagic);
        this.setMaxStackSize(1);
    }

    @Override
    public void onUpdate(ItemStack itemStack, World world, net.minecraft.entity.Entity entity, int slot, boolean isSelected) {
        // Apply levitation every tick while holding the staff
        if (!world.isRemote && entity instanceof EntityPlayer && isSelected) {
            EntityPlayer player = (EntityPlayer) entity;
            
            // Apply levitation effect every tick (keeps player floating)
            if (world.getTotalWorldTime() % 10 == 0) { // Update every half-second
                player.addPotionEffect(new PotionEffect(Potion.levitation, 15, 0, false, false));
            }
        }
    }
}
```

#### Testing Levitation

1. Compile with `CompileEPK.sh` and `CompileJS.sh`
2. In-game, use: `/give @s levitation_wand` to get the wand
3. Right-click to activate levitation
4. The player will float upward for 15 seconds
5. Fall damage is disabled while levitating

#### Levitation Duration Reference

| Duration (Ticks) | Seconds | Use Case |
|------------------|---------|----------|
| 20 | 1 | Very short float |
| 100 | 5 | Short burst |
| 200 | 10 | Medium duration |
| 300 | 15 | Default wand |
| 400 | 20 | Long potion |
| 600 | 30 | Extended spell |
| 1200 | 60 | Minute-long effect |

#### Levitation Amplifier Levels

| Amplifier | Level | Float Speed | Notes |
|-----------|-------|-------------|-------|
| 0 | I | Slow, gentle | Recommended for safety |
| 1 | II | Moderate | Standard levitation |
| 2 | III | Fast | Players rise quickly |
| 3+ | IV+ | Very fast | Can be disorienting |

#### Combining Levitation with Other Effects

Create a "Magic Flight" item with levitation + speed:

```java
@Override
public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
    if (!world.isRemote) {
        // Levitation effect
        player.addPotionEffect(new PotionEffect(Potion.levitation, 300, 0, true, true));
        
        // Speed boost for movement while floating
        player.addPotionEffect(new PotionEffect(Potion.moveSpeed, 300, 1, true, true));
        
        // Optional: Night vision to see while up high
        player.addPotionEffect(new PotionEffect(Potion.nightVision, 300, 0, false, false));
    }
    return itemStack;
}
```

## Creating Potions

Potions are special items that grant effects when consumed. You can create custom potions with unique effects and appearances.

### Step 1: Create a Custom Potion Class

**File to create:** `src/game/java/net/minecraft/potion/PotionCustom.java`

```java
package net.minecraft.potion;

import net.minecraft.entity.EntityLivingBase;
import net.minecraft.potion.Potion;

public class PotionCustom extends Potion {
    public PotionCustom(int id, boolean isBadEffect, int liquidColor) {
        super(id, isBadEffect, liquidColor);
        this.setPotionName("effect.customPotion"); // Name for localization
    }

    @Override
    public void performEffect(EntityLivingBase entity, int amplifier) {
        // Apply custom effects here
        if (entity instanceof EntityPlayer) {
            EntityPlayer player = (EntityPlayer) entity;
            // Example: Speed boost
            player.addPotionEffect(new PotionEffect(Potion.moveSpeed, 10, amplifier));
        }
    }

    @Override
    public boolean isReady(int duration, int amplifier) {
        // Determine when the effect should trigger (every X ticks)
        return duration % 10 == 0; // Every 10 ticks
    }
}
```

### Step 2: Register the Custom Potion

**File to edit:** `src/game/java/net/minecraft/potion/Potion.java`

Add your custom potion registration in the `registerPotions()` method (around line 150):

```java
public static Potion custom_potion = new PotionCustom(50, false, 0xFF00FF); // ID 50, not bad, purple color
```

### Step 3: Create a Potion Item

Create a drinkable potion item:

**File to create:** `src/game/java/net/minecraft/item/ItemCustomPotion.java`

```java
package net.minecraft.item;

import net.minecraft.entity.player.EntityPlayer;
import net.minecraft.potion.Potion;
import net.minecraft.potion.PotionEffect;
import net.minecraft.world.World;

public class ItemCustomPotion extends ItemPotion {
    public ItemCustomPotion() {
        super();
        this.setUnlocalizedName("customPotion");
        this.setCreativeTab(CreativeTabs.tabBrewing);
    }

    @Override
    public ItemStack onItemRightClick(ItemStack itemStack, World world, EntityPlayer player) {
        if (!world.isRemote) {
            // Apply the custom potion effect
            player.addPotionEffect(new PotionEffect(Potion.custom_potion, 3600, 0)); // 3 minutes duration, level 1
            
            // Decrease stack size (player drinks it)
            if (!player.capabilities.isCreativeMode) {
                itemStack.stackSize--;
                if (itemStack.stackSize <= 0) {
                    return new ItemStack(Items.glass_bottle); // Return empty bottle
                }
            }
        }
        return itemStack;
    }
}
```

### Step 4: Register the Potion Item

**File to edit:** `src/game/java/net/minecraft/item/Item.java`

Add the registration in `registerItems()`:

```java
registerItem(5050, "custom_potion", new ItemCustomPotion());
```

Add to `src/game/java/net/minecraft/init/Items.java`:

```java
public static Item custom_potion;
```

And in the `doBootstrap()` method:

```java
custom_potion = getRegisteredItem("custom_potion");
```

### Step 5: Create Potion Brewing Recipes

You can create potions through the brewing stand using custom recipes.

**File to edit:** `src/game/java/net/minecraft/potion/PotionBrewer.java`

Add brewing recipes in the constructor:

```java
// Add custom ingredient -> base potion recipe
this.addRecipe(new ItemStack(Items.awkward_potion), 
    new ItemStack(Items.golden_carrot), 
    new ItemStack(Items.custom_potion));

// Add potion modifier (like glowstone dust for Level II)
this.addRecipe(new ItemStack(Items.custom_potion), 
    new ItemStack(Items.glowstone_dust), 
    new ItemStack(Items.custom_potion)); // Level II version
```

### Step 6: Add Textures and Localization

**Texture:** Add `customPotion.png` to `assets/minecraft/textures/items/`

**Localization:** Edit `javascript/lang/en_US.lang` and add:

```
item.customPotion.name=Custom Potion
effect.customPotion=Custom Effect
```

### Preset Potion Effects

You can use existing Minecraft potion effects:

| Effect | Duration | Effect |
|--------|----------|--------|
| `Potion.moveSpeed` | Increased movement speed |
| `Potion.moveSlowdown` | Decreased movement speed |
| `Potion.digSpeed` | Faster mining (Haste) |
| `Potion.digSlowdown` | Slower mining (Mining Fatigue) |
| `Potion.damageBoost` | Increased damage (Strength) |
| `Potion.poison` | Damage over time |
| `Potion.heal` | Instant health |
| `Potion.harm` | Instant damage |
| `Potion.jump` | Increased jump height |
| `Potion.regeneration` | Health regeneration |
| `Potion.resistance` | Damage resistance |
| `Potion.fireResistance` | Fire immunity |
| `Potion.waterBreathing` | Underwater breathing |
| `Potion.invisibility` | Invisibility |
| `Potion.blindness` | Blindness |
| `Potion.nightVision` | Night vision |
| `Potion.hunger` | Hunger |
| `Potion.weakness` | Reduced damage |
| `Potion.wither` | Wither effect |
| `Potion.healthBoost` | Extra health |
| `Potion.absorption` | Temporary health |
| `Potion.saturation` | Food saturation |

### Example: Speed Potion

```java
public static Potion speed_potion = new Potion(51, false, 0x00FF00) {
    @Override
    public void performEffect(EntityLivingBase entity, int amplifier) {
        if (entity instanceof EntityPlayer) {
            EntityPlayer player = (EntityPlayer) entity;
            player.capabilities.setPlayerWalkSpeed(0.3F + (amplifier * 0.1F)); // Speed increases with level
        }
    }
};
```

### Example: Levitation Potion

```java
public static Potion levitation_potion = new Potion(52, false, 0x9900FF) {
    @Override
    public void performEffect(EntityLivingBase entity, int amplifier) {
        if (entity.motionY < 0) {
            entity.motionY = 0; // Stop falling
            entity.motionY += 0.05 * (amplifier + 1); // Float upward
        }
    }
};
```

### Potion Duration and Amplifier

When applying potion effects:
- **Duration**: Time in ticks (20 ticks = 1 second)
- **Amplifier**: Effect level (0 = level 1, 1 = level 2, etc.)

Example:
```java
// 3600 ticks = 3 minutes, level 2
player.addPotionEffect(new PotionEffect(Potion.moveSpeed, 3600, 1));
```

## Adding Crafting Recipes

Recipes are added using `CraftingManager.addRecipe()`.

### Shaped Recipes
**File to edit:** `src/game/java/net/minecraft/item/Item.java`

Add in the `registerItems()` method after item registrations:

```java
CraftingManager.getInstance().addRecipe(new ItemStack(Items.custom_food), new Object[] {
    "XXX",
    "XYX",
    "XXX",
    'X', Items.apple,
    'Y', Items.diamond
});
```

### Shapeless Recipes
**File to edit:** `src/game/java/net/minecraft/item/Item.java`

Add in the `registerItems()` method:

```java
CraftingManager.getInstance().addShapelessRecipe(new ItemStack(Items.custom_food), new Object[] {
    Items.apple, Items.diamond
});
```

### Crafting Recipes with Enchantments
To add an enchantment to the crafted item, create the `ItemStack`, apply the enchantment, and use it in the recipe.

**File to edit:** `src/game/java/net/minecraft/item/Item.java`

First, ensure the import is present at the top of the file:

```java
import net.minecraft.enchantment.Enchantment;
```

Then, in the `registerItems()` method:

```java
ItemStack enchantedSword = new ItemStack(Items.flaming_sword); // Assuming you have a custom sword item
enchantedSword.addEnchantment(Enchantment.fireAspect, 2); // Add Fire Aspect enchantment level 2
CraftingManager.getInstance().addRecipe(enchantedSword, new Object[] {
    "XXX",
    "XYX",
    "XXX",
    'X', Items.iron_ingot,
    'Y', Items.blaze_powder
});
```

This will craft an enchanted flaming sword when using the specified recipe.

## Adding Furnace Recipes

Furnace recipes are added using `FurnaceRecipes.instance().addSmelting()`.

**File to edit:** `src/game/java/net/minecraft/item/crafting/FurnaceRecipes.java`

In the constructor (around line 52), add:

```java
this.addSmelting(Items.custom_food, new ItemStack(Items.cooked_custom_food), 0.35F);
```

For blocks:

```java
this.addSmeltingRecipeForBlock(Blocks.custom_ore, new ItemStack(Items.custom_ingot), 1.0F);
```

## Building and Testing

After making changes, run these commands in the project root:

1. **Compile Java to TeaVM:**
   ```bash
   ./gradlew generateJavaScript
   ```

2. **Compile EPK (assets):**
   ```bash
   ./CompileEPK.sh
   ```

3. **Compile JavaScript:**
   ```bash
   ./CompileJS.sh
   ```

4. **Create offline download (optional):**
   ```bash
   ./MakeOfflineDownload.sh
   ```

5. **Test the client:**
   - Open `javascript/index.html` in a browser, or
   - Use the desktop runtime by importing `desktopRuntime/eclipseProject` into Eclipse/IntelliJ and running the debug configuration.

## Setting Up a Custom Server

EaglercraftX includes integrated server functionality for singleplayer and LAN multiplayer.

### Singleplayer Server
The singleplayer server runs automatically when you start a singleplayer world in the client. No additional setup is required.

### LAN Server
To host a LAN server:
1. Start the game in the desktop runtime or browser.
2. Create or load a world in singleplayer.
3. Open the pause menu (press Escape).
4. Click "Open to LAN".
5. Configure options like allow cheats, game mode, etc.
6. Click "Start LAN World".
7. Other players on the same network can join by selecting "Multiplayer" > "Direct Connect" and entering your IP address (usually shown in chat).

### Dedicated Multiplayer Server
For dedicated multiplayer servers, EaglercraftX uses BungeeCord or Velocity plugins. The server implementations are separate from this client repository.

To set up a custom dedicated server:
1. Download EaglercraftXBungee from the main EaglercraftX repository: https://github.com/lax1dude/eaglercraftx-1.8
2. Follow the setup instructions in the server's README.
3. Configure the server properties, plugins, and worlds as needed.
4. Run the server JAR with Java 8 or higher.

Note: This client repository does not include the server source code. For server modding or custom server development, refer to the main repository.

## Files Modified/Created Summary

| Step | File | Action |
|------|------|--------|
| Item Class | `src/game/java/net/minecraft/item/ItemCustomFood.java` or `ItemCustomIngot.java` | Create |
| Fireball Wand Item Class | `src/game/java/net/minecraft/item/ItemFireballWand.java` | Create |
| Teleport Crystal Item Class | `src/game/java/net/minecraft/item/ItemTeleportCrystal.java` | Create |
| Regeneration Amulet Item Class | `src/game/java/net/minecraft/item/ItemRegenerationAmulet.java` | Create |
| Block Placer Item Class | `src/game/java/net/minecraft/item/ItemBlockPlacer.java` | Create |
| Flaming Sword Item Class | `src/game/java/net/minecraft/item/ItemFlamingSword.java` | Create |
| Speed Boots Item Class | `src/game/java/net/minecraft/item/ItemSpeedBoots.java` | Create |
| Diamond Pickaxe Item Class | `src/game/java/net/minecraft/item/ItemDiamondPickaxe.java` | Create |
| Custom Mob Entity Class | `src/game/java/net/minecraft/entity/monster/EntityCustomMob.java` | Create |
| Item Registration | `src/game/java/net/minecraft/item/Item.java` | Edit |
| Static Reference | `src/game/java/net/minecraft/init/Items.java` | Edit |
| Item Texture | `assets/minecraft/textures/items/custom_food.png` | Create |
| Item Texture (Fireball Wand) | `assets/minecraft/textures/items/fireball_wand.png` | Create |
| Item Texture (Teleport Crystal) | `assets/minecraft/textures/items/teleport_crystal.png` | Create |
| Item Texture (Regeneration Amulet) | `assets/minecraft/textures/items/regeneration_amulet.png` | Create |
| Item Texture (Block Placer) | `assets/minecraft/textures/items/block_placer.png` | Create |
| Item Texture (Flaming Sword) | `assets/minecraft/textures/items/flaming_sword.png` | Create |
| Item Texture (Speed Boots) | `assets/minecraft/textures/items/speed_boots.png` | Create |
| Item Texture (Diamond Pickaxe) | `assets/minecraft/textures/items/diamond_pickaxe.png` | Create |
| Entity Texture | `assets/minecraft/textures/entity/custom_mob/custom_mob.png` | Create |
| Localization | `javascript/lang/en_US.lang` | Edit |
| Crafting Recipe | `src/game/java/net/minecraft/item/Item.java` | Edit |
| Furnace Recipe | `src/game/java/net/minecraft/item/crafting/FurnaceRecipes.java` | Edit |
| Custom Ingot Item Class | `src/game/java/net/minecraft/item/ItemCustomIngot.java` | Create |
| Custom Ingot Item Registration | `src/game/java/net/minecraft/item/Item.java` | Edit |
| Custom Ingot Static Reference | `src/game/java/net/minecraft/init/Items.java` | Edit |
| Custom Ingot Texture | `assets/minecraft/textures/items/custom_ingot.png` | Create |
| Block Class | `src/game/java/net/minecraft/block/BlockCustomOre.java` | Create |
| Duplicator Block Class | `src/game/java/net/minecraft/block/BlockDuplicator.java` | Create |
| Block Registration | `src/game/java/net/minecraft/block/Block.java` | Edit |
| Block Static Reference | `src/game/java/net/minecraft/init/Blocks.java` | Edit |
| Block Texture | `assets/minecraft/textures/blocks/custom_ore.png` | Create |
| Duplicator Block Texture | `assets/minecraft/textures/blocks/duplicator.png` | Create |
| Entity Registration | `src/game/java/net/minecraft/entity/EntityList.java` | Edit |
| Entity Spawn | `src/game/java/net/minecraft/world/biome/BiomeGenBase.java` | Edit |
| Container Class | `src/game/java/net/minecraft/inventory/ContainerCustom.java` | Create |
| GuiScreen Class | `src/game/java/net/minecraft/client/gui/GuiCustom.java` | Create |
| GuiMenu Class | `src/game/java/net/minecraft/client/gui/GuiCustomMenu.java` | Create |
| GUI Texture | `assets/minecraft/textures/gui/container/custom.png` | Create |

## Notes

- Ensure unique IDs for items and blocks (check existing ones in `Item.java` and `Block.java`).
- Recipes are registered during bootstrap, so add them in appropriate initialization code.
- For complex mods, consider creating separate recipe classes like `RecipesCustom.java`.
- The `classes.js` and `classes.js.map` files are generated during compilation and should not be edited manually.
- If you encounter build errors, check for missing imports or syntax issues.</content>
<parameter name="filePath">/workspaces/Phantomcraft/README_ItemRecipes.md