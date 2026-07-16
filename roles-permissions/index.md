# Sports360 — Authorization Model: Roles and Permissions

[⬅️ Back to Index](../README.md)

| Field | Value |
|---|---|
| **Version** | 1.0 |
| **Status** | Draft |
| **Author** | Ahmed Shamim — Junior Software Engineer |
| **Date** | 16 July 2026 |

---

## Overview

Proposed authorization model for Sports360 will a flexible Role-Based Access Control (RBAC) model that allows each club to manage user access according to its own organizational structure.

Instead of assigning access to individual users one by one, administrators create **roles** by grouping predefined **permissions**. Users are then assigned one or more roles, automatically receiving all permissions included in those roles.

This approach provides a scalable and easy-to-manage authorization system while allowing every club to define its own staff hierarchy.

---

## Core Concepts

### Permission

A permission represents the ability to perform a specific action within Sports360.

Examples include:

- View Players
- Create Players
- Edit Teams
- View Matches
- Manage Users
- Manage Roles

Permissions are the smallest unit of access control.

Permissions are defined and maintained by the Sports360 development team whenever new application features are introduced.

Club administrators cannot create new permissions.

---

### Role

A role is a collection of one or more permissions.

Examples of roles include:

- Administrator
- Tenant Manager
- Coach
- Accountant
- Medical Staff

Roles determine what a user is allowed to do within the application.

Each club may create its own custom roles based on its operational requirements.

---

### User

Users are assigned one or more roles.

The permissions granted to a user are automatically calculated from all assigned roles.

No permissions are assigned directly to users.

---

## Authorization Model

```text
                 Permission
            (Smallest Unit)
                    │
                    ▼
                 Role
      (Collection of Permissions)
                    │
                    ▼
                 User
          (Assigned One or More Roles)
                    │
                    ▼
         Accessible Features & Actions
```

---

## Responsibilities

### Sports360 Development Team

The development team is responsible for:

- Creating new permissions whenever new features are developed.
- Maintaining the list of available permissions.
- Protecting every feature using the appropriate permissions.

---

### Club Administrator

Each club administrator is responsible for:

- Creating custom roles.
- Selecting which permissions belong to each role.
- Assigning roles to users.
- Updating roles as club responsibilities evolve.

Administrators cannot create new permissions but have full control over how existing permissions are grouped.

---

## Example

Suppose Sports360 provides the following permissions:

```text
Players
• View Players
• Create Players
• Edit Players
• Delete Players

Teams
• View Teams
• Create Teams

Matches
• View Matches
```

The club administrator creates a role named:

```text
Tenant Manager
```

and selects the following permissions:

- View Players
- Create Players
- View Teams
- Create Teams
- View Matches

Any user assigned the **Tenant Manager** role automatically gains access to those features.

If additional permissions are later added to the role, all users assigned to that role immediately receive the updated access.

---

## User Experience

Sports360 automatically displays only the features that a user has permission to access.

For example:

- A Coach may only see Players, Teams, and Matches.
- An Accountant may only see Billing and Payments.
- An Administrator will see all administrative functions.

This reduces interface complexity while ensuring users only interact with features relevant to their responsibilities.

Even if a user attempts to access a restricted feature directly, Sports360 will prevent the action unless the required permission has been granted.

---

## Benefits

- Flexible access control for every club.
- No need for software changes when clubs create new organizational roles.
- Simple user management through reusable roles.
- Consistent security across the entire platform.
- Reduced administrative effort when onboarding or changing staff responsibilities.
- Scalable model suitable for clubs of any size.

---

## Typical Workflow

```text
Sports360 Development Team
            │
Creates new application feature
            │
Defines required permission
            │
────────────────────────────────────
            │
Club Administrator
            │
Creates a new role
            │
Selects required permissions
            │
Assigns the role to one or more users
            │
────────────────────────────────────
            │
Users automatically receive access
to the appropriate features
```

---

## Guiding Principles

- **Permissions** define what actions are available within Sports360.
- **Roles** group permissions into meaningful job responsibilities.
- **Users** receive access by being assigned one or more roles.
- Permissions are managed by the Sports360 platform.
- Roles are managed independently by each club.

