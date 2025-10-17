# Technical Details - Email Notifications

## Email Sending Process

The SupportCandy plugin handles email notifications through the `WPSC_Email_Notifications` class, located in `supportcandy/includes/class-wpsc-email-notifications.php`. The system supports both immediate (synchronous) and delayed (asynchronous) email delivery.

### 1. The `WPSC_Email_Notifications` Class

This class is the core component for managing email notifications. It is responsible for:
-   Assembling email components (headers, recipients, subject, body).
-   Validating notification rules and conditions.
-   Sending emails using the WordPress `wp_mail()` function.

### 2. Synchronous vs. Asynchronous Sending

-   **Synchronous Sending**: When an event triggers an email notification, an instance of `WPSC_Email_Notifications` can be created, populated with data, and its `send()` method can be called directly. This sends the email immediately as part of the same process that triggered the event.

-   **Asynchronous (Background) Sending**: To avoid delays in the user interface and to handle large volumes of emails efficiently, the plugin can queue emails to be sent in the background.
    -   The `send_background_emails()` static method is hooked to the `wpsc_cron_five_minute` and `wpsc_run_ajax_background_process` actions. This means it runs periodically (every five minutes via WP-Cron) or via AJAX-triggered background processes.
    -   This method fetches a batch of queued emails from the database (via the `WPSC_Background_Email` model).
    -   For each queued email, it creates a new `WPSC_Email_Notifications` object, populates it with the stored email data, and calls the `send()` method.
    -   After an email is successfully sent, its corresponding record is deleted from the queue in the database.

### 3. Email Delivery

-   **Core Function**: All emails, whether sent synchronously or asynchronously, are ultimately dispatched using the standard WordPress `wp_mail()` function.
-   **Headers**: The `send()` method constructs the necessary email headers, including `From`, `Reply-To`, `CC`, `BCC`, and `Content-Type`.
-   **Attachments**: The system can include attachments with notifications. It temporarily copies the attachment files to a `wpsc/temp` directory within the WordPress uploads folder, passes them to `wp_mail()`, and then cleans up the temporary files and directories.
-   **Blocked Emails**: The plugin maintains a configurable list of blocked email addresses. The `block_emails_list()` method loads these addresses, and the system ensures that no emails are sent to recipients on this list.

## Email Content Generation

The content of each notification email is dynamically generated based on templates and ticket data.

### 1. The `is_valid()` Method

Before an email is sent, the `is_valid()` method is called to perform several crucial setup and validation steps:

-   **Enable Check**: It first checks if the notification template is enabled (`is_enable`).
-   **Sender Information**: It sets the `From Name` and `From Email` based on the plugin's general email settings.
-   **Subject and Body**: It generates the email subject and body by processing the template and replacing macros.
-   **Recipients**: It determines the `To`, `CC`, and `BCC` recipients based on the template's configuration.
-   **Conditions**: It validates any configured conditions (e.g., send only if ticket priority is 'High') using the `WPSC_Ticket_Conditions` class.

If any of these checks fail (e.g., no recipients, conditions not met), the email is not sent.

### 2. Email Templates

-   Email templates are stored in the WordPress options table and define the structure and content for different types of notifications (e.g., 'New Ticket Created', 'Agent Reply').
-   Each template includes:
    -   Subject line (with macros).
    -   Body content (with macros).
    -   Recipient rules (e.g., send to customer, assigned agent, specific email addresses).
    -   Conditions for sending.

### 3. Subject and Body Creation

-   **Subject**: The `set_subject()` method retrieves the subject template, replaces any macros with ticket data, and prepends the ticket ID (e.g., `[Ticket-123] Your subject here`).
-   **Body**: The main body of the email is retrieved from the template. The `WPSC_Macros::replace()` function is then called to substitute all macro tags in the body with the relevant live data from the ticket.

## Macro System

Macros are placeholders (e.g., `{{ticket_id}}`) that can be inserted into email templates and are replaced with dynamic data when an email is generated. The macro system is managed by the `WPSC_Macros` class, located in `supportcandy/includes/class-wpsc-macros.php`.

### 1. The `WPSC_Macros` Class

This class is responsible for:
-   Defining the list of available macros.
-   Providing the logic to replace macro tags with actual data.

### 2. Macro Registration

-   The `load_macros()` method initializes a list of default macros, including ticket properties (ID, subject, status), customer details (name, email), and conversation history (`{{ticket_history}}`, `{{last_reply}}`).
-   It also dynamically adds macros for any custom fields that are configured to be available in macros.
-   **Extensibility**: The list of macros can be extended by other plugins or custom code using the `wpsc_macros` filter.

### 3. Macro Replacement

-   The `replace()` static method is the core of the macro system. It takes a string (like an email subject or body) and a ticket object as input.
-   It uses a `switch` statement to handle the replacement for each specific macro tag found in the string.
-   For each macro, it calls a corresponding internal method (e.g., `replace_ticket_url()`) or directly fetches the data from the ticket object or related custom fields.
-   **Extensibility**: The replacement logic can be extended to handle custom macros using the `wpsc_replace_macros` filter. If the `switch` statement does not find a match for a macro, this filter is applied, allowing other code to provide the replacement value.

### 4. Available Macros

The plugin provides a wide range of macros out-of-the-box, including but not limited to:
-   `{{ticket_id}}`: The numeric ID of the ticket.
-   `{{customer_name}}`: The full name of the customer.
-   `{{customer_first_name}}`: The first name of the customer.
-   `{{last_reply}}`: The content of the last reply to the ticket.
-   `{{ticket_history}}`: A formatted history of the ticket's conversation.
-   `{{ticket_url}}`: A clickable link to the ticket view page.
-   Custom field macros (e.g., `{{priority}}`, `{{category}}`).