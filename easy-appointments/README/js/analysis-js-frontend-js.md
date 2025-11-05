# File Analysis: easy-appointments/js/frontend.js

## High-Level Overview

`js/frontend.js` is a large jQuery plugin, named `eaStandard`, that provides the client-side logic for the "Standard" layout of the front-end booking form. Its role and functionality are nearly identical to `js/frontend-bootstrap.js`. Both files act as the engine for the user-facing booking process, managing a step-by-step workflow, fetching data dynamically via AJAX, and handling the final submission.

Architecturally, this file represents one of two available "themes" for the booking form. The key distinction between `frontend.js` (Standard) and `frontend-bootstrap.js` (Bootstrap) lies in the user interface layout, specifically in how available time slots are presented to the user after they select a date. The underlying business logic, data flow, and communication with the server are the same.

## Detailed Explanation

This script is a near-duplicate of `frontend-bootstrap.js`, sharing the same dependencies and overall structure. It is instantiated on elements with the class `.ea-standard`.

**Dependencies:**
-   jQuery & jQuery UI Datepicker
-   jQuery Validate
-   Moment.js
-   Underscore.js
-   Global objects `ea_settings` and `ea_ajaxurl`.

**Key Differentiating Feature: Time Slot Rendering**

The most significant difference is in the `dateChange` method, which dictates the UI for time selection.

-   **In `frontend.js` (Standard Layout):** After a date is selected, the script finds the next visible step in the form (a container with the class `.step`) and populates a child element (`.time`) with the available time slots. This results in a linear, multi-page-like workflow where the time slots appear in a separate section below the calendar.

    ```javascript
    // from js/frontend.js
    dateChange: function (dateString, calendar) {
        // ... AJAX call to get time slots ...
        var next_element = jQuery(calendar).parent().next('.step').children('.time');
        next_element.empty();
        // ... logic to append time slots into next_element ...
    }
    ```

-   **In `frontend-bootstrap.js` (Bootstrap Layout):** The script dynamically creates a new table row (`<tr>`) and injects it directly into the calendar widget's HTML, immediately below the row of the selected date. This creates an "inline" or "accordion-style" effect where the time slots are revealed within the calendar itself.

Apart from this key UI difference, the core logic for fetching options (`getNextOptions`), handling reservations (`appSelected`), and confirming bookings (`finalComformation`) is functionally identical to its bootstrap counterpart.

## Features Enabled

### Admin Menu

This file is purely for the front-end and does not add or modify any admin menus.

### User-Facing

This script enables the "Standard" theme for the `[easyappointments]` shortcode. It provides the complete interactive booking experience, including:
-   A step-by-step form for selecting Location, Service, and Worker.
-   A calendar for date selection.
-   A separate section for time slot selection.
-   Form validation and submission.
-   A final confirmation step with a booking overview.

## Extension Opportunities

The extension points are the same as `frontend-bootstrap.js`.

-   **Custom Events (Recommended):** The safest way to interact with the form is by listening for the custom browser events it fires:
    -   `easyappnewappointment`: Fired after a booking is successfully confirmed.
    -   `easyappslotselect`: Fired when a user clicks on a time slot.
    -   `ea-init:completed`: Fired after the plugin has finished initializing.

    ```javascript
    document.addEventListener('easyappnewappointment', function (e) {
        // This event contains no data in this version of the script
        console.log('A new appointment was booked!');
    });
    ```

-   **Risks & Limitations:**
    -   **Code Duplication:** The most significant architectural risk is the massive code duplication between `frontend.js` and `frontend-bootstrap.js`. This makes the plugin difficult to maintain, as any bug fix or feature enhancement must be manually applied to both files, doubling the effort and increasing the chance of inconsistencies.
    -   **Monolithic Design & Tight Coupling:** Like its counterpart, the file is a large, single plugin with logic that is tightly coupled to the HTML structure, making it fragile and difficult to modify safely.

## Next File Recommendations

The discovery of these two parallel front-end scripts makes understanding the server-side logic that chooses between them the highest priority.

1.  **`easy-appointments/src/frontend.php`**: This file is now the most critical one to analyze. It is the server-side controller that registers the `[easyappointments]` shortcode and, crucially, must contain the logic that determines whether to load `frontend.js` or `frontend-bootstrap.js` based on a plugin setting.
2.  **`easy-appointments/ea-blocks/ea-blocks.php`**: This remains a key file for understanding how the plugin integrates with the modern WordPress Block Editor, providing an alternative to the shortcode.
3.  **`easy-appointments/js/report.prod.js`**: Analyzing the JavaScript for the admin reporting page would provide a look into another distinct area of the plugin's functionality, showing how it handles data visualization.
