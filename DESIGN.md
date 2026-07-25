---
name: CampusGigs
description: A precise student-work marketplace where availability and demand data drive every match.
colors:
  primary: "#5980a6"
  primary-hover: "#597ea3"
  primary-active: "#416180"
  primary-deep: "#1d2d3d"
  primary-soft: "#eef6ff"
  primary-ink: "#2c455d"
  secondary: "#728fab"
  background: "#f2f2f3"
  surface: "#e9e9ea"
  text: "#1d1f20"
  muted-text: "#5d5d60"
  divider: "#1d1f2029"
typography:
  display:
    fontFamily: "Barlow Condensed, system-ui, sans-serif"
    fontSize: "42px"
    fontWeight: 600
    lineHeight: 1.12
    letterSpacing: "-0.015em"
  headline:
    fontFamily: "Barlow Condensed, system-ui, sans-serif"
    fontSize: "32px"
    fontWeight: 600
    lineHeight: 1.12
    letterSpacing: "-0.015em"
  title:
    fontFamily: "Barlow Condensed, system-ui, sans-serif"
    fontSize: "20px"
    fontWeight: 600
    lineHeight: 1.2
    letterSpacing: "-0.015em"
  body:
    fontFamily: "Barlow, system-ui, sans-serif"
    fontSize: "15px"
    fontWeight: 400
    lineHeight: 1.55
    letterSpacing: "normal"
  control:
    fontFamily: "Barlow Condensed, system-ui, sans-serif"
    fontSize: "14px"
    fontWeight: 600
    lineHeight: 1.2
    letterSpacing: "normal"
  field:
    fontFamily: "Barlow, system-ui, sans-serif"
    fontSize: "14px"
    fontWeight: 400
    lineHeight: 1.55
    letterSpacing: "normal"
  label:
    fontFamily: "Barlow, system-ui, sans-serif"
    fontSize: "12px"
    fontWeight: 500
    lineHeight: 1.4
    letterSpacing: "0.02em"
  micro:
    fontFamily: "Barlow, system-ui, sans-serif"
    fontSize: "11px"
    fontWeight: 400
    lineHeight: 1.4
    letterSpacing: "0.02em"
rounded:
  square: "0px"
  sm: "2px"
  md: "4px"
  lg: "7px"
spacing:
  space-1: "3.4px"
  space-2: "6.8px"
  space-3: "10.2px"
  space-4: "13.6px"
  space-6: "20.4px"
  space-8: "27.2px"
components:
  button-primary:
    backgroundColor: "{colors.primary-active}"
    textColor: "{colors.background}"
    typography: "{typography.control}"
    rounded: "{rounded.square}"
    padding: "6.8px 12.24px"
  button-primary-hover:
    backgroundColor: "{colors.primary-ink}"
    textColor: "{colors.background}"
    rounded: "{rounded.square}"
  button-primary-active:
    backgroundColor: "{colors.primary-deep}"
    textColor: "{colors.background}"
    rounded: "{rounded.square}"
  button-secondary:
    backgroundColor: "transparent"
    textColor: "{colors.text}"
    typography: "{typography.control}"
    rounded: "{rounded.square}"
    padding: "6.8px 12.24px"
  button-ghost:
    backgroundColor: "transparent"
    textColor: "{colors.primary}"
    typography: "{typography.control}"
    rounded: "{rounded.square}"
    padding: "6.8px 3.4px"
  input:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.text}"
    typography: "{typography.field}"
    rounded: "{rounded.square}"
    padding: "6px 10px"
    height: "36px"
  tag-accent:
    backgroundColor: "{colors.primary-soft}"
    textColor: "{colors.primary-ink}"
    typography: "{typography.micro}"
    rounded: "{rounded.square}"
    padding: "3px 10px"
  blueprint-card:
    backgroundColor: "transparent"
    textColor: "{colors.text}"
    rounded: "{rounded.square}"
    padding: "10.2px"
---

# Design System: CampusGigs

## 1. Overview

**Creative North Star: "The Working Blueprint"**

CampusGigs should feel like a working drawing spread across a drafting table: precise enough to trust, unfinished only where the user is meant to act, and visibly organized around real constraints. Barlow Condensed gives headings and actions an engineered economy; Barlow keeps dense task content readable. Registration marks, square controls, thin rules, and restrained blue-gray signals make the interface feel measured rather than ornamental.

