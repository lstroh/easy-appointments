# File Analysis: easy-appointments/src/uninstall.php

This document provides a detailed analysis of the `uninstall.php` file for the Easy Appointments plugin.

## High-Level Overview

`easy-appointments/src/uninstall.php` defines the `EAUninstallTools` class, a simple but critical utility responsible for cleaning up the plugin's data from the WordPress database. Its methods are designed to be called during two key lifecycle events:

1.  **Plugin Deletion:** When a user deletes the plugin from the WordPress dashboard, its methods are used to drop all custom tables and remove plugin-specific entries from the `wp_options` table.
2.  **Plugin Reset:** Its methods are also likely used by an internal "Reset" tool to truncate all data from the custom tables, returning the plugin to a clean slate without requiring reinstallation.

This class ensures that the plugin adheres to WordPress best practices by not leaving orphaned data, tables, or scheduled cron jobs in the user's database after deletion.

## Detailed Explanation

-   **Key Class:** `EAUninstallTools`
    -   This is a simple utility class with no constructor and only public methods. It relies on the global `$wpdb` object and core WordPress functions to perform its tasks.

-   **Key Functions & Database Interaction:**
    -   `drop_db()`: This is the most destructive method. It uses raw `$wpdb->query()` calls to execute `DROP TABLE IF EXISTS` for all 9 of the plugin's custom tables. This is called during the uninstallation process to completely remove the plugin's database footprint.
    -   `delete_db_version()`: This method uses the core `delete_option()` function to remove the `easy_app_db_version` key from the `wp_options` table, ensuring a fresh install if the plugin is ever re-added.
    -   `clear_database()`: This method is different from `drop_db()`. It uses `TRUNCATE TABLE` to empty all data from the plugin's tables while leaving the table structures intact. This is likely used for a "Reset" feature within the plugin's admin panel. It correctly disables and re-enables foreign key checks during the process.
    -   `clear_cron()`: This method uses the core `wp_clear_scheduled_hook()` function to remove any custom cron jobs scheduled by the plugin, ensuring no orphaned tasks are left behind.

## Features Enabled

This file is a backend lifecycle script and provides no visible features to users or administrators directly. Its role is to power essential cleanup operations.

### Admin Menu
-   This file has no impact on the admin menu.

### User-Facing
-   This file has no impact on the user-facing side of the site.

### Background Features
-   **Clean Uninstall:** This class enables the plugin to perform a clean uninstall. When a user deletes the plugin, the methods in this class ensure that all associated database tables, options, and cron jobs are properly removed.
-   **Plugin Reset:** The `clear_database()` method provides the logic for a "Reset to factory settings" tool, allowing an admin to clear all appointment data without deactivating the plugin.

## Extension Opportunities

-   **No Extension Points:** This class is not designed to be extended. It contains no WordPress actions or filters. Its purpose is highly specific and destructive, and altering its behavior is not recommended.
-   **Potential Risks:** The methods in this class are destructive by nature. If `drop_db()` or `clear_database()` were to be called accidentally, it would result in irreversible data loss for the plugin. The code is written safely (using `DROP TABLE IF EXISTS` to prevent errors), but its purpose is inherently dangerous and should only be invoked through the proper WordPress uninstall or plugin reset procedures.

## Next File Recommendations

Having analyzed all the primary PHP classes in the `src` directory, the next logical steps are to investigate the remaining specialized services and utilities to complete the analysis.

1.  **`easy-appointments/src/services/SlotsLogic.php`**: This is the most important remaining file for understanding the plugin's core functionality. As a dependency of `EALogic`, it contains highly specialized algorithms related to time slot generation and conflict resolution.
2.  **`easy-appointments/ea-blocks/ea-blocks.php`**: This file handles the plugin's integration with the modern WordPress block editor (Gutenberg). Analyzing it will reveal how the booking form is provided as a block, which is a key part of the modern WordPress user experience.
3.  **`easy-appointments/src/utils.php`**: This utility class has been referenced in other files for tasks like retrieving template paths. Understanding its methods is key to learning how to customize the plugin's front-end appearance through theme-based template overrides.
