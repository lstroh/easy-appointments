# File Analysis: easy-appointments/src/services/SlotsLogic.php

## High-Level Overview
The file `easy-appointments/src/services/SlotsLogic.php` contains the `EASlotsLogic` class, a specialized service that is core to the plugin's scheduling engine. Its sole responsibility is to determine when a time slot is unavailable or "busy".

This class encapsulates the complex and highly configurable logic for what constitutes a scheduling conflict. The plugin allows an administrator to define availability based on various rules (e.g., a worker is busy if they have *any* appointment, or only if they have an appointment for a specific service). `EASlotsLogic` translates the configured rule into a precise database query to find conflicting appointments. It is a critical component for preventing double-bookings and accurately calculating a provider's availability.

## Detailed Explanation
`EASlotsLogic` is a focused class that constructs SQL queries based on plugin settings. It is instantiated with the WordPress database object and the plugin's options manager.

```php
class EASlotsLogic {
    private $WORKER = '0';
    private $MATCH_ALL = '1';
    private $LOCATION = '2';
    private C$SERVICE = '3';
    private $EXCLUSIVE_WORKER = '4';

    // ...

    public function __construct($wpdb, $options) { ... }

    public function get_busy_slot_query($location, $service, $worker, $day, $app_id) { ... }

    // ...
}
```

- **Key Functions and Classes**:
  - `__construct($wpdb, $options)`: The constructor uses dependency injection to get the global `$wpdb` object and the `EAOptions` service, which are used to build queries and read settings.
  - `get_busy_slot_query(...)`: This is the central method. It dynamically builds a prepared SQL query to find existing appointments that would make a new slot unavailable. The logic hinges on the `multiple.work` setting:
    - **Mode-Based Logic**: It uses a `switch` statement to change the `WHERE` clause of the query based on the selected mode (`WORKER`, `LOCATION`, `SERVICE`, etc.). This determines the conditions under which a slot is considered busy.
    - **Database Interaction**: It queries the `{$wpdb->prefix}ea_appointments` table to find appointments on a specific day (`$day`) that are not canceled or abandoned. It correctly uses `$wpdb->prepare` to prevent SQL injection.
  - `is_exclusive_mode()`: A helper that returns true if the `multiple.work` setting is in `EXCLUSIVE_WORKER` mode.
  - `is_provider_is_busy(...)`: A helper method that appears to be used in exclusive mode to check if a provider's existing appointment is for a different location or service.

## Features Enabled
This is a backend service class and has no direct UI. It is a dependency for other features.

### Admin Menu
- This file does not add any admin menus. It implements the functionality of the "Multiple work" setting found in the plugin's admin settings pages.

### User-Facing
- This class is fundamental to the front-end booking calendar. When a user selects a date to see available times, an AJAX call to the backend will use this class to determine which slots are already filled. The results are used to visually distinguish available vs. unavailable slots on the calendar.

## Extension Opportunities
The logic in this class is hardcoded, which makes it powerful but rigid. The best way to improve its extensibility would be to add WordPress filters.

- **Recommended Improvement**: Introduce a filter on the generated SQL query. This would allow developers to create custom availability rules without modifying the plugin's core code.

  **Example: Adding a filter**
  ```php
  public function get_busy_slot_query($location, $service, $worker, $day, $app_id)
  {
      // ... existing query building logic ...

      $full_query = str_replace($this->PLACEHOLDER, $dynamic_part, $static_part);
      $prepared_query = $this->wpdb->prepare($full_query, $params);

      // Add a filter to allow full modification of the final prepared query
      return apply_filters('ea_busy_slot_query', $prepared_query, compact('location', 'service', 'worker', 'day', 'app_id'));
  }
  ```

- **Potential Risks**: The primary limitation is inflexibility. If a business has a unique scheduling rule not covered by the built-in modes (e.g., "a worker is busy only if they have more than 2 appointments in the same hour"), there is no way to implement it without editing the plugin directly. The use of numeric constants (`0`, `1`, `2`...) for the modes also makes the code harder to read than if it used descriptive strings.

## Next File Recommendations
`EASlotsLogic` tells us which time slots are busy. To get the full picture of availability, we need to see how this is combined with a provider's general working hours. The following files are the logical next step.

1.  **`src/services/ea_connections_service.php`**: In Easy Appointments, "Connections" define the working hours for a provider (which services they offer, at which locations, and on what days/times). This service is responsible for fetching that base availability data from the database, which is the starting point for any availability calculation.
2.  **`src/services/ea_appointments_service.php`**: This service likely handles the creation and modification of appointments. It would use `EASlotsLogic` to validate that a requested slot is actually available before saving a new appointment to the database.
3.  **`src/shortcodes/ea_bootstrap.php`**: This file is the entry point for the front-end booking form. Analyzing it would show how the entire booking process is initiated, how scripts are loaded, and how initial data (like settings and translations) is passed from the PHP backend to the client-side application.
