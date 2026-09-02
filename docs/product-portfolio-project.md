# Product Portfolio — GitHub Project blueprint

This document defines the standard Evergreen-IT Product Portfolio project. GitHub Projects does not support a native project-template YAML file, so create the project once using this blueprint and then mark that project as an organization template.

## Purpose

Use Product Portfolio for product, internal-tool, experiment, and research ideas before they become engineering delivery work.

Do not create a repository for an idea until it is validated and there is a concrete reason to start delivery.

## Status workflow

Use a single-select field named `Status` with these values in this order:

1. Idea
2. Exploring
3. Validated
4. Planned
5. Building
6. Review
7. Done
8. Rejected

## Custom fields

Create these fields:

| Field | Type | Values / purpose |
| --- | --- | --- |
| Status | Single select | Idea, Exploring, Validated, Planned, Building, Review, Done, Rejected |
| Priority | Single select | P0, P1, P2, P3 |
| Owner | Assignee / people | Person accountable for the initiative |
| Product | Text | Product or initiative name |
| Type | Single select | Product, Internal Tool, Experiment, Research |
| Effort | Single select | XS, S, M, L, XL |
| Confidence | Single select | Low, Medium, High |
| Target date | Date | Optional target or decision date |
| Repository | Repository | Add only when a repository exists |

## Views

### 1. Ideas

- Layout: Table
- Filter: `Status:Idea`
- Show: Title, Type, Owner, Priority, Confidence, Created date
- Sort: newest first

### 2. Discovery

- Layout: Board
- Group by: Status
- Filter: `Status:Exploring,Validated`
- Show: Owner, Priority, Confidence, Effort

### 3. Roadmap

- Layout: Roadmap
- Filter: `Status:Validated,Planned,Building,Review`
- Date field: Target date
- Group by: Product or Type

### 4. Delivery

- Layout: Board
- Group by: Status
- Filter: `Status:Planned,Building,Review`
- Show: Owner, Priority, Effort, Repository

### 5. All work

- Layout: Table
- No status filter
- Show all key fields

## Operating rules

### Idea

Capture the problem, target user, proposed solution, expected value, evidence, and unknowns. Avoid premature architecture or implementation tasks.

### Exploring

Gather evidence. Validate the problem, target user, market or internal need, feasibility, major risks, and expected value.

### Validated

There is enough evidence to justify planning. Define the smallest useful outcome and decide whether a repository or delivery project is required.

### Planned

Delivery has an agreed scope, owner, and priority. Create concrete GitHub Issues and dependencies here or in the product repository.

### Building

Implementation is actively underway. Engineering work should be represented by real Issues and Pull Requests rather than draft ideas.

### Review

The initiative or release is awaiting product, technical, security, user, or stakeholder validation.

### Done

The intended outcome is delivered or the discovery objective is complete.

### Rejected

The idea was deliberately stopped. Record the reason so the same idea is not repeatedly rediscovered without new evidence.

## Idea intake

Use the organization-wide `Product idea` issue form from `Evergreen-IT/.github` when an idea belongs in a repository. For ideas that do not yet belong to any repository, create a draft item directly in Product Portfolio using the same structure.

## Promotion to delivery

When an idea reaches `Validated`:

1. Define the smallest useful deliverable.
2. Decide whether an existing repository owns it.
3. Create a repository only if the initiative genuinely needs one.
4. Create a parent Issue for the initiative or first release.
5. Break delivery into executable Issues.
6. Link Pull Requests to Issues.
7. Move the portfolio item to `Planned` and then `Building` when implementation starts.

## Template setup

After creating the Product Portfolio project and configuring the fields, views, and built-in workflows above, mark the project as an organization project template. Future portfolio-style projects should be created from that template rather than rebuilt manually.
