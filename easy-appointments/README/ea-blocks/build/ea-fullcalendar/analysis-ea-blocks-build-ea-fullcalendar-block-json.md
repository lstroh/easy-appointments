# File Analysis: easy-appointments/ea-blocks/build/ea-fullcalendar/block.json

This is a static metadata file and does not contain executable code. Its purpose is to declare a WordPress Block Editor (Gutenberg) block.

## High-Level Overview

The `block.json` file is a standard WordPress convention for registering a block type. This specific file defines the **"EA Full Calendar View"** block. Its primary role is to make the Easy Appointments booking calendar available as a block that can be easily added to any post or page using the WordPress editor.

It acts as a manifest, telling WordPress about the block's properties, its settings (called attributes), and which JavaScript and CSS files to load for the editor experience and the user-facing view. It essentially bridges the plugin's core calendar functionality with the modern WordPress block editor.

## Detailed Explanation

This file contains a single JSON object with several key properties that WordPress uses to register and handle the block.

-   **`name`**: `"ea-blocks/ea-fullcalendar"`
    -   This is the unique machine-readable name for the block. It follows the standard `namespace/block-name` format.

-   **`title`, `category`, `icon`, `description`, `keywords`**:
    -   These properties define how the block appears in the block inserter UI within the editor, making it discoverable for content creators.

-   **`attributes`**:
    -   This is one of the most important sections. It defines the configurable settings for the block that a user can modify in the editor's sidebar. Each attribute has a defined type and a default value.
    -   `width`: Sets the container width.
    -   `scrollOff`: A boolean to disable scrolling the page on calendar navigation.
    -   `layoutCols`: Defines the column layout.
    -   `location`, `service`, `worker`: These allow the user to pre-filter the calendar to show availability for a specific location, service, or provider.
    -   `defaultDate`: Allows setting a specific start date for the calendar view.

-   **Script and Style Definitions**:
    -   `"editorScript": "file:./index.js"`: Specifies the JavaScript file to be loaded **only in the editor**. This script (typically a React component) renders the block's appearance and controls within the editor.
    -   `"editorStyle": "file:./index.css"`: The CSS for the block's appearance **in the editor**.
    -   `"style": "file:./style-index.css"`: The CSS for the block's appearance on both **the editor and the live site**.
    -   `"viewScript": "file:./frontend.js"`: Specifies the JavaScript file to be loaded when the block is viewed on the **user-facing side** of the website. This file contains the logic to make the calendar interactive.

This file does not interact with the database or use PHP hooks directly. It is purely declarative. The actual logic is handled by the JavaScript files it references and the server-side rendering function (likely found in `ea-blocks/ea-blocks.php`) that WordPress calls.

## Features Enabled

### Admin Menu

-   This file **does not** add any new admin menus, settings pages, or dashboard widgets.
-   Its sole function for administrators is to register the "EA Full Calendar View" block, making it available within the Block Editor when creating or editing posts and pages.

### User-Facing

-   Enables the display of the Easy Appointments booking calendar on any post or page where the block is used.
-   The `attributes` allow for the creation of multiple, uniquely configured calendar instances across the site without needing to remember and type out complex shortcode parameters. For example, you could have one page with a calendar pre-filtered for "Location A" and another for "Location B".

## Extension Opportunities

This file is the starting point for extending the block's functionality.

1.  **Adding New Settings**: The safest way to extend this block is to add a new custom setting.
    -   **Step 1**: Add a new key-value pair to the `attributes` object in this `block.json` file. For example, to add a setting for calendar color, you could add `"calendarColor": { "type": "string", "default": "#3a87ad" }`.
    -   **Step 2**: Modify the source file for the `editorScript` (likely `easy-appointments/ea-blocks/src/ea-fullcalendar/edit.js`) to add a UI control (e.g., a color picker) in the block's inspector controls (the sidebar) that updates this new attribute.
    -   **Step 3**: Modify the source file for the `viewScript` (likely `easy-appointments/ea-blocks/src/ea-fullcalendar/view.js` or `frontend.js`) to read the `calendarColor` attribute and apply it to the calendar instance.

2.  **Potential Risks**:
    -   The primary risk is not in this file but in how its attributes are used. Any server-side rendering logic must sanitize the attribute values (e.g., `location`, `service`, `worker`) before using them in database queries to prevent SQL injection. Likewise, any attributes rendered directly into the HTML must be escaped to prevent XSS attacks.

## Next File Recommendations

To continue your analysis, I recommend examining the files that implement the logic defined in this `block.json` file. The following files have **not** been analyzed yet (according to `completed_files.txt`) and are critical next steps:

1.  **`easy-appointments/ea-blocks/src/ea-fullcalendar/edit.js`** (or `index.js` in the same folder)
    -   **Reasoning**: This is the un-compiled source code for the `editorScript`. It contains the React component that defines the block's user interface within the WordPress editor. Analyzing this file will show you how the block's settings (attributes) are controlled and how it presents a preview to the content creator.

2.  **`easy-appointments/ea-blocks/src/ea-fullcalendar/view.js`** (or `frontend.js` in the same folder)
    -   **Reasoning**: This is the source code for the `viewScript`. It holds the frontend JavaScript that powers the interactive calendar that the site visitor uses. This is where the FullCalendar library is likely initialized and configured using the block's attributes to fetch and display appointments.

3.  **`easy-appointments/src/services/SlotsLogic.php`**
    -   **Reasoning**: The interactive calendar needs to fetch available time slots from the server. This PHP file is almost certainly the heart of the plugin's booking engine, containing the core business logic for calculating availability based on employee schedules, existing appointments, buffer times, and other constraints. Understanding this file is key to understanding how the entire booking system works.
