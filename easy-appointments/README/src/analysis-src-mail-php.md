# File Analysis: easy-appointments/src/mail.php

This document provides a detailed analysis of the `mail.php` file for the Easy Appointments plugin.

## High-Level Overview

`easy-appointments/src/mail.php` defines the `EAMail` class, which serves as the plugin's centralized notification system. This class manages all aspects of email communication, from constructing and sending notifications to processing user actions initiated from links within those emails.

Its core responsibilities include:
-   Sending status-based emails (e.g., pending, confirmed, canceled) to customers, administrators, and employees.
-   Generating secure "magic links" that allow users to confirm or cancel their appointments directly from their email client.
-   Parsing incoming web requests to handle these confirmation/cancellation actions.
-   Populating email templates with dynamic appointment data using a placeholder system (e.g., `#id#`, `#service_name#`).
-   Logging email sending errors to the database for later review.

Architecturally, `EAMail` is a high-level service that is triggered by actions elsewhere in the plugin (primarily in `EAAjax` after a booking is made or updated) to handle the crucial communication aspect of the booking lifecycle.

## Detailed Explanation

-   **Key Class:** `EAMail`
    -   The constructor uses dependency injection to receive all the major plugin services it needs to function, including `EADBModels` (to fetch appointment data) and `EAOptions` (to get email templates and settings).

-   **Key Functions & WordPress API Interaction:**
    -   **Sending Logic:**
        -   `send_notification(...)`: Sends emails to administrators and/or workers. It retrieves the recipient list from the plugin settings.
        -   `send_status_change_mail(...)`: Sends emails to the customer. It intelligently selects the correct email template from the plugin settings based on the appointment's current status (e.g., `mail.pending`, `mail.confirmed`).
        -   `send_email(...)`: A protected wrapper around the core `wp_mail()` function. It cleverly hooks into `wp_mail_failed` right before sending and removes the hook immediately after, allowing it to log errors for its own operations without interfering with other plugins.
    -   **Link Handling Logic:**
        -   `parse_mail_link()`: Hooked into the `wp` action, this method runs on every front-end page load to check for the plugin's unique URL parameters (`_ea-action`, `_ea-app`, `_ea-t`).
        -   `generate_token(...)`: Creates a unique MD5 hash for each action link based on a salt and the appointment's creation timestamp. This ensures that links are unique and cannot be easily guessed.
        -   When a valid link is detected, `parse_mail_link` validates the token, updates the appointment status in the database (via `EADBModels`), and displays a success or error message to the user.
    -   **Template System:**
        -   The class uses a simple but effective `#placeholder#` replacement system. Methods like `send_notification` fetch appointment data, format it, and then use `str_replace` to inject the dynamic values into the email subject and body templates retrieved from the database.

## Features Enabled

This file is a backend service and does not create any direct UI elements. However, it powers critical user-facing functionality.

### Admin Menu
-   This file has no direct impact on the admin menu. It does, however, provide the logic for the "Test Mail" feature and is responsible for populating the `ea_error_logs` table, which is likely displayed in the "Tools" section of the admin panel.

### User-Facing
-   **Email Notifications:** This class is responsible for every email a user or administrator receives about an appointment.
-   **One-Click Actions:** It enables the "magic link" functionality, allowing users to confirm or cancel their appointments directly from an email without needing to log in to the website. This is a significant feature for user convenience.

## Extension Opportunities

This class is rich with filters, making it one of the most extensible parts of the plugin.

-   **Customizing Email Content:**
    -   `apply_filters('ea_customer_mail_template', $body_template, ...)`: Allows you to completely change the body of the email sent to a customer, even on a per-appointment basis.
    -   `apply_filters('ea_admin_mail_template', $body_template)`: Allows for the same customization for admin emails.
    -   `apply_filters('ea_customer_mail_subject_template', $subject_template)`: Allows for customizing the customer email subject line.
-   **Adding Recipients or Attachments:**
    -   `apply_filters('ea_admin_mail_address_list', $emails, ...)`: Easily add CC or BCC recipients to admin notifications.
    -   `apply_filters('ea_admin_mail_attachments', $files, ...)` and `apply_filters('ea_user_mail_attachments', $files, ...)`: Programmatically add file attachments (like PDF tickets or invoices) to outgoing emails.
-   **Custom Redirects:**
    -   `apply_filters('ea_confirmed_redirect_url', get_home_url())` and `apply_filters('ea_cancel_redirect_url', get_home_url())`: Change the URL where a user is sent after they click a confirmation or cancellation link. This is useful for directing users to custom "Thank You" or "Sorry to see you go" pages.
-   **Potential Risks:** The token generation for the action links uses MD5, which is a cryptographically weak hashing algorithm. While the risk in this context is low (it mainly prevents accidental clicks or non-targeted abuse), using a more modern approach like a salted HMAC-SHA256 would be best practice.

## Next File Recommendations

Having analyzed the core PHP components of the plugin, the following files are the most logical next steps to complete the analysis:

1.  **`easy-appointments/src/services/SlotsLogic.php`**: The `EALogic` class has a dependency on `EASlotsLogic`. This class was likely created to offload some of the complexity of slot calculation from the main logic file. Analyzing it is essential to fully understand the availability algorithm.
2.  **`easy-appointments/ea-blocks/ea-blocks.php`**: This file is responsible for integrating the plugin with the modern WordPress block editor (Gutenberg). Understanding how the booking form is made available as a block is key to understanding the full scope of the plugin's front-end capabilities.
3.  **`easy-appointments/src/utils.php`**: This utility class has been referenced for handling template paths. Analyzing it will clarify how template overrides work and reveal any other helper functions that might be useful for developers.
