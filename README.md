# Platform Constitution

## Introduction

This document defines the governing principles for the w5d.io platform. These principles apply to all repositories in the [w5d.io](https://github.com/w5dio) GitHub organisation. They are intended to be applied by a coding agent when implementing or extending platform systems.

## Overview

```
                                     ┌────────────────┐
                                     │                │
                        ┌────────────┤ Validation     │────────────┐
                        │            │                │            │
                        │            └────────────────┘            │
                        │                                          │
                        ▼                                          ▼
┌────────┐      ┌────────────────┐      ┌──────────┐      ┌────────────────┐
│        │      │ Declarative    │      │          │      │                │
│ Edit   ├─────►│ Configuration  ├─────►│ Creation ├─────►│ Systems        │
│        │      │ (Intent)       │      │          │      │                │
└────────┘      └────────────────┘      └──────────┘      └────────────────┘
```

## Principles

### 1. Declarative Intent

The platform is defined through declarative configuration that expresses intent.

Declarative configuration:

- Expresses what the platform should be, not how to achieve it
- Is version-controlled
- Uses the format best suited for the purpose

---

### 2. Configuration as Documentation

Declarative configuration is the documentation. No separate documentation is written by default. Separate documentation is only written where there is no other way to convey information that is not covered by the declarative configuration data itself and is kept to a minimum.

- Each aspect of the platform is defined in exactly one place
- Updating an aspect requires changes in only that place
- Configuration must be self-explanatory: a reader with no prior context should be able to understand what it describes without consulting any external source
- Where the configuration data alone cannot convey full context or rationale, comments within the configuration are the preferred form of supplementary documentation

---

### 3. Independent Systems

The platform consists of independent systems, each realising a specific aspect of the intent.

Each system:

- Consumes the relevant parts of the declarative configuration
- Realises the intent for the aspect it owns
- Can be understood and reasoned about independently of other systems

---

### 4. Creation

Each system is brought into existence based on the relevant declarative configuration. The creation method is determined by the nature of the system and may be:

- **Automatic:** the configuration is applied directly (e.g. Terraform)
- **Semi-automatic:** some steps are automated, others are not
- **Manual:** a human performs the required actions

---

### 5. Validation

Every system must be validated against the intent defined in its declarative configuration.

Validation:

- Must be automated
- Must cover as much of the declarative configuration as possible; 100% coverage is the goal
- Must detect and report any inconsistency between the system and the intent
