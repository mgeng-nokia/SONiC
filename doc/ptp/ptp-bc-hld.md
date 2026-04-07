# PTP Boundary Clock Feature0

## High Level Design document

## Table of Contents

- [Revision](#revision)
- [About this manual](#about-this-manual)
- [Scope](#scope)
- [Abbreviations](#abbreviations)
- [1 Introduction](#1-introduction)
  - [1.1 Feature Overview](#11-feature-overview)
- [2 High Level Design](#2-high-level-design)
  - [2.1 Functionality](#21-functionality)
  - [2.2 Multi-ASIC extensions](#22-multi-asic-extensions)
  - [2.3 Chassis extensions](#23-chassis-extensions)

## Revision

| Rev | Date | Author     | Change Description |
| --- | ---- | ---------- | ------------------ |
| 0.1 |      | Maike Geng | Outline edition    |

## About this manual

This document provides an overview for PTPv2 boundary clock feature in SONiC.

## Scope

This document is the high level design document for running a SONiC switch as a PTPv2 boundary clock.  It provides an overview of feature configuration and operation.  More detailed design and implementation, configuration data models, state data models, and platform-specific attribue data models are not within the scope of the document.

## Abbreviations

| Term  | Meaning                                                             |
| ----- | ------------------------------------------------------------------- |
| ASIC  | Application-Specific Integrated Circuit                             |
| BC    | Boundary Clock                                                      |
| BMCA  | Best Master Clock Algorithm                                         |
| DB    | Database                                                            |
| CLI   | Command-line Interface                                              |
| pmc   | PTP Management Client; a linux-ptp executable                       |
| PHC   | Physical Hardware Clock; Linux timing sychronization infrastructure |
| PTP   | Precision Time Protocol                                             |
| PTPv2 | PTP Version 2; IEEE-1588 2008 specification with 2019 enhancements  |
| ptp4l | PTP daemon for Linux; a linux-ptp executable                        |
| SAI   | Switch Abstraction Interface                                        |
| SONiC | Software for Open Networking in the Cloud                           |
| YANG  | Yet Another Next Generation                                         |

# 1 Introduction

Timing synchronization across nodes in a data center has a variety of applications requiring achieving a corresponding level of precision and accuracy in synchronization. PTPv2 is the industry standand network protocol for achieving tight timing synchronization over an ethernet network.

One of the network architectures that can scale timing synchronization over an entire data center is running PTP boundary clocks on network switches. In said network architectures, deploying PTP BCs on network switches share the processing load of handling the PTPv2 protocol and redundancy protection for each other. 

The PTPv2 boundary clock feature will enable a data center operator to run PTP boundary clocks on network switches running the SONiC OS. It will support configuration, tuning, and telemetry monitoring required to achieve and maintain the targeted timing synchronization.

## 1.1 Feature Overview

The PTP BC is an [optional feature](../optional-feature-control/Optional-Feature-Control.md) that can be enabled or disabled.  When the PTP BC feature is enabled, SONiC will launch the PTP BC container on a per-asic namespace basis.  The PTP BC container will run a PTPv2 boundary clock compliant with the IEEE-1588 standard, use the default BMCA, and use open source ptp4l.

## 1.2 Architecture
``` mermaid

---
title: PTP Boundary Clock architecture
---
  flowchart TB
    direction TB

    subgraph User
      input[CLI
      NetApi
      NetConf
      RestConf]
    end

    subgraph SONIC
      direction LR

      subgraph Redis
          direction TB
          config_db[(CONFIG_DB)]
          state_db[(STATE_DB)]
          app_db[(APP_DB)]
          counter_db[(COUNTER_DB)]
      end

      subgraph ptp boundary service
          subgraph ptpbc container
              cfgptp(config manager)
              ptp4l(ptp4l)
              telemetry_(telemetry feed)
          end

          cfgptp-->ptp4l
          ptp4l-->telemetry_
      end

      subgraph SWSS container
          counter_syncd(syncd)
      end
    end

    subgraph Linux
      direction LR
      sai(SAI)
      eth_dev(Ethernet Interface)
      phc_dev(PHC Device)
      sai-->eth_dev
      eth_dev-->phc_dev
    end

    config_db-->counter_syncd
    config_db-->cfgptp
    cfgptp-->app_db
    counter_syncd-->sai
    input-->config_db
    input<-->state_db
    input<-->counter_db
    telemetry_-->state_db
    telemetry_-->counter_db
    ptp4l-->eth_dev
    ptp4l-->phc_dev

```

- Enable configuration of hardware characteristics.
- Enable configuration of network characteristics and PTPv2 masters.
- Enable configuration of message frequencies.
- Enable query of status.
- Enable telemetry feeds.

# 1.3 Extra Platform Specific Parameters

At each PTP BC, achieving certain levels of precision and accuracy will require hardware timestamping support in the switch ASIC, high frequency fidelity in timing crystals, and appropriate hardware design in the network device. These hardware capabilities will be reflected in the PTP configuration and known in the industry as a profile.

*See OCP Profile document for this profile and will be the default configuration values. *OCP Profile document appears to not have boundary clock sections so we may have to write it.*

# 2 High Level Design

The high level design will reflect the description and diagram found in [Architecture](#12-architecture) section.
The PTPv2 boundary clock application is packaged as a separate container.

Configuration for PTPv2 boundary clock will be stored in CONFIG DB. 

1. Will use linuxptp open source and integrate with the PHC infrastructure within Linux.
2. The executable, ptp4l, will run in a separate container and use a configuration file generated from SONiC ptp configurations at launch.
3. Configuration of switch ASIC through swss and SAI for relevant hardware timestamping functional settings.
4. Apply configuration without service interruption.
5. Run-time status queries and telemetry with pmc.

## 2.1 Functionality

1. Configuration driven by ptp4l configuration file for starting ptp service.
2. Status queries, configuration updates, telemetry without service interruption via pmc.
3. Per-interface PTP mode (enabled/disabled) without service interruption via pmc.

## 2.2 Multi-ASIC extensions

In case of multi ASIC SONiC devices, multiple instances of the PTP container will be running, one instance for each of the asic namespaces.  The ptp4l executables running within the containers will listen to the other ptp4l executables - all boundary clocks - over the internally connecting system port and may synchronize with any if selected via the BCMA.

## 2.3 Chassis extensions

In a chassis implementation, linecards will be PTP containers, one instance for each of the asic namespaces.  The ptp4l executables will listen to other boundary clocks in the chassis over the internally connecting system port and may synchronize with any  if selected via the BCMA.
