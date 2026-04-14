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


# Sonic DB


# Config_DB

# Feature Config

Feature config follows [optional feature standard](../optional-feature-control/Optional-Feature-Control.md).

```
FEATURE|ptp
     {
          "type": "hash",
          "value": {
               "auto_restart": ("enabled"|"disabled"),
               "delayed": "False",
               "has_global_scope": "False",
               "has_per_asic_scope": "True"
               "state": ("enabled"|"disabled"),
          }
     }
```

The PTP feature may be enabled or disabled in the "state" key.  Default is "disabled".  Auto-restart of the PTP feature may be enabled or disabled in "auto_restart" key.  Default is "enabled".

# Application Config

## ptp4l configurations
```
PTP_GROUP|{{asic_enumeration}}|ptp4l_config
     "ptp4l_cfg" :  {{ptp4l configuration file}}
```

The ptp4l_cfg 

### hardware related sections

### network related sections

### configuration helper


## telemetry configurations

```
PTP_GROUP|{{asic_enumeration}}|telemetry
     "telemetry_cfg" :  {{telemetry configuration object}}
```

# Application DB

# State DB


# ASIC DB



# Counters DB


