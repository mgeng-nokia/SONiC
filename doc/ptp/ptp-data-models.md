# PTP Feature Data Models

Describing the data models in implementation of the PTP feature.

## Table of Contents

- [Revision](#revision)
- [About this manual](#about-this-manual)
- [Scope](#scope)
- [Sonic DBs](#sonic-dbs)
- [Config_DB](#config_db)
  - [Feature Config](#feature-config)
  - [Application Config](#application-config)
    - [ptp4l configurations](#ptp4l-configurations)
    - [telemetry configurations](#telemetry-configurations)
- [APPL_DB](#appl_db)
  - [switch ptp port mode](#switch-ptp-port-mode)
  - [ptp port mode](#ptp-port-mode)
- [STATE_DB](#state_db)
- [ASIC_DB](#asic_db)
  - [Switch PTP Mode Configuration](#switch-ptp-mode-configuration)
  - [Port Specific PTP Mode Configuration](#port-specific-ptp-mode-configuration)
- [COUNTERS_DB](#counters_db)

## Revision

| Rev | Date | Author     | Change Description |
| --- | ---- | ---------- | ------------------ |
| 0.1 |      | Maike Geng | Initial version    |


## About this manual
This document defines the data models that are used during the operations of the PTP feature.

## Scope
This document is the definition document for data models introduced by the PTP feature.  See the [high level design document](./ptp-hld.md) for an overview.

# Sonic DBs

# Config_DB

## Feature Config

Feature config follows [optional feature standard](../optional-feature-control/Optional-Feature-Control.md) and its data model for defining a SONiC feature.  We introduce a new key for the PTP feature.

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

## PTP Application Config
Application configuration of PTP is per ASIC namespace.  We introduce a new key for PTP feature configuration in the global namespace.

```
PTP_GROUP|{{hostname}}|{{asic_enumeration}}|ptp_config
     "ptp4l_cfg" :  {{ptp4l configuration file}}
     "telemetry_cfg" :  {{telemetry configuration object}}
```

### ptp4l configurations

The ptp4l_cfg is a text in the format of a configuration file for ptp4l.  See [the documentation](https://linuxptp.nwtime.org/documentation/ptp4l/) for reference.

### telemetry configurations
* tbd

# APPL_DB
Application DB of PTP is per ASIC namespace.  We introduce some new keys for PTP feature in the asic namespace.

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

# STATE_DB
We introduce some new keys for PTP feature in the asic namespace.

```
PTP_SWITCH_TABLE:switch
     "ptp-mode": ("none"|"one-step|two-step"),
```

```
PTP_PORT_TABLE:{{ifname}}
     "ptp-mode": ("none"|"one-step|two-step"),
```

* tbd

# ASIC_DB
There are existing SAI definition for PTP timestamping mode and PTP feature uses these existing SAI definitions.

## Switch PTP Mode Configuration
PTP feature will add an attribute to existing objects in asic namespace.
```
ASIC_STATE:SAI_OBJECT_TYPE_SWITCH:oid:{{oid value}}
     "SAI_SWITCH_ATTR_PORT_PTP_MODE": ("SAI_PORT_PTP_MODE_NONE"|"SAI_PORT_PTP_MODE_SINGLE_STEP_TIMESTAMP"|         "SAI_PORT_PTP_MODE_TWO_STEP_TIMESTAMP")

```
This is the switch level configuration of PTP timestamping mode used in phase 1 and phase 2 of feature development.

## Port Specific PTP Mode Configuration
PTP feature will add an attribute to existing objects in asic namespace.
```
ASIC_STATE:SAI_OBJECT_TYPE_PORT:oid:{{oid value}}
{
     "SAI_PORT_ATTR_PTP_MODE": ("SAI_PORT_PTP_MODE_NONE"|"SAI_PORT_PTP_MODE_SINGLE_STEP_TIMESTAMP"|"SAI_PORT_PTP_MODE_TWO_STEP_TIMESTAMP")
}

```
This is the port specific configuration of PTP timestamping mode that can be used in phase 3 of feature development.

# COUNTERS_DB

* tbd
