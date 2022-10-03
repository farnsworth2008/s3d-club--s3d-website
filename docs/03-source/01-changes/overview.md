---
sidebar_position: 1
---

# Changes Module
## Overview
This module allows other Terraform modules to read data about themselves.

This module depends on scripts defined in
[s3d-scripts](https://go.s3d.club/scripts).

## Usage Notes
This module wrapper around a script exists to provide output about the module
in the directory where the script is executed. The following is an example of
the data structure that the script must produce.

```json
{
  "module": "terraform-aws-ec2",
  "release": "0.1.1",
  "latest": "0.1.1-1042",
  "is_final": "false"
}
```

Automated tagging allows teams to determine what versions of their
Terraform modules were involed in the the creation of resources.

In the context of the script output above, this module produces output for
`tags` adding a `pre-release-ec2: 0.1.1` entry. The prefix `pre-` is added
rather then usin gthe actual pre-release number in order to avoid what would
otherwise be a lot of _"noise"_ in plan output that would result where every
resource that is tagged is adjusted to show the small ticks in pre-release
numbering.  Release tags do not include the `pre-` prefix and in construction
of the tag name the substring `terraform-aws` is always removed.

Using this module, resources are tagged with version information for the
module(s) that define them.  The process is additive such that an ec2 instance
that exists as a result of several modules interacting is tagged with all
modules in the chain that defined it.

```json
{
  "pre-release-root-foohat-site-group": "0.0.922",
  "pre-release-foohat-info-site": "0.1.732",
  "release-site": "0.1.4",
  "release-ec2": "0.1.1"
}
```

## Implementation notes
### Using `local.data` to structure output
This module is a wrapper around the `s3d-flow-json` script. In the current
implementation, the script reads the `CHANGES.md` to determine version
information and issues a GIt command to determine the module name. The output
of the script is JSON.

At a future point, the implementation of the module will change. We may for
example implement a Terraform provider so we can provide this data without
resorting to using the `external` provider or future versions of Terraform may
provide the functionality we require.

Regardles of implementation, the module must provide sturctured output. Our use
of the `local.data` variable ensures that output of the module contains the
output we require and nothing else. If for some reason an implementation of
`s3d-flow-json` failed to provide a required output we need an error. If
`s3d-flow-json` provided _extra_ output we want to ignore it. The structure
ensures that only outputs listed in `local.data` will be supplied as module
outputs.  The `coalesce` function throws an error if one of the outputs is
missing.

[chge]: ./CHANGES.md
[code]: ./CODE-OF-CONDUCT.md
[cont]: ./CONTRIBUTING.md
[lice]: ./LICENSE.md