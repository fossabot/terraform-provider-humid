---
page_title: "Provider: Humid"
description: |-
  The humid provider is used to generate random, human friendly IDs.
---

# Humid Provider

The "humid" provider leverages the [humid package](https://github.com/kscarlett/humid) to generate random, human friendly IDs such as those used by Docker, Heroku and others. This is a *logical provider*, which means that it works entirely within Terraform's logic, and doesn't interact with any other services. As you can probably already tell, it has been modelled after the `terraform/random` provider.

Unconstrained randomness within a Terraform configuration would not be very useful, since Terraform's goal is to converge on a fixed configuration by applying a diff. Because of this, the "humid" provider provides an idea of *managed randomness*: it generate random values during creation and then holds those values steady until the inputs are changed.

## Resource "Keepers"

As noted above, the humid resource generates randomness only when it is created; the results produced are stored in the Terraform state and re-used until the inputs change, prompting the resource to be recreated.

The resource provides a map argument called `keepers` that can be populated with arbitrary key/value pairs that should be selected such that they remain the same until new random values are desired.

For example:

```terraform
resource "humid" "server" {
    keepers = {
        # Generate a new id each time we switch to a new AMI id
        ami_id = "${var.ami_id}"
    }

    list = "animals"
    adjectives = 2
    separator = "-"
    capitalize = false
}

resource "aws_instance" "server" {
    tags = {
        Name = "web-server ${humid.server.id}"
    }

    # Read the AMI id "through" the humid resource to ensure that
    # both will change together.
    ami = humid.server.keepers.ami_id

    # ... (other aws_instance arguments) ...
}
```

Resource "keepers" are optional. The other arguments to each resource must *also* remain constant in order to retain a random result.

`keepers` are *not* treated as sensitive attributes; a value used for `keepers` will be displayed in Terraform UI output as plaintext.

To force a random result to be replaced, the `taint` command can be used to produce a new result on the next run.