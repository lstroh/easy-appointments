# File Analysis: easy-appointments/ea-blocks/build/ea-fullcalendar/frontend-script.js

**Note**: This is a compiled and minified build artifact. The analysis is based on patterns found in the code, as the original source code is not directly readable here. It is not intended for direct modification.

## High-Level Overview

This file is the core JavaScript engine for the user-facing "EA Full Calendar View" block. As the `viewScript` defined in the block's `block.json`, it is loaded on the front end of the website wherever the calendar block is present. Its primary responsibility is to render the interactive booking calendar, handle user actions (like selecting dates and times), and communicate with the WordPress backend via the REST API to fetch available appointment slots.

## Detailed Explanation

Although the code is minified, several key implementation details are evident:

-   **Core Technologies**: The script clearly utilizes several key libraries provided by WordPress and the plugin:
    -   **React**: It depends on `react`, `react-dom`, and `wp-element`, indicating the entire block is a React application rendered on the frontend.
    -   **FullCalendar.js**: The code explicitly sets up a `FullCalendarVDom` object. This confirms the plugin uses the popular and powerful FullCalendar.js library to render the calendar grid and handle date/time interactions.
    -   **`wp-api-fetch`**: The script's dependency on `wp-api-fetch` confirms it communicates with the WordPress backend using the standard REST API infrastructure.

-   **Functionality**:
    1.  **Initialization**: The script finds the placeholder `<div>` rendered by the block on the page and mounts a React component inside it.
    2.  **Calendar Rendering**: It initializes a FullCalendar instance, configured by the attributes passed from the block (e.g., `defaultDate`, `location`, `service`, `worker`).
    3.  **Data Fetching**: It uses `apiFetch` to make asynchronous requests to a custom Easy Appointments REST API endpoint to retrieve data for available appointment slots.
    4.  **User Interaction**: It handles user events from the FullCalendar instance, such as clicking on a day, navigating between months, or selecting an available time slot, and then proceeds with the booking workflow.

## Features Enabled

### Admin Menu

-   This file has **no impact** on the WordPress Admin Menu. It is a frontend-only script.

### User-Facing

-   This file is **exclusively** for the user-facing side and is the primary driver of the block's functionality.
-   It renders the fully interactive booking calendar.
-   It allows site visitors to browse availability and select appointment times.
-   It provides the dynamic experience of fetching data from the server without page reloads, showing loading states, and displaying available, booked, or unavailable slots.

## Extension Opportunities

**Direct modification of this file is strongly discouraged**, as it is a build artifact and any changes will be overwritten by the next build. Safe extension should happen via two primary methods:

1.  **Modify the Source File**: The correct way to change this file's behavior is to edit its source, which is likely located at `easy-appointments/ea-blocks/src/ea-fullcalendar/view.js` or a similarly named file in that directory. After making changes, you would run the plugin's build process (`npm run build` or similar) to regenerate this `frontend-script.js` file.

2.  **Use Custom JavaScript Events**: A well-designed frontend script often dispatches custom JavaScript events on the `window` or `document` object during key lifecycle moments (e.g., `ea-calendar-loaded`, `ea-slot-selected`). You could write your own separate JavaScript file to listen for these events and trigger custom functionality, allowing you to extend the calendar's behavior without modifying the plugin's core files. You would need to inspect the source code (`view.js`) to see if such events are available.

## Next File Recommendations

To truly understand how this interactive calendar works, you must look at its source code and the backend services that support it. The following files are not yet in `completed_files.txt` and are the most logical next steps:

1.  **`easy-appointments/ea-blocks/src/ea-fullcalendar/view.js`** (or `frontend.js` in the same `src` folder)
    -   **Reasoning**: This is the un-compiled, human-readable source code for the file we just analyzed. It will reveal the complete implementation of the React component, how FullCalendar is configured, how attributes are used, and exactly which REST API endpoints are being called. This is the most critical file for understanding the frontend logic.

2.  **`easy-appointments/ea-blocks/src/ea-fullcalendar/edit.js`** (or `index.js` in the same `src` folder)
    -   **Reasoning**: This is the source code for the block's appearance and functionality *within the WordPress editor*. It will show you how the block settings in the sidebar are created and how they connect to the block's attributes, providing a complete picture of the block's authoring experience.

3.  **`easy-appointments/src/services/SlotsLogic.php`**
    -   **Reasoning**: This frontend script is useless without a backend to provide it with data. The `SlotsLogic.php` file is the most likely candidate for the server-side business logic that calculates which appointment slots are available. Analyzing it will explain the rules engine of the entire booking system.
