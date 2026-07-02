# GreySensor

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Language](https://img.shields.io/badge/language-C++20-orange.svg)](https://en.cppreference.com/)
[![Framework](https://img.shields.io/badge/Framework-LibXR-green)](https://github.com/Jiu-xiao/libxr)
[![GitHub stars](https://img.shields.io/github/stars/verdancy-org/GreySensor?style=social)](https://github.com/verdancy-org/GreySensor/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/verdancy-org/GreySensor?style=social)](https://github.com/verdancy-org/GreySensor/network/members)
[![GitHub issues](https://img.shields.io/github/issues/verdancy-org/GreySensor)](https://github.com/verdancy-org/GreySensor/issues)

**English** | [中文](README_CN.md)
**GreySensor** is a runtime-configurable digital grey sensor module designed for
line tracking and reflective optical sensor arrays.

## Module Introduction

GreySensor reads digital GPIO channels in the order provided by the application
layer and publishes a compact line state through LibXR `Topic`. It keeps the
driver independent from board-level pin names: the board or application registers
GPIO hardware aliases, then passes those alias strings to the module constructor.

The module calculates a weighted line position from active channels and keeps the
last valid position when the line is temporarily lost, simplifying upper-level
control loops that need stable direction feedback.

## Features

- Based on `LibXR::GPIO` for hardware abstraction, decoupling the module from
  platform-specific GPIO implementations.

- Follows `Application` framework specifications, supporting dependency
  injection and automated lifecycle management through MANIFEST.

- Supports runtime channel count detection from the `channel_names` initializer
  list. The current bitmap payload supports 1-8 channels.

- Publishes raw channel state, active channel state, weighted position, line-loss
  state, remembered position, and consecutive lost-line count.

- Uses a signed fixed-point position scale where the left side is negative, the
  right side is positive, and the center is zero.

## Hardware Requirements

- GPIO device nodes matching each string in the `channel_names` configuration
  must be provided.

## Constructor Parameters

- `channel_names`
  - GPIO alias list ordered from left to right. The list length determines the
    active channel count.

- `active_low`
  - Whether low GPIO level means the corresponding channel detects the line.

- `topic_name`
  - Topic name used to publish `GreySensor::Sample`.

- `publish_period_ms`
  - Minimum publish interval in milliseconds. Set to `0` to publish on every
    monitor cycle.

### API Reference

```cpp
struct Sample {
    uint8_t raw_mask;
    uint8_t active_mask;
    uint8_t changed_mask;
    uint8_t channel_count;
    uint8_t active_count;
    uint8_t line_detected;
    uint8_t line_lost;
    uint8_t lost_side;
    int16_t weighted_position;
    int16_t position;
    int16_t remembered_position;
    uint32_t lost_count;
    uint32_t sequence;
    std::array<uint8_t, MAX_CHANNEL_COUNT> raw;
    std::array<uint8_t, MAX_CHANNEL_COUNT> active;
};
```

For an 8-channel array, the default position scale maps channels to:

```text
-3500, -2500, -1500, -500, 500, 1500, 2500, 3500
```

When no channel is active, `line_lost` is set and `position` keeps the most
recent valid `weighted_position`. `lost_side` is `0` for unknown, `1` for left,
and `2` for right.

## Dependencies

- No dependencies except for the LibXR basic framework.


