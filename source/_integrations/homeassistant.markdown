---
title: Home Assistant Core Integration
description: Description of the homeassistant integration.
ha_release: 0.0
ha_category:
  - Other
ha_quality_scale: internal
ha_codeowners:
  - '@home-assistant/core'
ha_domain: homeassistant
ha_platforms:
  - scene
ha_integration_type: system
related:
  - docs: /docs/configuration/basic/
    title: Basic information
  - docs: /docs/configuration/
---

The **Home Assistant Core** {% term integration %} provides generic implementations like the generic `homeassistant.turn_on`.

## Editing the general settings in YAML

The Home Assistant Core integration is also responsible for the general settings. These settings are defined during onboarding, but you can change them later under {% my general title="**Settings** > **System** > **General**" %}. For the detailed steps, refer to [Basic settings](/docs/configuration/basic/).

If you prefer editing in YAML, you can define your general settings in the [`configuration.yaml` file](/docs/configuration/).
Note that for some of the settings, these can't be edited from the UI if they were defined in YAML. They will be grayed out or inaccessible.

<p class='img'>
    <img class="no-shadow" src='/images/docs/configuration/coordinates-defined-in-yaml.png' alt='Screenshot showing coordinates cannot be edited because they are defined in configuration.yaml file'>
    Screenshot showing coordinates cannot be edited because they are defined in configuration.yaml file.
</p>

To get started with the general settings in YAML, follow these steps:

1. Copy the following information to your [`configuration.yaml` file](/docs/configuration/).

    ```yaml
    homeassistant:
      name: Home
      latitude: 32.87336
      longitude: 117.22743
      elevation: 430
      unit_system: metric
      currency: USD
      country: US
      time_zone: "America/Los_Angeles"
      external_url: "https://www.example.com"
      internal_url: "http://homeassistant.local:8123"
      allowlist_external_dirs:
        - "/usr/var/dumping-ground"
        - "/tmp"
      allowlist_external_urls:
        - "http://images.com/image1.png"
      media_dirs:
        media: "/media"
        recordings: "/mnt/recordings"
    ```

2. Edit each entry to fit your home.

{% configuration %}
name:
  description: Name of the location where Home Assistant is running.
  required: false
  type: string
latitude:
  description: Latitude of your location required to calculate the time the sun rises and sets.
  required: false
  type: float
longitude:
  description: Longitude of your location required to calculate the time the sun rises and sets.
  required: false
  type: float
elevation:
  description: Altitude above sea level in meters. Impacts sunrise data.
  required: false
  type: integer
unit_system:
  description: "`metric` for Metric, `us_customary` for US Customary. This also sets temperature_unit, Celsius for Metric and Fahrenheit for US Customary"
  required: false
  type: string
temperature_unit:
  description: "Override temperature unit set by unit_system. `C` for Celsius, `F` for Fahrenheit."
  required: false
  type: string
time_zone:
  description: "Pick your time zone from the column **TZ** of [Wikipedia's list of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)"
  required: false
  type: string
currency:
  description: "Pick your currency code from the column **Code** of [Wikipedia's list of ISO 4217 active codes](https://en.wikipedia.org/wiki/ISO_4217#Active_codes)"
  required: false
  type: string
  default: "EUR"
external_url:
  description: "The URL that Home Assistant is available on from the internet. For example: `https://example.duckdns.org:8123`. Note that this setting may only contain a protocol, hostname and port; using a path is not supported."
  required: false
  type: string
internal_url:
  description: "The URL that Home Assistant is available on from your local network. For example: `http://homeassistant.local:8123`. Note that this setting may only contain a protocol, hostname and port; using a path is not supported."
  required: false
  type: string
customize:
  description: "[Customize](/docs/configuration/customizing-devices/) entities."
  required: false
  type: string
customize_domain:
  description: "[Customize](/docs/configuration/customizing-devices/) all entities in a domain."
  required: false
  type: string
customize_glob:
  description: "[Customize](/docs/configuration/customizing-devices/) entities matching a pattern."
  required: false
  type: string
allowlist_external_dirs:
  description: List of folders that can be used as sources for sending files.
  required: false
  type: list
allowlist_external_urls:
  description: List of external URLs that can be fetched. URLs can match specific resources (e.g., `http://10.10.10.12/images/image1.jpg`) or a relative path that allows access to resources within it (e.g., `http://10.10.10.12/images` would allow access to anything under that path)
  required: false
  type: list
media_dirs:
  description: A mapping of local media sources and their paths on disk.
  required: false
  type: map
language:
  description: "Default language used by Home Assistant. This may, for example, influence the language used by voice assistants. The language should be specified as an RFC 5646 language tag, and must be a language which Home Assistant is translated to."
  required: false
  type: string
  default: "en"
country:
  description: "Country in which Home Assistant is running. This may, for example, influence radio settings to comply with local regulations. The country should be specified as an ISO 3166.1 alpha-2 code. Pick your country from the column **Code** of [Wikipedia's list of ISO 31661 alpha-2 officially assigned code codes](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements)"
  required: false
  type: string
{% endconfiguration %}

