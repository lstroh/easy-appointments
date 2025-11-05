# File Analysis: easy-appointments/js/report.prod.js

## High-Level Overview

`js/report.prod.js` is a self-contained Backbone.js application that powers the legacy "Reports *OLD*" page within the WordPress admin dashboard. Its purpose is to provide administrators with tools to view and export appointment data. The file creates an interactive, single-page experience, allowing users to generate reports without full page reloads.

Architecturally, this file follows the same pattern as `js/admin.prod.js`, using Backbone.js to structure the application and jQuery for DOM manipulation and AJAX. It communicates exclusively with the server via WordPress's `admin-ajax.php` API. It represents an older, but functional, part of the plugin's admin-side codebase, which exists alongside the more modern React-based UI found in `js/bundle.js`.

## Detailed Explanation

The script initializes a global `EA` object and defines several Backbone Views to manage the reporting interface. It also includes its own copy of the `Backbone.ajax` override for `PUT`/`DELETE` method spoofing, meaning it does not depend on `backbone.sync.fix.js`.

**Key Components:**

1.  **`EA.ReportView` (Main View):** This acts as the entry point and router for the reports page. It renders the initial screen with two options: "Overview" and "Excel Export." When a user clicks one, it hides the main menu and instantiates the appropriate sub-view (`EA.OverviewReportView` or `EA.ExcelReportView`).

2.  **`EA.OverviewReportView`:** This view provides a calendar-based report to visualize slot availability.
    -   It uses jQuery UI Datepicker to display a full monthly calendar.
    -   It includes dropdown filters for Location, Service, and Worker.
    -   When a filter is changed or the user navigates to a new month, the `selectChange` method fires an AJAX request to `admin-ajax.php` with `action=ea_report`.
    -   The `refreshData` callback then takes the returned data and injects it into the calendar's day cells, showing the number of free slots for each day.

3.  **`EA.ExcelReportView`:** This view provides the interface for downloading appointment data.
    -   It contains date input fields to specify a date range for the export.
    -   The "Download" button is a link that points to an `admin-ajax.php` URL with `action=ea_export`. The server-side handler for this action generates and serves the CSV file.
    -   It also includes a feature to customize the columns in the export. User preferences for these columns are saved via a separate AJAX call (`action=ea_save_custom_columns`).

```javascript
// Example: AJAX call in the Overview report to fetch monthly data
selectChange: function (month, year) {
    // ...
    if (this.checkStatus()) { // checks if all filters are selected
        // ... build 'fields' object from form inputs ...
        fields.push({'name': 'action', 'value': 'ea_report'});
        fields.push({'name': 'report', 'value': 'overview'});
        // ...

        jQuery.get(ajaxurl, fields, function (result) {
            self.refreshData(result);
        }, 'json');
    }
}
```

## Features Enabled

### Admin Menu

This file provides the entire client-side functionality for the **Appointments -> Reports *OLD*** admin page. The features include:

-   An interactive monthly calendar report showing slot availability based on Location, Service, and Worker.
-   A data export tool to download appointments as a CSV file.
-   Options to filter the export by date range and customize the data columns.

### User-Facing

This is an administrator-only script and has no user-facing features.

## Extension Opportunities

Extending this legacy report page is challenging and may not be advisable, as the plugin has a newer reporting interface.

-   **Adding a New Report Type:** The most significant extension would be to add a new report. This would involve:
    1.  Using JavaScript to inject a new "card" into the main report selection screen.
    2.  Writing a new Backbone View (`EA.MyCustomReportView`) that contains the logic and template for your report.
    3.  Modifying the `reportSelected` method in `EA.ReportView` (via prototype extension) to instantiate your new view when your card is clicked.
    4.  Adding the necessary `wp_ajax_` hooks in PHP to provide data to your new report.
-   **Risks & Limitations:**
    -   **Legacy Feature:** The "*OLD*" designation in the menu implies this feature may be deprecated or removed in the future. Any time invested in extending it could be wasted.
    -   **Backbone.js Complexity:** Requires a good understanding of Backbone.js to extend properly.
    -   **No Formal Hooks:** Like the other Backbone apps in this plugin, it lacks formal, documented extension points, making any customization fragile.

## Next File Recommendations

With the analysis of the legacy JavaScript applications complete, the next logical steps are to investigate their server-side counterparts and the plugin's modern integration points.

1.  **`easy-appointments/src/report.php`**: This is the direct server-side counterpart to `report.prod.js`. It will contain the PHP code that registers the "Reports *OLD*" admin page, enqueues the script, renders the Underscore.js templates, and, most importantly, houses the AJAX handlers (`wp_ajax_ea_report`, `wp_ajax_ea_export`) that this script calls.
2.  **`easy-appointments/src/frontend.php`**: This remains a top priority. It is the controller for the entire front-end booking experience, responsible for registering the shortcode and deciding which JavaScript theme (`frontend.js` or `frontend-bootstrap.js`) to load.
3.  **`easy-appointments/ea-blocks/ea-blocks.php`**: To complete the picture of how this plugin is used on the front-end, analyzing its Gutenberg block integration is essential. This file is the entry point for that functionality.
