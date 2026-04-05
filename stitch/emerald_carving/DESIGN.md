# The Design System: Editorial Mobility

This document outlines the visual language and structural principles for the GEMA platform. Our goal is to transcend the "utility app" aesthetic and establish a premium, editorial experience that honors the craftsmanship of Jepara through modern digital sophistication.

---

## 1. Overview & Creative North Star: "The Heritage Modernist"

The Creative North Star for this design system is **The Heritage Modernist**. Jepara is world-renowned for its intricate wood carving; we do not treat this heritage as a "logo," but as a structural philosophy. 

This system rejects the rigid, boxy constraints of standard bootstrap-style layouts. Instead, we utilize **Intentional Asymmetry** and **Tonal Depth**. By overlapping elements and using extreme shifts in typography scale, we move away from "templates" toward a bespoke, curated digital environment. The interface should feel like a high-end lifestyle magazine—spacious, authoritative, and deeply rooted in local pride.

---

## 2. Color Philosophy: Tonal Depth & The "No-Line" Rule

We utilize a sophisticated palette inspired by the raw beauty of emerald stone and the polished finish of Jepara teak.

### The "No-Line" Rule
**Explicit Instruction:** Designers are prohibited from using 1px solid borders for sectioning content. Boundaries must be defined solely through:
1.  **Background Color Shifts:** Placing a `surface-container-low` card against a `surface` background.
2.  **Tonal Transitions:** Using subtle shifts in green saturation to denote change.
3.  **Negative Space:** Using generous padding to define groupings.

### Surface Hierarchy & Nesting
Treat the UI as physical layers of "Frosted Emerald Glass" and "Polished Vellum." 
*   **Lowest Layer:** `surface` (The canvas).
*   **Secondary Layer:** `surface-container-low` (The structural groupings).
*   **Primary Interaction Layer:** `surface-container-lowest` (The "Lifted" interactive cards).

### The "Glass & Gradient" Rule
To ensure the app feels "Custom" rather than "Generic," main CTAs and Hero sections must utilize **Signature Gradients**. Transition from `primary` (#006D36) to `primary-container` (#50C878) at a 135-degree angle. Floating action elements should utilize **Glassmorphism** (semi-transparent surface colors with a 20px backdrop-blur) to create a sense of lightness.

### Signature Textures
The background of major landing screens must incorporate the **Jepara Wood Carving Pattern**. This pattern should be set at **2% opacity** in Light Mode and **4% opacity** in Dark Mode, ensuring it remains a "subliminal discovery" rather than a visual distraction.

---

## 3. Typography: The Editorial Scale

We pair **Manrope** (Display/Headlines) with **Inter** (Body/Labels) to balance character with readability.

*   **Display & Headlines (Manrope):** These are our "Style Anchors." Use `display-lg` (3.5rem) with tight letter-spacing for high-impact landing screens. The bold, geometric nature of Manrope mirrors the precision of wood carving.
*   **Body & Titles (Inter):** Inter provides the "Functional Engine." Its high x-height ensures maximum readability for mobility data (prices, distances, and times).
*   **Visual Hierarchy:** Use `primary` or `on-surface-variant` for headlines to create a high-contrast, professional rhythm. Avoid "Grey" text; use the `on-surface-variant` (#3E4A3F) which is a deep, desaturated forest green to maintain tonal harmony.

---

## 4. Elevation & Depth: Tonal Layering

We move away from the "drop shadow" era. Depth in this system is achieved through light and material.

*   **The Layering Principle:** Stack `surface-container` tiers. A `surface-container-lowest` card placed on a `surface-container-low` background creates a natural, soft lift.
*   **Ambient Shadows:** If a shadow is required for a floating button (FAB), use a diffused blur (24px to 32px) at 6% opacity. The shadow color must be a tinted version of `primary` (e.g., a dark emerald shadow) to mimic natural light passing through a green gemstone.
*   **The "Ghost Border":** If accessibility requires a border (e.g., in high-glare outdoor situations), use the `outline-variant` token at **15% opacity**. Never use 100% opaque lines.
*   **Glassmorphism:** For top navigation bars and bottom sheets, use `surface` at 80% opacity with a `blur(12px)` effect. This allows the Jepara patterns and map data to bleed through, softening the UI edges.

---

## 5. Components: Refined Interaction

### Buttons
*   **Primary:** Uses the "Signature Gradient" (Primary to Primary-Container). `rounded-lg` (1rem) corner radius. No border.
*   **Secondary:** `surface-container-highest` background with `on-surface` text. This feels integrated, not loud.
*   **States:** On press, apply a 10% black overlay (Light Mode) or 10% white overlay (Dark Mode).

### Cards & Lists
*   **The "Forbid Dividers" Rule:** Never use lines to separate list items. Use 16px or 24px vertical spacing.
*   **Structure:** Use `surface-container-lowest` for the card body. Use `lg` (1rem) rounding.
*   **Interactive List Items:** On hover/tap, transition the background to `surface-container-high`.

### Input Fields
*   **Visual Style:** Subtle `surface-container-high` fill. No bottom line. No full border. Use a "Ghost Border" only when focused, using the `primary` color at 40% opacity.
*   **Feedback:** Error states use the `error` token (#BA1A1A) but keep the background of the field a soft `error-container` (#FFDAD6) to reduce visual stress.

### Mobility-Specific Components
*   **The "Eco-Tracker" Chip:** A high-gloss, semi-transparent chip using `secondary-container` to highlight green-friendly travel choices.
*   **Dynamic Bottom Sheets:** Use `xl` (1.5rem) rounding on the top corners only. The sheet must utilize backdrop-blur to maintain a "Glass" aesthetic.

---

## 6. Do’s and Don'ts

### Do
*   **Do** use asymmetrical margins (e.g., 24px left, 32px right) for headline elements to create an editorial feel.
*   **Do** favor vertical white space over structural lines.
*   **Do** use the `tertiary` (Warning Yellow) sparingly—only for high-priority alerts or "Premium" status levels.
*   **Do** ensure text on `primary` buttons is always `on-primary` (#FFFFFF) for AA accessibility.

### Don’t
*   **Don't** use pure #000000 for shadows or text. Always use the green-tinted neutrals.
*   **Don't** use standard "Material Design" blue or grey defaults. Every color must feel like it belongs to the Emerald/Earth palette.
*   **Don't** clutter the screen. If a screen feels full, increase the spacing and hide secondary actions in a "More" menu.
*   **Don't** use the Jepara pattern at high opacity. It is a texture, not an illustration.