The workspace is a product surface, not a campaign. Information density is welcome when it helps a student compare demand, capacity, price, and compatible work windows. Familiar controls must remain familiar. The system explicitly rejects a generic Fiverr or Upwork clone, a childish student portal, futuristic AI-dashboard theatrics, and any decorative “smart” effect that competes with the scheduling and market evidence.

New and revised workspace surfaces must reflow into one complete column at 380px without horizontal page scrolling. No ambient animation belongs in the foundation; future motion must communicate state, respect reduced-motion preferences, and never delay the task.

**Key Characteristics:**

- Flat, structural surfaces separated by hairline rules.
- Square, compact controls with unmistakable states.
- Restrained drafting blue reserved for actions, selection, and evidence.
- Condensed headings paired with a legible workhorse body face.
- Responsive density that reveals structure without hiding information.

**The Evidence-First Rule.** Decoration never outranks availability, demand, pay, duration, or compatibility.

## 2. Colors

The palette is an industrial blue-gray register: cool enough to feel precise, muted enough to keep data and action states in control.

### Primary

- **Drafting Blue** (`#5980a6`): Selected controls, links, borders, and active evidence. Its restraint is the point; it is never a decorative wash or a small-text button fill.
- **Drafting Blue—Mid** (`#597ea3`): Supporting indicators and large-format state fills where the text contrast requirement is not implicated.
- **Drafting Blue—Active** (`#416180`): The accessible baseline for filled primary actions, selected emphasis, and high-confidence data bars.
- **Blueprint Ink** (`#2c455d`): Hovered primary actions and text placed on Blueprint Wash.
- **Drafting Deep** (`#1d2d3d`): Pressed primary actions and the darkest blue evidence state.
- **Blueprint Wash** (`#eef6ff`): Low-intensity selected tags, availability blocks, and informational backgrounds.

### Secondary

- **Plate Blue** (`#728fab`): A secondary tonal role for supporting states. It must not compete with Drafting Blue for primary-action ownership.

### Neutral

- **Workbench Ground** (`#f2f2f3`): The page background and modal ground.
- **Instrument Surface** (`#e9e9ea`): Inputs and the restrained second surface layer.
- **Graphite Ink** (`#1d1f20`): Primary text and high-emphasis labels.
- **Pencil Note** (`#5d5d60`): Secondary metadata and explanatory text that still meets the contrast requirement at its deployed size.
- **Construction Line** (`#1d1f2029`): Sixteen-percent graphite for borders, dividers, and registration geometry.

**The One Drafting Blue Rule.** Drafting Blue owns action, selection, and evidence; Plate Blue may support it but never creates a second primary-action language.

**The Contrast Is Structural Rule.** Body copy, placeholders, metadata, and button labels must meet WCAG 2.2 AA. Workbench Ground text on base Drafting Blue is only 3.71:1 and is prohibited for small controls; filled primary actions start at Drafting Blue—Active.

## 3. Typography

**Display Font:** Barlow Condensed (with system-ui and sans-serif fallbacks)  
**Body Font:** Barlow (with system-ui and sans-serif fallbacks)

**Character:** The pairing is compact, direct, and practical. Condensed type gives headings and actions a technical cadence; the regular-width body face carries instructions, market evidence, and form content without turning the workspace into a poster.

### Hierarchy

- **Display** (600, 42px, 1.12): Rare top-level workspace titles. Screen-specific overrides may step down to 30–34px when density demands it.
- **Headline** (600, 32px, 1.12): Major page and panel headings.
- **Title** (600, 20px, 1.2): Card titles, dialog titles, and strong data labels.
- **Body** (400, 15px, 1.55): Core instructions and content. Long prose is capped at 70ch.
- **Control** (600, 14px, 1.2): Buttons and compact actions in Barlow Condensed.
- **Field** (400, 14px, 1.55): Input values, search, and editable profile content.
- **Label** (500, 12px, 1.4, 0.02em): Field labels, metadata, table headers, and compact navigation states.
- **Micro** (400, 11px, 1.4, 0.02em): Tags, card metadata, and supplemental captions.

**The Condensed-for-Control Rule.** Barlow Condensed belongs to headings, amounts, and actions; form values, descriptions, and dense metadata remain in Barlow.

**The No-Poster Rule.** Workspace headings remain fixed-size and task-scaled. Fluid hero typography and oversized marketing type are prohibited inside the product.

## 4. Elevation

The Working Blueprint is flat by default. Borders, tonal surfaces, overlap, and registration geometry communicate structure. Shadows are functional escape hatches for elements that must sit above the working plane—drawers, dialogs, and other temporary overlays—not decoration for resting cards.

### Shadow Vocabulary

