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

Use the command buffer pattern to add interactions to entities:

```java
Interactions interactions = commandBuffer.getComponent(
    entityRef,
    Interactions.getComponentType()
);

if (interactions == null) {
    interactions = new Interactions();
    commandBuffer.putComponent(entityRef,
        Interactions.getComponentType(), interactions);
}

interactions.setInteractionId(InteractionType.Use, "my_custom_interaction");
interactions.setInteractionHint("Press E to interact");
```

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

## Related Files

- [02-ecs-system.md](./02-ecs-system.md) - Component system
- [05-events.md](./05-events.md) - Event registration
