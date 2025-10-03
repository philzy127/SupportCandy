# SupportCandy Programmer's Codebook

This document provides a high-level overview of the SupportCandy plugin's codebase, intended for developers who wish to understand its structure and extend its functionality.

## High-Level Overview

SupportCandy is a feature-rich support ticket system for WordPress. Its codebase is structured to be modular and extensible, separating core logic, data models, and presentation layers (both for the admin backend and the customer-facing frontend).

## Key Files in Root Directory

-   `supportcandy.php`: The main plugin file. It handles initialization, defines constants, loads internationalization files, and includes all other necessary PHP files.
-   `class-wpsc-installation.php`: Manages the plugin's installation and activation process, including the creation of necessary database tables.
-   `global-functions.php`: Contains globally accessible helper functions used throughout the plugin.
-   `readme.txt`: The official WordPress plugin readme file, containing metadata and a public-facing description.

## Directory Structure

### `asset/`

This directory contains all the static assets for the plugin.

-   `css/`: Stylesheets for both the admin and frontend views.
-   `js/`: JavaScript files for interactive elements.
-   `images/`: Image files used in the plugin's interface.

### `framework/`

Contains the core application framework that provides foundational services like database interaction, settings management, and other low-level functionalities that the rest of the plugin builds upon.

### `i18n/`

Internationalization (i18n) files. It contains the `.pot` template file (`supportcandy.pot`) that can be used to translate the plugin's strings into different languages.

### `includes/`

This is the most important directory, containing the core logic of the plugin, which is further divided into subdirectories.

#### `includes/admin/`

All backend-related functionality resides here. It manages the WordPress admin interface for agents, including:
-   Ticket list and management views.
-   Plugin settings pages.
-   Customer and agent profiles.
-   Email notification templates.

#### `includes/custom-field-types/`

This directory is crucial for understanding how ticket data is structured. It contains a separate class for each type of field that can be associated with a ticket. This includes both default fields (like `class-wpsc-df-subject.php` and `class-wpsc-df-status.php`) and custom field types (like `class-wpsc-cf-text-field.php` and `class-wpsc-cf-checkbox.php`).

#### `includes/frontend/`

Handles what the end-user or customer sees. It primarily consists of classes that implement the shortcodes used to display the support system on the website (e.g., ticket submission forms, ticket history).

#### `includes/models/`

Defines the data models for the plugin's core objects. Each file corresponds to a key entity in the system, such as:
-   `class-wpsc-ticket.php`: The main ticket object.
-   `class-wpsc-thread.php`: Represents a single entry in a ticket (a customer reply, an agent note, etc.).
-   `class-wpsc-customer.php`: The customer object.
-   `class-wpsc-agent.php`: The agent object.
-   `class-wpsc-custom-field.php`: Manages the registration and behavior of custom fields.

#### `includes/rest-api/`

Contains the logic for the plugin's REST API endpoints, allowing for integration with external services or custom frontend applications.

### `upgrade/`

This directory holds the scripts for handling database migrations and upgrades between different versions of the plugin. When a new version of the plugin requires a change to the database schema, a script is added here to manage the transition smoothly.

## Core Concepts for Developers

### Data Models

The plugin's data is organized into clear object models (found in `includes/models/`). The primary objects are `WPSC_Ticket`, which represents a support ticket, and `WPSC_Thread`, which represents each interaction within that ticket. Understanding these models is key to manipulating ticket data.

### Custom Fields

The custom field system is highly flexible. Every piece of data on a ticket, including the subject and status, is treated as a field type. You can find their definitions in `includes/custom-field-types/`. This modular approach makes it easy to add new types of data to tickets.

### Extensibility via Hooks (Actions and Filters)

The plugin is designed to be extensible. It uses WordPress's standard hook system (`add_action()` and `add_filter()`) to allow other plugins to modify its behavior without changing the core code. Key hooks to look for are:

-   `wpsc_macros`: Allows you to add new macros to the email notification system.
-   `wpsc_replace_macros`: Allows you to define the replacement logic for your custom macros.

By using these hooks, you can build powerful add-ons for SupportCandy.

## Example: How to Add a Custom Email Macro

To demonstrate the plugin's extensibility, here is a guide on how to add a custom email macro from your own separate plugin.

### Step 1: Add Your Macro to the List

Use the `wpsc_macros` filter to add your macro to the list that appears in the email editor.

```php
/**
 * Add a custom macro to the list.
 */
function my_custom_macros( $macros ) {
    $macros[] = array(
        'tag'   => '{{a_custom_tag}}',
        'title' => esc_attr__( 'My Custom Macro Title', 'my-text-domain' ),
    );
    return $macros;
}
add_filter( 'wpsc_macros', 'my_custom_macros' );
```

### Step 2: Define the Replacement Logic

Use the `wpsc_replace_macros` filter to tell the system what to replace your macro tag with when an email is sent.

```php
/**
 * Replace the custom macro tag with its value.
 */
function my_custom_macro_replacement( $str, $ticket, $macro ) {
    if ( $macro === 'a_custom_tag' ) {
        // Your logic to get the replacement value goes here.
        // You can use the $ticket object to get context-specific data.
        $replacement_value = 'This is the replacement for a custom tag!';
        $str = str_replace( '{{a_custom_tag}}', $replacement_value, $str );
    }
    return $str;
}
add_filter( 'wpsc_replace_macros', 'my_custom_macro_replacement', 10, 3 );
```
This codebook should serve as a solid starting point for any developer looking to work with the SupportCandy plugin.