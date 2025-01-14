# Migration Guide

## Version 0.10.x to 2.0

::: tip NOTE
If you are using a version prior 0.10, please use the [update guide (manual mode)](https://docs.froxlor.org/v0.10/general/update/) to update to the latest 0.10.x version first
:::

## 1. Requirements

* froxlor **0.10+** (updates from 0.9 are not supported)
* **PHP 7.4+**
* Required PHP extensions:
  * session, ctype, xml, filter, posix, mbstring, curl, gmp, json
* Suggested PHP extensions:
  * bcmath, zip

## 2. New features

### Settings mode

Many settings froxlor introduced over the years are very specific and just for edge cases. To not overwhelm users that don't need these, we have split the settings in to a **Basic** and an **Advanced** mode.
The default for new installations will be the **Basic** mode. Updaters will be set to **Advanced** in case any of the advanced settings were adjusted and not to confuse these users with possible "missing" settings.

<UiBrowser :src="$withBase('/img/frx_settings_mode.png')" alt="Settings mode: advanced" />

### Global search

Users can now globally search all the resources that belong to them using the **Global search** on the top. For admins, this even includes all **Settings**!

<UiBrowser :src="$withBase('/img/frx_global_search.png')" alt="Search everything from everywhere" />

### Customize visibility of table-columns

In all the resource overviews, it is now possible to adjust the visibile columns. To change, simply click on the <i class="fa fa-cog"></i> icon in the top-right corner of the listing as shown below:

<UiBrowser :src="$withBase('/img/frx_mig_columns_icon.png')" alt="frx_mig_columns_icon" />

A click opens a modal window where the shown columns can be managed:

<UiBrowser :src="$withBase('/img/frx_mig_columns_modal.png')" alt="frx_mig_columns_modal" />

### New command line tools

Our crons and helper scripts were migrated and can now be accessed via `bin/froxlor-cli`. More details see [Froxlor console scripts (CLI)](../admin-guide/cli-scripts).

```shell
bin/froxlor-cli froxlor:api-call admin Froxlor.generatePassword '{"length":20}'

{
    "data": "aUP6wfigEr5p84dB7lvO"
}
```

### MysqlServer API command

Easy manage multiple database server instances for your customers with the new MysqlServer API command via webinterface or directly via API-call (admin only). You can also specifiy which servers are available for each customer individually. The mysql-servers do not have to meet any special requirements. You will need a privileged user that has the ability to create databases and users, see [step 3 on install guide](installation/tarball#_3-create-privileged-database-user) for the required permissions.

<UiBrowser :src="$withBase('/img/frx_mysqlserver_add.png')" alt="Add new MySQL server for your customers" />

::: tip NOTE
All mysqls server credentials are stored in the `lib/userdata.inc.php` file and **no cleartext** passwords are stored in the database
:::

## 3. Important changes

### HTTP-Authentication for API / API Endpoints

API authentication now requires the use of the HTTP-Authentication Header. The old login method ( `header`-array in Request-body) is no longer supported.

Optionally, you can now call API endpoints in the form of `api.php?/module/function/` instead of having module.function in the Request body.

```shell
AUTH=$(echo -ne "$FROXLOR_API_KEY:$FROXLOR_API_SECRET" | base64 --wrap 0)

curl \
  --header "Content-Type: application/json" \
  --header "Authorization: Basic $AUTH" \
  --request POST \
  --data  '{"key1":"value1", "key2":"value2"}' \
  https://froxlor.example.com/api.php
```

### Auto-update via Webinterface

The `auto-update` feature must now be enabled explicitly in `lib/config.inc.php`. This was introduced to avoid updating froxlor via webinterface manually in case it was installed via debian/ubuntu package. See also [settings in config.inc.php](../admin-guide/settings/#_3-settings-in-config-inc-php).

### Cron / Helper scripts

The main cron file in `scripts/froxlor_master_cronjob.php` as well as the helper scripts in `scripts/` and `install/scripts/` have been migrated to the new [command line tools](../admin-guide/cli-scripts).

| Old script                           | New command                              |
|--------------------------------------|------------------------------------------|
| install/scripts/config-services.php  | bin/froxlor-cli froxlor:config-services  |
| install/scripts/switch-server-ip.php | bin/froxlor-cli froxlor:switch-server-ip |
| scripts/php-sessionclean.php         | bin/froxlor-cli froxlor:php-sessionclean |
| scripts/froxlor_master_cronjob.php   | bin/froxlor-cli froxlor:cron             |

### Services configuration changes

#### Dovecot / dovecot-sql.conf.ext

The sql-configuration for dovecot in `/etc/dovecot/dovecot-sql.conf.ext` is needed for dovecot to communicate with the froxlor database and to know about existing email accounts. As froxlor now uses more recent password hashes for the stored passwords, it is required to remove/comment out the following option in the given config file to allow the new password-hashes to be used correctly.

```diff
- default_pass_scheme = CRYPT
+ #default_pass_scheme = CRYPT
```

::: warning ATTENTION
E-mail users might not be able to login if this setting is not adjusted
:::

#### ProFTPd / sql.conf

The sql-configuration for proftpd in `/etc/proftpd/sql.conf` is used by proftpd to read the virtual users from froxlors database. As froxlor now uses more recent password hashes for the stored passwords, it is required to adjust the possible auth-types to allow new password-hashes to be used correctly.

```diff
- SQLAuthTypes Crypt
+ SQLAuthTypes Crypt OpenSSL
```

::: warning ATTENTION
FTP users might not be able to login if this setting is not adjusted
:::

### Debian Stretch / Ubuntu Xenial and CentOS

As of version 2.0, froxlor removed the configuration templates for Debian Stretch / Ubuntu Xenial and CentOS.
