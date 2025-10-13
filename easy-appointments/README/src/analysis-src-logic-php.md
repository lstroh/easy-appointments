# File Analysis: easy-appointments/src/logic.php

This document provides a detailed analysis of the `logic.php` file for the Easy Appointments plugin.

## High-Level Overview

`easy-appointments/src/logic.php` defines the `EALogic` class, which can be considered the "brain" of the plugin. This class is responsible for executing the core business logic, with its most critical function being the calculation of appointment availability.

It sits between the database layer (`EADBModels`) and the presentation layers (`EAAjax`, `EAFrontend`). It fetches raw data about working hours, breaks, and existing appointments from the database and applies a series of complex rules and filters to determine which time slots are actually available for a user to book. It also handles validation rules, such as preventing booking spam.

## Detailed Explanation

-   **Key Class:** `EALogic`
    -   The constructor uses dependency injection to receive instances of `$wpdb`, `EADBModels`, `EAOptions`, and `EASlotsLogic`. This shows it's a high-level service that orchestrates other, more specialized services.

-   **Key Functions:**
    -   `get_open_slots(...)`: This is the most important and complex method in the class. It determines which time slots are available for a given location, service, worker, and day. Its process is as follows:
        1.  **Fetch Rules:** It queries the database (via its dependencies) to get all working-hour "connection" rules for the specified day.
        2.  **Generate All Possible Slots:** It loops from the start of a working day to the end, incrementing by the service's `slot_step`, to generate a list of all theoretically possible appointment times.
        3.  **Handle Recurrence:** It correctly processes weekly recurrence rules (`repeat_week`) to ensure that appointments are only shown on the correct weeks (e.g., every 2nd week).
        4.  **Filter Closed Slots:** It calls a helper, `remove_closed_slots`, which fetches "non-working" time rules (like lunch breaks) and removes any slots that fall within those periods.
        5.  **Filter Reserved Slots:** It calls another helper, `remove_reserved_slots`, which fetches all confirmed appointments for the day and removes any slots that conflict with them. This step also accounts for "block before" and "block after" time buffers and daily booking limits for a service.
        6.  **Format Output:** Finally, it passes the remaining, truly available slots to the `format_time` method to prepare them for display on the front end (e.g., in 12-hour or 24-hour format).
    -   `can_make_reservation($data)` and `can_make_reservation_by_user($data)`: These methods enforce business rules to prevent abuse. They check if a user (identified by IP address or logged-in user ID) has exceeded the daily limit of pending bookings and return a boolean result.

-   **WordPress API & Database Interaction:**
    -   The class interacts heavily with the database, but always **indirectly** through its injected dependencies, `EADBModels` and `EASlotsLogic`. It does not contain any raw SQL itself but rather calls methods like `$this->wpdb->prepare` and `$this->slots_logic->get_busy_slot_query`.
    -   It also interacts with WordPress options indirectly via its `EAOptions` dependency to fetch settings like time format and booking limits.

## Features Enabled

This file is a pure backend library and does not directly create any UI elements. However, it is the foundational logic that enables the core functionality of the plugin.

### Admin Menu
-   This file has no direct impact on the admin menu.

### User-Facing
-   **Availability Calculation:** This class is solely responsible for the "Available time slots" list that a user sees on the booking form. Every time a user selects a date, an AJAX request is made that ultimately triggers the `get_open_slots` method.
-   **Booking Validation:** It enforces the daily booking limits per user/IP, preventing spam and ensuring fair access. If a user sees a "Daily limit of booking request has been reached" error, that message originates from the logic in this file.

## Extension Opportunities

-   **Custom Booking Validation:** The most powerful extension points in this file are the filters on the reservation validation methods:
    -   `apply_filters('ea_can_make_reservation', $result, $data)`
    -   `apply_filters('ea_can_update_reservation', $result, $appointment, $data)`
    -   A developer can hook into these filters to add custom validation logic. For example, you could prevent a user from booking if they are not in a specific user role, or if the appointment is for a date that is too far in the future.
    ```php
    // Example: Prevent booking if the user's email is from a blocked domain.
    add_filter('ea_can_make_reservation', function($result, $data) {
        if (strpos($data['email'], '@blocked-domain.com') !== false) {
            $result['status'] = false;
            $result['message'] = 'Bookings from this email provider are not allowed.';
        }
        return $result;
    }, 10, 2);
    ```
-   **Potential Risks:** The `get_open_slots` method is the heart of the plugin and is highly complex. Any modification to this method, or to a class that overrides it, must be done with extreme care. A small logical error could lead to incorrect availability being shown (e.g., showing no slots at all, or showing slots that are already booked), which would be a critical failure for the plugin.

## Next File Recommendations

Having analyzed the core PHP components of the plugin, the following files are the most logical next steps to complete the analysis:

1.  **`easy-appointments/src/services/SlotsLogic.php`**: The `EALogic` class has a dependency on `EASlotsLogic`. This class was likely created to offload some of the complexity of slot calculation from the main logic file. Analyzing it is essential to fully understand the availability algorithm.
2.  **`easy-appointments/src/mail.php`**: This file contains the `EAMail` class, which is responsible for a critical piece of functionality: sending email notifications to both customers and administrators. Understanding how it constructs and sends these emails is important for customization.
3.  **`easy-appointments/src/utils.php`**: This utility class has been referenced for handling template paths. Analyzing it will clarify how template overrides work and reveal any other helper functions that might be useful for development and customization.
