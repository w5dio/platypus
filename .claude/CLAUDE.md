# CLAUDE.md — Platform Constitution

This repository contains the governing principles for the w5d.io platform — a shared platform that provides common services and resources across multiple project and product realms (weibeld, W5D Labs, Rye & Cheese). The principles defined here apply to platform system repositories in the [w5d.io](https://github.com/w5dio) GitHub organisation — not to meta-level repositories such as this one.

This is a meta-level repository. It does not implement any platform system itself — it defines the framework that governs how all platform systems are built and maintained.

The constitution is injected into Claude Code sessions in platform system repositories via a session-start hook — keep it self-contained and concise, as unnecessary length has a real cost.

## Keep principles broad

Principles must be broad enough to govern implementations across the full spectrum — from fully automatic to fully manual. Do not narrow a principle to a specific tool, technology, or workflow. Deliberate technology choices belong in a dedicated Platform Standards section.

## Keep principles actionable

Each principle should be concrete enough that Claude Code implementing a platform system can make decisions based on it. Avoid purely philosophical statements that do not translate into observable behaviour or constraints.

## Prefer fewer, more fundamental principles

Resist adding principles for every concern. Prefer a small set of principles that are genuinely governing over a longer list that includes implementation guidance or edge-case rules.

## Do not over-specify

The principles intentionally leave room for a variety of implementations. Only constrain implementation choices deliberately — choices not explicitly constrained by a principle or platform standard should remain open.
