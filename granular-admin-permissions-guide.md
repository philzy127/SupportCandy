# Technical Guide: Implementing Granular Admin Permissions for SupportCandy

This guide provides a complete architectural and implementation plan for creating a granular, role-based permission system for the SupportCandy admin area.

**Objective:**
To allow a site administrator to grant specific WordPress roles access to specific SupportCandy admin pages (like "Settings" or "Customers") without granting them the full `manage_options` capability, which is a significant security risk.

---

### **Part 1: Architectural Overview & Core Concepts**

The solution is a two-part system that works with WordPress's core APIs to create a secure and seamless user experience.

1.  **UI & Custom Capabilities:** Your addon will create its own set of granular capabilities (e.g., `scp_access_settings`). You will provide an admin UI where an administrator can assign these new, safe capabilities to any WordPress role.
2.  **Security & Access Control:** Your addon will use the `user_has_cap` filter to act as a "bridge." It will dynamically and temporarily grant the powerful `manage_options` capability to a user *only* when they are trying to access a SupportCandy page for which they have been given the corresponding custom capability.
3.  **UI Hiding:** Your addon will use the `admin_menu` action to dynamically hide any SupportCandy submenu items that a user should not see, preventing them from ever clicking a link to a page they cannot access.

**Why this architecture?**
SupportCandy uses two different capabilities to control access: `wpsc_agent` for the ticket list and `manage_options` (the administrator-level capability) for all other settings pages. Simply giving a role `manage_options` is not secure. This architecture allows you to grant access *conditionally*, which is the correct and secure WordPress approach.

---

### **Part 2: Step-by-Step Implementation Plan**

#### **Step 1: Create the Custom Capabilities and the Admin UI**

1.  **Define Your Custom Capabilities:** Decide on a consistent, prefixed set of capabilities. For each SupportCandy admin page you want to control, you will need one capability.
    *   `scp_access_settings` (for the "Settings" page)
    *   `scp_access_customers` (for the "Customers" page)
    *   `scp_access_agents` (for the "Support Agents" page)
    *   `scp_access_reports` (for a future "Reports" page, etc.)

2.  **Build the Admin UI:** Create a new settings page in your addon.
    *   Programmatically get a list of all editable WordPress roles using `get_editable_roles()`.
    *   For each role, display a series of checkboxes, one for each of your custom capabilities defined above (and one for the built-in `wpsc_agent`).
    *   The UI will look something like this:
        *   **Editor**
            *   [✓] Can Access Ticket List (`wpsc_agent`)
            *   [✓] Can Access Settings (`scp_access_settings`)
            *   [ ] Can Access Customers (`scp_access_customers`)
        *   **Author**
            *   [✓] Can Access Ticket List (`wpsc_agent`)
            *   [ ] Can Access Settings (`scp_access_settings`)
            *   [ ] Can Access Customers (`scp_access_customers`)

3.  **Implement the Save Logic:** When the administrator saves this page:
    *   Loop through each role and each capability checkbox.
    *   Use the standard WordPress functions `get_role()` along with `$role->add_cap()` and `$role->remove_cap()` to add or remove your custom capabilities based on whether the boxes were checked.

    ```php
    // Example for saving the "Editor" role's settings
    $editor_role = get_role('editor');
    if ( $editor_can_access_settings ) {
        $editor_role->add_cap('scp_access_settings');
    } else {
        $editor_role->remove_cap('scp_access_settings');
    }
    ```

#### **Step 2: Implement the Capability Bridging Filter (The Security Layer)**

This is the core of the security model. Hook a function into the `user_has_cap` filter.

```php
add_filter( 'user_has_cap', 'my_addon_bridge_capabilities', 10, 3 );

function my_addon_bridge_capabilities( $allcaps, $caps, $args ) {

    // This filter runs on every capability check. We only care about the check for 'manage_options'.
    if ( ! in_array( 'manage_options', $caps ) ) {
        return $allcaps;
    }

    // Check if the user is trying to access a SupportCandy admin page.
    // This is a helper function you will need to write (see below).
    $current_sc_page = get_current_supportcandy_page_slug();
    if ( ! $current_sc_page ) {
        return $allcaps;
    }

    // This is our mapping of SupportCandy page slugs to our custom capabilities.
    $capability_map = array(
        'wpsc-settings'         => 'scp_access_settings',
        'wpsc-customers'        => 'scp_access_customers',
        'wpsc-support-agents'   => 'scp_access_agents',
        // Add all other page slugs and their corresponding custom caps here...
    );

    // If we have a map for the current page...
    if ( isset( $capability_map[ $current_sc_page ] ) ) {

        // ...and the current user has the granular capability for that page...
        $required_cap = $capability_map[ $current_sc_page ];
        if ( ! empty( $allcaps[ $required_cap ] ) ) {

            // ...then we dynamically grant them 'manage_options' for this check only.
            $allcaps['manage_options'] = true;
        }
    }

    return $allcaps;
}

/**
 * Helper function to identify the current SupportCandy admin page.
 *
 * @return string|false The page slug (e.g., 'wpsc-settings') or false.
 */
function get_current_supportcandy_page_slug() {
    if ( ! is_admin() || empty( $_GET['page'] ) ) {
        return false;
    }

    $page = sanitize_text_field( $_GET['page'] );

    // Check if the page belongs to SupportCandy.
    if ( strpos( $page, 'wpsc-' ) === 0 ) {
        return $page;
    }

    return false;
}
```

#### **Step 3: Implement the Menu Hiding Logic (The UI Layer)**

This ensures the user only sees links to pages they can actually access.

```php
// Use a high priority to run after SupportCandy builds its menu.
add_action( 'admin_menu', 'my_addon_hide_unauthorized_menus', 99 );

function my_addon_hide_unauthorized_menus() {

    // The parent menu slug is always 'wpsc-tickets'.
    $parent_slug = 'wpsc-tickets';

    // Map of submenu slugs to the custom capabilities that grant access.
    $submenu_map = array(
        'wpsc-settings'       => 'scp_access_settings',
        'wpsc-customers'      => 'scp_access_customers',
        'wpsc-support-agents' => 'scp_access_agents',
        // ... add all other submenu slugs here
    );

    // Loop through our map and check permissions for each page.
    foreach ( $submenu_map as $submenu_slug => $required_cap ) {

        // If the current user does NOT have the required custom capability...
        if ( ! current_user_can( $required_cap ) ) {

            // ...remove the menu item so they don't see it.
            remove_submenu_page( $parent_slug, $submenu_slug );
        }
    }
}
```

By following these three steps, your addon will create a secure, granular, and user-friendly permission system that works seamlessly with both WordPress and SupportCandy's architectures.
