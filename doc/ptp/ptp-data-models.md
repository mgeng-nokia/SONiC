# PTP Feature Data Models

## Data Model for PTP Feature

## Table of Contents

## Revision

| Rev | Date | Author     | Change Description |
| --- | ---- | ---------- | ------------------ |
| 0.1 |      | Maike Geng | Initial edition    |


## About this manual
This document defines the data models to be used in the PTP feature.

## Scope
This document is the definition document for data models introduced by the PTP feature.  See the [high level design document](./ptp-hld.md) for an overview.


# Sonic DBs


# Config_DB

## Feature Config

Feature config follows [optional feature standard](../optional-feature-control/Optional-Feature-Control.md).

```
FEATURE|ptp
     {
          "auto_restart": ("enabled"|"disabled"),
          "delayed": "False",
          "has_global_scope": "False",
          "has_per_asic_scope": "True"
          "state": ("enabled"|"disabled"),
     }
```

The PTP feature may be enabled or disabled in the "state" key.  Default is "disabled".  Auto-restart of the PTP feature may be enabled or disabled in "auto_restart" key.  Default is "enabled".

## Application Config
```
PTP_GROUP|{{hostname}}|{{asic_enumeration}}|ptp4l_config
     "ptp4l_cfg" :  {{ptp4l configuration file}}
     "telemetry_cfg" :  {{telemetry configuration object}}
```

### ptp4l configurations

The ptp4l_cfg is a text in the format of a configuration file for ptp4l.  See [the documentation](https://linuxptp.nwtime.org/documentation/ptp4l/) for reference.

### telemetry configurations
* this is tbd *

# APPL_DB

## switch ptp port mode

```
PTP_SWITCH_TABLE:switch
     "ptp-mode": ("none"|"one-step|two-step"),
```

## ptp port mode

```
PTP_PORT_TABLE:{{ifname}}
     "ptp-mode": ("none"|"one-step|two-step"),
```

# State DB

```
PTP_SWITCH_TABLE:switch
     "ptp-mode": ("none"|"one-step|two-step"),
```

```
PTP_PORT_TABLE:{{ifname}}
     "ptp-mode": ("none"|"one-step|two-step"),
```

* this is tbd will be more state and counters *

# ASIC DB

This is using existing SAI definition for PTP timestamping mode.

## Switch PTP Mode Configuration
```
ASIC_STATE:SAI_OBJECT_TYPE_SWITCH:oid:{{oid value}}
     "SAI_SWITCH_ATTR_PORT_PTP_MODE": ("SAI_PORT_PTP_MODE_NONE"|"SAI_PORT_PTP_MODE_SINGLE_STEP_TIMESTAMP"|         "SAI_PORT_PTP_MODE_TWO_STEP_TIMESTAMP")

```
This is the switch level configuration of PTP timestamping mode used in phase 1 and phase 2 of feature development.

## Port Specific PTP Mode Configuration
```
ASIC_STATE:SAI_OBJECT_TYPE_PORT:oid:{{oid value}}
{
     "SAI_PORT_ATTR_PTP_MODE": ("SAI_PORT_PTP_MODE_NONE"|"SAI_PORT_PTP_MODE_SINGLE_STEP_TIMESTAMP"|"SAI_PORT_PTP_MODE_TWO_STEP_TIMESTAMP")
}

```
This is the port specific configuration of PTP timestamping mode that can be used in phase 3 of feature development.

# Counters DB

* this is tbd *