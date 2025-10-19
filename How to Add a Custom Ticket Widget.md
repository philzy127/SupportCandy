# How to Add a Custom Ticket Widget

This guide explains how to register a custom ticket widget from an external add-on plugin using the `wpsc_register_ticket_widgets` filter.

## 1. Create Your Widget Class

First, you need to create the PHP class for your widget. This class must have a static method named `print_widget()` that will be called to render the widget's HTML.

**Example `My_Custom_Widget.php`:**

```php
<?php
class My_Custom_Widget {
    /**
     * Renders the HTML for the custom widget.
     *
     * @param WPSC_Ticket $ticket The current ticket object.
     * @param array $settings The settings for this specific widget instance.
     */
    public static function print_widget( $ticket, $settings ) {
        // Check permissions if necessary
        $current_user = WPSC_Current_User::$current_user;
        if ( !$current_user->is_agent ) {
            return;
        }
        ?>
        <div class="wpsc-it-widget my-custom-widget">
            <div class="wpsc-widget-header">
                <h2><?php echo esc_html( $settings['title'] ); ?></h2>
                <span class="wpsc-itw-toggle" data-widget="my-custom-widget"><?php WPSC_Icons::get( 'chevron-up' ); ?></span>
            </div>
            <div class="wpsc-widget-body">
                <p>This is my custom widget content!</p>
                <p>Current Ticket ID: <?php echo intval( $ticket->id ); ?></p>
            </div>
        </div>
        <?php
    }
}
```

## 2. Register Your Widget Using the Filter

Next, in your main plugin file, you need to hook into the `wpsc_register_ticket_widgets` filter. This will add your widget to the array of available widgets.

```php
/**
 * Register our custom widget with SupportCandy.
 *
 * @param array $widgets The existing array of registered widgets.
 * @return array The modified array of widgets.
 */
function my_addon_register_custom_widget( $widgets ) {

    // The unique key for your widget.
    $widget_slug = 'my-custom-widget';

    // Only add the widget if it hasn't been registered already.
    if ( ! isset( $widgets[ $widget_slug ] ) ) {

        $widgets[ $widget_slug ] = array(
            'title'               => 'My Custom Widget',
            'is_enable'           => 1, // Enabled by default
            'allow-customer'      => 0, // Not visible to customers by default
            'allowed-agent-roles' => array( 1, 2 ), // Default roles (Admin, Agent)
            'callback'            => 'my_custom_widget_settings_modal()', // A JS function to open your settings modal
            'class'               => 'My_Custom_Widget', // The name of your widget class
        );
    }

    return $widgets;
}
add_filter( 'wpsc_register_ticket_widgets', 'my_addon_register_custom_widget' );
```

## 3. Handling Widget Settings (Optional)

If your widget needs its own settings, you must:
1.  Define a JavaScript function with the same name as the `callback` you registered (e.g., `my_custom_widget_settings_modal()`).
2.  This function should make an AJAX call to a custom action that you define.
3.  The AJAX action should return the HTML for a modal window containing your settings fields.
4.  You will also need to create another AJAX action to save these settings. It is recommended to save your custom widget's settings in your own `wp_options` entry, not directly into the SupportCandy widget array, to avoid complexity.