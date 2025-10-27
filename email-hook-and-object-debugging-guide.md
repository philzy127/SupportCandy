# Technical Guide: Debugging and Modifying New Ticket Emails

This guide provides a definitive best practice for addon developers who need to modify the content of new ticket email notifications. It also explains the common pitfalls related to debugging the `WPSC_Ticket` object within this context and provides a safe, reliable protocol for doing so.

## Summary of Findings

1.  **Correct Hook:** The `wpsc_create_ticket_email_data` filter is the correct, officially intended hook for modifying the data of a new ticket email before it is sent.
2.  **Execution Context:** This filter runs **synchronously** during the user-facing ticket creation process. It is a **blocking operation**. Any code attached to it must be lightweight and fast to avoid slowing down the user experience.
3.  **Asynchronous Handoff:** After your synchronous filter runs, the plugin passes the final email data to its own built-in asynchronous background processor (`WPSC_Background_Email`). This means you do not need to build your own asynchronous system; you only need to prepare the data quickly.
4.  **The `WPSC_Ticket` Object:** The hanging issue observed when logging this object is caused by its "active record" design. The object uses lazy-loading and contains circular references (e.g., a ticket has threads, and threads have a ticket). Standard debugging functions like `print_r()` or `var_dump()` will enter an infinite loop trying to resolve these references, causing the script to exhaust memory and time out.

---

## Best Practice: Adding Custom Data to the New Ticket Email

Your goal is to add a formatted HTML table of ticket data to the email. The following is the recommended, most efficient, and most stable method.

### Step 1: Add Your Callback to the Filter

In your addon's main PHP file, attach your function to the `wpsc_create_ticket_email_data` filter.

```php
<?php
add_filter( 'wpsc_create_ticket_email_data', 'my_addon_add_ticket_table_to_email', 10, 2 );

function my_addon_add_ticket_table_to_email( $data, $thread ) {

    // The $thread object is passed to the hook.
    // The ticket object is a property of the thread.
    $ticket = $thread->ticket;

    // 1. Safely gather the ticket data.
    $ticket_info = array(
        'ID'       => $ticket->id,
        'Subject'  => $ticket->subject,
        'Status'   => $ticket->status->name,
        'Priority' => $ticket->priority->name,
        'Customer' => $ticket->customer->name,
    );

    // 2. Build your custom HTML table.
    $html_table = '<h3>Ticket Details</h3><table border="1" cellpadding="5" cellspacing="0"><tbody>';
    foreach ( $ticket_info as $label => $value ) {
        $html_table .= '<tr><td style="background-color:#f7f7f7;"><strong>' . esc_html( $label ) . '</strong></td><td>' . esc_html( $value ) . '</td></tr>';
    }
    $html_table .= '</tbody></table>';

    // 3. Replace a placeholder in the email body with your table.
    //    (Assumes you have added `[my_custom_ticket_table]` to your email template)
    $data['body'] = str_replace( '[my_custom_ticket_table]', $html_table, $data['body'] );

    // 4. Return the modified data array for background processing.
    return $data;
}
```

### Step 2: Add a Placeholder to Your Email Template

1.  Navigate to `SupportCandy -> Email Notifications`.
2.  Edit the "Create Ticket" email template.
3.  In the body of the template, insert the placeholder `[my_custom_ticket_table]` where you want your table to appear.

This approach is clean, performant, and correctly uses the plugin's built-in systems.

---

## Safe Protocol for Debugging the `WPSC_Ticket` Object

To avoid crashing the ticket submission process, **never use `print_r($ticket)` or `var_dump($ticket)` directly.** Instead, use one of the following safe methods.

### Method 1: Get Raw Database Values with `to_array()`

The `WPSC_Ticket` class has a built-in `to_array()` method that returns the raw, unprocessed data from the database. This is the safest and most comprehensive way to see the ticket's core data without triggering the infinite loop.

```php
// This is 100% safe and will not hang.
$ticket_raw_data = $ticket->to_array();

// Safely log this simple array to your debug file.
error_log( print_r( $ticket_raw_data, true ) );
```

### Method 2: Build a Custom Debug Array

The best practice for targeted debugging is to build a simple array containing only the specific data points you need. This gives you a clear and safe snapshot of the ticket's state.

```php
// Create a simple, safe array with only the data you need.
$debug_snapshot = array(
    'ticket_id'      => $ticket->id,
    'subject'        => $ticket->subject,
    'status_name'    => $ticket->status->name,     // Safely gets the status name
    'customer_email' => $ticket->customer->email,  // Safely gets the customer email
    'raw_db_data'    => $ticket->to_array()       // Include the raw data for reference
);

// This is completely safe to log and provides rich information.
error_log( print_r( $debug_snapshot, true ) );
```
