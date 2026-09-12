# OpenHMI Architecture

## Overview

OpenHMI is an open-source platform for industrial Human Machine Interface (HMI), operational monitoring, remote I/O, visualization and automation-related integration.

The project was initially created for aviation fueling systems but is designed as a generic and extensible platform.

Core principles:

* Modular architecture
* Protocol independence
* Open software
* Industrial deployment
* Remote maintenance
* Hardware abstraction
* Long-term maintainability

---

## System Layers

```text
Applications / Interfaces
          ↑
Health / Events / Process State
          ↑
Adapters / Communication
          ↑
Hardware / Infrastructure Sources
```

---

## Applications

Applications provide user-facing functionality.

Examples:

* FuelDisplay
* Web interfaces
* Monitoring applications
* Maintenance tools
* Reporting interfaces

Applications must not directly depend on field protocols.

---

## Process-State and Data Abstraction

OpenHMI separates field communication from application logic.

Applications receive normalized data structures regardless of the underlying protocol or hardware source.

Possible sources and adapters include:

* Modbus / RS485
* MQTT
* REST APIs
* JSON
* proprietary protocol adapters
* pulse/counter modules
* industrial controllers

Example:

```json
{
  "product": "AVGAS 100LL",
  "unitPrice": 2.89,
  "liters": 123.45,
  "totalPrice": 356.77,
  "status": "fueling"
}
```

---

## Health and Events

OpenHMI evaluates operational state independently from individual user interfaces.

The common health model uses:

```text
OK
WARN
ERROR
```

Health information can include system resources, service state, communication availability, process-source state, data age, network state and infrastructure diagnostics.

Persistent event history and reliable notification delivery are part of the platform direction.

---

## Communication Layer

Communication between field devices, gateways and controllers may use technologies such as:

* RS485 / Modbus
* Ethernet
* WiFi where explicitly required
* MQTT
* CAN
* I²C for local hardware diagnostics
* HTTP / REST
* WebSocket

The communication technology remains an implementation detail behind adapters wherever possible.

---

## Linux Edge Platform

The Linux edge controller hosts the local OpenHMI runtime.

Current primary platform:

* Raspberry Pi

Functions include:

* application hosting
* local web server
* process-state acquisition
* system health monitoring
* notification services
* remote access
* data aggregation
* integration adapters

The architecture is intended to remain portable to other Linux systems.

---

## Remote Service

Current preferred remote network technology:

* Tailscale

Administrative access uses standard OpenSSH over the Tailscale network.

Goals:

* secure remote access
* no public inbound SSH requirement
* predictable recovery paths
* simplified deployment
* reduced maintenance effort

---

## Fuel-Station Architecture

FuelDisplay is the first productive OpenHMI application.

The fundamental separation is:

```text
Field device / protocol
        ↓
Adapter / Gateway
        ↓
Normalized Process State
        ↓
FuelDisplay / Web / Health / Reporting
```

This allows the data source to change without forcing the application to understand the field protocol directly.

---

## OpenHMI Ecosystem

```text
Open / reusable platform
│
├── OpenHMI Core
├── OpenHMI Services
├── Public Documentation
└── selected reusable hardware/software components

Project-specific / commercial
│
├── industry applications
├── customer integrations
├── hardware appliances
└── proprietary hardware or software where appropriate
```

The platform is designed so that reusable infrastructure can remain open while application-specific or commercial solutions can use their own licensing model.
