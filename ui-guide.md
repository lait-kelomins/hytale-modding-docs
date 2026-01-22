# Hytale UI System

> **⚠️ Work In Progress**
> This documentation is based on community research and may be incomplete or contain inaccuracies. The Hytale modding API is still evolving.

---

## Quick Setup

### 1. Project Structure

Place UI files in this exact folder structure:
```
src/main/resources/
├── manifest.json
└── Common/
    └── UI/
        └── Custom/
            └── Pages/
                └── YourPage.ui
```

### 2. Manifest Configuration

Add to your `manifest.json`:
```json
{
  "IncludesAssetPack": true
}
```

### 3. Gradle Configuration

Ensure resources are bundled in your JAR:
```groovy
tasks.jar {
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE
    from("src/main/resources")
}
```

### 4. Import Common.ui

At the top of your `.ui` file, import the built-in Common.ui:
```
$C = "../Common.ui";
```

The path is relative to your file's location:
- From `Pages/MyPage.ui` → `$C = "../Common.ui";`
- From `Pages/Sub/MyPage.ui` → `$C = "../../Common.ui";`

---

## Understanding the UI System

### Two Worlds: UI Files and Java Code

Hytale UIs are **declarative** - you describe what should appear on screen, not how to draw it.

| UI Files (`.ui`) | Java Code |
|------------------|-----------|
| Layout & positioning | Logic & actions |
| Colors & styling | Data handling |
| Text content | Event responses |

### Communication Between Worlds

**Java → UI:**
- Load UI files
- Update text/values
- Show/hide elements

**UI → Java:**
- Button clicks
- Form submissions
- Input changes

Elements are connected via **IDs** (e.g., `#myButton`).

---

## Guide: Creating a Basic Dialog

### Step 1: Create the UI File

`src/main/resources/Common/UI/Custom/Pages/MyDialog.ui`:
```
$C = "../Common.ui";

Group {
  LayoutMode: Center;

  Group #panel {
    Anchor: (Width: 300, Height: 150);
    Background: #1a1a1a(0.95);
    LayoutMode: Top;
    Padding: (Full: 20);

    Label #title {
      Text: "Hello World";
      Style: (
        FontSize: 20,
        TextColor: #ffffff,
        Alignment: Center,
        RenderBold: true
      );
    }

    TextButton #closeButton {
      Text: "Close";
      Anchor: (Top: 20, Width: 100, Height: 36);
      Padding: (Left: 12, Right: 12, Top: 8, Bottom: 8);
    }
  }
}
```

### Step 2: Create the Java Page Class

For a simple display without input handling, use `BasicCustomUIPage`:

```java
import com.hypixel.hytale.protocol.packets.interface_.CustomPageLifetime;
import com.hypixel.hytale.server.core.entity.entities.player.pages.BasicCustomUIPage;
import com.hypixel.hytale.server.core.ui.builder.UICommandBuilder;
import com.hypixel.hytale.server.core.universe.PlayerRef;

public class MyDialogPage extends BasicCustomUIPage {

    public MyDialogPage(PlayerRef playerRef) {
        super(playerRef, CustomPageLifetime.CanDismiss);
    }

    @Override
    public void build(UICommandBuilder cmd) {
        cmd.append("Pages/MyDialog.ui");
    }
}
```

### Step 3: Open the Page

```java
Player player = // get player instance
PlayerRef playerRef = player.getPlayerRef();
Ref<EntityStore> ref = player.getReference();
Store<EntityStore> store = ref.getStore();

MyDialogPage page = new MyDialogPage(playerRef);
player.getPageManager().openCustomPage(ref, store, page);
```

---

## Guide: Interactive Forms with Input

For buttons and text inputs, use `InteractiveCustomUIPage`.

### Step 1: Create Event Data Class

```java
public static class FormEventData {
    public String inputValue;

    public static final BuilderCodec<FormEventData> CODEC = BuilderCodec
        .builder(FormEventData.class, FormEventData::new)
        .append(new KeyedCodec<>("@inputValue", new StringCodec()),
            (obj, val) -> obj.inputValue = val,
            obj -> obj.inputValue)
        .add()  // REQUIRED after each append
        .build();
}
```

**Key points:**
- The `@` prefix binds fields to UI element values
- Must call `.add()` after each `.append()`
- Use `StringCodec` for text fields

### Step 2: Create the Interactive Page

