# File Analysis: easy-appointments/src/templates/ea_bootstrap.tpl.php

## High-Level Overview
`ea_bootstrap.tpl.php` is the master template file for the primary user-facing feature of the plugin: the front-end booking form. This template defines the entire HTML structure and layout for the multi-step booking wizard that gets rendered by the `[easyappointments]` shortcode (aliased as `[ea_bootstrap]`).

It functions as a hybrid template, combining server-side PHP execution with client-side Underscore.js templating. It lays out the step-by-step process—from selecting a location, service, and provider, to picking a date and time, and finally filling in personal details. Crucially, it also includes clearly defined PHP filters that act as extension points, most notably for integrating various payment gateways.

## Detailed Explanation
This file provides a single, large Underscore.js template (`ea-bootstrap-main`) that is controlled by a parent PHP class and a front-end JavaScript application.

```php
<script type="text/template" id="ea-bootstrap-main">
    <div class="ea-bootstrap">
        <form class="form-horizontal">
            <!-- Step 1: Location Dropdown -->
            <div class="step form-group">
                <select ...>
                    <?php echo $this->get_options('locations', ...); ?>
                </select>
            </div>

            <!-- Step 2, 3... Service, Worker -->

            <!-- Step 4: Calendar (JS-injected) -->
            <div class="step calendar">
                <div class="date"></div>
            </div>

            <!-- Final Step: Custom Fields -->
            <% _.each(settings.MetaFields, function(item,key,list) { %>
                <div class="form-group">
                    <label><%- item.label %></label>
                    <input id="<%- item.slug %>" ... >
                </div>
            <% });%>

            <!-- Payment Gateway Extension Points -->
            <?php echo apply_filters('ea_payment_select', ''); ?>

            <!-- Submit Buttons -->
            <button class="ea-btn ea-submit">Submit</button>
        </form>
    </div>
</script>
```

- **Key Elements**:
  - **Hybrid Template**: The file is a `.php` file that gets included on the server. It uses PHP (`<?php ... ?>`) to define the `ea_ajaxurl` variable and, most importantly, to call `$this->get_options(...)` to populate the initial dropdowns for locations, services, and staff. The rest of the template uses Underscore.js syntax (`<% ... %>`) for client-side rendering.
  - **Multi-Step Form**: The UI is broken into sections with the class `.step`, which the controlling JavaScript application will show and hide sequentially to guide the user through the booking process.
  - **Dynamic Fields**: It dynamically generates the customer information form by looping through the `settings.MetaFields` array, creating inputs for all the custom fields defined by the administrator.
  - **Filter-Based Extensibility**: The template includes several `apply_filters` calls, such as `ea_payment_select` and `ea_checkout_button`. This is a robust, well-designed pattern that allows third-party add-ons to inject payment forms and buttons directly into the booking process.

## Features Enabled

### Admin Menu
- This file has no effect on the WordPress admin panel.

### User-Facing
- This template is the foundation for the entire `[easyappointments]` shortcode, which is the plugin's main feature.
- It renders the complete step-by-step booking wizard that allows a customer to book an appointment.
- It integrates various features configured in the admin panel, such as custom fields, GDPR consent, reCAPTCHA, and more.
- It provides the structure for payment gateway integrations.

## Extension Opportunities
This file is one of the most extensible parts of the plugin, thanks to its use of filters.

- **Payment Gateways**: The primary extension method is using the provided filters. A developer can easily build a payment gateway add-on by hooking into `ea_payment_select` to add their payment fields and `ea_checkout_button` to modify the final submission button.
  ```php
  // In a custom plugin:
  add_filter('ea_payment_select', 'my_custom_payment_fields');
  function my_custom_payment_fields($content) {
      $content .= '<div><label>Credit Card:</label><input type="text"></div>';
      return $content;
  }
  ```
- **Template Overriding**: While not implemented by default, a significant improvement would be to allow theme-based overrides for this template. This would let developers completely customize the booking form's HTML structure without editing plugin files.
- **Potential Risks**: The hybrid nature of the template, mixing PHP calls within a client-side template structure, can be confusing. A developer modifying it needs to be aware of what is rendered on the server versus what is rendered by JavaScript in the browser.

## Next File Recommendations
We have now analyzed the core front-end template. The next logical steps are to examine the PHP class that controls it and the JavaScript application that brings it to life.

1.  **`src/shortcodes/ea_bootstrap.php`**: This is the highest priority. It is the PHP class that registers the `[easyappointments]` shortcode, includes this template, and contains the `get_options` method used within it. It is the server-side controller for the entire booking form.
2.  **`js/frontend-bootstrap.js`**: This is the most likely candidate for the front-end JavaScript application that controls the step-by-step logic of the form, makes AJAX calls to get available time slots, and renders the booking overview.
3.  **`src/services/ea_appointments_service.php`**: This service class is the final piece of the puzzle. It handles the server-side logic when the booking form is submitted, responsible for validating the data and creating the appointment in the database.
