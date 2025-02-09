---
id: sentinel
title: Sentinel Plugin
---

The sentinel plugin is by default shipped with your CrowdSec installation. The following guide shows how to configure, test and enable it.

## Configuring the plugin

By default there would be a sentinel config at these default location per OS:

- **Linux** `/etc/crowdsec/notifications/sentinel.yaml`
- **FreeBSD** `/usr/local/etc/crowdsec/notifications/sentinel.yaml`
- **Windows** `C:\ProgramData\CrowdSec\config\notifications\sentinel.yaml`

### Base configuration

You will need to specify:
 - customer_id
 - shared_key
 - log_type 

Example config: 

```yaml
type: sentinel          # Don't change
name: sentinel_default  # Must match the registered plugin in the profile

# One of "trace", "debug", "info", "warn", "error", "off"
log_level: info
# group_wait:         # Time to wait collecting alerts before relaying a message to this plugin, eg "30s"
# group_threshold:    # Amount of alerts that triggers a message before <group_wait> has expired, eg "10"
# max_retry:          # Number of attempts to relay messages to plugins in case of error
# timeout:            # Time to wait for response from the plugin before considering the attempt a failure, eg "10s"

#-------------------------
# plugin-specific options

# The following template receives a list of models.Alert objects
# The output goes in the http request body
format: |
  {{.|toJson}}

customer_id: XXX-XXX
shared_key: XXXXXXX
log_type: crowdsec

```

**Note** that the `format` is a [go template](https://pkg.go.dev/text/template), which is fed a list of [Alert](https://pkg.go.dev/github.com/crowdsecurity/crowdsec@master/pkg/models#Alert) objects.

### Configuration options

#### customer_id

Also known as the `workspace id`.
You can get it from the azure portal in `Log Analytics workspace` -> `YOUR_WORKSPACE` -> `Settings` -> `Agents`

#### shared_key

Also known as the `primary key`.
You can get it from the azure portal in `Log Analytics workspace` -> `YOUR_WORKSPACE` -> `Settings` -> `Agents`

#### log_type

The log type is the name of the log that will be sent to azure.

Assuming you chose `crowdsec`, it will appear as `crowdsec_CL` in azure.

## Testing the plugin

Before enabling the plugin it is best to test the configuration so the configuration is validated and you can see the output of the plugin. 

```bash
cscli notifications test sentinel_default
```

:::note
If you have changed the `name` property in the configuration file, you should replace `sentinel_default` with the new name.
:::

## Enabling the plugin

In your profiles you will need to uncomment the `notifications` key and the `sentinel_default` plugin list item.

```
#notifications:
# - sentinel_default
```

:::note
If you have changed the `name` property in the configuration file, you should replace `sentinel_default` with the new name.
:::

:::warning
Ensure your YAML is properly formatted the `notifications` key should be at the top level of the profile.
:::

<details>

<summary>Example profile with sentinel plugin enabled</summary>

```yaml
name: default_ip_remediation
#debug: true
filters:
 - Alert.Remediation == true && Alert.GetScope() == "Ip"
decisions:
 - type: ban
   duration: 4h
#duration_expr: Sprintf('%dh', (GetDecisionsCount(Alert.GetValue()) + 1) * 4)
#highlight-next-line
notifications:
#highlight-next-line
  - sentinel_default
on_success: break
```

</details>

## Final Steps

Let's restart crowdsec

```bash
sudo systemctl restart crowdsec
```
