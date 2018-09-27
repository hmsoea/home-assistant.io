---
layout: page
title: "Command line Switch"
description: "Instructions on how to have switches call command line commands."
date: 2015-06-10 22:41
sidebar: true
comments: false
sharing: true
footer: true
logo: command_line.png
ha_category: Switch
ha_release: pre 0.7
ha_iot_class: "Local Polling"
---


The `command_line` switch platform issues specific commands when it is turned on and off. This might very well become our most powerful platform as it allows anyone to integrate any type of switch into Home Assistant that can be controlled from the command line, including calling other scripts!

To enable it, add the following lines to your `configuration.yaml`:

```yaml
# Example configuration.yaml entry
switch:
  - platform: command_line
    switches:
      kitchen_light:
        command_on: switch_command on kitchen
        command_off: switch_command off kitchen
```

Configuration variables:

- **switches** (*Required*): The array that contains all command switches.
  - **identifier** (*Required*): Name of the command switch as slug. Multiple entries are possible.
    - **command_on** (*Required*): The action to take for on.
    - **command_off** (*Required*): The action to take for off.
    - **command_state** (*Optional*): If given, this command will be run. Returning a result code `0` will indicate that the switch is on.
    - **value_template** (*Optional*): If specified, `command_state` will ignore the result code of the command but the template evaluating to `true` will indicate the switch is on.
    - **friendly_name** (*Optional*): The name used to display the switch in the frontend.

A note on `friendly_name`:

When set, the `friendly_name` had been previously used for API calls and backend configuration instead of the `object_id` ("identifier"), but [this behavior is changing](https://github.com/home-assistant/home-assistant/pull/4343) to make the `friendly_name` for display purposes only. This allows users to set an `identifier` that emphasizes uniqueness and predictability for API and config purposes but have a prettier `friendly_name` still show up in the UI. As an additional benefit, if a user wanted to change the `friendly_name` / display name (e.g., from "Kitchen Lightswitch" to "Kitchen Switch" or "Living Room Light", or remove the `friendly_name` altogether), he or she could do so without needing to change existing automations or API calls. See aREST device below for an example.

## {% linkable_title Examples %}

In this section you find some real life examples of how to use this switch.

### {% linkable_title aREST device %}

The example below is doing the same as the [aREST switch](/components/switch.arest/). The command line tool [`curl`](http://curl.haxx.se/) is used to toggle a pin which is controllable through REST.

```yaml
# Example configuration.yaml entry
switch:
  platform: command_line
  switches:
    arest_pin_four:
      command_on: "/usr/bin/curl -X GET http://192.168.1.10/digital/4/1"
      command_off: "/usr/bin/curl -X GET http://192.168.1.10/digital/4/0"
      command_state: "/usr/bin/curl -X GET http://192.168.1.10/digital/4"
      value_template: '{% raw %}{{ value == "1" }}{% endraw %}'
      friendly_name: Kitchen Lightswitch
```

Given this example, in the UI one would see the `friendly_name` of "Kitchen Light". However, the `identifier` is `arest_pin_four`, making the `entity_id` `switch.arest_pin_four`, which is what one would use in [`automation`](/components/automation/) or in [API calls](/developers/).

### {% linkable_title Shutdown your local host %}

This switch will shutdown your system that is hosting Home Assistant.

<p class='note warning'>
This switch will shutdown your host immediately, there will be no confirmation.
</p>


```yaml
# Example configuration.yaml entry
switch:
  platform: command_line
  switches:
    home_assistant_system_shutdown:
      command_off: "/usr/sbin/poweroff"
```

Additional notes:

When the command_line platform issues a command, it does so as the user homeassistant, with the permissions that this user has. When Home Assistant runs under Hassbian (not sure about other environments), the `poweroff` command requires root privileges, and and would need to be issued in the form `sudo poweroff`. By default, the `sudo` command will ask for a password, so you would need to configure `sudo` to allow the user homeassistant to run specific or all commands without a password.

You do that by logging on to a Hassbian command line (e.g. through SSH) and modifying the sudoers file with
`sudo visudo`.

Then add the following line in the user privilege specification section:

`homeassistant ALL=(ALL) NOPASSWD: /sbin/poweroff`

If you want to give the user homeassistant the permission to run all commands as root - which is not recommended for security reasons - you would replace the specific command (`/sbin/poweroff`) with `ALL`. You can also list multiple commands, separating them with comma and space. You can use the `whereis` command to find the location of a specific command on the system: `whereis poweroff`. For example, on a Hassbian system, the `poweroff` command is at `/sbin/poweroff`.
Once you have made these changes, on Hassbian the command to shut down the system is:
`command_off: "sudo poweroff"`

When you use this switch from within Home Assistant, you may get an error message even though it works correctly, because the system is powered off immediately and cannot give a proper response.

### {% linkable_title Control your VLC player %}

This switch will control a local VLC media player ([Source](https://community.home-assistant.io/t/vlc-player/106)).


```yaml
# Example configuration.yaml entry
switch:
  platform: command_line
  switches:
    vlc:
      command_on: "cvlc 1.mp3 vlc://quit &"
      command_off: "pkill vlc"
```

### {% linkable_title Control Foscam Motion Sensor %}

This switch will control the motion sensor of Foscam Webcams which Support CGI Commands ([Source](http://www.ipcamcontrol.net/files/Foscam%20IPCamera%20CGI%20User%20Guide-V1.0.4.pdf)). This switch supports statecmd, which checks the current state of motion detection.

```yaml
# Example configuration.yaml entry
switch:
  platform: command_line
  switches:
    foscam_motion:
      command_on: 'curl -k "https://ipaddress:443/cgi-bin/CGIProxy.fcgi?cmd=setMotionDetectConfig&isEnable=1&usr=admin&pwd=password"'
      command_off: 'curl -k "https://ipaddress:443/cgi-bin/CGIProxy.fcgi?cmd=setMotionDetectConfig&isEnable=0&usr=admin&pwd=password"'
      command_state: 'curl -k --silent "https://ipaddress:443/cgi-bin/CGIProxy.fcgi?cmd=getMotionDetectConfig&usr=admin&pwd=password" | grep -oP "(?<=isEnable>).*?(?=</isEnable>)"'
      value_template: {% raw %}'{{ value == "1" }}'{% endraw %}
```

- Replace admin and password with an "Admin" privileged Foscam user
- Replace ipaddress with the local IP address of your Foscam
