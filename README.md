# GreySensor

Runtime-configurable digital grey sensor module for LibXR.

The module reads GPIO channels in the order passed by `channel_names`, so the
application or board layer owns the hardware alias mapping. For example,
`grey_sensor_gpio_0` should be the leftmost channel and `grey_sensor_gpio_7`
the rightmost channel for an 8-channel array.

The channel count is inferred from the initializer list length. The current
sample payload uses `uint8_t` masks, so supported channel count is 1-8.

## Topic

Samples are published to the `grey_sensor` topic by default.

- `raw_mask`: raw GPIO level bitmap
- `active_mask`: active-detection bitmap after `active_low` conversion
- `active_count`: number of active channels
- `weighted_position`: weighted average line position
- `position`: current position, or remembered position when line is lost
- `line_lost`: set when no channel is active
- `lost_side`: 0 unknown, 1 left, 2 right
- `lost_count`: consecutive lost-line sample count

Position uses `POSITION_SCALE = 1000`, with the left side negative and the right
side positive. For 8 channels, the channel positions are:

```text
-3500, -2500, -1500, -500, 500, 1500, 2500, 3500
```

## Example

```cpp
static GreySensor grey_sensor(
    hw, app,
    {"grey_sensor_gpio_0", "grey_sensor_gpio_1", "grey_sensor_gpio_2",
     "grey_sensor_gpio_3", "grey_sensor_gpio_4", "grey_sensor_gpio_5",
     "grey_sensor_gpio_6", "grey_sensor_gpio_7"},
    false, "grey_sensor", 10);
```

## MSPM0G3507 Binding Used By This Template

- PB0 -> `grey_sensor_gpio_0`
- PB1 -> `grey_sensor_gpio_1`
- PB2 -> `grey_sensor_gpio_2`
- PB3 -> `grey_sensor_gpio_3`
- PB4 -> `grey_sensor_gpio_4`
- PB5 -> `grey_sensor_gpio_5`
- PB6 -> `grey_sensor_gpio_6`
- PB7 -> `grey_sensor_gpio_7`