- **Low Lift** (`0 1px 2px color-mix(in srgb, #2b2b2d 14%, transparent)`): Small transient separation only.
- **Panel Lift** (`0 3px 10px color-mix(in srgb, #2b2b2d 16%, transparent)`): Floating controls that genuinely overlap content.
- **Overlay Lift** (`0 12px 32px color-mix(in srgb, #2b2b2d 22%, transparent)`): Drawers and dialogs above a dimmed backdrop.

**The Flat Working Plane Rule.** Cards and page sections stay shadowless. If a resting surface needs a shadow to be understood, its border, spacing, or hierarchy is wrong.

## 5. Components

### Buttons

- **Shape:** Square and tool-like (0px radius), with a one-pixel construction-line border.
- **Primary:** Drafting Blue—Active background with Workbench Ground text; compact 6.8px × 12.24px padding.
- **Hover / Focus:** Hover moves to Blueprint Ink; active moves to Drafting Deep. Keyboard focus uses a two-pixel Drafting Blue outline with a two-pixel offset.
- **Secondary / Ghost:** Secondary buttons stay transparent with a full construction-line border. Ghost actions use Drafting Blue text and only 3.4px inline padding. Disabled buttons retain their geometry at 45% opacity.

### Chips

- **Style:** Compact rectangular tags use Blueprint Wash with Blueprint Ink. Outline tags use Drafting Blue as a one-pixel stroke.
- **State:** Selected filters use Drafting Blue with Workbench Ground text; unselected filters stay transparent. Selection must remain legible without color through weight, stroke, or an explicit state label.

### Cards / Containers

- **Corner Style:** Square (0px radius).
- **Background:** Transparent on Workbench Ground; Instrument Surface is reserved for form controls and deliberate secondary layers.
- **Shadow Strategy:** None at rest.
- **Border:** One-pixel Construction Line.
- **Internal Padding:** 10.2px by default, increasing to 13.6px only for larger panels.
- **Signature:** Blueprint containers may place four small registration marks outside their corners. They are structural punctuation, not decoration to repeat on every nested element.

### Inputs / Fields

- **Style:** Instrument Surface fill, one-pixel Construction Line border, square corners, 6px × 10px padding, and a 36px minimum height.
- **Focus:** Border moves to Drafting Blue; the global two-pixel focus ring remains visible when it is not clipped.
- **Error / Disabled:** Error and disabled tokens are not yet committed. New states must use text or icon cues in addition to color and preserve the input’s dimensions.

### Navigation

The navigation is a flat sticky rail on Workbench Ground. The brand uses Barlow Condensed at 18px; navigation links use the body face at 14px. The active state moves to Drafting Blue and may add a two-pixel underline. At narrow widths, search, mode selection, and cart controls must wrap or collapse deliberately rather than widening the page.

### Availability Timeline

The day track is a 26px-high bordered work lane with time divisions. Availability blocks use Blueprint Wash, a Drafting Blue—Active border, and Blueprint Ink text. Dragging, resizing, deletion, and keyboard alternatives must be visible instructions; the chart cannot rely on pointer precision alone.

**The One Control Vocabulary Rule.** Buttons, inputs, segmented controls, tabs, and dialogs share the same square geometry, focus language, and restrained state colors across both client and freelancer modes.

## 6. Do's and Don'ts

### Do:

- **Do** use Drafting Blue—Active (`#416180`) for filled primary actions and Drafting Blue (`#5980a6`) for selected controls, borders, and evidence—not ambient decoration.
- **Do** keep cards flat with a one-pixel Construction Line (`#1d1f2029`) and 10.2px default padding.
- **Do** use Barlow Condensed for headings and actions, while keeping descriptions, fields, and metadata in Barlow.
- **Do** reflow the workspace into one complete column at 380px with no horizontal page scroll.
- **Do** provide keyboard, text, and icon equivalents for interactions that currently depend on dragging, pointer precision, or color.

### Don't:

- **Don't** turn CampusGigs into a generic Fiverr or Upwork clone.
- **Don't** use childish “student portal” visuals.
- **Don't** add futuristic AI-dashboard theatrics, marketplace theatre, or decorative “smart” effects.
- **Don't** use full-card shadows, soft ghost cards, glassmorphism, gradient text, or oversized rounded containers.
- **Don't** promote the striped image placeholder into a brand motif; replace it with meaningful assets or a quiet empty state.
- **Don't** add onboarding or marketing-page patterns to the workspace foundation; that later surface uses its own brand/marketing register.
