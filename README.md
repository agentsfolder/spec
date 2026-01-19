# .agents Specification
Intended status: Standard                                        

The .agents Repository Folder Specification (AGENTS-1)
Canonical Agent Configuration, Modes, Policies, and Skills

## Abstract

This document specifies a standardized repository directory, “.agents/”,
that provides a canonical, agent-agnostic configuration model for AI coding
agents. The specification defines directory structure, file formats, and a
deterministic resolution algorithm (profiles, scopes, overlays). It is
designed to enable interoperability across multiple agent implementations by
establishing a single canonical source-of-truth for project agent guidance,
safety policy, and tool/skill definitions.

## Status of This Document

This document is a work in progress. It may be updated, replaced, or
obsoleted by other documents at any time. It is inappropriate to treat this
document as final unless marked as such by the publishers.

Copyright Notice

Copyright (c) 2026. All rights reserved.

Table of Contents
1.	Introduction
2.	Conventions and Requirements Language
3.	Terminology
4.	Design Goals and Non-Goals
5.	Repository Structure
6.	Identifiers, Versioning, and Compatibility
7.	Canonical Data Model
8.	File Formats
8.1.  manifest.yaml
8.2.  prompts/*
8.3.  modes/.md (Front Matter)
8.4.  policies/.yaml
8.5.  skills/**/skill.yaml
8.6.  scopes/.yaml
8.7.  profiles/.yaml
8.8.  state/state.yaml
8.9.  schemas/
9.	Resolution Algorithm
9.1.  Inputs
9.2.  Precedence
9.3.  Merging Rules
9.4.  Scope Matching
9.5.  Conflict Handling
10.	Determinism Requirements
11.	Security Considerations
12.	Privacy Considerations
13.	Registry Considerations
14.	References

# Introduction

AI coding agents vary in how they discover repository instructions,
configuration, permissions, and tooling integration. Many ecosystems employ
agent-specific dotfolders and files, leading to multiple sources of truth
within a repository.

This document standardizes a repository folder, “.agents/”, that serves as a
single canonical source-of-truth for agent-related project guidance, safety
policy, and tool/skill definitions. The folder is intended to be consumed by
tools and agents directly or via auxiliary tooling, without requiring each
agent to define a bespoke repository structure.

# Conventions and Requirements Language

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”,
“SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “NOT RECOMMENDED”, “MAY”, and
“OPTIONAL” in this document are to be interpreted as described in RFC 2119
and RFC 8174 when, and only when, they appear in all capitals.

# Terminology

Canonical Configuration:
The effective configuration computed from “.agents/” inputs using the
resolution algorithm in Section 9.

Mode:
A named behavior profile that composes prompts, binds a policy, and
enables/disables skills.

Policy:
A declarative set of constraints and safety gates (filesystem, exec,
network, redaction, confirmations, limits).

Skill:
A declared tool or harness capability. Skills may be instruction-only,
MCP-provided tools, or CLI shims.

Scope:
A set of overrides applying to matching repository paths.

Profile:
A named overlay (e.g., “dev”, “ci”) that modifies canonical configuration.

State:
A local, non-committed selection of mode/profile/backend and other
convenience choices for developer UX. State is advisory and does not
change canonical repository configuration.

# Design Goals and Non-Goals

4.1.  Goals
	•	Interoperability: Repositories can define agent configuration once in a
canonical location.
	•	Determinism: Given the same inputs, the computed canonical configuration
MUST be identical.
	•	Monorepo support: Scopes permit path-specific behavior without duplicating
configuration folders.
	•	Explainability: Tooling can explain how the canonical configuration was
derived from inputs.
	•	Extensibility: The model supports vendor extensions without forking.

4.2.  Non-Goals
	•	This specification does not mandate any particular agent runtime or tool.
	•	This specification does not define how tools should emulate or render
alternative configurations for specific agents.
	•	This specification does not define a public registry for skills/plugins.
	•	This specification does not require automatic reverse synchronization from
non-canonical artifacts into canonical configuration.

# Repository Structure

5.1.  Required Layout

A repository implementing this specification MUST include a directory named
“.agents” at repository root.

The following files and directories are REQUIRED unless stated otherwise:

