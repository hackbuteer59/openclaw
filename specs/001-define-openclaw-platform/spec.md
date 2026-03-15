# Feature Specification: OpenClaw Personal Assistant Platform

**Feature Branch**: `[001-define-openclaw-platform]`  
**Created**: 2026-03-14  
**Status**: Clarified  
**Input**: User description: "Define OpenClaw as a secure single-user, multi-channel personal AI assistant platform with CLI-first onboarding, plugin-based extensibility, and companion apps across desktop and mobile."

## User Scenarios & Testing _(mandatory)_

### User Story 1 - Reach One Assistant Everywhere (Priority: P1)

As an operator, I want to talk to the same assistant through the channels and
apps I already use so I can rely on one personal assistant instead of managing
separate bots or identities.

**Why this priority**: This is the product's core promise and the reason the
project exists.

**Independent Test**: Connect at least one supported surface, send a request,
and confirm the assistant replies in that same surface while preserving the
same operator identity.

**Acceptance Scenarios**:

1. **Given** an operator has connected a supported surface (a messaging channel
   or the local terminal), **When** they send a request from that surface,
   **Then** the assistant responds in the same conversation.
2. **Given** an operator has connected more than one supported surface,
   **When** they contact the assistant from any connected surface,
   **Then** each surface reaches the same assistant identity without requiring a
   separate setup.
3. **Given** a configured surface becomes unavailable, **When** the operator
   tries to use it, **Then** the system reports that surface as unavailable and
   provides a recovery path.

---

### User Story 2 - Setup With Explicit Trust Decisions (Priority: P2)

As an operator, I want guided setup and clear permission choices so I can bring
the assistant online without losing control over sensitive capabilities.

**Why this priority**: The repo's vision prioritizes security, trust, and
first-run reliability over hidden convenience.

**Independent Test**: Start from a new installation, complete onboarding, make
required trust decisions, and reach a ready state without editing source code.

**Acceptance Scenarios**:

1. **Given** a new operator has not finished setup, **When** they start the
   onboarding flow, **Then** the system guides them through account connection,
   permissions, assistant activation, and first-use readiness.
2. **Given** a requested action requires elevated or risky capability,
   **When** that capability has not been explicitly enabled, **Then** the system
   blocks the action and asks for explicit operator approval.
3. **Given** setup or authentication fails, **When** the flow stops,
   **Then** the operator receives actionable recovery guidance instead of a dead
   end.

---

### User Story 3 - Use The Assistant Across Devices (Priority: P3)

As an operator, I want to access my assistant from desktop and mobile surfaces
in addition to messaging channels so I can keep using the same assistant in the
context that fits my day.

**Why this priority**: Cross-device access expands usefulness without changing
the assistant's identity or trust model.

**Independent Test**: Use the assistant from one messaging surface and one
companion surface, and confirm both reach the same configured assistant.

**Acceptance Scenarios**:

1. **Given** an operator has configured a companion surface,
   **When** they interact from that device, **Then** they reach the same
   assistant already available through messaging surfaces.
2. **Given** an operator switches between supported devices,
   **When** they continue using the assistant, **Then** they do not need to
   create a second assistant or duplicate setup.

---

### User Story 4 - Add Optional Capabilities Without Core Bloat (Priority: P4)

As an operator or contributor, I want optional channels, memory providers, and
skills to be add-ons so I can tailor the assistant to my needs without making
the core product heavier for everyone.

**Why this priority**: The repo explicitly keeps core lean and treats optional
capability as extension-first.

**Independent Test**: Use the baseline assistant without optional add-ons, then
enable one optional add-on and confirm the new capability appears without
breaking the baseline flow.

**Acceptance Scenarios**:

1. **Given** no optional add-ons are enabled, **When** the operator uses the
   baseline assistant, **Then** core messaging and control flows still work.
2. **Given** an operator enables an optional add-on,
   **When** the add-on becomes available, **Then** its capability appears as an
   optional extension rather than a mandatory part of core setup.

---

### Edge Cases

- A configured messaging surface is disconnected while others remain available.
- An operator starts onboarding but stops before activation is complete.
- A risky capability is requested before explicit operator approval exists.
- An optional add-on fails to load after being selected.
- The operator uses multiple surfaces in close succession and expects one
  consistent assistant identity.
- A surface's per-surface access policy changes while a conversation is in
  progress on that surface.
