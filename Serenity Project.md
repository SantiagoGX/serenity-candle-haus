# Shopify Optimization Task: Project Serenity

## Context
Improve section customizability and implement pre-order logic via tags. All edits must be done via schema settings to avoid hardcoded text. 

## 1. Section: Featured Product Custom (`sections/featured-product-custom.liquid`)
- **Current state:** Product image is dynamic, but badges (ingredients, scent, etc.) and labels are hardcoded.
- **Task:** - Identify all hardcoded strings in the HTML/Liquid.
    - Add corresponding `text` or `richtext` fields to the `schema`.
    - Replace hardcoded text with `{{ section.settings.field_name }}`.
    - Ensure all CTA buttons have editable labels and URL fields.

## 2. Section: Scroll Gallery (`sections/scroll-gallery.liquid`)
- **Current state:** Background text contains "Lorem Ip Zoom Dollar Seed" under titles.
- **Task:** - Remove the hardcoded Lorem Ipsum text.
    - Add a `text` field in the schema for "Background/Supportive Text".
    - Update the Liquid template to render this new setting.

## 3. Section: Footer (`sections/footer.liquid`)
- **Task A (Menus):** Replace static text columns with functional Shopify Menus.
    - Implement dynamic iteration: `for link in linklists[section.settings.menu].links`.
    - Allow the client to select the menu from the Theme Editor settings.
- **Task B (Newsletter):** Ensure the `<form>` uses `customer` context. 
    - Required: `<input type="hidden" name="contact[tags]" value="newsletter">`.
    - Validate that the email input correctly saves the user to the Shopify Customers database.
- **Task C (Branding):** - Add a setting for "Agency Display Text" and "Agency URL".
    - Default URL: https://snc-designs.com/
    - Display text should be editable, e.g., "Creado con amor por SNC Designs".

## 4. Pre-Order Logic (Product Page - `sections/main-product.liquid`)
- **Core Logic:** Trigger UI changes if `product.tags` contains the string 'pre-order'.
- **Task A (Badge & Button):** - Display a "Pre-order" badge in the Hero section.
    - **Optimization:** If the 'pre-order' tag is present, change the primary Buy Button text from "Add to Cart" to "Pre-order Now".
- **Task B (Delivery Date Customization):**
    - Locate the current "Delivery date" logic (auto-calculating +3 to +7 days).
    - Create a conditional: If `pre-order` tag exists, hide the auto-calculation.
    - Show instead: `{{ section.settings.pre_order_delivery_text }}`.
    - Add this `text` setting to the `main-product` schema.

## General Best Practices (Review before finishing)
- **Scannability:** Maintain existing CSS classes and BEM naming conventions if applicable.
- **Fallbacks:** Ensure that if a schema field is empty, the section doesn't break or show empty HTML tags.
- **Clean Code:** Use Liquid comments to mark the beginning and end of your edits.