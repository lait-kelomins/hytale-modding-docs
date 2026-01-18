# Hytale Interaction System

> **Source:** https://hytale-docs.com/docs/api/server-internals/modules/interactions

---

## Overview

The interaction system manages mouse button events through the `InteractionModule`.

**Interaction Types:**
- **Primary**: Left mouse button actions
- **Secondary**: Right mouse button actions
- **Use**: Context-sensitive actions (E key)
- **Scroll**: Mouse scroll wheel actions

---

## Core Classes

**Package:** `com.hypixel.hytale.server.core.modules.interaction`

### Interaction (Base Class)

```java
// com.hypixel.hytale.server.core.modules.interaction.interaction.config.Interaction
public abstract class Interaction {
    String id;
    float runTime;
    List<Effect> effects;
    List<Rule> rules;

    protected abstract void tick0(
        boolean firstRun,
        float time,
        InteractionType type,
        InteractionContext context,
        CooldownHandler cooldownHandler
    );
}
```

### SimpleInteraction

Extends base class with chainable interactions:

```java
public class SimpleInteraction extends Interaction {
    String next;    // Interaction to run on success
    String failed;  // Interaction to run on failure
}
```

---

## Entity Interactions

### UseEntityInteraction

`com.hypixel.hytale.server.core.modules.interaction.interaction.config.client.UseEntityInteraction`

Retrieves the `Interactions` component from the target entity and executes the appropriate interaction.

### Interactions Component

```java
// com.hypixel.hytale.server.core.modules.interaction.Interactions
public class Interactions implements Component<EntityStore> {
    Map<InteractionType, String> interactions;  // Type to interaction ID
    String interactionHint;                     // UI hint text

    public String getInteractionId(InteractionType type);
    public void setInteractionId(InteractionType type, String interactionId);
}
```

---

## Setting Up Entity Interactions

> **WORKAROUND:** Modifying entity interactions at runtime via reflection works but is not ideal. The preferred approach would be to define interactions in NPC Role assets, but this requires asset overrides for existing entities.

```java
// Get the Interactions component type
Class<?> interactionsClass = Class.forName(
    "com.hypixel.hytale.server.core.modules.interaction.Interactions");
Object interactionsType = interactionsClass.getMethod("getComponentType").invoke(null);

// Ensure the component exists and get it
Object interactions = store.ensureAndGetComponent(entityRef, interactionsType);

// Get InteractionType.Use enum value
Class<?> interactionTypeClass = Class.forName("com.hypixel.hytale.protocol.InteractionType");
Object useType = Arrays.stream(interactionTypeClass.getEnumConstants())
    .filter(e -> e.toString().equals("Use")).findFirst().orElse(null);

// Set the interaction ID (must match a registered RootInteraction)
Method setIntId = interactions.getClass().getMethod("setInteractionId",
    interactionTypeClass, String.class);
setIntId.invoke(interactions, useType, "Root_MyCustomInteraction");

// Optionally set hint text (uses localization keys for keybind icons)
Method setHint = interactions.getClass().getMethod("setInteractionHint", String.class);
setHint.invoke(interactions, "server.interactionHints.generic");
```

> **NOTE:** The `commandBuffer` pattern shown in official docs is **untested**. The reflection approach above is confirmed working.

---

## Event Registration

### Block Interaction Events

```java
// Register in setup() method
getEventRegistry().register(UseBlockEvent.Pre.class, this::onUseBlockPre);
getEventRegistry().register(UseBlockEvent.Post.class, this::onUseBlockPost);
```

**Key difference:** Use `register()` not `registerGlobal()` for these events!

### Event Types

| Event | When Fired |
|-------|------------|
| `UseBlockEvent.Pre` | Before block interaction is processed |
| `UseBlockEvent.Post` | After block interaction is processed |

---

## InteractionTarget Enum

```java
public enum InteractionTarget {
    USER,    // Entity that triggered the interaction
    OWNER,   // Entity that owns the interaction chain
    TARGET   // Target entity of the interaction
}
```

---

## Console Commands

| Command | Description |
|---------|-------------|
| `/interaction run <id>` | Execute specific interaction |
| `/interaction clear` | Clear current interactions |

---

## Registering Custom Interactions

> **VERIFIED WORKING:** Custom interactions must be registered with the CodecMapRegistry during plugin setup.

```java
@Override
protected void setup() {
    // Register custom interaction class
    getCodecMapRegistry()
        .register("MyCustomInteraction", MyCustomInteraction.class, MyCustomInteraction.CODEC);
}
```

### Custom Interaction Class

```java
public class MyCustomInteraction extends SimpleInteraction {
    public static final BuilderCodec<MyCustomInteraction> CODEC =
        BuilderCodec.builder(MyCustomInteraction.class, MyCustomInteraction::new, SimpleInteraction.CODEC)
            .build();

    @Override
    protected void tick0(
        boolean firstRun,
        float time,
        InteractionType type,
        InteractionContext context,
        CooldownHandler cooldownHandler
    ) {
        if (firstRun) {
            // Your custom logic here
            // Access target entity: context.getTargetEntity()
            // Access held item: context.getHeldItem()
        }
        super.tick0(firstRun, time, type, context, cooldownHandler);
    }
}
```

---

## Asset File Paths

> **VERIFIED WORKING PATHS:**

| Asset Type | Path |
|------------|------|
| RootInteraction | `Server/Item/RootInteractions/{id}.json` |
| Interaction | `Server/Item/Interactions/{id}.json` |

> **WARNING:** Paths like `Server/Interaction/RootInteraction/` do NOT work. Use the paths above.

### RootInteraction Asset Example

```json
// Server/Item/RootInteractions/Root_MyCustom.json
{
  "Interactions": ["MyCustomInteraction"],
  "RequireNewClick": true
}
```

> **Note:** Use `"Interactions"` (not `"InteractionIds"`). The `"NeedsRemoteSync"` field is optional.

### Interaction Asset Example

```json
// Server/Item/Interactions/MyCustomInteraction.json
{
  "Type": "MyCustomInteraction",
  "Effects": {
    "ItemAnimationId": "Eat",
    "WaitForAnimationToFinish": true
  }
}
```

> **Note:** The `"Effects"` block is optional. Use `"ItemAnimationId"` for animations like `"Eat"`, `"Swing"`, etc.

---

## Related Files

- [02-ecs-system.md](api/02-ecs-system.md) - Component system
- [05-events.md](api/05-events.md) - Event registration
