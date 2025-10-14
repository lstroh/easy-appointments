# File Analysis: easy-appointments/src/api/vacation.php

## High-Level Overview

This file, `vacation.php`, defines the `EAVacationActions` class, which provides the REST API endpoints for managing the plugin's "Vacations" or "Days Off" functionality. It allows administrators to create, read, and update vacation rules for employees.

Architecturally, this class serves as the backend API for the "Vacation" settings page in the WordPress admin dashboard. Interestingly, the vacation data is not stored in its own dedicated database table. Instead, all vacation rules are stored as a single JSON array within one row of the plugin's main options table (`wp_ea_options`). This file provides the necessary endpoints to safely read and write to that JSON data structure.

## Detailed Explanation

-   **Key Class:** `EAVacationActions`
    -   The constructor receives the `EADBModels` and `EAOptions` services via dependency injection, allowing it to interact with the database options.
    -   **`register_routes()`**: This method registers two endpoints for the same route, `/easy-appointments/v1/vacation`, both requiring administrator privileges (`manage_options`):
        1.  `GET /vacation`: This endpoint calls the `get_vacations` method to fetch the current vacation rules.
        2.  `EDITABLE /vacation` (handles POST, PUT, PATCH): This endpoint calls the `update_vacations` method to overwrite the vacation rules with a new set.
    -   **`get_vacations()`**: This is the "read" method. It retrieves the `vacations` option string from the database (via the `EAOptions` service), JSON-decodes it into a PHP array, and sends it to the client using `wp_send_json`.
    -   **`update_vacations(WP_REST_Request $request)`**: This is the "write" method. It takes the raw JSON payload from the request body, sanitizes it using the private `process_data` helper, and saves the validated and re-encoded JSON string back to the `vacations` option in the database (via the `EADBModels` service).
    -   **`process_data($data)`**: This private helper function acts as a security and validation layer. It ensures the incoming data is a valid JSON array of objects and that each object contains the required keys (`id`, `title`, `tooltip`, `workers`, `days`). This prevents malformed or incomplete data from being saved.

## Features Enabled

### Admin Menu

-   This file provides the backend API that powers the **Settings > Vacation** page in the WordPress admin.
-   The UI on that page uses `GET /vacation` on page load to display the list of configured vacation days.
-   When the administrator saves their changes, the UI sends the entire set of rules as a JSON payload to `POST /vacation` to be stored in the database.

### User-Facing

-   This file has no direct user-facing features.
-   However, the data it manages is **critically important** to the front-end booking form. The core availability engine (`EALogic`) reads the vacation rules set by this API to determine which days are blocked off. This prevents users from booking appointments when an employee is on vacation.

## Extension Opportunities

-   **No Direct Hooks**: The class itself contains no WordPress actions or filters, which makes it difficult to extend. For example, there is no easy way to trigger a notification when vacations are updated or to add a custom validation rule without modifying the plugin's code. A `do_action('ea_vacations_updated', $data)` hook in the `update_vacations` method would be a valuable addition.
-   **Data Storage Model**: The choice to store all vacation rules as a single JSON blob in the options table is simple but has scalability limitations. It is not possible to perform efficient database queries on this data (e.g., "get all vacations for a specific worker"). For sites with a very large number of employees and vacation rules, migrating this to a dedicated database table would be a more robust solution.
-   **Potential Risks**: The code is reasonably safe. The `process_data` function provides a good layer of sanitization, preventing corrupt data from being saved. The primary limitation is its inflexibility due to the lack of hooks.

## Next File Recommendations

We have now analyzed the entire REST API layer. The next logical steps are to dive into the last remaining core service and then explore the front-end implementation details.

1.  **`easy-appointments/src/services/SlotsLogic.php`**: This is the most important unreviewed file. It is a key dependency of `EALogic` and contains the deepest, most complex algorithms for calculating time slot availability, accounting for things like breaks, existing appointments, and buffer times. A full understanding of the plugin's core feature is not possible without analyzing this file.
2.  **`easy-appointments/ea-blocks/ea-blocks.php`**: This file is the entry point for the plugin's integration with the modern WordPress block editor (Gutenberg). It will reveal how the booking form is registered and configured as a block, which is a crucial part of the modern user experience.
3.  **`easy-appointments/src/shortcodes/fullcalendar.php`**: In contrast to the modern block, this file implements the legacy shortcode for displaying the full calendar view. Analyzing it will show how the `apifullcalendar` REST endpoints are consumed by a shortcode-based implementation.
