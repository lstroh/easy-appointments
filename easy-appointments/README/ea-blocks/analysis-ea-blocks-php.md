# File Analysis: easy-appointments/ea-blocks/ea-blocks.php

## High-Level Overview

This file serves as the primary integration point between the Easy Appointments plugin and the modern WordPress Block Editor (Gutenberg). Its main purpose is to register, render, and provide data for custom blocks, allowing administrators to embed complex booking functionalities (like a full calendar view) directly into posts and pages.

It acts as a backend-for-the-frontend, creating a series of REST API endpoints that the blocks' JavaScript components consume to fetch data (like locations, services, and appointments) and to power their interactive settings in the editor.

## Detailed Explanation

### Key Functions, Classes, and Hooks

-   **`create_block_ea_blocks_block_init()`**: Hooked into `init`, this is the main registration function. It uses `register_block_type` to register all blocks defined in the `/build` directory, guided by a manifest file. For the `ea-fullcalendar` block, it specifies a custom server-side render callback.
-   **`render_ea_fullcalendar_block( $attributes )`**: This is a server-side render (SSR) function for the Full Calendar block. Instead of rendering the full calendar in PHP, it creates a placeholder `<div>` and enqueues the necessary JavaScript (`frontend.js`) and CSS. It passes block attributes (like `location`, `service`, `worker`) to the script using `wp_add_inline_script`, effectively bootstrapping a client-side rendered application.
-   **`add_action('rest_api_init', ...)`**: This hook is used multiple times to register several custom REST endpoints under the `wp/v2/eablocks` namespace.
-   **`ea_blocks_get_options(WP_REST_Request $request)`**: This function powers the block editor's dropdowns. It dynamically fetches locations, services, or staff from the database based on the `type` parameter and the other selected filters.
-   **`get_ea_appointments(WP_REST_Request $request)`**: This public-facing endpoint fetches and returns appointment data, which is then used by the frontend calendar to display booked slots. It uses a helper function, `get_all_appointments`, to perform the database query.
-   **`get_all_appointments($data)`**: A robust function that uses `$wpdb->prepare` to safely query the `ea_appointments` table. It's capable of filtering by location, service, and worker, and it also fetches and attaches custom field data to the results.

### Database Interaction

The file directly interacts with several custom database tables:
-   `ea_appointments`: To fetch appointment details.
-   `ea_fields` & `ea_meta_fields`: To retrieve custom field values for appointments.
-   `ea_locations`, `ea_services`, `ea_staff`: To populate the block's configuration options.
-   `ea_connections`: A crucial pivot table used to determine valid combinations of locations, services, and workers.

## Features Enabled

### Admin Menu

-   **Gutenberg Blocks**: The file registers all blocks necessary for the plugin to be used in the Block Editor. The primary one discussed is `ea-fullcalendar`.
-   **Interactive Block Controls**: Within the editor's sidebar, this file populates the dropdowns for selecting Location, Service, and Worker. When an admin changes one selection (e.g., picks a location), an API call is made to this file's endpoints to refresh the options in the other dropdowns based on available connections.

### User-Facing

-   **Full Calendar Booking System**: The `ea-fullcalendar` block renders a complete, interactive booking calendar on any page or post where it's placed. This is a client-side React application that gets its data from the REST endpoints defined in this file.
-   **REST Endpoints**:
    -   `GET /wp/v2/eablocks/get_ea_options/`: Used by the block editor.
    -   `GET /wp/v2/eablocks/ea_appointments/`: Used by the frontend calendar to display existing appointments.
    -   `POST /wp/v2/eablocks/render_shortcode`: An endpoint for rendering shortcode previews, likely for other blocks.

## Extension Opportunities

-   **Adding New Blocks**: You can create new blocks by adding them to the `src` directory and running the project's build process. The `create_block_ea_blocks_block_init` function is designed to automatically register any new blocks found in the manifest.
-   **Extending REST API**: You could add more filters or data to the existing REST endpoints. For example, you could modify `get_all_appointments` to accept a date range for filtering appointments.
-   **Custom Actions/Filters**: This file does not expose its own custom WordPress actions or filters. Extension is primarily achieved by interacting with the REST APIs or by creating new blocks that use them.
-   **Potential Risks & Improvements**:
    -   **Security**: The `get_ea_options` and `get_ea_appointments` endpoints have `permission_callback` set to `__return_true`, making them publicly accessible. This could leak information about business operations (locations, staff, booked times). These should be reviewed and potentially restricted, for example, to authenticated users or by using nonces.
    -   **SQL Injection**: The queries in `ea_blocks_get_options` use string concatenation to build the SQL query. While there is a check for `is_numeric`, it would be significantly safer to rewrite these queries using `$wpdb->prepare` to prevent any potential SQL injection vulnerabilities.

## Next File Recommendations

1.  **`easy-appointments/ea-blocks/build/blocks-manifest.php`** — This file is auto-generated and contains a manifest of all blocks that `ea-blocks.php` will register. Reviewing it will provide a complete inventory of the available blocks and their properties, giving a full picture of the scope of the Gutenberg integration.
2.  **`easy-appointments/ea-blocks/src/ea-fullcalendar/edit.js`** — Since `ea-blocks.php` provides the backend for the blocks, the next logical step is to see how the block editor UI is constructed. This file controls what the admin sees when adding or configuring the Full Calendar block. It contains the logic for the settings sidebar and consumes the `/get_ea_options/` REST endpoint.
3.  **`easy-appointments/ea-blocks/src/ea-fullcalendar/frontend.js`** — This is the client-side application that gets rendered on the public-facing website. Analyzing this file is critical to understanding the user's booking experience, how the calendar is rendered, and how it interacts with the `/ea_appointments/` endpoint to display availability.
