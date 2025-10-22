# Technical Guide: Accessing Standard Date Properties on the WPSC_Ticket Object

This guide provides a definitive list of property names to access the raw `DateTime` objects for standard date fields on the `WPSC_Ticket` object within filter callbacks.

## Context

When using the `wpsc_ticket_field_val_*` filters to customize date formats in the ticket list, you receive the `$ticket` object as a parameter. While `$ticket->date_created` is a straightforward property, other standard date fields use different property names.

These property names correspond directly to the slugs defined in the `create_default_custom_fields()` method in `supportcandy/class-wpsc-installation.php`.

## Definitive Property List

Here are the correct properties to access the `DateTime` objects for the specified fields:

*   **Date Created:**
    *   **Filter:** `wpsc_ticket_field_val_date_created`
    *   **Property:** `$ticket->date_created`

*   **Date Updated:**
    *   **Filter:** `wpsc_ticket_field_val_date_updated`
    *   **Property:** `$ticket->date_updated`

*   **Last Reply On:**
    *   **Filter:** `wpsc_ticket_field_val_last_reply_on`
    *   **Property:** `$ticket->last_reply_on`

*   **Date Closed:**
    *   **Filter:** `wpsc_ticket_field_val_date_closed`
    *   **Property:** `$ticket->date_closed`

## Example Implementation

Here is an updated code example demonstrating how to correctly access these properties within your addon.

```php
<?php
/**
 * Plugin Name: SupportCandy Custom Date Formats Addon
 * Description: Demonstrates how to access various standard date properties on the ticket object.
 * Version: 1.1
 * Author: Your Name
 */

// Ensure this code runs only within the WordPress context
if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

add_action( 'plugins_loaded', 'wpsc_custom_date_properties_addon_init' );

function wpsc_custom_date_properties_addon_init() {

    // --- Date Updated ---
    add_filter( 'wpsc_ticket_field_val_date_updated', 'wpsc_addon_get_date_updated', 10, 3 );
    function wpsc_addon_get_date_updated( $value, $cf, $ticket ) {
        // Correctly access the DateTime object using the 'date_updated' property
        $date_updated_obj = $ticket->date_updated;
        if ( is_object( $date_updated_obj ) ) {
            // Your custom formatting logic here...
            // For example, return a custom format.
            // return 'Updated: ' . $date_updated_obj->format('Y-m-d');
        }
        return $value; // Return original value if property doesn't exist or isn't a DateTime object
    }

    // --- Last Reply On ---
    add_filter( 'wpsc_ticket_field_val_last_reply_on', 'wpsc_addon_get_last_reply_on', 10, 3 );
    function wpsc_addon_get_last_reply_on( $value, $cf, $ticket ) {
        // Correctly access the DateTime object using the 'last_reply_on' property
        $last_reply_obj = $ticket->last_reply_on;
        if ( is_object( $last_reply_obj ) ) {
            // Your custom logic here...
        }
        return $value;
    }

    // --- Date Closed ---
    add_filter( 'wpsc_ticket_field_val_date_closed', 'wpsc_addon_get_date_closed', 10, 3 );
    function wpsc_addon_get_date_closed( $value, $cf, $ticket ) {
        // Correctly access the DateTime object using the 'date_closed' property
        $date_closed_obj = $ticket->date_closed;
        if ( is_object( $date_closed_obj ) ) {
            // Your custom logic here...
        }
        return $value;
    }
}
```
This guide should resolve the issue of accessing the correct date properties, allowing you to proceed with your addon development.
