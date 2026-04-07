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

The PTPv2 boundary clock feature enables a data center operator to run PTP boundary clocks on network switches running the SONiC OS. It supports the configuration, tuning, and telemetry monitoring required to achieve and maintain targeted timing synchronization.

## 1.1 Feature Overview

The PTP BC is an [optional feature](../optional-feature-control/Optional-Feature-Control.md) that can be enabled or disabled.  When the PTP BC feature is enabled, SONiC will launch the PTP BC container on a per-asic namespace basis.  The PTP BC container runs a PTPv2 boundary clock compliant with the IEEE-1588 standard, use the default BMCA, and uses open source ptp4l.

## 1.2 Architecture
``` mermaid

---
title: PTP Boundary Clock architecture
---
  flowchart TB
    direction TB

    subgraph User and Management
      input[CLI
      NetConf Interface
      RestConf Interface]
    end

    subgraph SONIC
      direction LR

      subgraph Redis
          direction TB
          config_db[(CONFIG_DB)]
          state_db[(STATE_DB)]
          app_db[(APPL_DB)]
          counter_db[(COUNTER_DB)]
      end

      subgraph ptpbc_service [ptp boundary clock]
          subgraph ptpbc [ptp bc container]
              appcfg(app manager)
              ptp4l(ptp4l)
              telemetry_(telemetry feed)
          end

          appcfg-->ptp4l
          ptp4l-->telemetry_
      end

      subgraph syncd container
          syncd(syncd)
          sai(SAI)

          syncd --> sai
      end

      subgraph swss_service [swss container]
          ptp_orchagent(ptp orchagent)
      end
    end

    subgraph Linux
      direction LR
      asic_dev(ASIC drivers)
      eth_dev(Ethernet Device)
      phc_dev(PHC Device)
      asic_dev --> eth_dev
      eth_dev-->phc_dev
    end

    config_db -->syncd
    config_db --> appcfg
    appcfg --> app_db
    sai --> asic_dev
    input-->config_db
    input<-->state_db
    input<-->counter_db
    telemetry_-->state_db
    telemetry_-->counter_db
    ptp4l-->eth_dev
    ptp4l-->phc_dev

```

# 1.3 Additional Platform-Specific Parameters

Achieving certain levels of precision and accuracy will require hardware timestamping support in the switch ASIC, high frequency fidelity in timing crystals, appropriate hardware design, and other tuning parameters. These options exist. The specific options fall outside the scope of this document.

# 2 High Level Design

The high level design will reflect the description and diagram found in [Architecture](#12-architecture) section.
The key piece of the PTPv2 boundary clock feature is packaged as a container, PTP BC container.  When PTPv2 boundary clock feature is enabled, SONiC will launch the PTP BC service; SONiC launches an instance of the PTP BC container for every ASIC namespace.

When the PTP BC container starts, ptp app manager will read from CONFIG_DB and generate a configuration for ptp4l. When ptp app manger finds that a PTP port in the ASIC namespace is enabled, ptp app manager will generate a ptp4l configuration and launch the ptp4l executable.  The ptp4l executable is configured to run in boundary clock mode.  The ptp4l executable interacts with Linux Ethernet devices and with Linux PHC devices under linux. The Linux Ethernet devices and Linux PHC devices will be created for ptp4l by syncd via SAI calls.

After the ptp app manager launches ptp4l executable, ptp app manager monitors CONFIG_DB and STATE_DB for incremental configuration changes and launches a telemetry feed executable.  All incremental configuration are applied to the running ptp4l without service interruption via pmc.  The telemetry feed will subscribe to telemetry update with pmc and push status and statistics to STATE_DB and COUNTER_DB.

## 2.1 Base Functionality

The base case is SONiC running on a single device with a single ASIC. In this use case, one instance of the PTP BC container is launched by SONiC.

## 2.1.1 Multi-ASIC extensions

In case of multi ASIC SONiC devices, multiple instances of the PTP container will be running, one instance for each of the asic namespaces.  The ptp4l executables running within the containers will listen to the other ptp4l executables - all boundary clocks - over the internally connecting system port and may synchronize with any if selected via the BCMA.  The interally connecting system port is enabled in the ASIC namespace by default.

## 2.1.2 Chassis extensions

In a chassis implementation, linecards will be PTP containers, one instance for each of the asic namespaces.  The ptp4l executables will listen to other boundary clocks in the chassis over the internally connecting system port and may synchronize with any  if selected via the BCMA. The interally connecting system port is enabled in the ASIC namespace by default.

## 2.2 Modules

# SAI
The SWSS will affect needed configuration changes on the ASIC device via the SAI, a vendor agnostic middleware interface. The vendor-specific implementation of the SAI will interact with the vendor-specific ASIC device driver.

## 2.2.1 ASIC Device
The ASIC device driver is a vendor-specific component that will create, configure, and maintain linux ethernet device and linux PHC devices.

## 2.2.2 Linux Ethernet Device
The ethernet device is a standard linux infracture for ethernet ports. When applicable, the ethernet device will advertise hardware timestamping capability and have an associated linux PHC device.  ptp4l interacts directly with the linux ethernet device.

## 2.2.2 PHC Device
The PHC device is a standard linux infracture for ptp clocks. ptp4l interacts directly with the linux PHC device.
