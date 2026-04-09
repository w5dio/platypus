# Platform Constitution

## Introduction

This document defines the governing principles for the w5d.io platform. These principles apply to platform system repositories in the [w5d.io](https://github.com/w5dio) GitHub organisation — not to meta-level repositories such as this one. They are intended to be applied by a coding agent when implementing or extending platform systems.

## Overview

```
┌──────────────────────────────────────────┐     ┌─────────────┐
│                                          │     │             │
│                Validation                ├────►│             │
│                                          │     │             │
└────────.────────────────────────.────────┘     │             │
         ▼                        ▼              │ Remediation │
┌─────────────────┐      ┌─────────────────┐     │             │
│                 │      │                 │     │             │
│ Configuration   ├─────►│ Systems         │◄────┤             │
│                 │      │                 │     │             │
└─────────────────┘      └─────────────────┘     └─────────────┘
```

## Principles

### 1. Configuration

Each system is governed by declarative configuration that:

- Expresses intent — what the system should be, not how to achieve it
- Is declarative — structured state rather than procedural steps
- Is version-controlled
- Serves as the single source of truth — no parallel documentation is maintained; where configuration data alone cannot convey full context, inline comments are the preferred form of supplementary documentation

#### 1.1 Configuration Format

The configuration format is determined by the nature of the system and may be:

- Framework-based (e.g. IaC)
- Ad-hoc (e.g. YAML)

---

### 2. Systems

Each configuration drives a system that realises the declared state.

Each system:

- Owns its declarative configuration
- Can be understood and reasoned about independently of other systems

#### 2.1 Creation Method

The way the system is created depends on the nature of the system and may be:

- Framework automation (e.g. IaC tool)
- Ad-hoc automation (e.g. scripts)
- Manual
- Hybrid (any combination of the above)

---

### 3. Validation

Every system must be validated to verify that its actual state conforms to its declared configuration.

Validation:

- Runs fully automatically — no manual steps, checks, or acknowledgements are permitted
- Covers every item declared in the configuration
- Operates at a level of detail sufficient to determine conformance

#### 3.1 Triggers

Validation must run under all of the following conditions:

- Periodically, at a frequency appropriate to the system
- On every change to the declarative configuration
- On demand, when triggered manually

---

### 4. Remediation

> **Note:** This principle is still a work in progress.

Every system must have a remediation strategy in place that is triggered on failed validations and that eventually brings the system back to successful validation.

The remediation strategy may either:

- Automatically correct the validation failure
- Alert the user for manual correction of the validation failure

---

### 5. Platform Standards

- A repository may define one or multiple systems.
- All automated execution runs on GitHub Actions.
- When IaC is required, Terraform is used with HCP Terraform for state storage.
