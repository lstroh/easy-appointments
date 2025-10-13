# File Analysis: easy-appointments/main.php

This document provides a detailed analysis of the `main.php` file for the Easy Appointments plugin.

## High-Level Overview

`easy-appointments/main.php` is the primary entry point for the Easy Appointments WordPress plugin. It acts as the central orchestrator, responsible for bootstrapping the entire plugin.

Its key responsibilities include:
-   Defining essential constants for file paths and URLs.
-   Setting up the Composer autoloader for class dependencies.
-   Initializing the main `EasyAppointment` class, which manages the plugin's lifecycle.
-   Hooking into WordPress core actions for activation, deactivation, and uninstallation.
-   Setting up a Dependency Injection (DI) container to manage and provide all core plugin services (like database models, logic handlers, and admin panels).
-   Conditionally loading components for either the WordPress admin area or the user-facing front end.

Architecturally, this file establishes a modular and service-oriented pattern by using a DI container, which promotes separation of concerns and makes the plugin more maintainable and extensible.

## Detailed Explanation

The file's logic is primarily contained within the `EasyAppointment` class.

-   **Key Class:** `EasyAppointment`
    -   The `init()` method is the engine of the plugin. It registers all necessary WordPress hooks and initializes the various components (admin, frontend, AJAX, mail, etc.).
    -   The `init_container()` method is the architectural core. It instantiates a `tad_EA52_Container` (a Dependency Injection container) and registers all the plugin's core classes as services. This allows any part of the plugin to access a shared instance of a service (e.g., the database model) through the container.

    ```php
    // Example of the DI container setup in init_container()
    $this->container = new tad_EA52_Container();
    $this->container['wpdb'] = $wpdb;
    $this->container['utils'] = new EAUtils();

    $this->container['options'] = function($container) {
        return new EAOptions($container['wpdb']);
    };

    $this->container['logic'] = function ($container) {
        return new EALogic($container['wpdb'], $container['db_models'], $container['options'], $container['slots_logic']);
    };
    ```

-   **Key Hooks & WordPress API Interaction:**
    -   **Lifecycle Hooks:**
        -   `register_activation_hook`: Calls the `install()` method to set up the database and schedule cron events.
        -   `register_deactivation_hook`: Calls `remove_scheduled_event()` to un-schedule cron jobs.
        -   `register_uninstall_hook`: Calls the static `uninstall()` method to completely remove plugin data (tables, options) and cron jobs.
    -   **Runtime Hooks:**
        -   `add_action('plugins_loaded', ...)`: Used to run the `update()` method, which handles database migrations and loads the plugin's text domain for translation.
        -   `add_action('init', ...)`: Listens for a special GET parameter (`_ea-action=clear_reservations`) for debugging purposes.
        -   `add_action('rest_api_init', ...)`: Registers the plugin's custom REST API endpoints by initializing the `EAMainApi` class.
    -   **Custom Cron Hooks:**
        -   `add_action('easyapp_hourly_event', ...)`: Triggers `delete_reservations()` to clean up incomplete bookings.
        -   `add_action('ea_gdpr_auto_delete', ...)`: Triggers `delete_old_data()` for GDPR compliance.

-   **Database Interaction:**
    -   This file does not interact with the database directly. It delegates all database operations to classes registered in the DI container, primarily `EADBModels`, `EAOptions`, and `EAInstallTools`.
    -   The `install()` and `uninstall()` methods confirm that the plugin creates and removes its own custom database tables.

## Features Enabled

### Admin Menu
-   This file is responsible for initializing the admin-side experience but does not create UI elements directly. It instantiates the `EAAdminPanel` class when `is_admin()` is true.
    ```php
    // In the init() method
    if (is_admin()) {
        /** @var EAAdminPanel $admin */
        $admin = $this->container['admin_panel'];
        $admin->init();
    }
    ```
-   The `EAAdminPanel` class (defined in `src/admin.php`) is responsible for creating all admin menus, settings pages, and meta boxes.
-   It also sets up the background cron jobs (`easyapp_hourly_event`, `ea_gdpr_auto_delete`) that run independently of user interaction.

### User-Facing
-   **Shortcodes, Blocks, and Scripts:** When on the front end (`!is_admin()`), the file initializes several key classes:
    -   `EAFrontend`: Manages the main front-end booking form, scripts, and styles.
    -   `EAFullCalendar`: Manages the display and logic for the full calendar view, likely implemented via a shortcode or block.
    -   `EAUserFieldMapper`: Maps appointment form fields to WordPress user profile fields.
-   **Gutenberg Blocks:** The line `require_once dirname(__FILE__) . '/ea-blocks/ea-blocks.php';` indicates that the plugin registers custom Gutenberg blocks for use in the block editor.
-   **REST Endpoints:** By hooking into `rest_api_init`, this file enables a custom REST API for the plugin, allowing modern JavaScript front ends or external applications to interact with the appointment system.

## Extension Opportunities

-   **DI Container:** The most powerful and intended way to extend the plugin is by interacting with the DI container. You could potentially replace a core service with your own custom implementation. However, there is no clean, built-in way to access the container from outside this file. A modification would be needed to expose it.
-   **WordPress Hooks:** You can use standard WordPress action and filter hooks to interact with the plugin's lifecycle. For example, you could hook into `plugins_loaded` with a later priority to run code after the plugin has been initialized.
-   **Potential Risks:**
    -   Directly modifying `main.php` is risky as any changes will be lost upon a plugin update.
    -   The plugin's core `EasyAppointment` class is instantiated directly in the global scope. This makes it difficult to remove its actions or decorate its functionality without modifying the file itself.
    -   The lack of a global accessor for the DI container (`$ea_app->get_container()`) means you cannot easily leverage the plugin's own services for your custom extensions. You would need to create your own instance of those services.

## Next File Recommendations

To gain a deeper understanding of the plugin's functionality, I recommend analyzing the following files next:

1.  **`easy-appointments/src/logic.php`**: This file likely contains the `EALogic` class, which appears to be the heart of the plugin's business logic for managing appointments, time slots, and availability. Understanding this file is critical to understanding how the core booking system works.
2.  **`easy-appointments/src/admin.php`**: This file contains the `EAAdminPanel` class. Analyzing it will reveal how the plugin creates its settings pages, manages locations, services, and workers, and what admin-specific hooks and filters are available.
3.  **`easy-appointments/src/frontend.php`**: This file contains the `EAFrontend` class, which is responsible for rendering the user-facing booking form. It will show you how shortcodes are implemented, how scripts and styles are enqueued, and how the booking process is handled from the user's perspective.