```java
import com.hypixel.hytale.codec.KeyedCodec;
import com.hypixel.hytale.codec.builder.BuilderCodec;
import com.hypixel.hytale.codec.codecs.simple.StringCodec;
import com.hypixel.hytale.component.Ref;
import com.hypixel.hytale.component.Store;
import com.hypixel.hytale.protocol.packets.interface_.CustomPageLifetime;
import com.hypixel.hytale.protocol.packets.interface_.CustomUIEventBindingType;
import com.hypixel.hytale.protocol.packets.interface_.Page;
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.entity.entities.Player;
import com.hypixel.hytale.server.core.entity.entities.player.pages.InteractiveCustomUIPage;
import com.hypixel.hytale.server.core.ui.builder.EventData;
import com.hypixel.hytale.server.core.ui.builder.UICommandBuilder;
import com.hypixel.hytale.server.core.ui.builder.UIEventBuilder;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import com.hypixel.hytale.server.core.universe.world.storage.EntityStore;

public class MyFormPage extends InteractiveCustomUIPage<MyFormPage.FormEventData> {

    public static class FormEventData {
        public String inputValue;

        public static final BuilderCodec<FormEventData> CODEC = BuilderCodec
            .builder(FormEventData.class, FormEventData::new)
            .append(new KeyedCodec<>("@inputValue", new StringCodec()),
                (obj, val) -> obj.inputValue = val,
                obj -> obj.inputValue)
            .add()
            .build();
    }

    public MyFormPage(PlayerRef playerRef) {
        super(playerRef, CustomPageLifetime.CanDismiss, FormEventData.CODEC);
    }

    @Override
    public void build(Ref<EntityStore> ref, UICommandBuilder cmd, UIEventBuilder events, Store<EntityStore> store) {
        cmd.append("Pages/MyForm.ui");

        // Set dynamic text (property names are case-sensitive!)
        cmd.set("#title.Text", "Enter Your Name");

        // Bind submit button - captures input value
        events.addEventBinding(CustomUIEventBindingType.Activating, "#submitButton",
            new EventData().append("@inputValue", "#nameInput.Value"));

        // Bind cancel button - empty data
        events.addEventBinding(CustomUIEventBindingType.Activating, "#cancelButton",
            new EventData());
    }

    @Override
    public void handleDataEvent(Ref<EntityStore> ref, Store<EntityStore> store, FormEventData data) {
        Player player = store.getComponent(ref, Player.getComponentType());

        if (data.inputValue != null && !data.inputValue.trim().isEmpty()) {
            // Submit was clicked with input
            player.sendMessage(Message.raw("Hello, " + data.inputValue).color("#55FF55"));
        }

        // Close the page
        player.getPageManager().setPage(ref, store, Page.None);
    }
}
```

**Important differences from BasicCustomUIPage:**
- `build()` has 4 parameters: `(ref, cmd, events, store)`
- `handleDataEvent()` must be `public`, not `protected`
- Close pages with `Page.None`

---

## Quick Reference

### UI File Syntax

```
ElementType #elementId {
  Property: Value;
  Property: (SubProp1: Value1, SubProp2: Value2);

  ChildElement {
    Property: Value;
  }
}
```

### Colors

```
#ffffff           // White
#1a1a1a           // Dark gray (panels)
#2a2a2a           // Input backgrounds
#93844c           // Gold (titles)
#888888           // Gray (subtitles)
#55ff55           // Green
#ff5555           // Red

#ffffff(0.9)      // With 90% opacity
```

### Layout Modes

| Mode | Description |
|------|-------------|
| `Top` | Stack children vertically from top |
| `Left` | Stack children horizontally from left |
| `Center` | Center children |
| `TopScrolling` | Vertical scrollable list |

### Centering Buttons

Use empty Labels with `FlexWeight` as spacers:
```
Group {
  LayoutMode: Left;

  Label { FlexWeight: 1; }
  TextButton #btn { Text: "Click"; Anchor: (Width: 100, Height: 36); }
  Label { FlexWeight: 1; }
}
```

### Event Binding Types

| Type | Use Case |
|------|----------|
| `Activating` | Button clicks |
| `ValueChanged` | Input changes |
| `MouseEntered` | Hover start |
| `MouseExited` | Hover end |

---

## UI Element Reference

### Group

Container element (like HTML `<div>`).

```
Group #myContainer {
  Anchor: (Width: 400, Height: 300);
  LayoutMode: Top;
  Background: #1a1a1a;
  Padding: (Full: 10);
  FlexWeight: 1;
}
```

| Property | Values | Description |
|----------|--------|-------------|
| `Anchor` | Width, Height, Top, Bottom, Left, Right | Position and size |
| `LayoutMode` | Top, Left, Center, TopScrolling | Child arrangement |
| `Background` | Hex color or texture path | Background fill |
| `Padding` | Full, Horizontal, Vertical, Top/Bottom/Left/Right | Inner spacing |
| `FlexWeight` | Number | Flexible sizing ratio |

### Label

Text display element.

```
Label #title {
  Text: "Hello World";
  Anchor: (Top: 20);
  Style: (
    FontSize: 24,
    TextColor: #ffffff,
    Alignment: Center,
    RenderBold: true
  );
}
```

