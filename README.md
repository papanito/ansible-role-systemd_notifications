# Ansible role "papanito.systemd-notifiers" <!-- omit in toc -->

[![Ansible Role](https://img.shields.io/ansible/role/55671)](https://galaxy.ansible.com/papanito/systemd_notifiers) [![Ansible Quality Score](https://img.shields.io/ansible/quality/55671)](https://galaxy.ansible.com/papanito/systemd_notifiers) [![Ansible Role](https://img.shields.io/ansible/role/d/55671)](https://galaxy.ansible.com/papanito/systemd_notifiers) [![GitHub issues](https://img.shields.io/github/issues/papanito/ansible-role-systemd_notifiers)](https://github.com/papanito/ansible-role-systemd_notifiers/issues) [![GitHub pull requests](https://img.shields.io/github/issues-pr/papanito/ansible-role-systemd_notifiers)](https://github.com/papanito/ansible-role-systemd_notifiers/pulls)

When systemd services fail you usually don't really notice it unless it's a critical service. However there may be situations where it would be nice that you are mare aware of failing services. This role will setup systemd jobs of type `oneshot` which you can the use in your critical services, by using them in the [`OnFailure`][OnFailure]:

> A space-separated list of one or more units that are activated when this unit enters the “failed” state.

See also [Notifications for failing Systemd services](https://wyssmann.com/blog/2021/07/notifications-for-failing-systemd-services/) for the idea behind this role.


## Requirements

N/A

## Role Variables

There are common role variables and service specific ones. Most variables should be fine (as tested). Thus it's recommended to only define these variables

| Parameter                    | Description                                                                                                   | Default Value    |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------- | ---------------- |
| `sn_install_desktop_notifier_service`   | Whether to install the service for desktop notification - `No` will uninstall it if previously installed.     | `No`  |
| `sn_install_googlechat_notifier_service`| Whether to install the service for google chat notification - `No` will uninstall it if previously installed. | `No`  |
| `sn_install_mail_notifier_service`      | Whether to install the service for mail notification - `No` will uninstall it if previously installed.        | `No`  |
| `sn_systemd_script_mode`     | Mode of the script file                                  | `0774`                        |
| `sn_systemd_user`            | User under which the service runs.                       | `root`                        |
| `sn_systemd_group`           | User group under which the service runs.                 | `systemd-journal`             |
| `sn_systemd_service_mode`    | Mode of the service file                                 | `0644`                        |

The following role variables are fixed and cannot be overriden:

| Parameter                    | Description                                              | Default Value                 |
| ---------------------------- | -------------------------------------------------------- | ----------------------------- |
| `systemd_script_target`      | Location where to place script files                     | `/usr/local/bin/`             |
| `systemd_target_dir`         | Location where to place `.service` files                 | `/etc/systemd/system/`        |
| `sn_service_name_googlechat` | Name of the systemd service for google chat notification | `systemd-googlechat-notifier` |
| `sn_script_name_googlechat`  | Name of the script for google chat notification          | `notify-google-chat.sh`       |
| `sn_service_name_mail`       | Name of the systemd service for mail notification        | `systemd-mail-notifier`       |
| `sn_script_name_mail`        | Name of the script for mail notification                 | `notify-mail.sh`              |
| `sn_service_name_desktop`    | Name of the systemd service for desktop notification     | `systemd-desktop-notifier`    |
| `sn_script_name_desktop`     | Name of the script for desktop notification              | `notify-desktop.sh`           |

### Desktop Notification

If `sn_install_desktop_notifier_service=true` will install the desktop notifier, which allows you to send desktop notifications to a user `{{ sn_username }}`

| Parameter                    | Description                                                                                                   | Default Value    |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------- | ---------------- |
| `sn_username`                | [Mandatory] User to which send the desktop notifications and under which user the notifier runs               | `root`           |

To use this notifier in your critical systemd service files, do this:

```
[Unit]
OnFailure=systemd-desktop-notifier@%N.service
```

### Google Chat Notification

If `sn_install_googlechat_notifier_service=true` will install the googlechat notifier, which allows you to send notifications to a google channel specified via `sn_googlechat_url`


| Parameter                    | Description                                                                                                   | Default Value    |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------- | ---------------- |
| `sn_googlechat_url`        | Url to google endpoint (webhook)                                                                              | `-`              |

The message is [formatted][google-chat-message-format] as follows:

```json
{'text':'*$SERVICE@$(hostname)*\n\`\`\`$status\`\`\`'}
```

To use this notifier in your critical systemd service files, do this:

```
[Unit]
OnFailure=systemd-googlechat-notifier@%N.service
```

### Mail Notifier

If `sn_install_mail_notifier_service=true` will install the mail notifier, which allows you to send notifications to a `sn_mail_recepient` using the `sn_mail_notify_command`


| Parameter                    | Description                                                                                                   | Default Value    |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------- | ---------------- |
| `sn_mail_recepient`             | E-Mail to where to send notification mails                                                                    | `-`              |
| `sn_mail_notify_command`     | Command to send mail. **CAUTION** the role currently does not install or configure any mail service, you have to do this by yourself       | `sendmail -i -t` |

To use this notifier in your critical systemd service files, do this:

```
[Unit]
OnFailure=systemd-mail-notifier@%N.service
```

## Dependencies

n/a

## Testing

TBD

## Example Playbook

The following playbook installs the systemd notification service for google chat notifications

```yml
- name: Install googlechat notifier service
  hosts: localhost
  vars:
    sn_install_googlechat_service: true
    sn_googlechat_url: https://chat.googleapis.com/v1/spaces/xxxxxxxxxx/messages\?key\=xxxxxxxxxxxxxxxxxxxx\&token\=xxxxxxxxxxx%3D

  roles:
     - papanito.systemd_notifications
```

```yml
- name: Install systemd desktop notifier
  hosts: localhost
  vars:
    sn_install_desktop_notifier_service: true

  roles:
     - papanito.systemd_notifications
```

## License

This is Free Software, released under the terms of the Apache v2 license.

## Author Info

Written by [Papanito](https://wyssmann.com) - [Gitlab](https://gitlab.com/papanito) / [Github](https://github.com/papanito)


[google-chat-message-format]: https://developers.google.com/chat/reference/message-formats/basic
[OnFailure]: https://manpages.debian.org/stretch/systemd/systemd.service.5.en.htm