## Services

The `homeassistant` integration provides services for controlling Home Assistant itself, as well as generic controls for any entity.

### Service `homeassistant.check_config`

Reads the configuration files and checks them for correctness, but **does not** load them into Home Assistant. Creates a persistent notification and log entry if errors are found.

### Service `homeassistant.reload_all`

Reload all YAML configuration that can be reloaded without restarting Home Assistant.

It calls the `reload` service on all domains that have it available. Additionally,
it reloads the core configuration (equivalent to calling
`homeassistant.reload_core_config`), themes (`frontend.reload_themes`), and custom Jinja (`homeassistant.reload_custom_templates`).

Prior to reloading, a basic configuration check is performed. If that fails, the reload
will not be performed and will raise an error.

### Service `homeassistant.reload_custom_templates`

Reload all Jinja templates in the `config/custom_templates` directory. Changes to these templates
will take effect the next time an importing template is rendered.

### Service `homeassistant.reload_config_entry`

Reloads an integration config entry.

| Service data attribute | Description                                                |
| ---------------------- | ---------------------------------------------------------- |
| `entity_id`            | List of entity ids used to reference a config entry.       |
| `area_id`              | List of area ids used to reference a config entry.         |
| `device_id`            | List of device ids used to reference a config entry.       |
| `entry_id`             | A single config entry id used to reference a config entry. |

### Service `homeassistant.reload_core_config`

Reloads the core configuration under `homeassistant:` and all linked files. Once loaded the new configuration is applied. New `customize:` information will be applied the next time the state of the entity gets updated.

### Service `homeassistant.restart`

Restarts the Home Assistant instance (also reloading the configuration on start).

This will also do a configuration check before doing a restart. If the configuration check fails then Home Assistant will not be restarted, instead a persistent notification with the ID `persistent_notification.homeassistant_check_config` will be created. The logs will show details on what failed the configuration check.

### Service `homeassistant.stop`

Stops the Home Assistant instance. Home Assistant must be restarted from the Host device to run again.

### Service `homeassistant.set_location`

Update the location of the Home Assistant default zone (usually "Home").

| Service data attribute | Optional | Description                 |
| ---------------------- | -------- | --------------------------- |
| `latitude`             | no       | Latitude of your location.  |
| `longitude`            | no       | Longitude of your location. |
| `elevation`            | yes      | Elevation of your location. |

#### Example

```yaml
action:
  service: homeassistant.set_location
  data:
    latitude: 32.87336
    longitude: 117.22743
    elevation: 120
```

### Service `homeassistant.toggle`

Generic service to toggle devices on/off. Same usage as the
`light.toggle`, `switch.toggle`, etc. services. The difference with this
service compared the others, is that is can be used to mix different domains,
for example, a light and a switch can be toggled in a single service call.

| Service data attribute | Optional | Description                                   |
| ---------------------- | -------- | --------------------------------------------- |
| `entity_id`            | yes      | The entity_id of the device to toggle on/off. |

#### Example

```yaml
action:
  service: homeassistant.toggle
  target:
    entity_id: 
      - light.living_room
      - switch.tv
```

### Service `homeassistant.turn_on`

Generic service to toggle devices on. Same usage as the
`light.turn_on`, `switch.turn_on`, etc. services. The difference with this
service compared the others, is that is can be used to mix different domains,
for example, a light and a switch can be turned on in a single service call.

| Service data attribute | Optional | Description                             |
| ---------------------- | -------- | --------------------------------------- |
| `entity_id`            | yes      | The entity_id of the device to turn on. |

#### Example

```yaml
action:
  service: homeassistant.turn_on
  target:
    entity_id:
      - light.living_room
      - switch.tv
```

### Service `homeassistant.turn_off` 

Generic service to toggle devices off. Same usage as the
`light.turn_off`, `switch.turn_off`, etc. services. The difference with this
service compared the others, is that is can be used to mix different domains,
for example, a light and a switch can be turned off in a single service call.

| Service data attribute | Optional | Description                              |
| ---------------------- | -------- | ---------------------------------------- |
| `entity_id`            | yes      | The entity_id of the device to turn off. |

#### Example

```yaml
action:
  service: homeassistant.turn_off
  target:
    entity_id:
      - light.living_room
      - switch.tv
```

### Service `homeassistant.update_entity`

Force one or more entities to update its data rather than wait for the next scheduled update.

| Service data attribute | Optional | Description                                             |
| ---------------------- | -------- | ------------------------------------------------------- |
| `entity_id`            | no       | One or multiple entity_ids to update. It can be a list. |

#### Example

```yaml
action:
  service: homeassistant.update_entity
  target:
    entity_id:
    - light.living_room
    - switch.coffe_pot
```

### Service `homeassistant.save_persistent_states`

Save the persistent states (for entities derived from RestoreEntity) immediately.
Maintain the normal periodic saving interval.

Normally these states are saved at startup, every 15 minutes and at shutdown.
