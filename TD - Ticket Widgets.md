# Technical Details - Admin Ticket Widgets

In SupportCandy, "Ticket Widgets" refer to the modular blocks of information and functionality displayed in the right-hand sidebar of the individual ticket page within the WordPress admin area. This system allows administrators to customize the layout and content of this sidebar to suit their workflow.

## Core Widget System

The entire Ticket Widget system is managed through the WordPress admin dashboard and relies on a combination of a settings page, a database option, and dynamically loaded classes.

### 1. Management Interface

-   **Location**: The central management interface for these widgets is located at **SupportCandy > Settings > Ticket Widgets**.
-   **Functionality**: On this page, administrators can:
    -   **Enable or Disable** individual widgets.
    -   **Reorder** the widgets using a drag-and-drop interface.
    -   Access the specific **settings** for each widget by clicking the edit icon.

### 2. Data Storage

-   The configuration for the ticket widgets, including their order, enabled/disabled status, and individual settings, is stored in a single entry in the WordPress `wp_options` table.
-   **Option Name**: `wpsc-ticket-widget`
-   **Data Format**: The value of this option is a serialized array. Each element in the array represents a widget and contains its properties, such as the class name, title, and configuration settings. The order of the elements in this array dictates the display order on the ticket page.

### 3. Settings Management

-   The settings page itself is handled by the `WPSC_Ticket_Widgets` class (`supportcandy/includes/admin/settings/class-wpsc-ticket-widgets.php`).
-   This class is responsible for rendering the list of widgets on the settings page (`get_ticket_widget()`) and for saving the new order when it's changed (`set_tw_load_order()`). It acts as the controller for the admin-facing configuration.

## Widget Registration and Rendering

The system is designed to be modular. While the settings page manages the configuration, the actual definition and rendering of the widgets happen elsewhere.

### 1. Widget Class Location

-   Each individual widget is defined by its own class.
-   These classes are located in the directory: `supportcandy/includes/admin/tickets/widgets/`.
-   Each file in this directory (e.g., `class-wpsc-itw-assigned-agents.php`) corresponds to a single widget available in the system. The `itw` prefix stands for "Individual Ticket Widget."

### 2. Rendering Process

-   The rendering of the individual ticket page in the admin area is controlled by the `WPSC_Individual_Ticket` class (`supportcandy/includes/admin/tickets/class-wpsc-individual-ticket.php`).
-   When an admin views a ticket, the `layout()` method of this class is triggered via an AJAX call.
-   Inside this class, the `load_widget_html()` method is responsible for rendering the widget sidebar.
-   **The process is as follows:**
    1.  The method fetches the `wpsc-ticket-widget` option from the database, which contains the ordered and configured list of widgets.
    2.  It iterates through this list.
    3.  For each widget that is marked as `is_enable`, it dynamically calls a static method named `print_widget()` on the corresponding widget class (e.g., `WPSC_ITW_Assigned_Agents::print_widget()`).
    4.  The `print_widget()` method generates and echoes the HTML for that specific widget.
    5.  The combined HTML for all enabled widgets is captured and displayed in the ticket's sidebar.

## Available Widgets

The following widgets are available to be displayed on the individual ticket page in the admin area. Each is defined in its own class file in the `supportcandy/includes/admin/tickets/widgets/` directory.

-   **`Additional Recipients`** (`class-wpsc-itw-additional-recipients.php`)
    -   **Purpose**: Displays and allows agents to manage a list of CC'd email addresses for a ticket. These recipients will receive the same email notifications as the customer.
    -   **Functionality**: Shows the current list of additional recipients and provides an "edit" button to open a modal where agents can add or remove email addresses.

-   **`Agent Only Fields`** (`class-wpsc-itw-agentonly-fields.php`)
    -   **Purpose**: Displays the values of any custom fields that are designated as "agent only."
    -   **Functionality**: This widget shows a list of agent-only custom fields and their corresponding values for the current ticket. It includes an "edit" button for agents with the appropriate permissions to modify these values.

-   **`Assigned Agents`** (`class-wpsc-itw-assigned-agents.php`)
    -   **Purpose**: Shows which agent(s) are currently assigned to the ticket.
    -   **Functionality**: Displays the names and avatars of the assigned agents. An "edit" button allows authorized agents to change the assignment, with options to filter and sort available agents.

-   **`Biographical Info`** (`class-wpsc-itw-biographical-info.php`)
    -   **Purpose**: Displays the "Biographical Info" from the WordPress user profile of the ticket's creator.
    -   **Functionality**: This is a read-only widget that shows the description from the user's WordPress profile, providing extra context about the customer.

-   **`Change Status`** (`class-wpsc-itw-change-status.php`)
    -   **Purpose**: Displays the ticket's core properties: Status, Category, and Priority.
    -   **Functionality**: Shows the current values for the ticket's status, category, and priority. An "edit" button opens a modal allowing authorized agents to change these values.

-   **`Raised By`** (`class-wpsc-itw-raisedby.php`)
    -   **Purpose**: Displays information about the customer who created the ticket.
    -   **Functionality**: Shows the customer's name and avatar. It also provides action icons for agents to view more information about the customer, see their other tickets, or open their WordPress user profile.

-   **`Ticket Fields`** (`class-wpsc-itw-ticket-fields.php`)
    -   **Purpose**: Displays the values of standard custom fields associated with the ticket.
    -   **Functionality**: Similar to the "Agent Only Fields" widget, but for regular, non-agent-only custom fields. It lists the fields and their values and provides an "edit" button for modification.

-   **`Ticket Info`** (`class-wpsc-itw-ticket-info.php`)
    -   **Purpose**: Shows technical metadata about the ticket's creation.
    -   **Functionality**: This is a read-only widget that displays the customer's IP address, the source of the ticket (e.g., web, email), and the browser and operating system used.

-   **`Ticket Tags`** (`class-wpsc-itw-ticket-tags.php`)
    -   **Purpose**: Manages the tags associated with a ticket.
    -   **Functionality**: Displays the current list of tags. An "edit" icon allows agents to add or remove tags, with an autocomplete feature for existing tags and the ability to create new tags on the fly.

## Configuration and Customization

The primary way to configure the ticket widgets is through the settings page at **SupportCandy > Settings > Ticket Widgets**.

### 1. Enabling, Disabling, and Reordering

-   **Enable/Disable**: Each widget can be toggled on or off. Disabled widgets will not be rendered on the ticket page. This is controlled by the `is_enable` property in the `wpsc-ticket-widget` database option.
-   **Reordering**: Administrators can drag and drop the widgets in the settings list to change their display order in the ticket sidebar. This order is saved directly to the `wpsc-ticket-widget` array in the database.

### 2. Widget-Specific Settings

-   Most widgets have their own configuration options, which are accessible by clicking the "edit" icon next to the widget on the settings page. This opens a modal window with settings specific to that widget.
-   **Common Settings**:
    -   **Title**: Allows the administrator to change the display title of the widget.
    -   **Allowed Agent Roles**: Provides a multi-select box to control which agent roles can see the widget.
    -   **Allowed for Customer**: Some widgets have a toggle to allow them to be visible to customers on the frontend ticket view.
-   **Saving Settings**: When an administrator saves the settings from the modal, an AJAX request is sent (e.g., `wpsc_set_tw_ticket_info`), and the corresponding PHP function in the widget's class file updates the `wpsc-ticket-widget` option in the database with the new configuration.