.agents/
manifest.yaml                  ; REQUIRED
prompts/                       ; REQUIRED
modes/                         ; REQUIRED
policies/                      ; REQUIRED
skills/                        ; REQUIRED (MAY be empty)
scopes/                        ; REQUIRED (MAY be empty)
profiles/                      ; REQUIRED (MAY be empty)
schemas/                       ; REQUIRED (MAY be generated)
state/                         ; REQUIRED

5.2.  State and Version Control

The “state/” directory MUST contain a “.gitignore” entry excluding
“state.yaml” from version control.

Canonical source files are those within “.agents/” excluding “schemas/” and
“state/state.yaml”. Tools MAY generate or update schema files under
“.agents/schemas/” but MUST NOT modify canonical sources unless explicitly
requested by a user.

# Identifiers, Versioning, and Compatibility

6.1.  specVersion

The manifest MUST declare a “specVersion” string. The versioning scheme
SHOULD follow Semantic Versioning (SemVer).

6.2.  IDs

Mode IDs, Policy IDs, Skill IDs, Scope IDs, and Profile IDs MUST be stable
identifiers. Implementations SHOULD restrict IDs to the following pattern:

id = ALPHA / DIGIT *( ALPHA / DIGIT / “.” / “_” / “-” )

Case sensitivity SHOULD be treated as significant.

6.3.  Forward Compatibility

Unknown fields in “x” namespaces (Section 7.5) MUST be preserved during
processing. Unknown top-level fields outside “x” namespaces MUST cause
validation failure unless explicitly allowed by the schema version in use.

# Canonical Data Model

7.1.  Components

Canonical configuration comprises:
	•	Effective Mode
	•	Effective Policy
	•	Enabled Skills
	•	Prompt Composition (base, project, snippets)
	•	Matched Scopes
	•	Active Profile

7.2.  Policy Semantics

Policy is a set of constraints. Where enforcement is feasible, tooling SHOULD
enforce; otherwise tooling MUST represent policy as explicit guidance.

“Deny overrides allow” MUST be the default behavior for:
	•	denied paths vs allowed paths
	•	denied hosts vs allowed hosts
	•	denied commands vs allowed commands

7.3.  Skill Activation

Skills MUST declare one activation mode:
	•	instruction_only
	•	mcp_tool
	•	cli_shim

Tooling that executes or exposes skills MUST define the operational meaning
of activation. This document only standardizes the declaration.

7.4.  Scopes

Scopes provide path-based overrides. Scopes MUST NOT redefine specVersion.

7.5.  Extensions

Any component MAY include an “x” object permitting arbitrary key/value
extensions. Extension keys SHOULD be vendor-namespaced (e.g., “x.vendor.*”).

# File Formats

8.0.  General Requirements

Canonical configuration files MUST be encoded in UTF-8. YAML files SHOULD be
YAML 1.2. Markdown files SHOULD be CommonMark compliant.

8.1.  manifest.yaml

The manifest file MUST exist at “.agents/manifest.yaml” and MUST include:
	•	specVersion (string)
	•	defaults (object): mode (string), policy (string), optional profile
	•	enabled (object): arrays of modes, policies, skills

The manifest MAY include:
	•	project metadata (name, description, languages, frameworks)
	•	resolution options (enableUserOverlay, denyOverridesAllow, onConflict)
	•	profiles map (optional)
	•	x extensions

8.2.  prompts/*

The “prompts/” directory MUST exist and MUST contain:
	•	base.md        ; REQUIRED
	•	project.md     ; REQUIRED
	•	snippets/      ; REQUIRED (MAY be empty)

Prompt files MUST be Markdown. Tools SHOULD treat the content as plain text
instructions to be composed by modes and scopes.

8.3.  modes/*.md (Front Matter)

Each mode MUST be a Markdown file in “.agents/modes/”. A mode file MAY include
YAML front matter. If present, it MUST validate against the Mode Front Matter
schema and SHOULD include:
	•	id
	•	title
	•	policy
	•	enableSkills / disableSkills
	•	includeSnippets
	•	toolIntent (allow/deny)

The Markdown body MUST represent human-readable instructions for that mode.

8.4.  policies/*.yaml

Each policy file MUST include:
	•	id
	•	description
	•	capabilities: filesystem, exec, network, mcp (each optional sub-objects)
	•	paths: allow, deny, redact
	•	confirmations: requiredFor
	•	limits (optional)
	•	x (optional)

8.5.  skills/*/skill.yaml

