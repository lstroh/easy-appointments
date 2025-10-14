# File Analysis: easy-appointments/src/api/apifullcalendar.php

## High-Level Overview

This file, `apifullcalendar.php`, implements a set of REST API endpoints for the Easy Appointments plugin. Its primary role is to serve appointment data to a front-end calendar, likely the [FullCalendar.io](https://fullcalendar.io/) library, given the class name `EAApiFullCalendar`.

It acts as the bridge between the plugin's database (containing appointments, services, workers, etc.) and a visual calendar interface. It provides methods to fetch a collection of appointments within a specific date range and to retrieve the details of a single appointment, including a form for editing it.

## Detailed Explanation

The file defines the `EAApiFullCalendar` class, which encapsulates all the logic for the REST endpoints.

-   **Class `EAApiFullCalendar`**:
    -   **`__construct($db_models, $options, $mail)`**: The constructor takes several dependency-injected objects:
        -   `$db_models`: An object for database interactions (likely defined in the reviewed `src/dbmodels.php`).
        -   `$options`: An object for retrieving plugin settings.
        -   `$mail`: An object for handling email-related functionalities, like generating confirmation/cancellation links.
    -   **`register_routes()`**: This is the key method that registers two WordPress REST API endpoints under the `easy-appointments/v1` namespace.
        1.  `GET /appointments`: Fetches a list of appointments. It accepts query parameters like `start`, `end`, `location`, `worker`, and `service` to filter the results. The data is formatted into a structure that FullCalendar can consume directly.
        2.  `GET /appointment/(?P<id>\d+)`: Fetches data for a single appointment. Unconventionally for a REST API, this endpoint returns raw HTML, not JSON. It displays appointment details and, for authorized users, an editable form.
    -   **`get_items_permissions_check($request)`**: This function checks if the current user has permission to view the appointments. Access is granted to users with the `read` capability. It also includes a filter, `ea_calendar_public_access`, which allows developers to grant access to non-logged-in users.
    -   **`get_items($request)`**: The callback for the `/appointments` endpoint. It retrieves appointments using `$this->db_models->get_all_appointments()` and maps the raw data to a format suitable for a calendar, including `start`, `end`, `title`, and `color` properties.
    -   **`get_item($request)`**: The callback for the `/appointment/<id>` endpoint. It retrieves a single appointment and uses a template engine (`Leuffen\TextTemplate`) to render an HTML view. If the user has permission (is the appointment owner or an admin), it also renders a complete HTML form to edit the appointment's custom fields. This endpoint terminates execution with `exit;` after printing the HTML.

## Features Enabled

### Admin Menu

-   This file does not directly add any new admin menus or pages.
-   However, the REST endpoints it creates are likely consumed by a JavaScript application within the WordPress admin area (e.g., on a dashboard or a dedicated appointments page) to display a comprehensive calendar of all bookings.
-   The `get_item` endpoint provides the HTML form for in-line editing of appointments directly from a calendar view.

### User-Facing

-   **REST Endpoints**: The primary user-facing feature is the set of REST endpoints that allow a front-end calendar to be displayed via a shortcode or a Gutenberg block.
    -   `GET easy-appointments/v1/appointments`
    -   `GET easy-appointments/v1/appointment/<id>`
-   **Dynamic Calendar**: It enables a dynamic, filterable calendar on the front end where users can see available/booked slots.
-   **Appointment Details**: It allows users to click on an event in the calendar to see more details.
-   **Self-Service Editing/Cancellation**: The `get_item` endpoint generates links for users to confirm or cancel their own appointments (`link_cancel`, `link_confirm`). It also provides the editing form if the user is logged in and owns the appointment.

## Extension Opportunities

-   **Public Calendar Access**: The easiest extension point is the `ea_calendar_public_access` filter. You can use it to make the calendar visible to the public without requiring them to log in.
    ```php
    add_filter( 'ea_calendar_public_access', '__return_true' );
    ```
-   **Modify Calendar Data**: To modify the data sent to the calendar, you would need to intercept it. Since there isn't a dedicated filter in the `get_items` method's mapping function, a safe way to extend it is limited. A good improvement would be to add a filter inside the `array_map` callback:
    ```php
    // Suggestion for improvement in get_items()
    $result = apply_filters('ea_fullcalendar_appointment_data', $result, $element);
    return $result;
    ```
-   **Refactor `get_item`**: The `get_item` endpoint could be refactored to return JSON data instead of HTML. This would decouple the back-end from the front-end presentation, making it more flexible. A separate JavaScript component could then consume this JSON to render the view, which is a more modern and maintainable approach.
-   **Potential Risks**: The `get_item` endpoint's use of `exit;` can interfere with other plugins or WordPress hooks that might run after the REST API response is generated. The direct output of HTML from a REST endpoint is unconventional and can make client-side implementation more complex than consuming standard JSON.

## Next File Recommendations

To understand how these API endpoints are used, I recommend analyzing the following files next. These have not been reviewed according to `completed_files.txt`.

1.  **`easy-appointments/src/shortcodes/fullcalendar.php`**: This file is the most logical next step. It likely contains the shortcode `[ea_full_calendar]` (or similar) that, when placed on a page, renders the calendar and uses the REST endpoints from `apifullcalendar.php` to display appointments.
2.  **`easy-appointments/js/frontend.js`**: This JavaScript file is probably responsible for the client-side logic. It should contain the code that initializes the FullCalendar library, makes the AJAX requests to the REST endpoints you just analyzed, and handles user interactions like clicking on events or filtering the calendar.
3.  **`easy-appointments/ea-blocks/src/ea-fullcalendar/`**: This directory likely contains the source code for a Gutenberg block that embeds the calendar. Analyzing its contents (e.g., `index.js`, `edit.js`) will reveal how the calendar is integrated into the modern WordPress block editor and how it uses the API.
