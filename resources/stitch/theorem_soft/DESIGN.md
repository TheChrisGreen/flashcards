# Design System Strategy: The Elevated Academic

## 1. Overview & Creative North Star
The "Creative North Star" for this design system is **"The Kinetic Library."** 

Math is often perceived as static and rigid; this system aims to flip that narrative. We are moving away from the "generic ed-tech" look (heavy borders, flat primary colors, and stiff grids). Instead, we are building an environment that feels like a premium, physical workspace—think high-end stationery meets a fluid, digital playground. 

By leveraging **intentional asymmetry**, **glassmorphism**, and **tonal depth**, we create an experience where information doesn't just sit on a screen—it floats in a curated, breathable space. We use "Minimalist-Playful" as our guiding light: the minimalism ensures cognitive load is low for complex problem solving, while the playfulness comes through tactile depth and vibrant motion.

---

## 2. Colors: Tonal Architecture
We move beyond "coloring in the lines" to using color as a structural material.

### The "No-Line" Rule
Standard 1px borders are strictly prohibited for sectioning. To define boundaries, you must use **background shifts**. A `surface-container-low` (#f1f4f6) section should sit directly on a `surface` (#f7f9fb) background. This creates "soft edges" that make the math app feel approachable rather than clinical.

### Surface Hierarchy & Nesting
Treat the UI as a series of nested, physical layers.
*   **Base:** `surface` (#f7f9fb)
*   **Secondary Content Areas:** `surface-container` (#eaeef1)
*   **Interactive Cards:** `surface-container-lowest` (#ffffff)
*   **Highlighted Workspace:** `surface-bright` (#f7f9fb)

### The Glass & Gradient Rule
To achieve a "Signature" look, main CTA buttons or progress indicators should not be flat. Use a subtle linear gradient from `primary` (#0060ad) to `primary-container` (#9ac3ff) at a 135-degree angle. For floating navigation or over-content modals, use `surface-container-lowest` with a 70% opacity and a `24px` backdrop-blur to create a "frosted glass" effect.

---

## 3. Typography: Editorial Logic
We utilize a high-contrast scale to create an "Editorial Math" feel, separating the *problem* from the *instruction*.

*   **Display & Headlines (Plus Jakarta Sans):** These are our "Brand Voice." The wide apertures and geometric curves of Plus Jakarta Sans provide the "playful" element of the system. Use `display-md` for achievement milestones and `headline-sm` for lesson titles.
*   **Body & Labels (Inter):** This is our "Functional Voice." Inter is used for mathematical expressions and instructions where legibility is non-negotiable. Use `body-lg` for the core math problems to ensure maximum clarity.
*   **Scale Contrast:** Always pair a `headline-lg` with a `body-md`. The significant jump in scale creates an expensive, intentional layout that guides the eye naturally.

---

## 4. Elevation & Depth: The Layering Principle
We reject the standard "Material" drop shadow. Depth is achieved through **Tonal Layering**.

*   **Natural Lift:** To make a card "float," place a `surface-container-lowest` (#ffffff) card on top of a `surface-container-low` (#f1f4f6) background. The contrast is enough to define the shape without visual clutter.
*   **Ambient Shadows:** If a card requires a "floating" interactive state (e.g., a draggable math block), use a shadow with a `48px` blur, 0px offset, and 6% opacity of the `on-surface` (#2d3337) color. It should look like a soft glow of light, not a dark stain.
*   **The Ghost Border:** If high-contrast accessibility is required, use a "Ghost Border": `outline-variant` (#acb3b7) at **15% opacity**.

---

## 5. Components

### Interactive Cards (The Workspace)
*   **Style:** Corner radius `lg` (2rem). Background `surface-container-lowest`. No borders.
*   **Layout:** Use `spacing-6` (2rem) for internal padding to give the math equations "room to breathe."

### Primary Action Buttons
*   **Style:** `primary` gradient background. Corner radius `full` (9999px).
*   **Text:** `title-sm` in `on-primary` (#f8f8ff).
*   **State:** On hover, increase the elevation using an **Ambient Shadow**, never change the background color to a darker shade.

### Progress Orbs (The Vibrant Accent)
*   **Style:** Use `tertiary-container` (#ffd709) for pending progress and `secondary` (#006e36) for completion. 
*   **Visual Soul:** Apply a soft inner-glow to progress bars using a 10% lighter tint of the base color to make them look "liquid."

### Input Fields
*   **Style:** Background `surface-container-high` (#e3e9ec). Corner radius `md` (1.5rem).
*   **Interaction:** On focus, transition the background to `surface-container-lowest` and add the "Ghost Border."

### Forbid Divider Lines
Lists must never use horizontal rules. Separate list items using `spacing-2` (0.7rem) of vertical white space or by alternating background tints between `surface` and `surface-container-low`.

---

## 6. Do’s and Don’ts

### Do:
*   **Do** use asymmetrical margins. Offsetting a headline by `spacing-4` relative to the body text creates a sophisticated, editorial feel.
*   **Do** use the `xl` (3rem) corner radius for large lesson containers to emphasize the "friendly" nature of the app.
*   **Do** use `tertiary` (#705e00) as a "spark" color—sparingly, only for rewards or "Eureka!" moments.

### Don’t:
*   **Don’t** use pure black (#000000) for text. Always use `on-surface` (#2d3337) to maintain the soft, premium aesthetic.
*   **Don’t** stack more than three layers of surfaces. It creates "visual mud." Keep the hierarchy shallow: Base > Container > Card.
*   **Don’t** use the `none` (0px) roundedness scale. Even the smallest element should have a `sm` (0.5rem) radius to stay on-brand.