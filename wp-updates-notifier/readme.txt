=== WP Updates Notifier ===
Contributors: l3rady
Donate link: https://wpun.cariss.dev/#support-the-project
Tags: updates, notifications, email, monitor
Requires at least: 6.7
Tested up to: 7.1
Requires PHP: 7.4
Stable tag: 2.0.0
License: GPL-2.0-or-later
License URI: https://www.gnu.org/licenses/gpl-2.0.html

Get an email when your WordPress site has core, plugin or theme updates waiting.

== Description ==

WP Updates Notifier checks your site on a schedule and emails you when WordPress core, a plugin or a theme has an update waiting, so you know without logging in. It is made for sites you rarely visit and for sites you look after for clients.

Each new version is reported once, not on every check. Version 2.0.0 is a ground-up rewrite of a plugin first released in 2011: the same simple promise, with the testing, accessibility and security work a plugin on this many sites deserves.

= What it does =

* Checks for core, plugin and theme updates hourly, twice daily, daily or weekly.
* Tells you about each new version once, with optional daily or weekly reminders about updates you haven't installed yet.
* Sends email to one or more addresses, with a test send to check it arrives.
* Lets you set up more than one email channel, each with its own recipients, its own choice of updates and its own reminders.
* Can watch only the plugins and themes you have active.
* Runs a check on demand with **Check for updates and notify now**.
* Hides the WordPress update notice from users who cannot update WordPress.
* Takes its channels and settings from code if you like: `wp-config.php` constants, environment variables or your own configuration source, so a fleet of sites can share one setup.
* Comes in 11 languages besides English: Chinese (Simplified), Dutch, French, German, Hebrew, Italian, Japanese, Polish, Portuguese (Brazil), Russian and Spanish.

= Built for developers =

