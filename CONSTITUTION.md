# Platform Constitution

## Introduction

This document defines the governing principles for the w5d.io platform. These principles apply to all repositories in the [w5d.io](https://github.com/w5dio) GitHub organisation. They are intended to be applied by a coding agent when implementing or extending platform systems.

## Overview

```
┌────────────────┐      ┌────────────────┐
│ Declarative    │      │                │
│ Intent         ├─────►│ Systems        │
│ (Configuration)│      │                │
└────────────────┘      └────────────────┘
    ▲                      ▲    ▲ 
    .                      .    │
    .        . . . . . . . .    │
    .        .                  │
┌────────────────┐      ┌────────────────┐
│                │      │                │
│ Validation     ├─────►│ Remediation    │
│                │      │                │
└────────────────┘      └────────────────┘

Legend: 

──────► : Main flow

. . . ► : Read
```


## Principles

### 1. Declarative Intent

The platform is defined through declarative configuration that expresses intent.

Declarative configuration:

- Expresses what the platform should be, not how to achieve it
- Is version-controlled
- Uses the format best suited for the purpose

#### 1.1 Single Source of Truth

Declarative configuration is the single source of truth for the platform. No parallel documentation is maintained. Where the configuration data alone cannot convey full context or rationale, comments within the configuration are the preferred form of supplementary documentation.

---

### 2. Systems

The platform consists of independent systems, each realising a specific aspect of the intent.

Each system:

- Consumes the relevant parts of the declarative configuration
- Realises the intent for the aspect it owns
- Can be understood and reasoned about independently of other systems

#### 2.1 Realisation Method

The realisation method is determined by the nature of the system and may be:

- Automatic: the configuration is applied directly (e.g. Terraform)
- Semi-automatic: some steps are automated, others are not
- Manual: a user performs the required actions

---

### 3. Validation

Every system must be validated to detect any inconsistency between its actual state and the intent defined in its declarative configuration.

Validation:

- Runs fully automatically — no manual steps, checks, or acknowledgements are permitted
- Must cover every configuration item that the system realises
- Operates at a level of detail appropriate for determining that the intent is correctly realised

#### 3.1 Triggers

Validation must run under all of the following conditions:

- Periodically, at a frequency appropriate to the system
- On every change to the declarative configuration
- On demand, when triggered manually

---

### 4. Remediation

Every system must have a remediation strategy in place that is triggered on failed validations and that eventually brings the system back to successful validation.

The remediation strategy may either:

- Automatically correct the validation failure
- Alert the user for manual correction of the validation failure
