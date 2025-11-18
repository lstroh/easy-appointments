# File Analysis: easy-appointments/ea-blocks/build/blocks-manifest.php

## High-Level Overview

This file is a generated PHP array that serves as a manifest for all Gutenberg blocks provided by the Easy Appointments plugin. It contains the metadata (properties, attributes, script/style dependencies) for each block, allowing the `ea-blocks.php` file to efficiently register them with WordPress. It is a crucial component for defining the structure and behavior of the plugin's custom blocks.

## Detailed Explanation

### Key Structure and Content

-   **PHP Array Return**: The file's sole purpose is to return a PHP associative array. The top-level keys of this array are the block names (e.g., `'ea-blocks'`, `'ea-fullcalendar'`).
-   **Block Metadata**: Each block's entry in the array is another associative array containing its metadata, which largely mirrors the structure of a `block.json` file:
    -   `$schema`, `apiVersion`, `name`, `version`, `title`, `category`, `icon`, `description`, `example`, `supports`, `keywords`, `textdomain`: These fields define the block's identity, how it appears in the WordPress editor, and its basic capabilities.
    -   `attributes`: This is a critical section that defines the configurable properties of the block. For example, both blocks define attributes like `width`, `scrollOff`, `layoutCols`, `location`, `service`, `worker`, and `defaultDate`. These attributes store the user's settings for the block.
    -   `editorScript`, `editorStyle`, `style`, `viewScript`: These fields specify the paths to the JavaScript and CSS assets that WordPress should enqueue for the block.
        -   `editorScript`: The JavaScript file loaded specifically for the block within the WordPress editor (e.g., `./index.js`).
        -   `editorStyle`: The CSS file loaded specifically for the block within the WordPress editor (e.g., `./index.css`).
        -   `style`: The CSS file loaded for the block in both the editor and on the frontend (e.g., `./style-index.css`).
        -   `viewScript`: The JavaScript file loaded for the block on the public-facing frontend of the website (e.g., `./view.js` for `ea-blocks`, `./frontend.js` for `ea-fullcalendar`).

### Interaction with WordPress Core or Database

-   This file itself does not directly interact with WordPress core APIs or the database. It is a static data structure.
-   Its content is consumed by `easy-appointments/ea-blocks/ea-blocks.php`, which then uses WordPress functions like `register_block_type` to register the blocks based on the metadata provided in this manifest.

### No Hooks or Classes

-   The file is purely a data definition and does not contain any PHP functions, classes, or WordPress action/filter hooks.

## Features Enabled

### Admin Menu

-   **Block Editor Integration**: This file enables the "EA Booking Form" and "EA Full Calendar View" blocks to appear in the WordPress Block Editor's inserter. The `title`, `category`, `icon`, and `description` fields dictate their presentation.
-   **Block Settings**: The `attributes` defined within this manifest determine the configurable options available to users in the block's settings sidebar within the editor.

### User-Facing

-   **Frontend Functionality**: The `viewScript` and `style` entries ensure that the necessary JavaScript and CSS are loaded on the frontend when these blocks are used on a page or post. This enables the interactive and visual aspects of the booking form and full calendar view for site visitors.

## Extension Opportunities

-   **Adding New Blocks**: To introduce a new custom block, you would typically create a new directory under `easy-appointments/ea-blocks/src/` for your block, define its `block.json` file (which is the source for this manifest), and implement its `edit.js`, `save.js`, and `view.js` (or `frontend.js`) files. The plugin's build process would then regenerate this `blocks-manifest.php` file to include your new block.
-   **Modifying Existing Block Metadata**: Changes to existing block properties (e.g., adding new attributes, updating titles, changing script dependencies) should be made in the respective source `block.json` files located in `easy-appointments/ea-blocks/src/`.
-   **No Direct Modification**: It is crucial to understand that this file is **generated**. Any manual changes made directly to `blocks-manifest.php` will be overwritten the next time the plugin's build process runs. Always modify the source `block.json` files and then run the build script.

## Next File Recommendations

1.  **`easy-appointments/ea-blocks/src/ea-fullcalendar/edit.js`** — This JavaScript file is responsible for rendering the "EA Full Calendar View" block within the WordPress editor. Analyzing it will reveal how the block's settings are presented to the user, how it interacts with the REST API endpoints (defined in `ea-blocks.php`) to fetch dynamic options (like locations, services, workers), and how it manages the block's attributes.
2.  **`easy-appointments/ea-blocks/src/ea-fullcalendar/frontend.js`** — This JavaScript file is the client-side application that powers the "EA Full Calendar View" block on the public-facing website. It's essential for understanding the user experience, how the calendar is displayed, how appointments are fetched from the `/ea_appointments/` REST endpoint, and any interactive booking logic.
3.  **`easy-appointments/ea-blocks/src/ea-blocks/edit.js`** — This file is the editor-side JavaScript for the "EA Booking Form" block. Similar to `ea-fullcalendar/edit.js`, it will show how the booking form's settings are managed in the WordPress editor and how it utilizes the plugin's REST APIs.
