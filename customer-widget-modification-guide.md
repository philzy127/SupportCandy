# SupportCandy Addon Development: A Guide to Modifying the Customer Widget

## 1. Executive Summary

This document provides a technical guide for addon developers who wish to add custom information to the "Customer" widget on the individual ticket view. This is the ideal approach for integrating related data, such as contact information from a company directory, directly into the customer's profile block.

**The Solution:** The SupportCandy plugin provides a dedicated PHP action hook, `wpsc_itw_raisedby`, which allows external addons to inject content directly into the body of this widget. This is a clean, stable, and direct method that does not require any JavaScript manipulation.

**Important Note:** The widget you see as "Customer" in the UI is internally named the "Raised By" widget (`wpsc-itw-raised-by`). The plugin's settings allow administrators to change the display title, which is a common customization. The hook `wpsc_itw_raisedby` is the correct one to use for modifying this widget, regardless of its displayed title.

## 2. Analysis of the "Raised By" Widget

The widget is rendered by the `print_widget()` method inside the `WPSC_ITW_Raisedby` class (`/includes/admin/tickets/widgets/class-wpsc-itw-raisedby.php`).

Within this method, after the customer's name and avatar are rendered, the plugin explicitly calls a hook for addons to use:

```php
<?php do_action( 'wpsc_itw_raisedby', $ticket ); ?>
```

This hook is perfectly positioned for appending additional information. It passes the current `$ticket` object, which in turn provides access to the customer's data via `$ticket->customer`, allowing you to fetch and display related information.

## 3. Step-by-Step Implementation Guide

The following PHP code demonstrates how to use the `wpsc_itw_raisedby` action hook to display custom contact information from an external source (e.g., your company directory plugin).

You can add this code to your addon's main PHP file.

```php
<?php
/**
 * Hooks into the SupportCandy "Raised By" widget to add custom information.
 *
 * @param WPSC_Ticket $ticket The current ticket object.
 */
add_action( 'wpsc_itw_raisedby', function( $ticket ) {

    // Retrieve the customer object from the ticket.
    $customer = $ticket->customer;
    if ( ! $customer ) {
        return;
    }

    // --- Your Custom Logic --- //
    // This is a placeholder for your function that would fetch data
    // from your company directory based on the customer's email or ID.
    // It should return an array of contact details or null if none are found.
    $contact_info = my_company_directory_get_contact_info( $customer->email );

    // If no contact info is found, do nothing.
    if ( ! $contact_info ) {
        return;
    }

    // --- Display the Information --- //
    // The output is already inside the widget body, so we just need to
    // add our own structured content. Using the 'info-list-item' format
    // will help it blend in with other SupportCandy widgets.
    ?>
    <hr class="wpsc-widget-hr">

    <div class="info-list-item">
        <div class="info-label"><?php echo esc_html( 'Company:' ); ?></div>
        <div class="info-val"><?php echo esc_html( $contact_info['company_name'] ); ?></div>
    </div>

    <div class="info-list-item">
        <div class="info-label"><?php echo esc_html( 'Phone:' ); ?></div>
        <div class="info-val"><?php echo esc_html( $contact_info['phone_number'] ); ?></div>
    </div>

    <?php
});

/**
 * Placeholder function for fetching contact information.
 *
 * In your addon, you would replace this with your actual data retrieval logic.
 *
 * @param string $customer_email The email of the customer.
 * @return array|null An associative array of contact info or null.
 */
function my_company_directory_get_contact_info( $customer_email ) {

    // Example: Replace this with a real database query or API call.
    $directory = [
        'john.doe@example.com' => [
            'company_name' => 'Acme Inc.',
            'phone_number' => '555-1234'
        ],
        'jane.smith@example.com' => [
            'company_name' => 'Globex Corp.',
            'phone_number' => '555-5678'
        ]
    ];

    if ( isset( $directory[ $customer_email ] ) ) {
        return $directory[ $customer_email ];
    }

    return null;
}
```

## 4. Conclusion

Using the `wpsc_itw_raisedby` action hook is the recommended and most robust method for augmenting the "Customer" / "Raised By" widget. This pure PHP approach is efficient, easy to implement, and ensures that your addon will remain compatible with future versions of the SupportCandy plugin.
