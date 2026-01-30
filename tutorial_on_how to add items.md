# Tutorial: Adding a Test Item with White Texture

This tutorial documents the process of adding a new custom item to the Phantomcraft game (a Minecraft-like application). The example item created is called `test_item` and has a simple white texture.

## Overview

Adding a new item to the game involves:
1. Declaring the item in the Items registry
2. Registering it with a unique ID and properties
3. Adding texture rendering support
4. Creating the texture asset

## Step 1: Declare the Item in Items.java

**File:** `src/game/java/net/minecraft/init/Items.java`

Add a public static field to declare the new item:

```java
public static Item test_item;
```

This field will hold a reference to your registered item and can be accessed throughout the codebase as `Items.test_item`.

**Location in file:** Add this declaration at the end of the item declarations, before the `getRegisteredItem()` method (around line 225).

## Step 2: Initialize Item in Bootstrap

**File:** `src/game/java/net/minecraft/init/Items.java`

In the `doBootstrap()` static method, add initialization code:

```java
test_item = getRegisteredItem("test_item");
```

This retrieves the registered item from the item registry using its string identifier. The bootstrap method is called when the game starts to populate all item references.

**Location in file:** Add this at the end of the `doBootstrap()` method, just before the closing brace (around line 420).

## Step 3: Register the Item in Item.java

**File:** `src/game/java/net/minecraft/item/Item.java`

In the static `registerItems()` method, register your item with a unique ID:

```java
registerItem(2268, (String) "test_item", (new Item()).setUnlocalizedName("testItem")
        .setCreativeTab(CreativeTabs.tabMisc));
```

**Key components:**
- **2268**: Unique numeric ID for the item (must not conflict with existing items)
- **"test_item"**: String identifier used in registries and texture lookups
- **new Item()**: Creates a new basic Item instance
- **setUnlocalizedName("testItem")**: Sets the translation key for the item's display name
- **setCreativeTab(CreativeTabs.tabMisc)**: Places the item in the Miscellaneous creative tab

**Location in file:** Add this before the closing brace of the `registerItems()` method (around line 1044), after the last record item registration.

## Step 4: Register Texture Rendering in RenderItem.java

**File:** `src/game/java/net/minecraft/client/renderer/entity/RenderItem.java`

In the `registerItems()` method, add texture registration:

```java
this.registerItem(Items.test_item, "test_item");
```

This tells the renderer to use the texture file named `test_item.png` for rendering the item in the game.

**Location in file:** Add this in the `registerItems()` method after other item registrations (around line 1072), before the `enchanted_book` registration.

## Step 5: Create the Texture Asset

**File location:** `desktopRuntime/resources/assets/minecraft/textures/items/test_item.png`

Create a 16x16 pixel PNG image. For this example, a solid white texture was created.

### Creating the Texture

You can use any image editor, or create it programmatically:

```python
from PIL import Image

# Create a 16x16 white image
img = Image.new('RGBA', (16, 16), (255, 255, 255, 255))

# Save the image
img.save('test_item.png')
```

**Texture Requirements:**
- Format: PNG with transparency support (RGBA)
- Size: 16x16 pixels (standard Minecraft item texture size)
- Location: Must be in `desktopRuntime/resources/assets/minecraft/textures/items/` directory
- Filename: Should match the string identifier (e.g., `test_item.png` for `"test_item"`)

## How Item Registration Works

The item registration system in this codebase follows a pattern common in Minecraft:

1. **Item Declaration** → A static reference field is created in the Items class
2. **Item Registration** → The Item is registered with a unique ID in the Item class
3. **Bootstrap** → On startup, the Items class retrieves registered items and populates the static fields
4. **Texture Registration** → The renderer is told which texture file to use for display
5. **Asset File** → The actual texture PNG file is stored in the assets directory

### Registry Lookup

The workflow:
- `registerItem(id, name, itemInstance)` → Stores the item in the global registry
- `getRegisteredItem(name)` → Retrieves the item from the registry by string name
- `Items.test_item` → Access the item via the static field

## Testing Your Item

Once all changes are made and the game is compiled:

1. Launch the game
2. Open creative mode inventory
3. Navigate to the Miscellaneous tab
4. Find "Test Item" (or the localized name if translations are available)
5. The item should display with your white texture

## File Locations Summary

| File | Changes | Line Reference |
|------|---------|-----------------|
| `src/game/java/net/minecraft/init/Items.java` | Added field declaration + bootstrap initialization | ~225, ~420 |
| `src/game/java/net/minecraft/item/Item.java` | Added item registration | ~1044 |
| `src/game/java/net/minecraft/client/renderer/entity/RenderItem.java` | Added texture registration | ~1072 |
| `desktopRuntime/resources/assets/minecraft/textures/items/test_item.png` | Created new texture asset | N/A |

## Next Steps

To customize the item further, you can:
- **Change the texture**: Create a different PNG image
- **Add special properties**: Use different Item subclasses (e.g., `ItemFood`, `ItemTool`)
- **Set max stack size**: Use `.setMaxStackSize(n)` in registration
- **Make it damageable**: Use `.setMaxDamage(durability)` for tools/weapons
- **Add creative tab**: Use `.setCreativeTab(CreativeTabs.tabXXX)` to place in different tabs
- **Localize the name**: Add translations in language files

For more complex items, extend the `Item` class to create custom behavior!