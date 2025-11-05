# File Analysis: easy-appointments/js/frontend-bootstrap.js

## High-Level Overview

`frontend-bootstrap.js` is the client-side engine that powers the entire user-facing booking form. It is structured as a large, stateful jQuery plugin named `eaBootstrap`. This file is responsible for rendering the booking widget, managing the step-by-step user workflow, dynamically fetching availability via AJAX, and handling the final form submission.

Architecturally, this script represents a traditional, yet powerful, approach to building interactive web components. It is tightly integrated with its server-side PHP counterpart, which provides the initial HTML templates and settings. All dynamic data and actions are handled through AJAX calls to WordPress's `admin-ajax.php` API. This single file contains the complete logic for the end-user's booking journey.

## Detailed Explanation

The script is instantiated on the `.ea-bootstrap` element, which acts as the container for the entire booking widget. It relies on several global objects and libraries.

**Dependencies:**
-   **jQuery & jQuery UI:** Used for all DOM manipulation, event handling, and the datepicker widget.
-   **jQuery Validate:** Used for client-side validation of the final submission form.
-   **Moment.js:** Used for parsing and formatting dates and times.
-   **Underscore.js:** Used for its templating engine (`_.template`) to render the main form and booking overview from HTML `<script>` tags in the DOM.
-   **Global Objects:** It heavily depends on `ea_settings` (for configuration), `ea_vacations` (for blocking dates), and `ea_ajaxurl` (for server communication).

**Key Logic & Workflow:**

1.  **Initialization (`init`):** The plugin compiles the Underscore templates, sets up the datepicker with custom logic in `beforeShowDay` to block vacation days, and attaches change event listeners to the form's `<select>` elements.
2.  **Step-by-Step Navigation:** The core user experience is a multi-step form (Location -> Service -> Worker -> Date -> Time). The `getNextOptions` method is triggered on each selection. It makes an AJAX call (`action=ea_next_step`) to fetch the relevant choices for the subsequent step.
3.  **Date & Time Selection:** When a date is selected, the `dateChange` method fires an AJAX call (`action=ea_date_selected`) to get all available time slots for that day. When a time slot is clicked, `selectTimes` handles the logic for selecting a block of time appropriate for the service duration.
4.  **Booking & Confirmation:** The process can be one-step or two-step, based on the `pre.reservation` setting:
    -   **Two-Step:** A preliminary reservation is made (`action=ea_res_appointment`), and the user is shown a final confirmation step. The booking is only finalized when the user submits their personal details (`action=ea_final_appointment`).
    -   **One-Step:** The entire form is submitted at once, creating the appointment and user details in a single AJAX request.
5.  **Custom Events:** The script dispatches custom browser events (`ea-init:completed`, `ea-timeslot:selected`, `easyappnewappointment`) at key moments, providing clean hooks for other scripts to interact with the booking process.

```javascript
// Example of a core AJAX call to get the next step's options
this.callServer = function (options, next_element) {
    var plugin = this;
    options.action = 'ea_next_step';
    // ...

    this.placeLoader(next_element.parent());

    var req = jQuery.get(ea_ajaxurl, options, function (response) {
        // ... logic to populate the next <select> dropdown
    }, 'json');
    // ...
};
```

## Features Enabled

### Admin Menu

This file has no role in the WordPress admin menu; it is a front-end only script.

### User-Facing

This file is the primary driver of the plugin's main user-facing feature: the booking form. It enables:
-   A complete, step-by-step appointment booking process.
-   Dynamic filtering of services, workers, and available time slots.
-   An interactive calendar that visually displays availability.
-   Client-side form validation to prevent incorrect submissions.
-   A booking overview and confirmation step.
-   Support for custom form fields created in the admin panel.
-   Custom redirects after a successful booking.
-   A customer search feature (if enabled) to pre-fill form data.
-   Custom recurrence options for repeat bookings.

## Extension Opportunities

-   **Custom Events (Recommended):** The safest and most decoupled way to extend the booking form is to listen for the custom events it dispatches to the `document`.
    ```javascript
    // Listen for the final appointment confirmation
    document.addEventListener('easyappnewappointment', function (e) {
        console.log('New appointment was booked:', e.detail);
        // Trigger analytics, show a custom popup, etc.
    });

    // Listen for when a time slot is selected
    document.addEventListener('ea-timeslot:selected', function (e) {
        console.log('A time slot was selected.');
    });
    ```
-   **jQuery Plugin Access:** It is technically possible to access the plugin instance and its methods, but this creates a tighter coupling and is more likely to break with updates.

-   **Risks & Limitations:**
    -   **Monolithic Design:** The file is very large and handles a wide range of functionality, making it difficult to navigate and maintain.
    -   **Legacy AJAX:** The reliance on `admin-ajax.php` for all server communication is less performant than the modern REST API, as each call has to load a significant portion of the WordPress admin environment.
    -   **Tight Coupling:** The JavaScript logic is tightly coupled to the specific HTML structure and class names rendered by the server-side PHP. Changes to the templates could easily break the script.

## Next File Recommendations

Analyzing the front-end logic in this file leads directly to its server-side counterparts and other key integration points.

1.  **`easy-appointments/src/frontend.php`**: This is the most critical file to analyze next. It is the server-side controller for the booking form. It registers the `[easyappointments]` shortcode, enqueues this `frontend-bootstrap.js` script, and renders the initial HTML and Underscore.js templates that this script depends on.
2.  **`easy-appointments/ea-blocks/ea-blocks.php`**: This file handles the integration with the Gutenberg Block Editor. It's essential for understanding how the booking form is made available as a native WordPress block, which is a core feature for modern site building.
3.  **`easy-appointments/js/report.prod.js`**: This script likely powers the admin reporting page. Analyzing it would provide a good contrast to `frontend-bootstrap.js`, showing how the plugin handles data visualization and reporting in the admin area.
