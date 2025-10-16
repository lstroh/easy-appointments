# File Analysis: easy-appointments/src/services/UserFieldMapper.php

## High-Level Overview
The file `easy-appointments/src/services/UserFieldMapper.php` implements a "smart pre-fill" feature for the booking form. Its purpose is to improve the user experience for logged-in WordPress users by automatically populating form fields with their existing profile information.

This class, `EAUserFieldMapper`, hooks into the plugin's form generation process. It allows an administrator to map booking form fields to standard WordPress user fields (like `user_email`) or custom user meta fields (like `billing_phone`). When a logged-in user views the form, this service fetches their data and fills in the mapped fields, saving the user time and reducing potential typing errors.

## Detailed Explanation
`EAUserFieldMapper` works by filtering the array of form fields right before they are rendered. It uses the "Default Value" property of a field as a key to look up data in the current user's profile.

```php
class EAUserFieldMapper
{
    public function init()
    {
        add_filter('ea_form_rows', array($this, 'process_fields'));
    }

    public function process_fields($fields)
    {
        $current_user = wp_get_current_user();
        // ... merge user data and meta ...

        foreach ($fields as $field) {
            if (array_key_exists($field->default_value, $user_data)) {
                $field->default_value = $user_data[$field->default_value];
            }
        }
        return $fields;
    }

    public static function all_field_keys() { ... }
}
```

- **Key Functions and Classes**:
  - `init()`: This method hooks the class into the plugin's architecture by adding the `process_fields` method to the `ea_form_rows` filter.
  - `process_fields($fields)`: This is the core logic. It fetches the logged-in user's data using `wp_get_current_user()` and `get_user_meta()`, creating a comprehensive array of all available user information. It then iterates through the form fields. If a field's `default_value` (set by the admin) matches a key in the user's data array, it replaces that `default_value` with the user's actual data.
  - `all_field_keys()`: A static helper method that generates a comma-separated list of all possible keys from the user object and meta fields. This is likely used in the admin UI to show the administrator which values are available for mapping.

- **WordPress Integration**: The class is tightly integrated with the WordPress User system, making direct use of `wp_get_current_user` and `get_user_meta`. It also demonstrates good plugin citizenship by using a custom filter (`ea_form_rows`) to apply its logic, rather than modifying the form generation code directly.

## Features Enabled

### Admin Menu
- This file does not add any new admin pages.
- It provides the underlying logic for a feature configured within the custom field editor (likely under **Settings > Customize**). The `all_field_keys()` method supports this by providing a list of available placeholders to the admin.

### User-Facing
- This class directly enhances the front-end booking form for logged-in users.
- It automatically pre-populates fields such as Name, Email, and Phone, creating a more seamless and efficient booking process.

## Extension Opportunities
The current implementation is a direct 1-to-1 mapping. It could be made more flexible.

- **Recommended Improvement**: Add a filter to allow for data transformation. This would enable more complex mappings, such as combining `first_name` and `last_name` from user meta into a single "Full Name" field.

  **Example: Adding a transformation filter**
  ```php
  // Inside process_fields() foreach loop
  if (array_key_exists($field->default_value, $user_data)) {
      $value = $user_data[$field->default_value];
      
      // Allow the mapped value to be transformed
      $field->default_value = apply_filters('ea_user_field_map_value', $value, $field->default_value, $user_data, $field);
  }
  ```

- **Potential Risks**: The logic for guest users (not logged in) clears the default value for most fields. This could be problematic if an admin wants to set a global default value for a field that should apply to everyone. The behavior could be made more nuanced, for example, by only clearing default values that look like user data keys (e.g., start with `user_` or `billing_`).

## Next File Recommendations
`EAUserFieldMapper` modifies form fields by hooking into the `ea_form_rows` filter. The most logical next step is to investigate where these form fields are created and how the form itself is constructed and processed.

1.  **`src/metafields.php`**: This file is the strongest candidate for being the origin of the form fields. The name suggests it's responsible for managing the custom fields of the booking form. It is likely the place where the `ea_form_rows` filter is defined and applied, making it the orchestrator of the form-building process.
2.  **`src/services/ea_appointments_service.php`**: After the form is pre-filled by this mapper and submitted by the user, the data needs to be saved. This service is likely responsible for handling the submitted data and creating a new appointment in the database.
3.  **`src/shortcodes/ea_bootstrap.php`**: This file represents the beginning of the entire front-end process. It's responsible for registering the shortcode that displays the booking form, loading necessary scripts, and triggering the form generation that `EAUserFieldMapper` plugs into.
