# OpenHMI Project Rules

Engineering Principles

## Purpose

OpenHMI shall remain a modular and protocol-independent platform for industrial HMI and automation applications.

---

## General Principles

### Keep It Simple

Prefer simple and robust solutions over complex solutions.

---

### Modular Design

Applications shall be independent from hardware and communication protocols.

---

### Protocol Independence

Applications must never directly depend on field protocols.

Use abstraction layers whenever possible.

---

### Industrial First

Solutions shall be suitable for real industrial environments.

---

### Documentation Before Development

Major architecture decisions shall be documented before implementation.

---

### ADR Requirement

Every significant architecture decision shall be documented as an ADR.

---

### Open Core Strategy

Core software components are open source.

Commercial hardware designs may remain proprietary.

---

### Remote Maintenance

Remote maintenance shall be considered from the beginning of system design.

---

### No Vendor Lock-In

Avoid dependencies on proprietary cloud services whenever possible.

---

### Hardware Remains Replaceable

Hardware choices should not unnecessarily bind the platform to a single supplier or implementation.

---

### Field Reliability Over Features

Reliability is more important than feature count.

---

### Practical Engineering

Real-world experience has priority over theoretical elegance.
