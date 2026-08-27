---
name: naming-conventions
description: Naming rules for new Terraform templates in this project. Use whenever creating a new Terraform template, module, or .tf file that declares resources — asks the user for a base name and appends the mandatory "-loihde" suffix to every resource name.
---

# Terraform naming conventions

## When this applies

Any time you are about to create a new Terraform template — a new `.tf` file, a new module, or a set of resources added to an existing template. It does not apply when only editing logic in resources that already exist and already follow the convention.

## Step 1 — ask for the base name

Before writing any Terraform, ask the user what to use as the base name for all resources in the template. Ask with AskUserQuestion where a small set of sensible candidates can be inferred (repo name, directory name, workload name), otherwise ask in plain text. Do not guess and proceed — the base name is the user's call, and every name in the template depends on it.

If the user has already given the base name in their request, use it and skip the question.

## Step 2 — append `-loihde` to every name

Every resource name carries the `-loihde` suffix. The pattern is:

```
<base-name>[-<qualifier>]-loihde
```

- `<base-name>` is what the user supplied in step 1.
- `<qualifier>` is optional and distinguishes multiple resources of the same or different kind (`-web`, `-db`, `-primary`, `-01`). Add it only when the template has more than one resource that would otherwise collide.
- `-loihde` is always last, on every name, with no exceptions.

Apply it to both the Terraform block labels and the actual cloud resource names/tags:

```hcl
resource "azurerm_resource_group" "main" {
  name     = "payments-loihde"
  location = var.location
}

resource "azurerm_storage_account" "logs" {
  name                = "payments-logs-loihde"
  resource_group_name = azurerm_resource_group.main.name
  # ...
}
```

Prefer wiring the base name through a variable so the suffix is applied in one place:

```hcl
variable "base_name" {
  description = "Base name for all resources in this template."
  type        = string
}

locals {
  name = "${var.base_name}-loihde"
}
```

Then reference `local.name` (or `"${var.base_name}-<qualifier>-loihde"`) rather than repeating literals.

## Constraints to watch

Some providers restrict names in ways that fight the suffix. Handle these explicitly rather than silently dropping `-loihde`:

- **No hyphens allowed** (e.g. Azure storage accounts, which are lowercase-alphanumeric only): use `loihde` as a plain suffix — `paymentslogsloihde`.
- **Length limits**: shorten the qualifier or the base name, never the `-loihde` suffix. If the base name itself cannot fit, say so and ask the user for a shorter one.

## Before finishing

Re-read the template and confirm every resource name ends in `-loihde` (or the hyphen-free variant). Mention the base name you used in your summary to the user.
