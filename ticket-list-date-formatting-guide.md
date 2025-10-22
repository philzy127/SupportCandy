# Technical Guide: Customizing Date and Time Formats in the Ticket List

This document explains how an addon developer can modify the date and time format for any date-based column in the SupportCandy Ticket List view.

## Summary

SupportCandy uses a flexible system for formatting date and time fields, such as "Date Created," "Last Reply On," or any custom `datetime` fields you create. While administrators can set a global date format in the general settings, developers can override this for specific fields using dedicated PHP filters. This allows for powerful customizations, like displaying a human-readable time difference (e.g., "2 hours ago") or showing only the time for recent tickets.

## Key Files Analyzed

*   `supportcandy/includes/admin/tickets/class-wpsc-ticket-list.php`: The main class that renders the ticket list table. The `print_tickets()` method is responsible for iterating through tickets and their columns.
*   `supportcandy/includes/custom-field-types/class-wpsc-cf-datetime.php`: The class that handles user-created custom fields of the "Datetime" type.
*   `supportcandy/includes/custom-field-types/class-wpsc-df-date-created.php`: The class for the default "Date Created" system field. (Similar classes exist for other default date fields).

## The Formatting Mechanism

The rendering process for each date field in the ticket list culminates in a call to a `get_ticket_field_val()` method within the field's corresponding class. This method performs two key actions:

1.  **Format Selection:** It first checks if a custom format is defined for that specific field. If not, it falls back to the global "Default Date Format" configured in `SupportCandy -> Settings -> General -> General Settings`.
2.  **Filter Application:** Crucially, before returning the final formatted date string, it passes the value through a dynamic WordPress filter. This is the intended entry point for developers.

## Available Filter Hooks

There is a unique filter for each date-based field. The filter names are constructed as follows:

`wpsc_ticket_field_val_{field_type_slug}`

Based on our investigation, here are the key filters an addon developer can use:

*   **For Custom "Datetime" Fields:**
    ```php
    apply_filters( 'wpsc_ticket_field_val_datetime', $value, $cf, $ticket, $module );
    ```

*   **For the "Date Created" Field:**
    ```php
    apply_filters( 'wpsc_ticket_field_val_date_created', $value, $cf, $ticket, $module );
    ```
    *(Note: Similar filters exist for `date_updated`, `date_closed`, `last_reply_on`, etc.)*

### Filter Parameters

*   `$value` (string): The already-formatted date string.
*   `$cf` (WPSC_Custom_Field): The custom field object, containing all its properties (e.g., `$cf->slug`, `$cf->name`).
*   `$ticket` (WPSC_Ticket): The complete ticket object.
*   `$module` (string): The context from which the function is being called (e.g., 'ticket-list').

## Implementation Example

The following code demonstrates how an addon can modify the format for the "Date Created" column to show the time only if the ticket was created today, and the full date otherwise.

This code should be placed in your addon's main PHP file.

```php
<?php
/**
 * Plugin Name: SupportCandy Custom Date Formats Addon
 * Description: Customizes the date format for the 'Date Created' column in the ticket list.
 * Version: 1.0
 * Author: Your Name
 */

// Ensure this code runs only within the WordPress context
if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

add_action( 'plugins_loaded', 'wpsc_custom_date_format_addon_init' );

function wpsc_custom_date_format_addon_init() {

    /**
     * Hooks into the filter for the 'Date Created' field.
     *
     * This function checks if a ticket was created today. If it was, it returns
     * only the time. Otherwise, it returns the original, fully formatted date string.
     */
    add_filter( 'wpsc_ticket_field_val_date_created', 'wpsc_addon_customize_date_created_format', 10, 4 );

    function wpsc_addon_customize_date_created_format( $value, $cf, $ticket, $module ) {

        // Ensure we are only modifying the format in the ticket list view.
        if ( $module !== 'ticket-list' ) {
            return $value;
        }

        // The date object is available in the $ticket object.
        $date_created = $ticket->date_created;

        if ( ! is_object( $date_created ) ) {
            return $value; // Return original value if date is invalid.
        }

        // Get the current date and the ticket creation date in the site's timezone.
        $now = new DateTime( 'now', wp_timezone() );
        $ticket_date = new DateTime( $date_created->format('Y-m-d'), wp_timezone() );

        // Check if the ticket was created today.
        if ( $now->format('Y-m-d') === $ticket_date->format('Y-m-d') ) {

            // Return only the time, formatted according to WordPress settings.
            return wp_date( get_option( 'time_format' ), $date_created->getTimestamp() );

        } else {

            // For older tickets, return the original formatted value.
            return $value;
        }
    }
}
```

By using this approach, developers can safely and reliably customize the appearance of any date field in the ticket list without altering core plugin files.
