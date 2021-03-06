<h2 style="padding-top: 0">Configuration</h2>

Each MOS application has a configuration file `config.yaml` that's required to have, in order to run.

What this file does:

1. Sets meta information for the application, which is used in the <a href="http://apps.matrix.one" target="_blank">MATRIX App Store</a>.
1. Defines a schema for [sending data](data-types.md) from the device.
1. Identifies which [sensors](../reference/sensors/#configuration), services<!--[services](../reference/computer-vision/#configuration)-->, [events](../reference/crosstalk.md ) and [integrations](../reference/integrations/#configuration) an application utilizes.
1. Defines dynamic [settings](../reference/system/#settings) variables, which are available to the application and end-user modifiable.
1. Defines the [widgets](../reference/widgets.md) and their layout in the [dashboard](dashboard.md)
1. Generates a policy on install, asking the user for specific access to their device.

### Configuration Options
* `name` - Name of the application that appears as part of the application store when viewing it.
* `description` - Short description that appears as part of the application when deploying or pushing to App Store.
* `keywords` - a few values about the application. ex: `face, door, unlock`
* `version` - this field will be updated by the CLI on publish.
* `configVersion` - This field defines what version is the configuration. don't touch this. 
* `settings` - runtime variables for MOS applications
* `dataTypes` - schema for structured data from an application
* `sensors` - which sensors to activate for this application
* `integrations` - which external integrations does this application support ie. `ifttt`
* `events` - which events this application has listeners for 
* `screens` - layout for widgets. see [Layout](dashboard.md#layout)
* `widgets` - widget definitions. see [Widgets](dashboard.md#widgets)
* `services` - service definitions. <!--see [Computer Vision](../reference/computer-vision.md)-->

```language-yaml
#example config.yaml

name: example-app
displayName: The Example App
description: An example application config.yaml.

imageUrls:
  - http://image.com/1

keywords:
  - example
  - configuration

categories:
  - entertainment

sensors:
  - temperature
  - gyroscope
```

### Usage Notes
The `config.yaml` is standardized before installation, which will modify it somewhat, especially in the widget definations. Do not be alarmed if your installed `config.yaml` does not match the original, this is normal.

### More info
See the helper library with examples at: <https://github.com/matrix-io/matrix-app-config-helper>



