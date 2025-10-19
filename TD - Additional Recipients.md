# Technical Details - Additional Recipients

The "Additional Recipients" feature in SupportCandy is fundamentally a **notification feature**, not an access control system. Its primary purpose is to CC or BCC additional email addresses on ticket-related communications.

## Core Functionality & Data Storage

The mechanism is straightforward and does not involve complex relationships between users and tickets.

### 1. Data Storage

-   **Location**: The data is stored directly on the ticket object, which corresponds to a row in the `{$wpdb->prefix}psmsc_tickets` table.
-   **Field**: The specific database column is `add_recipients`.
-   **Format**: The data is stored as a serialized PHP array of email strings. For example: `a:2:{i:0;s:18:"email1@example.com";i:1;s:18:"email2@example.com";}`.
-   **No User Linking**: It's critical to note that this is just a list of email addresses. The system does not link these emails to any `WPSC_Customer` or WordPress `WP_User` objects in the database.

### 2. Management

-   The **Additional Recipients** widget in the admin ticket view, controlled by `class-wpsc-itw-additional-recipients.php`, provides the UI for agents to view and edit this simple array of email strings.

## The Notification Process

The sole purpose of the `add_recipients` array is to populate the recipient fields (`To:`, `CC:`, `BCC:`) when sending email notifications.

1.  **Trigger**: When an event occurs (e.g., a ticket reply), SupportCandy initializes the `WPSC_Email_Notifications` class.
2.  **Recipient List Building**: The `set_to_addresses()`, `set_cc_addresses()`, and `set_bcc_addresses()` methods are called.
3.  **Matching the Rule**: Inside these methods, a `switch` statement checks the rules defined in the email template. If a rule is set to send to "Additional Recipients", the case for `'add-recipients'` is matched.
4.  **Iteration and Addition**: The code then retrieves the `add_recipients` array from the ticket object, iterates through the email addresses, and adds each one to the appropriate list (`$this->to`, `$this->cc`, or `$this->bcc`) for the email being sent.

## Access Control Implications

This is the most critical aspect to understand for development purposes. Being an "additional recipient" does **not** grant a user formal access rights to the ticket within the SupportCandy system.

-   **No Direct Ticket Access**: A user who is an additional recipient cannot log in and see the ticket in their "My Tickets" list. The query that generates the ticket list is based on the primary `customer` of the ticket, and the `add_recipients` field is not checked.
-   **Access is via Authenticated URL**: The only way an additional recipient can view the ticket is if the email notification they receive contains the direct, authenticated URL (e.g., `.../?ticket-id=123&auth-code=ABCDEFG`). This link provides temporary, guest-like access to view the ticket thread. It does not grant them any other permissions.
-   **No "Shared With Me" View**: There is no built-in interface for a user to see a list of all tickets on which they are an additional recipient.

## Opportunities for Enhancement (for Add-on Development)

The limitations of the default feature provide clear opportunities for a third-party add-on to create a more robust ticket-sharing system.

### 1. Granting Formal Access

An add-on can bridge the gap between notification and permission by using the `wpsc_non_agent_ticket_customers_allowed` filter.

-   **Hook**: `wpsc_non_agent_ticket_customers_allowed`
-   **Purpose**: This filter allows you to programmatically add customer IDs to the list of users who are allowed to view a specific ticket.
-   **Logic**:
    1.  Hook into the `wpsc_change_ticket_add_recipients` action, which fires when the list of recipients is modified.
    2.  In your callback, get the new list of email addresses.
    3.  For each email, check if it corresponds to a registered user (`get_user_by('email', ...)`).
    4.  If it does, store this user-to-ticket link in your own custom table or as ticket metadata.
    5.  Then, hook into the `wpsc_non_agent_ticket_customers_allowed` filter. In your callback for this filter, if the ticket being viewed is one you've linked to the current user, add their customer ID to the array of allowed customers.

This would make the "shared" ticket appear in the user's "My Tickets" list.

### 2. Visually Distinguishing Shared Tickets

Once a user has access, it's important to show them *why*. You can add a visual indicator to the ticket list to differentiate between tickets they own and tickets shared with them.

-   **Method**: The recommended approach is to add a custom CSS class to the ticket's row (`<tr>`) in the list view.
-   **Hook**: Use a filter like `wpsc_ticket_list_row_class` (if available) or a JavaScript-based DOM manipulation approach if no such filter exists.
-   **Logic**:
    1.  When the ticket list row is being rendered, check if the current user's ID matches the ticket's original customer ID (`$ticket->customer->user->ID`).
    2.  If they do not match, it's a shared ticket. Add a custom class like `wpsc-shared-with-me`.
    3.  Use CSS in your add-on's stylesheet to target this class, for example, by adding a unique background color or a "shared" icon.

```css
/* Example CSS for a shared ticket */
tr.wpsc-shared-with-me {
    background-color: #f0f5fa !important; /* A light blue highlight */
    border-left: 3px solid #1e88e5; /* A distinct blue left border */
}

tr.wpsc-shared-with-me td.subject::before {
    content: "\\f504"; /* WordPress Dashicon for "share" */
    font-family: "dashicons";
    display: inline-block;
    margin-right: 8px;
    color: #1e88e5;
}
```