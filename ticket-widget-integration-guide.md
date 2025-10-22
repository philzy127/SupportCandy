# SupportCandy Addon Development: A Guide to Integrating Custom Ticket Widgets

## 1. Executive Summary

This document provides a technical guide for addon developers who wish to insert a custom information block—a "pseudo-widget"—into the individual ticket view of the SupportCandy plugin.

**The Challenge:** The core plugin's architecture does not provide a direct PHP action or filter hook for an addon to register and render a new widget *within* the existing stack of widgets. The widget system is managed by a single database option (`wpsc-ticket-widget`) and is not designed for runtime additions by external code.

**The Solution:** A stable and safe integration can be achieved using a two-step process that combines PHP and JavaScript:
1.  **Render via PHP:** Use an available PHP action hook (`wpsc_after_ticket_widget`) to print your widget's HTML content onto the page. Initially, this will place it after all other widgets.
2.  **Reposition via JavaScript:** Enqueue a custom JavaScript file that relocates your widget in the DOM, moving it to its final, configured position (e.g., before or after a specific core widget).

This method is robust, forward-compatible, and does not require any modifications to the core plugin files, making it ideal for a distributable addon.

## 2. Analysis of the Core Widget System

The rendering of widgets on the individual ticket page is handled by the `load_widget_html()` method within the `WPSC_Individual_Ticket` class (`/includes/admin/tickets/class-wpsc-individual-ticket.php`).

This method performs the following steps:
1.  Retrieves the widget layout and configuration from a single database option named `wpsc-ticket-widget`.
2.  Loops through the array of enabled widgets.
3.  Calls a static `print_widget()` method on the corresponding class for each widget.

Crucially, there are no dynamic hooks *inside* the rendering loop. However, two hooks are available just outside of it:
*   `wpsc_before_ticket_widget`: Fires before the widget rendering loop begins.
*   `wpsc_after_ticket_widget`: Fires after the widget rendering loop has completed.

Using these hooks will render content on the page but outside the primary widget container. The `wpsc_after_ticket_widget` hook is the ideal entry point for our purposes.

## 3. Core Widget CSS Selectors

To correctly position your custom widget with JavaScript, you must be able to target the core "factory" widgets. Each widget is wrapped in a `<div>` that includes a base class (`wpsc-it-widget`) and a unique CSS class.

The following table lists the default widgets and their unique selectors, which you can use to configure your addon's placement logic.

| Widget Name                 | CSS Selector                      |
| --------------------------- | --------------------------------- |
| Additional Recipients       | `.wpsc-itw-add-rec`               |
| Agent Only Fields           | `.wpsc-itw-agentonly-fields`      |
| Assigned Agents             | `.wpsc-itw-assignee`              |
| Biographical Info           | `.wpsc-itw-bio-info`              |
| Change Status               | `.wpsc-itw-ticket-status`         |
| Raised By                   | `.wpsc-itw-raised-by`             |
| Ticket Custom Fields        | `.wpsc-itw-ticket-fields`         |
| Ticket Info                 | `.wpsc-itw-ticket-info`           |
| Ticket Tags                 | `.wpsc-itw-ticket-tags`           |

## 4. Step-by-Step Implementation Guide

This section provides the PHP and JavaScript code required to implement the pseudo-widget.

### Step 1: Render the Custom Widget HTML (PHP)

First, hook into `wpsc_after_ticket_widget` to render your widget's HTML. Give your widget a unique ID so it can be easily identified by your JavaScript.

```php
<?php
/**
 * Action to render the custom widget's HTML.
 *
 * @param WPSC_Ticket $ticket The current ticket object.
 */
add_action( 'wpsc_after_ticket_widget', function( $ticket ) {

    // You can add logic here to check if the widget should be displayed
    // for the current ticket, user, etc.

    // Your widget's dynamic content would be generated here.
    $widget_content = 'This is my custom widget content for ticket #' . $ticket->id;

    // Output the widget HTML.
    // Note the unique ID 'my-addon-custom-widget' for JavaScript targeting
    // and the 'wpsc-it-widget' class for styling consistency.
    ?>
    <div id="my-addon-custom-widget" class="wpsc-it-widget">
        <div class="wpsc-widget-header">
            <h2>My Custom Widget</h2>
            <span class="wpsc-itw-toggle" data-widget="my-addon-custom-widget">
                <!-- You can use the SupportCandy icon helper -->
            </span>
        </div>
        <div class="wpsc-widget-body">
            <?php echo esc_html( $widget_content ); ?>
        </div>
    </div>
    <?php
});
```

### Step 2: Enqueue and Write the Positioning Script (PHP & JavaScript)

Next, properly enqueue a JavaScript file that will execute on the ticket page. This script will handle the DOM manipulation.

#### A. Enqueue the Script (PHP)

Add the following to your addon's main PHP file or includes.

```php
<?php
/**
 * Enqueue the custom JavaScript for widget positioning.
 */
add_action( 'wp_enqueue_scripts', function() {

    // Optional: Add a check here to ensure this script only loads on the
    // individual ticket page to improve performance.

    wp_enqueue_script(
        'my-addon-widget-positioner',
        plugin_dir_url( __FILE__ ) . 'js/widget-positioner.js', // The path to your JS file
        array( 'jquery' ), // Depends on jQuery
        '1.0.0',
        true // Load in the footer
    );
});
```

#### B. Write the Positioning Script (JavaScript)

Create the `js/widget-positioner.js` file. This script will find your widget and move it relative to a target core widget.

```javascript
// js/widget-positioner.js
document.addEventListener('DOMContentLoaded', function() {

    // --- Configuration --- //
    // In your addon, these values would come from your settings.

    // The unique selector of the core widget to position against.
    // Example: '.wpsc-itw-ticket-info' (the "Ticket Info" widget).
    const targetSelector = '.wpsc-itw-ticket-info';

    // The placement: 'after' or 'before'.
    const placement = 'after';

    // --- Logic --- //

    const customWidget = document.getElementById('my-addon-custom-widget');
    const targetWidget = document.querySelector(targetSelector);

    // Ensure both widgets exist on the page before trying to move them.
    if (!customWidget || !targetWidget) {
        console.log('My Addon: Could not find custom or target widget.');
        return;
    }

    // Use the appropriate DOM manipulation method based on configuration.
    if (placement === 'after') {
        // The insertAfter() method is not native, so we use its equivalent:
        targetWidget.parentNode.insertBefore(customWidget, targetWidget.nextSibling);
    } else if (placement === 'before') {
        targetWidget.parentNode.insertBefore(customWidget, targetWidget);
    }
});
```

## 5. Conclusion

By following this guide, you can create a custom widget that seamlessly integrates with the SupportCandy individual ticket view. This PHP-and-JavaScript approach provides maximum flexibility for positioning your widget while respecting the core plugin's architecture, ensuring your addon remains stable and maintainable through future updates.
