---
layout: default
title: Standards
nav_order: 1
parent: Organization
permalink: /organization/standards
---

# Organization Standards

## Trello

When creating a board:

- Copy the template from this [board](https://trello.com/b/p2iycBpD)
- Add the `Scaled by Screenful` plugin, which adds functionality for:
    - Defining epics.
    - Adding dependencies or blocks between tasks.
    - Priorities and estimates.
- For epics, add a column only with the epics.

## Discord

When starting a new project:

- Create a new role for the project.
- Create a category and edit the permissions:
    - Remove the `view` permission for the role `everyone`.
    - Add permission for the role and include viewing.

## Clockify

When creating a new project:

- Create the corresponding client if it's not registered.
- Project color, if there's a distinctive color that already exists such as the project logo's color or the primary color of the client, use that. If not, use one that references the project's industry.
- If a total development estimate in hours was made, load it.
- Provide access to the project for each user who needs it. And if a price per hour was agreed upon, load it in the access table.
- Always use tags whenever possible for tracking.

## GitHub

- Repository names are in `kebab-case`. First, the project identifier, followed by the role identifier in the system, for example:
    - bavastro-backend
    - signacheck-desktop
    - falcode-web
- Handle access based on teams.
- Add the admin group as admins to the repo.
- Individual repository accesses are for exceptional cases of clients who want to see the code and shouldn't be in the organization for any other reason. They only have reading access to the repo directly.
- Create `main`, `staging`, `develop` branches
- Protect and manage permissions on the `main` and `staging` branches
