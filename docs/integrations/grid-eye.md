# Grid-EYE

**Integration Key:** `gridEye`

The Grid-EYE sensor by Panasonic is a thermopile sensor that can be used to measure a 8x8 matrix of temperatures in front of its lense. This data can be used to detect human presence in the room, even if the person is stationary. This is done by analyzing the temperature readings for localized peaks, i.e. certain spots have a notably higher temperature than the rest. SparkFun sells a [version of the sensor](https://www.sparkfun.com/products/14607) that can be easily connected to Linux boards and will also work with this component.

This integration will provide a sensor that contains a count of people in the room. It also includes their coordinates (relative to the sensor view) in the sensor attributes.

## Requirements

The sensor needs to be connected to the I<sup>2</sup>C pins on your machine.

### Running with NodeJS

For Raspberry Pi devices the I<sup>2</sup>C interface also needs to be enabled using `sudo raspi-config` and then enabling the I<sup>2</sup>C option under Advanced Options.

### Running with Docker

Your i2c device needs to be enabled on the host and mapped into the container as a device like shown below.

::: details Example docker-compose.yml

```yaml
version: '3'
services:
  room-assistant:
    image: mkerix/room-assistant
    network_mode: host
    volumes:
      - /var/run/dbus:/var/run/dbus
    devices:
      - /dev/i2c-1
    environment:
      NODE_CONFIG: >
        {
          "global": {
            "instanceName": "changeme",
            "integrations": ["gridEye"]
          }
        }
```

:::

### Running with Hass.io

You will need to enable the i2c interface by following the [official Hass.io guide](https://www.home-assistant.io/hassio/enable_i2c/). The `config.txt` file that you create should also contain an additional option, leading to the following contents:

```
dtparam=i2c1=on
dtparam=i2c_arm=on
dtparam=i2c_baudrate=10000
```

Reboot after you imported your config in the supervisor.

## Sensor placement

When placing your sensor you need to consider a few factors to get reliable results:

- Don't face the sensor towards a source of heat that is less than 5 meters away, for example radiators or windows.
- Make sure the sensor has a clear view of all areas that you want to track.
- Consider the range of the sensor, the further away people are the harder it will be to reliably detect them.
- Consider the field of view as stated in the datasheet.

## Settings

| Name             | Type   | Default | Description                                                  |
| ---------------- | ------ | ------- | ------------------------------------------------------------ |
| `busNumber`      | Number | `1`     | I<sup>2</sup>C bus number of your machine that the sensor is connected to. |
| `address`        | Number | `0x69`  | I<sup>2</sup>C address of the D6T sensor that you want to use. |
| `deltaThreshold` | Number | `2`     | Minimum temperature difference between average and single temperature pixel in &deg;C for it to be considered as human presence. Increase if you are seeing false positives, decrease if you are seeing false negatives. |

::: details Example Config

```yaml
global:
  integrations:
    - gridEye
gridEye:
  deltaThreshold: 2
```

:::