Each skill MUST reside in “.agents/skills//skill.yaml” and include:
	•	id, version, title, description
	•	activation
	•	interface (type, entrypoint, args/env optional)
	•	contract (inputs/outputs schema fragments)
	•	requirements (capabilities and path needs/writes)
	•	assets (mount/materialize lists optional)
	•	compatibility (optional)
	•	x (optional)

8.6.  scopes/*.yaml

Each scope file MUST include:
	•	id
	•	applyTo: array of glob patterns
	•	overrides: mode/policy and skill/snippet adjustments

Scope patterns MUST use doublestar (”**”) glob semantics consistent across
tooling. If the implementation uses a different glob engine, it MUST document
any deviations.

8.7.  profiles/*.yaml

Profiles are named overlays. Profile files SHOULD define overrides that merge
into the canonical configuration. The overlay shape MUST be deterministic and
validated by schema.

8.8.  state/state.yaml

“.agents/state/state.yaml” is OPTIONAL and MUST be excluded from version
control. If present, it MUST include at least:
	•	mode
and MAY include:
	•	profile, backend, scopes

State MUST NOT change canonical configuration; it only selects among defined
canonical options.

8.9.  schemas/*

“.agents/schemas/” contains JSON Schema definitions used by validation.
Tooling MAY generate these files. If present, schemas MUST correspond to the
specVersion declared in manifest.yaml.

# Resolution Algorithm

9.1.  Inputs

The resolution algorithm consumes:
	•	Repo canonical inputs under “.agents/”
	•	Optional user overlay (e.g., “~/.agents/”) if enabled
	•	Optional state selection (”.agents/state/state.yaml”)
	•	Tool and CLI overrides (implementation-specific; treated as highest priority)

9.2.  Precedence

Effective configuration MUST be computed in the following order (highest
priority first):
	1.	Tool/CLI overrides (if any)
	2.	Repo “.agents/” base configuration
	3.	Matched scopes (most specific wins; see Section 9.4)
	4.	User overlay (if enabled)

If a state selection is present, it MAY be used to choose a mode/profile
within the canonical configuration, but MUST NOT override canonical values.

9.3.  Merging Rules

Merging MUST be deterministic and MUST preserve stable ordering.
	•	Scalars: higher precedence overwrites lower precedence.
	•	Arrays: MUST be merged by stable union unless explicitly marked as
replace-only by schema for that field.
	•	Policies: deny lists MUST override allow lists.
	•	Unknown “x” extensions: MUST be preserved and merged by deep-merge.

9.4.  Scope Matching

A scope matches a repository path if any applyTo glob matches. If multiple
scopes match, implementations MUST choose the most specific scope using a
deterministic specificity metric. If specificity ties, “priority” (if
present) MUST break ties; remaining ties MUST be resolved by lexicographic
order of scope id.

9.5.  Conflict Handling

If “resolution.onConflict” is “error” (default), ambiguous references (e.g.,
unknown mode id, unknown policy id, duplicate skill id definitions) MUST fail
validation and MUST prevent use of the configuration. If “warn”, tooling MAY
proceed but MUST produce a machine-readable warning report.

# Determinism Requirements

Conforming tooling MUST:
	•	compute identical canonical configuration for identical inputs
	•	use stable sorting for maps and arrays where order is not semantically
meaningful
	•	serialize JSON and YAML deterministically when producing derived views
(if any), even though such views are outside the scope of this document

# Security Considerations

The “.agents” model can declare capabilities to agents (execution, network
access, file writes). Tools SHOULD:
	•	default to conservative policies (confirm destructive operations)
	•	support redaction patterns to reduce accidental leakage of secrets into
prompts or logs
	•	warn when enabling network or unrestricted execution
	•	treat user overlays as potentially sensitive and avoid writing secrets into
repository-scoped guidance unless explicitly configured

# Privacy Considerations

Tools should avoid collecting or transmitting repository contents unless
explicitly requested. If telemetry exists, it MUST be opt-in and MUST NOT
include source content by default.

# Registry Considerations

This specification does not define a public registry for skills/plugins.
Tools MAY implement private registries or distribution mechanisms; such
mechanisms are out of scope.

# References
TODO: fix references