- A companion device loses its pairing and needs to re-establish access without
  the operator redoing the full setup.

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: The system MUST allow one operator to connect at least one
  supported interaction surface to a single assistant identity.
- **FR-002**: The system MUST let the operator send a request from any connected
  surface and receive the response in that same surface.
- **FR-003**: The system MUST preserve one shared operator identity across all
  connected surfaces so duplicate assistant setup is not required.
- **FR-004**: The system MUST provide a terminal-driven guided onboarding flow
  that takes an operator from first launch to ready-to-message state, requiring
  no manual file editing to complete setup.
- **FR-005**: The system MUST require explicit operator approval before first
  use of any capability that can affect local devices, external accounts, or
  personal contacts.
- **FR-006**: The system MUST show whether each configured surface or add-on is
  ready, unavailable, or needs operator action.
- **FR-007**: The system MUST provide recovery guidance when onboarding,
  authentication, connection, or activation fails.
- **FR-008**: The system MUST support companion access on desktop and mobile by
  allowing paired devices to connect to the same assistant identity as
  additional surfaces alongside messaging channels.
- **FR-009**: The system MUST keep baseline assistant use available when no
  optional add-ons are installed.
- **FR-010**: The system MUST allow optional add-ons to extend channels,
  memory, or skills without making them mandatory for baseline use.
- **FR-011**: The system MUST treat one gateway instance as one trusted
  operator boundary and MUST not present shared multi-user operation on the same
  instance as the standard workflow.
- **FR-012**: The system MUST continue supporting the primary assistant flow
  even when one optional surface or add-on is unavailable.
- **FR-013**: The system MUST allow the operator to set a per-surface access
  policy that controls whether that surface accepts requests from any party,
  only an approved set of known contacts, or no one.
- **FR-014**: The system MUST treat the local terminal as a supported
  interaction surface that can be used to reach the assistant directly,
  subject to the same per-surface access controls as other surfaces.

### Key Entities _(include if feature involves data)_

- **Operator**: The trusted person who installs, configures, approves, and uses
  the assistant.
- **Assistant Identity**: The single personal assistant instance the operator
  reaches across all connected surfaces.
- **Interaction Surface**: Any supported messaging channel, local terminal
  session, or paired companion app the operator uses to access the assistant.
  Each surface may have its own access policy controlling who can use it.
- **Gateway Instance**: The operator-owned control boundary that hosts the
  assistant and connected surfaces.
- **Optional Add-On**: An installable extension that adds channels, memory, or
  skills beyond the baseline assistant.
- **Capability Approval**: The explicit operator consent required before the
  assistant can use sensitive or higher-risk capabilities, such as actions that
  affect local files, run system commands, or interact with external accounts.
- **Surface Access Policy**: The operator-configured rule for each interaction
  surface that determines which parties are permitted to send requests through
  that surface.

### Assumptions

- The standard deployment is one trusted operator per gateway instance.
- Baseline value comes from secure personal use, not team tenancy or shared
  operator administration.
- Operators expect to reuse existing communication surfaces rather than learn a
  brand new primary interface.
- The local terminal is the primary operator interface during initial setup and
  ongoing administration; it is also a valid surface for interacting with the
  assistant directly.
- Optional add-ons expand capability but are never required for the first
  successful assistant conversation.

### Scope Boundaries

- In scope: personal assistant access across supported surfaces (messaging
  channels, local terminal, and companion apps), guided terminal-driven setup,
  trust controls and per-surface access policies, companion device pairing,
  and optional extensibility.
- Out of scope: enterprise multi-tenant administration, shared-team ownership,
  mandatory installation of every optional surface or add-on, and managing
  multiple independent assistant identities on one gateway.

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: A new operator can complete initial setup and reach a first
  successful assistant response in under 15 minutes.
- **SC-002**: At least 90% of successful requests sent from configured surfaces
  receive a response in that same surface on the first attempt.
- **SC-003**: An operator can actively use the same assistant from at least two
  different supported surfaces without creating duplicate assistant identities.
- **SC-004**: 100% of higher-risk capabilities require explicit operator
  approval before first use.
- **SC-005**: At least 90% of onboarding or connection failures present
  recovery guidance that allows the operator to retry without reinstalling.
- **SC-006**: Baseline assistant use remains available when optional add-ons are
  absent, disabled, or temporarily unavailable.
