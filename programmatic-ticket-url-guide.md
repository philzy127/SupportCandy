# SupportCandy Addon Development: A Guide to Programmatically Retrieving Ticket URLs

## 1. Executive Summary

This document provides the official, developer-intended method for programmatically retrieving the unique front-end URL for an individual ticket. This is a critical function for any addon that needs to link back to a specific ticket from another part of the WordPress admin area, a custom dashboard, or an external system.

**The Challenge:** The exact format of a ticket's URL can vary significantly from one site to another. It is determined by the administrator's choices in **Settings > General Settings > Page Settings**, where they can select different WordPress pages to serve as the base for ticket links. Manually reading these settings and attempting to construct the URL is unreliable and likely to break.

**The Solution:** The only reliable and future-proof method is to use the built-in `get_url()` method available on the `WPSC_Ticket` object. This method contains all the internal logic to account for the user's settings and will always return the correct URL.

## 2. Analysis of the `get_url()` Method

The `get_url()` method is part of the `WPSC_Ticket` class, located in `/includes/models/class-wpsc-ticket.php`. Its sole purpose is to abstract away the complexity of URL generation.

When called, the method performs these critical steps:
1.  **Fetches Page Settings:** It retrieves the `wpsc-gs-page-settings` option from the database.
2.  **Checks Configuration:** It reads the `ticket-url-page` setting to determine whether the administrator has chosen to use the main "Support Page" or a separate "Open Ticket Page" for ticket URLs.
3.  **Gets the Base Permalink:** It uses the WordPress function `get_permalink()` to get the correct base URL for the configured page.
4.  **Builds the Final URL:** It uses `add_query_arg()` to append the correct parameters (e.g., `wpsc-section`, `ticket-id`, `auth-code`) to the base URL, producing a valid link.

By using this method, your addon does not need to be aware of the site's page structure. It will always produce the correct link, whether the URL format is `/?wpsc-section=ticket-list&ticket-id=123` or `/?ticket-id=123`, or any other format the plugin may use in the future.

## 3. Step-by-Step Implementation Guide

The following code provides a robust, copy-paste-ready function for your addon. It takes a ticket ID as input and returns the full, correct URL.

```php
<?php
/**
 * Retrieves the full, front-end URL for a given SupportCandy ticket ID.
 *
 * This is the only reliable method for generating a ticket link, as it uses
 * the core plugin's internal functions, which respect the administrator's
 * Page Settings.
 *
 * @param int $ticket_id The ID of the ticket.
 * @return string The full ticket URL, or an empty string if the ticket is not found.
 */
function my_addon_get_supportcandy_ticket_url( $ticket_id ) {

    // Sanitize the input to ensure it's an integer.
    $ticket_id = intval( $ticket_id );
    if ( ! $ticket_id ) {
        return '';
    }

    // Before proceeding, ensure the core SupportCandy class is available.
    if ( ! class_exists( 'WPSC_Ticket' ) ) {
        // You could add error logging here if desired.
        // error_log('My Addon Error: WPSC_Ticket class not found.');
        return '';
    }

    // Instantiate the ticket object using its ID.
    $ticket = new WPSC_Ticket( $ticket_id );

    // Check if the ticket was found and is a valid object with an ID.
    if ( $ticket && $ticket->id ) {

        // Call the built-in method to get the URL.
        return $ticket->get_url();
    }

    return '';
}

// --- HOW TO USE THE FUNCTION ---

// Example 1: Get the URL for ticket with ID 45.
$ticket_id_to_find = 45;
$ticket_url = my_addon_get_supportcandy_ticket_url( $ticket_id_to_find );

if ( $ticket_url ) {
    // Always escape URLs when outputting them in HTML.
    printf(
        '<a href="%s" target="_blank">View Ticket %d</a>',
        esc_url( $ticket_url ),
        $ticket_id_to_find
    );
} else {
    printf(
        'Could not generate a URL for Ticket ID %d.',
        $ticket_id_to_find
    );
}
```

## 4. Conclusion

For maximum compatibility and reliability, your addon must **never** attempt to build a ticket URL manually. Always instantiate the `WPSC_Ticket` object for the desired ticket and call its `get_url()` method. This is the official, supported, and forward-compatible approach.
