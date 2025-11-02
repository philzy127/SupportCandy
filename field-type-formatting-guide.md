# Technical Guide: SupportCandy Field Types & Data Formatting

This guide provides a definitive reference for addon developers who need to safely access and format any ticket field for display.

#### **Core Principle**

The `WPSC_Ticket` object uses a `__get` magic method to return values. For simple types like text, it returns a string. For complex types like a dropdown, it automatically fetches the related object (e.g., a `WPSC_Option` object). Your code should always access the property directly (e.g., `$ticket->my_custom_dropdown`) and then format the returned value based on its PHP type as described below.

---

### **Complete Field Type Reference**

| Field `type` in Database | PHP Data Type Received | How to Get Display Name |
| :--- | :--- | :--- |
| **--- Primitives ---** | | |
| `cf_textfield` | `string` | Use the value directly. |
| `cf_textarea` | `string` | Use the value directly. |
| `cf_email` | `string` | Use the value directly. |
| `cf_url` | `string` | Use the value directly. |
| `cf_number` | `string` (or `int`/`float`) | Use the value directly. |
| `cf_time` | `string` (e.g., "14:30")| Use the value directly. |
| `cf_html` | `string` (contains HTML) | Use the value directly. **Do not escape.** |
| **--- Date/Time Objects ---** | | |
| `cf_date` | `DateTime` object |`$value->format( get_option('date_format') )` |
| `cf_datetime`| `DateTime` object |`$value->format( get_option('date_format') . ' ' . get_option('time_format') )` |
| **--- Single Object References ---** | | |
| `cf_single_select` | `WPSC_Option` object | `$value->name` |
| `cf_radio_button` | `WPSC_Option` object | `$value->name` |
| `cf_file-attachment-single`| `WPSC_Attachment` object | `$value->name` |
| **--- Array of Object References ---** | | |
| `cf_multi_select`| `array` of `WPSC_Option` objects | Loop through the array and get the `.name` property of each object. Then `implode(', ', $names)`. |
| `cf_checkbox`| `array` of `WPSC_Option` objects | Loop through the array and get the `.name` property of each object. Then `implode(', ', $names)`. |
| `cf_file-attachment-multiple`| `array` of `WPSC_Attachment` objects | Loop through the array and get the `.name` property of each object. Then `implode(', ', $names)`. |
| **--- Default Fields (Primitives) ---** | | |
| `df_id` | `string` (or `int`) | Use the value directly. |
| `df_subject` | `string` | Use the value directly. |
| `df_description`| `string` (contains HTML) | Use the value directly. |
| `df_ip_address` | `string` | Use the value directly. |
| `df_browser` | `string` | Use the value directly. |
| `df_os` | `string` | Use the value directly. |
| `df_source` | `string` | Use the value directly. |
| `df_last_reply_source`| `string` | Use the value directly. |
| `df_user_type` | `string` ("guest" or "registered") | Use the value directly. |
| `df_customer_name`| `string` | Use the value directly. |
| `df_customer_email`| `string` | Use the value directly. |
| **--- Default Fields (Date/Time Objects) ---** | | |
| `df_date_created`| `DateTime` object |`$value->format( get_option('date_format') . ' ' . get_option('time_format') )` |
| `df_date_updated`| `DateTime` object |`$value->format( get_option('date_format') . ' ' . get_option('time_format') )` |
| `df_date_closed`| `DateTime` object |`$value->format( get_option('date_format') . ' ' . get_option('time_format') )` |
| `df_last_reply_on`| `DateTime` object |`$value->format( get_option('date_format') . ' ' . get_option('time_format') )` |
| **--- Default Fields (Single Object References) ---** | | |
| `df_status` | `WPSC_Status` object | `$value->name` |
| `df_priority` | `WPSC_Priority` object | `$value->name` |
| `df_category` | `WPSC_Category` object | `$value->name` |
| `df_customer` | `WPSC_Customer` object | `$value->name` |
| `df_agent_created`| `WPSC_Agent` object | `$value->name` |
| `df_last_reply_by`| `WPSC_Customer` object | `$value->name` |
| **--- Default Fields (Array of Object References) ---** | | |
| `df_assigned_agent`| `array` of `WPSC_Agent` objects | Loop through the array and get the `.name` property of each object. Then `implode(', ', $names)`. |
| `df_prev_assignee`| `array` of `WPSC_Agent` objects | Loop through the array and get the `.name` property of each object. Then `implode(', ', $names)`. |
| `df_tags`| `array` of `WPSC_Ticket_Tags` objects | Loop through the array and get the `.name` property of each object. Then `implode(', ', $names)`. |
| `df_additional_recipients`| `array` of `WPSC_Customer` objects | Loop through the array and get the `.name` property of each object. Then `implode(', ', $names)`. |

---
This guide should provide everything your developer needs to build a robust and reliable value formatting function for the Unified Ticket Macro.