| Property | Values | Description |
|----------|--------|-------------|
| `Text` | String | Display text |
| `Anchor` | Width, Height, Top, etc. | Position and size |
| `Style` | See below | Text styling |
| `FlexWeight` | Number | Flexible sizing |

**Style properties:**
- `FontSize` - Number (pixels)
- `TextColor` - Hex color
- `Alignment` - Center, Left, Right
- `RenderBold` - true/false
- `RenderUppercase` - true/false

### TextField

Text input field.

```
TextField #nameInput {
  Anchor: (Width: 200, Height: 40);
  Style: (
    FontSize: 14,
    TextColor: #ffffff
  );
}
```

| Property | Values | Description |
|----------|--------|-------------|
| `Anchor` | Width, Height, Top, etc. | Position and size |
| `Style` | FontSize, TextColor | Text styling |
| `Background` | Hex color | Background fill |

**Reading value in Java:** `#nameInput.Value`

### TextButton

Clickable button with text.

```
TextButton #confirmButton {
  Text: "Confirm";
  Anchor: (Width: 100, Height: 36);
  Padding: (Left: 12, Right: 12, Top: 8, Bottom: 8);
}
```

| Property | Values | Description |
|----------|--------|-------------|
| `Text` | String | Button label |
| `Anchor` | Width, Height, Top, etc. | Position and size |
| `Padding` | Full, Top/Bottom/Left/Right | Inner spacing |
| `Style` | TextButtonStyle(...) | Button states |

**Advanced styling:**
```
Style: TextButtonStyle(
  Default: (Background: #3a3a3a, LabelStyle: (FontSize: 14, TextColor: #ffffff)),
  Hovered: (Background: #4a4a4a, LabelStyle: (FontSize: 14, TextColor: #ffffff)),
  Pressed: (Background: #2a2a2a, LabelStyle: (FontSize: 14, TextColor: #888888)),
  Disabled: (Background: #1a1a1a, LabelStyle: (FontSize: 14, TextColor: #555555))
);
```

### Anchor Properties

```
Anchor: (
  Width: 400,       // Fixed width in pixels
  Height: 300,      // Fixed height in pixels
  Top: 20,          // Offset from top
  Bottom: 10,       // Offset from bottom
  Left: 15,         // Offset from left
  Right: 15,        // Offset from right
  Horizontal: 0,    // Horizontal centering
  Vertical: 0       // Vertical centering
);
```

### Padding Properties

```
Padding: (Full: 10);                              // All sides
Padding: (Horizontal: 20, Vertical: 10);          // H and V
Padding: (Top: 5, Bottom: 5, Left: 10, Right: 10); // Individual
```

---

## Java Class Reference

### Page Types

| Class | Use Case |
|-------|----------|
| `BasicCustomUIPage` | Static display, no user input |
| `InteractiveCustomUIPage<T>` | Buttons, text inputs, forms |

### CustomPageLifetime

| Value | Description |
|-------|-------------|
| `CanDismiss` | Player can close with ESC |
| `CantClose` | Cannot be closed by player |
| `CanDismissOrCloseThroughInteraction` | ESC or interaction closes |

### Key Imports

```java
// Codec
import com.hypixel.hytale.codec.KeyedCodec;
import com.hypixel.hytale.codec.builder.BuilderCodec;
import com.hypixel.hytale.codec.codecs.simple.StringCodec;

// UI
import com.hypixel.hytale.protocol.packets.interface_.CustomPageLifetime;
import com.hypixel.hytale.protocol.packets.interface_.CustomUIEventBindingType;
import com.hypixel.hytale.protocol.packets.interface_.Page;
import com.hypixel.hytale.server.core.entity.entities.player.pages.InteractiveCustomUIPage;
import com.hypixel.hytale.server.core.ui.builder.EventData;
import com.hypixel.hytale.server.core.ui.builder.UICommandBuilder;
import com.hypixel.hytale.server.core.ui.builder.UIEventBuilder;
import com.hypixel.hytale.server.core.universe.PlayerRef;
```

---

## Common Issues

| Issue | Solution |
|-------|----------|
| UI not showing | Check `IncludesAssetPack: true` in manifest |
| File not found | Verify path in `cmd.append()` is relative to `Common/UI/Custom/` |
| Property not updating | Property names are **case-sensitive** (use `Text`, not `text`) |
| Events not firing | Use `events.addEventBinding()`, not `cmd` |
| Codec errors | Call `.add()` after each `.append()` in codec |
| handleDataEvent not called | Must be `public`, not `protected` |

---

## Resources

- [HytaleModding.dev UI Guide](https://hytalemodding.dev/en/docs/guides/plugin/ui)
- [AdminUI Plugin](https://github.com/Buuz135/AdminUI) - Complex UI examples
- [Hytale-Sandbox-Plugin](https://github.com/underscore95/Hytale-Sandbox-Plugin) - Basic examples