* A public, versioned API lets add-ons bring their own notification channels, item types, routing rules and configuration sources. See the [developer guide](https://wpun.cariss.dev/latest/developers/).
* Every action and filter is documented in the [hook reference](https://wpun.cariss.dev/latest/developers/hooks/).

= Accessible and security conscious =

* The settings screen is checked against WCAG 2.2 AA with axe and keyboard tests on every change.
* A published [threat model](https://wpun.cariss.dev/latest/security/threat-model/), security tests for every request handler, static and taint analysis on every change, and signed releases with build provenance you can verify.

= Free and open source =

GPL-2.0-or-later and developed in the open on [GitHub](https://github.com/l3rady/wp-updates-notifier). No tracking, no analytics and no account to sign up for.

The full documentation, including the [user guide](https://wpun.cariss.dev/latest/users/), is at [wpun.cariss.dev](https://wpun.cariss.dev/).

= Requirements =

WordPress 6.7 or newer and PHP 7.4 or newer, on a single site. Version 2.0.0 does not support multisite networks.

== Installation ==

1. In WordPress, go to **Plugins → Add New**, search for "WP Updates Notifier", click **Install Now**, then **Activate**.
2. Go to **Settings → Updates Notifier**. The first check runs on one of the next page loads of your site, and emails go to the administration email address in **Settings → General**.
3. On the **Channels** tab, click **Send test** under **Email** to check that the email arrives. Click **Edit** to change where it goes.

You can also install it with Composer through [WordPress Packagist](https://wpackagist.org/) (`composer require wpackagist-plugin/wp-updates-notifier`), or upload the zip attached to each [GitHub release](https://github.com/l3rady/wp-updates-notifier/releases).

The [getting started guide](https://wpun.cariss.dev/latest/users/getting-started/) walks through the rest.

== Frequently Asked Questions ==

= Why did I not get an email about an update? =

Check these in turn:

1. **Was it already reported?** Each channel is told about each version once. See [How notifications work](https://wpun.cariss.dev/latest/users/how-notifications-work/).
2. **Is the channel on?** On the **Channels** tab, **Enabled** must say **On**.
3. **Does a setting leave it out?** On the **General** tab, the setting for that kind of update must not be **No**, and if it is limited to active plugins or themes, the plugin or theme must be active. Check the channel's **Routing** column too.
4. **Did the send fail?** **Last delivery** shows **Failed** and the reason.
5. **Has the check run?** **Last checked** on the **General** tab says when. If it is old, see the next entry.
6. **Is the data old?** See [Why is the update data hours old?](https://wpun.cariss.dev/latest/users/troubleshooting/#why-is-the-update-data-hours-old).

= Emails only arrive after someone visits the site =

The schedule is run by WP-Cron. WP-Cron is not a real clock. WordPress only looks for due tasks when a page is requested, so on a site that few people visit, a check due at 03:00 can wait until someone arrives at 09:00. A forum user put it as "I only receive emails after someone has logged in".

To fix it, run WP-Cron from the server's own clock. See [How do I run the check from a system cron?](https://wpun.cariss.dev/latest/users/troubleshooting/#how-do-i-run-the-check-from-a-system-cron).

= How do I run the check from a system cron? =

There are two ways. Use the first unless you have a reason not to.

**Request wp-cron.php**

Stop WordPress starting WP-Cron on page loads by adding this to `wp-config.php`:

`
define( 'DISABLE_WP_CRON', true );
`

Then have the server request `wp-cron.php` on a regular schedule, for example every 15 minutes:

`
*/15 * * * * curl -fsS https://example.com/wp-cron.php > /dev/null
`

WP-Cron then runs the plugin's check at the **Frequency to check** you chose, and WordPress's own update tasks too.

**Fire the check yourself**

The plugin's check can also be started from code with the `sc_wpun_run_check` action. For example, with WP-CLI:

`
*/30 * * * * cd /path/to/site && wp eval "do_action( 'sc_wpun_run_check' );"
`

This runs the same check as the schedule, and anything already reported is not sent again. It does not move the WP-Cron schedule, so **Next check** on the **General** tab does not change. If you rely on this alone, the time of your cron entry is the schedule.

Things to know:

* The plugin does not lock checks against each other. If this overlaps another run, the same update can be sent twice. This is a known trade-off, and rare. See [How notifications work](https://wpun.cariss.dev/latest/users/how-notifications-work/#overlapping-checks).
* A check started this way is not a WP-Cron run, so WordPress lets plugin and theme data get up to 12 hours old before it refreshes it. A check started by `wp-cron.php` allows 2 hours. See [Why is the update data hours old?](https://wpun.cariss.dev/latest/users/troubleshooting/#why-is-the-update-data-hours-old).

= Why is the update data hours old? =

On every check the plugin asks WordPress to refresh what it knows about updates. WordPress asks api.wordpress.org, but it does not do so every time. It skips the request when it has asked recently:

* The schedule, or `wp-cron.php` - Core updates: 1 minute; Plugins and themes: 2 hours
* **Check for updates and notify now**, or `wp eval` - Core updates: 1 minute; Plugins and themes: 12 hours

So a scheduled check can see plugin and theme data up to two hours old, and a manual check up to twelve. WordPress also asks again early if the plugins or themes you have installed, or their versions, have changed.

To get current data before a manual check, open **Dashboard → Updates** in WordPress first. That page refreshes the data if it is more than a minute old.

**Note:** These limits belong to WordPress, not to this plugin. They keep your site from calling api.wordpress.org too often.

= Why does the email not arrive? =

**Send test** on the **Channels** tab says "Test message sent" when WordPress has handed the email to your mail system. That does not mean the message was delivered. Check the spam folder first, then look at how your site sends mail.

* **The From address.** With **From address** empty, the email uses the WordPress default, which is `wordpress@` followed by your site's domain. If you set one, it must be at a domain your server is allowed to send mail for. Mail that claims to come from a domain the server has no permission for is often rejected or sent to spam.
* **SPF, DKIM and DMARC.** These DNS records tell mail providers which servers may send for your domain. If your domain publishes a strict DMARC policy and the From address does not line up with the server that sends, providers can reject the message. Ask your host or mail provider for the right records.
* **The server's mail.** Many hosts block or throttle the basic mail function WordPress uses. Install an SMTP plugin, or a plugin for a mail service, so WordPress sends through an authenticated mail server. Then send a test again.
* **Upgraded from 1.x?** Versions before 2.0.0 sent from the administration email address unless you set another. If your From address was that same address, the upgrade cleared it, so mail goes from the WordPress default. Set **From address** if your mail setup needs one.

If WordPress itself fails to send, the reason appears in the notice after the test and under **Last delivery**.

= Why does a message show the wrong address? =

The address in a message is your site's home address, the **Site Address (URL)** in **Settings → General**. The link to the **Dashboard → Updates** screen is built from the **WordPress Address (URL)**.

Both can be wrong if `wp-config.php` sets `WP_HOME` or `WP_SITEURL` from the request, for example from `$_SERVER['HTTP_HOST']`. The check then runs in a request from a system cron or a visitor, and the address follows whichever host that request used. Set `WP_HOME` and `WP_SITEURL` to fixed addresses, or remove them and use the settings in **Settings → General**.

= Why did Slack notifications stop? =

WP Updates Notifier 2.0.0 does not send Slack notifications, and your Slack settings are kept. If a Slack channel was switched on, the plugin tells you once by email, shows a notice on its settings page that you can dismiss, and adds an item to **Tools → Site Health**. The notice and the item stay while a Slack channel is switched on.

= Will I be told about every version? =

Each channel is told about each new version once. If you don't install it, nothing more is sent unless you switch on daily or weekly reminders for that channel. See [How notifications work](https://wpun.cariss.dev/latest/users/how-notifications-work/).

= Does it work on multisite? =

Not yet. Version 2.0.0 supports single sites only.

= How do I send WordPress's automatic update emails to another address? =

Version 2.0.0 leaves WordPress's own automatic update emails alone, so they go to the admin email under Settings > General. Change that address, or set the recipient with the `auto_core_update_email` filter:

`add_filter( 'auto_core_update_email', function ( $email ) { $email['to'] = 'ops@example.org'; return $email; } );`

The `auto_plugin_theme_update_email` filter does the same for plugin and theme automatic updates.

= How do I report a security problem? =

Please don't post it in the support forum or in a public GitHub issue. Report it privately through GitHub: on the [repository's Security tab](https://github.com/l3rady/wp-updates-notifier/security), choose "Report a vulnerability". The [security policy](https://github.com/l3rady/wp-updates-notifier/blob/main/SECURITY.md) has the details, and an email address if you can't use GitHub.

== Screenshots ==

1. The General tab: when the last and next check run, a button to check now, and the settings.
2. The Channels tab: each channel with what it receives and when it last delivered.
3. Editing an email channel; a value set in code is shown as read-only.
4. An update email listing the updates that are waiting.

== Changelog ==

= 2.0.0 =

**Breaking changes**

* WordPress's core auto-update emails now always go to the site admin email. Sites that sent them to another address must change the admin email or use the auto_core_update_email filter. The notify_automatic setting, its SC_WPUN_SETTING_NOTIFY_AUTOMATIC constant and the sc_wpun_default_settings key are no longer read.
* removed Settings_Page methods sc_wpun_settings_email_notifications_field_email_notifications(), sc_wpun_settings_email_notifications_field_notify_to(), sc_wpun_settings_email_notifications_field_notify_from(), sc_wpun_settings_email_notifications_field_notify_reply_to(), email_enabled_schema(), email_read_only(), email_note(), instance_setting(), instance_enabled(), save_instance() and value_attribute(), and the SC_WP_Updates_Notifier wrappers sc_wpun_settings_email_notifications_text(), sc_wpun_settings_email_notifications_field_email_notifications(), sc_wpun_settings_email_notifications_field_notify_to() and sc_wpun_settings_email_notifications_field_notify_from(). The keys notify_to, notify_from, notify_reply_to and email_notifications in a sc_wpun_settings save are ignored: they are not migrated or forwarded.
* Channel::validate_settings() takes a second parameter, array $stored = array(), the instance's stored settings. A channel that overrides it must declare the same signature.
* the sc_wpun_settings option stores notify_automatic and hide_updates as booleans instead of 1/0, and the sc_wpun_default_settings filter documents them as bool. Code reading the option directly sees true/false. A settings save that leaves out notify_automatic, hide_updates or email_notifications no longer turns them off.
* the default Channel::validate_settings() now rejects a `checkbox` or `boolean` field whose value is not true/false, 1/0, '1'/'0' or '', returning a WP_Error naming the field, and stores the value as a bool.
* the sc_wpun_settings option stores notify_plugins and notify_themes as 'none', 'all' or 'active' instead of 0, 1 and 2, and the sc_wpun_default_settings filter and SC_WPUN_SETTING_NOTIFY_PLUGINS / SC_WPUN_SETTING_NOTIFY_THEMES take only those keys. Code that reads the option directly must compare the keys.
* the main file is now wp-updates-notifier.php. Sites updating from 1.5 or 1.6 must activate the plugin again; settings are kept.
* channel instance IDs may no longer contain "__" or start or end with "_", so they map to unambiguous constant names.
* the plugin no longer removes already-notified items through sc_wpun_plugins_need_update and sc_wpun_themes_need_update, so callbacks on those filters receive every pending item on each run. SC_WP_Updates_Notifier::check_plugins_against_notified() and check_themes_against_notified() are deprecated.
* the email subject is now "WP Updates Notifier: updates available for <site name>" and the body uses the new plain-text layout. sc_wpun_email_content receives unescaped text and both email filters get the notification and channel instance as extra arguments. No From address is set unless one is configured; a 1.x From equal to the admin email is cleared on upgrade. The "Save settings with test email" button is replaced by "Send test email". send_email_message() and the sc_wpun_wp_mail_* methods of SC_WP_Updates_Notifier are deprecated.
* Slack notifications are no longer sent. The Slack settings are kept in the slack channel instance but are not used. SC_WP_Updates_Notifier::send_slack_message(), send_test_slack() and the Slack settings field callbacks are deprecated and do nothing.
* the email and Slack settings (email_notifications, notify_to, notify_from, slack_notifications, slack_webhook_url, slack_channel_override) are no longer in the sc_wpun_settings option. Code that read them directly should use sc_wpun_channels() instead.
* the plugin now requires PHP 7.4 or newer and WordPress 6.7 or newer. WordPress.org does not offer the update to sites below those versions, so they stay on 1.6.0.

**Features**

* add a boolean field type with checkbox, radio or select display
* add a check now button to the settings screen
* add a plugin header for the api version an add-on needs
* add an action to run the update check on demand
* add an enum field type and store notify settings as keys
* add and edit channel instances from schema-driven forms
* add length-limited rendering for channel modules
* add settings screen tabs with general and channels tabs
* add text, textarea, url, email_list, secret, number field types
* add the notification module API, registry and dispatcher
* add the public hook API for checks, notifications and delivery
* add version constraints for the public api
* add an about tab, hidden by a code-only setting
* link the documentation from the plugin's row
* mark required fields with "(required)" and aria-required
* name the view and flag errors in the page title
* add optional reminders for pending updates
* defer a channel instance until its retry-after time
* drop Slack notifications and keep their settings
* bundle translations for 11 locales
* let add-ons check the plugin API version they need
* let modules declare the api version they need
* list channel instances on the channels tab with row actions
* load channel settings and whole channels from outside the database
* load global settings from constants, env or custom sources
* move the email settings to the channels tab
* route update items to each notification channel
* send email through a built-in notification module
* add a global setting for wordpress core updates
* stop redirecting wordpress core auto-update emails
* store each channel instance's last delivery result
* store notification channels as instances
* track what each notification channel was told

**Bug fixes**

* describe saved secrets and name each clear checkbox
* escape the settings api titles on the general tab
* explain empty item types and global settings in routing
* give every settings view a heading outline
* head the error summary and link groups to a control
* hide the update notice by the update_core capability
* let only super admins manage the plugin on multisite
* link set-by and check now notes to their controls
* show check times in the site timezone, with the next one
* show general tab validation errors on their fields
* stop the channel list row header holding row actions
* hide outside values from custom field types by default
* redact channel instances when serialized
* check the capability on the settings page and harden the autoloader
* read the core update data safely
* correct settings screen markup and accessibility
* offer fixed frequencies and handle missing schedules
* schedule the update check when the plugin is activated
* send HTML email as a full document with lang and dir
* stop the auto-update email filter from fatally erroring
* attach translators comments to the right strings
* stop translating on plugins_loaded and tidy strings
* keep resolved channel secrets out of dumps, forms and custom types
* let the local site reach its own url
* meet the WordPress.org Plugin Check rules
* list only http and https links in plain text
* only link http(s) urls in markdown and keep item values on one line
* send auto-update emails through an enabled email channel only
* normalise settings and keep the run state separately
* stop the settings sanitizer dying outside the form
* tidy the channel list into core-style row actions
* remove the cron event, transients and multisite data
* read files only for installed plugins and themes
* count child themes and network plugins as active
* link plugins correctly and cope with other updaters
* work out plugin compatibility from the update data
* word notification versions and name the item in link text

**Code refactoring**

* draw settings screen from schemas with consistent displays
* make wp-updates-notifier.php the main file
* require PHP 7.4 and WordPress 6.7

= 1.6 =
* Added slack notifications and testing button
* Separated notifications so they can be email, slack, or both
* Reorganized settings page
* Changed validation to not turn on notifications for email or slack unless required fields are valid
* Test email or slack message are now triggered with a temporary transient
* Removed javascript functions

= 1.5.1 =
* Removed ability to trigger with external cron job (a potential security risk)

The full changelog is in [CHANGELOG.md](https://github.com/l3rady/wp-updates-notifier/blob/main/CHANGELOG.md).

== Upgrade Notice ==

= 2.0.0 =
From 1.5 or 1.6: activate the plugin again after updating (main file renamed, settings kept). Slack is no longer supported; its settings are kept. WordPress's auto-update emails now go to the admin email. Emails have a new subject, the default From unless set, and no HTML entities.
