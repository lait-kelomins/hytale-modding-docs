# UI Styling Guide

How to style your custom UIs to match Hytale's default visual design.

---

## The Golden Rule

{% hint style="danger" %}
**Direct texture paths do NOT work from plugins.**

```
// This shows RED X (missing texture)
Background: PatchStyle(TexturePath: "Common/ContainerPatch.png", Border: 20);
```
{% endhint %}

{% hint style="success" %}
**Use Common.ui variables instead** - they resolve to built-in textures at runtime.

```
$C = "../Common.ui";

// This works!
Background: $C.@InputBoxBackground;
Style: $C.@DefaultTextButtonStyle;
```
{% endhint %}

---

## Available Variables

Import Common.ui at the top of your `.ui` file:

```
$C = "../Common.ui";
```

### Button Styles

Use these with `TextButton` elements:

| Variable | Description |
|----------|-------------|
| `$C.@DefaultTextButtonStyle` | Primary blue button (Confirm, Submit) |
| `$C.@SecondaryTextButtonStyle` | Gray button (Cancel, Back) |
| `$C.@TertiaryTextButtonStyle` | Alternative style |
| `$C.@CancelTextButtonStyle` | Red destructive button (Delete) |
| `$C.@SmallDefaultTextButtonStyle` | Smaller primary button |
| `$C.@SmallSecondaryTextButtonStyle` | Smaller secondary button |

**Example:**
```
TextButton #confirmButton {
  Text: "CONFIRM";
  Anchor: (Width: 120, Height: 40);
  Padding: (Horizontal: 20);
  Style: $C.@DefaultTextButtonStyle;
}
```

### Input Backgrounds

Use these with `Group` containers wrapping `TextField`:

| Variable | Description |
|----------|-------------|
| `$C.@InputBoxBackground` | Default input field |
| `$C.@InputBoxHoveredBackground` | Hovered state |
| `$C.@InputBoxPressedBackground` | Pressed state |
| `$C.@InputBoxSelectedBackground` | Focused state |

**Example:**
```
Group {
  Anchor: (Width: 290, Height: 40);
  Background: $C.@InputBoxBackground;
  Padding: (Left: 12, Right: 12, Top: 8, Bottom: 8);

  TextField #nameInput {
    Anchor: (Width: 266, Height: 24);
    Style: (FontSize: 14, TextColor: #e5e7eb);
  }
}
```

---

## Color Palette

For panel backgrounds, use hex colors (no exported variables available):

### Panels

| Element | Color | Usage |
|---------|-------|-------|
| Title Bar | `#1e2a3a` | Header background |
| Stroke | `#3a4a5a` | Separator line |
| Content | `#232d3f` | Main body |
| Dark | `#1a2230` | Darker areas |

### Text

| Element | Color | Usage |
|---------|-------|-------|
| Title | `#bfcdd5` | Headers |
| Body | `#e5e7eb` | Main text |
| Muted | `#6b7a8c` | Descriptions |
| Subtle | `#6b7280` | Hints |

### Accents

| Element | Color | Usage |
|---------|-------|-------|
| Gold | `#c9953c` | Highlights |
| Blue | `#3a5070` | Borders |

---

## Dialog Structure

Hytale dialogs use a **Title Bar + Stroke + Content** pattern:

```
Group #dialogPanel {
  Anchor: (Width: 360, Height: 220);

  // 1. Title bar (darker background)
  Group #Title {
    Anchor: (Height: 38, Top: 0);
    Padding: (Top: 8);
    Background: #1e2a3a;

    Label #title {
      Text: "DIALOG TITLE";
      Style: (
        FontSize: 16,
        TextColor: #bfcdd5,
        Alignment: Center,
        RenderBold: true,
        RenderUppercase: true
      );
    }
  }

  // 2. Stroke line (separator)
  Group #Stroke {
    Anchor: (Height: 2, Top: 38);
    Background: #3a4a5a;
  }

  // 3. Content area
  Group #Content {
    LayoutMode: Top;
    Anchor: (Top: 40);
    Padding: (Left: 24, Right: 24, Top: 16, Bottom: 24);
    Background: #232d3f;

    // Your content here
  }
}
```

---

## Button Guidelines

