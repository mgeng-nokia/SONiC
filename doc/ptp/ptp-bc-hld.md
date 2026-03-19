# PTP Boundary Clock

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

This document provides an overview on PTPv2 boundary clock in SONiC.

## Scope

This document is the high level design document for running a SONiC switch as a PTPv2 boundary clock.  It enumerates the high level requirements and provides a high level overview of boundary clock operations such as starting and stopping .  Detailed design and implementation of boundary clock and the configuration platform specific parametersare not within the scope of the document.

## Abbreviations

| Term  | Meaning                                                             |
| ----- | ------------------------------------------------------------------- |
| ASIC  | Application-Specific Integrated Circuit                             |
| BC    | Boundary Clock                                                      |
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

## 1.1 Feature Overview

Timing synchronization across nodes in a data center has a variety of applications requiring their corresponding levels of precision and accuracy. PTPv2 is a network protocol that enables network devices to achieve timing synchronization from a clock source over an ethernet network.

One of the network architectures that achieves timing synchronization across the entire data center to a desired precision and accuracy is running PTP BCs on network switches.  In this network architecture, the many PTP BCs scale the timing synchronization, reduce PTPv2 processing load on the PTP GMs, and provide redundancy for each other.  *See OCP Profile document.*
*example diagram at https://infocenter.nokia.com/public/7750SR217R1A/topic/com.nokia.Basic_System_Configuration_Guide_21.7.R1/ptp_boundary_cl-d321e3962.html?cp=2_7_3_6_5*

At each PTP BC, achieving certain levels of precision and accuracy will hardware timestamping support in the switch ASIC, high frequency fidelity in timing crystals, and appropriate hardware design in the network device. These hardware capabilities will be reflected in the PTP configuration and known in the industry as a profile. *See OCP Profile document for this profile and will be the default configuration values.*

**OCP Profile document appears to not have boundary clock sections so we may have to write it.**

This document describes the design of SONiC components in order to deploy and operate switches as PTPv2 boundary clocks. 

- Enable configuration of hardware characteristics.
- Enable configuration of network characteristics and PTPv2 masters.
- Enable configuration of message frequencies.
- Enable query of status.
- Enable telemetry feeds.

# 2 High Level Design

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

In case of multi ASIC SONiC devices, a separate instance of the PTP container can run in each of the asic namespaces.  The ptp4l executables may synchronize with another  

## 2.3 Chassis extensions

In a chassis implementation, each linecard will run independent PTP containers.  PTP 
Chassis will enable the recycle interfaces and allow PTP operation over recycle ports.  This will allow the 

