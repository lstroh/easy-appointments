# File Analysis: easy-appointments/ea-blocks/build/ea-fullcalendar/index.js

**Note**: This is a compiled and minified JavaScript file. It is a build artifact and is not intended for direct modification. This analysis is based on patterns observed in the compiled code.

## High-Level Overview

This `index.js` file is the `editorScript` for the "EA Full Calendar View" block. It contains all the JavaScript logic required to render and manage the block within the WordPress Block Editor (Gutenberg). Its responsibilities include registering the block, creating the settings panel in the editor's sidebar (the Inspector), and displaying a preview of the calendar within the editor's main content area.

## Detailed Explanation

Although minified, the code reveals its core functionality and dependencies. It is a standard, modern Gutenberg block implementation using React.

-   **Core Dependencies**: The script utilizes the standard suite of WordPress block editor packages, loaded from the `window` object:
    -   `wp.blocks`: For registering the block type (`registerBlockType`).
    -   `wp.i18n`: For making strings in the editor translatable (e.g., the labels for settings).
    -   `wp.blockEditor`: Provides key React components for the editor, such as `InspectorControls` (for the sidebar).
    -   `wp.components`: A library of UI elements (like `TextControl`, `ToggleControl`, etc.) used to build the settings panel.
    -   `wp.element`: The WordPress abstraction for React.

-   **Live Preview Implementation**: A key finding is that this script also initializes `FullCalendarVDom`, just like the frontend script. This indicates that instead of showing a static image or a server-rendered placeholder, the editor provides a **live, interactive preview** of the calendar. This gives the content creator a high-fidelity WYSIWYG experience.

-   **Settings Management**: The code uses components like `InspectorControls` to create the settings sidebar. It renders UI elements (e.g., text boxes for `width`, toggles for `scrollOff`, and likely dropdowns for `location`, `service`, and `worker`) that, when changed, update the block's attributes.

## Features Enabled

### Admin Menu

-   This file is **fundamental to the block editor experience**. It doesn't add a new admin menu page, but it:
    -   Registers the "EA Full Calendar View" block so it appears in the block inserter.
    -   Renders the block's settings panel in the editor sidebar.
    -   Displays a live preview of the calendar directly in the editor canvas.

### User-Facing

-   This script has **no impact** on the user-facing side of the website. It only runs within the WordPress admin dashboard when a user is editing a post or page.

## Extension Opportunities

As a compiled asset, **this file should not be edited directly**. Any manual changes will be erased during the next build.

1.  **Modify the Source File**: To customize the editor experience (e.g., add a new setting, change a label, alter the preview), you must edit the original, un-minified source file. This is almost certainly located at **`easy-appointments/ea-blocks/src/ea-fullcalendar/edit.js`** (or `index.js` in that `src` directory). After editing, you must run the plugin's build command (e.g., `npm run build`) to regenerate this `index.js` file.

2.  **Use WordPress JS Filters**: For more advanced, decoupled extensions, you can use WordPress's JavaScript filters. For example, the `blocks.registerBlockType` filter can be used to programmatically modify a block's settings *after* it has been registered by this script. This allows you to alter a block from your own custom plugin or theme's JavaScript without touching the original files.

## Next File Recommendations

To see the human-readable code behind the functionality observed here, you must analyze the source files. The following unreviewed files (checked against `completed_files.txt`) are the critical next steps:

1.  **`easy-appointments/ea-blocks/src/ea-fullcalendar/edit.js`** (or `index.js` in the `src` folder)
    -   **Reasoning**: This is the un-compiled source code for the file we just analyzed. It contains the React components that define the editor UI, the settings controls, and the live preview. It is the most important file for understanding the block's authoring experience.

2.  **`easy-appointments/ea-blocks/src/ea-fullcalendar/view.js`** (or `frontend.js` in the `src` folder)
    -   **Reasoning**: This is the source code for the user-facing part of the block. Analyzing it will show how the live calendar is implemented for site visitors, complementing the editor analysis.

3.  **`easy-appointments/src/services/SlotsLogic.php`**
    -   **Reasoning**: The live preview in the editor likely needs to fetch real (or sample) availability data to be useful. This PHP file contains the backend business logic for calculating appointment availability, making it a crucial component supporting both the editor preview and the final frontend calendar.