| Property | Value |
|----------|-------|
| Minimum width | 120px |
| Height | 40px |
| Padding | `(Horizontal: 20)` |
| Gap between buttons | 12px |
| Text style | UPPERCASE |

### Centering Buttons

Use `FlexWeight` spacers:

```
Group {
  LayoutMode: Left;

  Label { FlexWeight: 1; }  // Left spacer

  TextButton #cancelButton {
    Text: "CANCEL";
    Anchor: (Width: 120, Height: 40, Right: 12);
    Padding: (Horizontal: 20);
    Style: $C.@SecondaryTextButtonStyle;
  }

  TextButton #confirmButton {
    Text: "CONFIRM";
    Anchor: (Width: 120, Height: 40);
    Padding: (Horizontal: 20);
    Style: $C.@DefaultTextButtonStyle;
  }

  Label { FlexWeight: 1; }  // Right spacer
}
```

---

## Typography

### Title Text
- Font size: **16px**
- Color: `#bfcdd5`
- Style: Bold, Uppercase, Center

### Subtitle
- Font size: **12px**
- Color: `#6b7a8c`
- Style: Regular, Center

### Input Text
- Font size: **14px**
- Color: `#e5e7eb`

---

## Spacing Reference

| Element | Size |
|---------|------|
| Title bar height | 38px |
| Stroke height | 2px |
| Content padding | 24px horizontal, 16-24px vertical |
| Gap between elements | 16px |
| Gap between input and buttons | 20px |
| Button gap | 12px |

---

## Complete Example

A fully styled dialog with input and buttons:

```
$C = "../Common.ui";

Group {
  LayoutMode: Center;

  Group #dialogPanel {
    Anchor: (Width: 360, Height: 220);

    // Title bar
    Group #Title {
      Anchor: (Height: 38, Top: 0);
      Padding: (Top: 8);
      Background: #1e2a3a;

      Label #titleLabel {
        Text: "NAME YOUR PET";
        Style: (
          FontSize: 16,
          TextColor: #bfcdd5,
          Alignment: Center,
          RenderBold: true,
          RenderUppercase: true
        );
      }
    }

    // Stroke
    Group #Stroke {
      Anchor: (Height: 2, Top: 38);
      Background: #3a4a5a;
    }

    // Content
    Group #Content {
      LayoutMode: Top;
      Anchor: (Top: 40);
      Padding: (Left: 24, Right: 24, Top: 16, Bottom: 24);
      Background: #232d3f;

      // Subtitle
      Label {
        Text: "Enter a name for your companion";
        Style: (FontSize: 12, TextColor: #6b7a8c, Alignment: Center);
      }

      // Input field
      Group {
        Anchor: (Top: 16, Width: 290, Height: 40);
        Background: $C.@InputBoxBackground;
        Padding: (Left: 12, Right: 12, Top: 8, Bottom: 8);

        TextField #input {
          Anchor: (Width: 266, Height: 24);
          Style: (FontSize: 14, TextColor: #e5e7eb);
        }
      }

      // Buttons
      Group {
        Anchor: (Top: 20);
        LayoutMode: Left;

        Label { FlexWeight: 1; }

        TextButton #cancelButton {
          Text: "CANCEL";
          Anchor: (Width: 120, Height: 40, Right: 12);
          Padding: (Horizontal: 20);
          Style: $C.@SecondaryTextButtonStyle;
        }

        TextButton #confirmButton {
          Text: "CONFIRM";
          Anchor: (Width: 120, Height: 40);
          Padding: (Horizontal: 20);
          Style: $C.@DefaultTextButtonStyle;
        }

        Label { FlexWeight: 1; }
      }
    }
  }
}
```

---

## Texture Files (Reference)

The actual textures are in the game at `Assets/Common/UI/Custom/Common/`. Files have `@2x` suffix for high-DPI.

These are **reference only** - you cannot use these paths from plugins.

| File | Description |
|------|-------------|
| `ContainerPatch@2x.png` | Panel background |
| `ContainerHeader@2x.png` | Header with runes |
| `InputBox@2x.png` | Input field |
| `Buttons/Primary@2x.png` | Blue button |
| `Buttons/Secondary@2x.png` | Gray button |
| `Buttons/Destructive@2x.png` | Red button |
