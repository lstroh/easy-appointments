# File Analysis: easy-appointments/src/api/gdpr.php

## High-Level Overview

This file, `gdpr.php`, defines the `EAGDPRActions` class, which provides a specific tool for General Data Protection Regulation (GDPR) compliance. Its sole purpose is to create a REST API endpoint that allows an administrator to delete old, personally identifiable information from the database.

Architecturally, it's a small, self-contained service that hooks into the WordPress REST API. It's designed to be called either manually by an administrator or by an automated process (like a cron job) to enforce a data retention policy.

## Detailed Explanation

-   **Key Class:** `EAGDPRActions`
    -   The constructor receives the `EADBModels` service via dependency injection, giving it access to the database layer.
    -   **`register_routes()`**: This method registers a single REST API endpoint:
        -   **Route:** `DELETE /wp-json/easy-appointments/v1/gdpr`
        -   **Permissions:** The endpoint is protected and can only be accessed by authenticated users with the `manage_options` capability (typically administrators).
        -   **Callback:** The request is handled by the `clear_old_custom_data` method.
    -   **`clear_old_custom_data()`**: This is the action-performing method.
        -   It constructs and executes a raw SQL `DELETE` query.
        -   The query targets the `wp_ea_fields` table, which stores the data entered into custom form fields by users.
        -   It deletes records from `wp_ea_fields` that are linked to appointments in `wp_ea_appointments` where the appointment's `end_date` is more than 6 months in the past.
        -   **Crucially, it only deletes the custom field data, not the core appointment record itself.** This preserves the appointment history for statistical purposes while removing potentially sensitive personal information.

## Features Enabled

### Admin Menu

-   This file does not add any visible UI elements to the admin menu.
-   It provides a backend API endpoint that is likely triggered by a button or process initiated from one of the plugin's "Tools" or "Settings" pages. It also provides the logic for the automated daily cron job `ea_gdpr_auto_delete` (which is scheduled in `src/options.php`).

### User-Facing

-   This file has **no direct impact** on the user-facing side of the website. Its function is purely for backend data management and privacy compliance.

## Extension Opportunities

-   **Hardcoded Retention Period:** The biggest limitation is that the 6-month data retention period is hardcoded directly into the SQL query. This is not flexible. A significant improvement would be to make this value a configurable setting in the plugin's options and use that value to build the query dynamically.
-   **No Hooks:** The class is not designed for extension; it contains no WordPress actions or filters. To modify its behavior, one would need to either edit the file directly (not recommended) or deregister the REST route and register a custom one with a different callback. Adding a `do_action('ea_before_gdpr_data_deleted', $query)` hook would allow developers to add logging or other actions before the data is removed.
-   **Limited Scope:** The function is very specific and only deletes data from the `ea_fields` table. It does not clear the customer's name, email, or phone number from the main `ea_appointments` table. A filter on the query or a more comprehensive deletion function would be needed to expand its scope.
-   **Potential Risks:** As with any `DELETE` query, there is an inherent risk of accidental data loss if the logic were ever modified incorrectly. The current implementation appears safe for its intended purpose.

## Next File Recommendations

To continue exploring the plugin's architecture, especially its modern components, I recommend analyzing the following unreviewed files:

1.  **`easy-appointments/src/services/SlotsLogic.php`**: This is the most critical unreviewed file for understanding the plugin's core business logic. It is a dependency of `EALogic` and contains the specialized, complex algorithms for calculating time slot availability.
2.  **`easy-appointments/src/api/mainapi.php`**: This file is the central router for the plugin's v1 REST API. Analyzing it will show how different API controllers (like `EAGDPRActions` and `EAApiFullCalendar`) are loaded and registered, giving a complete overview of the modern API structure.
3.  **`easy-appointments/ea-blocks/ea-blocks.php`**: This file is the entry point for the plugin's integration with the WordPress block editor (Gutenberg). It's essential for understanding how the booking form is made available as a modern, configurable block.
