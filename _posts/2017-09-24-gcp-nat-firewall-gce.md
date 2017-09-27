# Overview

GCP really fast, and as terrform release new version, and introduce [registry](https://registry.terraform.io/)
You can download terraform modules, most of them are provided by official. for instance, like GCP, the google cloud platform team release the offical module on github.com

## Settings

As what I understand, GCE based on 3 things, server, firewall, routes.
We need to have a bastion host, and which works as nat gateway.
I'm really suffering with the infrastructure setup, unfortuantely it doesn't work. I'm trying to use the CLI way to run this up and use the offical terraform module.
So what's wrong with the settings.
It turns out the settings with firewall, you really have to set the correct source pool.

```bash
resource "google_compute_firewall" "nat-gateway" {
  name    = "nat-${var.region}"
  network = "${var.subnet_name}"

  allow {
    protocol = "all"
  }
  source_tags = "${compact(concat(list("nat-${var.region}")))}"
}
```