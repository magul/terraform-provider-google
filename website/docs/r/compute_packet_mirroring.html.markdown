---
# ----------------------------------------------------------------------------
#
#     ***     AUTO GENERATED CODE    ***    Type: MMv1     ***
#
# ----------------------------------------------------------------------------
#
#     This file is automatically generated by Magic Modules and manual
#     changes will be clobbered when the file is regenerated.
#
#     Please read more about how to change this file in
#     .github/CONTRIBUTING.md.
#
# ----------------------------------------------------------------------------
subcategory: "Compute Engine"
page_title: "Google: google_compute_packet_mirroring"
description: |-
  Packet Mirroring mirrors traffic to and from particular VM instances.
---

# google\_compute\_packet\_mirroring

Packet Mirroring mirrors traffic to and from particular VM instances.
You can use the collected traffic to help you detect security threats
and monitor application performance.


To get more information about PacketMirroring, see:

* [API documentation](https://cloud.google.com/compute/docs/reference/rest/v1/packetMirrorings)
* How-to Guides
    * [Using Packet Mirroring](https://cloud.google.com/vpc/docs/using-packet-mirroring#creating)

<div class = "oics-button" style="float: right; margin: 0 0 -15px">
  <a href="https://console.cloud.google.com/cloudshell/open?cloudshell_git_repo=https%3A%2F%2Fgithub.com%2Fterraform-google-modules%2Fdocs-examples.git&cloudshell_working_dir=compute_packet_mirroring_full&cloudshell_image=gcr.io%2Fgraphite-cloud-shell-images%2Fterraform%3Alatest&open_in_editor=main.tf&cloudshell_print=.%2Fmotd&cloudshell_tutorial=.%2Ftutorial.md" target="_blank">
    <img alt="Open in Cloud Shell" src="//gstatic.com/cloudssh/images/open-btn.svg" style="max-height: 44px; margin: 32px auto; max-width: 100%;">
  </a>
</div>
## Example Usage - Compute Packet Mirroring Full


```hcl
resource "google_compute_instance" "mirror" {
  name = "my-instance"
  machine_type = "e2-medium"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = google_compute_network.default.id
    access_config {
    }
  }
}

resource "google_compute_network" "default" {
  name = "my-network"
}

resource "google_compute_subnetwork" "default" {
  name = "my-subnetwork"
  network       = google_compute_network.default.id
  ip_cidr_range = "10.2.0.0/16"

}

resource "google_compute_region_backend_service" "default" {
  name = "my-service"
  health_checks = [google_compute_health_check.default.id]
}

resource "google_compute_health_check" "default" {
  name = "my-healthcheck"
  check_interval_sec = 1
  timeout_sec        = 1
  tcp_health_check {
    port = "80"
  }
}

resource "google_compute_forwarding_rule" "default" {
  depends_on = [google_compute_subnetwork.default]
  name       = "my-ilb"

  is_mirroring_collector = true
  ip_protocol            = "TCP"
  load_balancing_scheme  = "INTERNAL"
  backend_service        = google_compute_region_backend_service.default.id
  all_ports              = true
  network                = google_compute_network.default.id
  subnetwork             = google_compute_subnetwork.default.id
  network_tier           = "PREMIUM"
}

resource "google_compute_packet_mirroring" "foobar" {
  name = "my-mirroring"
  description = "bar"
  network {
    url = google_compute_network.default.id
  }
  collector_ilb {
    url = google_compute_forwarding_rule.default.id
  }
  mirrored_resources {
    tags = ["foo"]
    instances {
      url = google_compute_instance.mirror.id
    }
  }
  filter {
    ip_protocols = ["tcp"]
    cidr_ranges = ["0.0.0.0/0"]
    direction = "BOTH"
  }
}
```

## Argument Reference

The following arguments are supported:


* `name` -
  (Required)
  The name of the packet mirroring rule

* `network` -
  (Required)
  Specifies the mirrored VPC network. Only packets in this network
  will be mirrored. All mirrored VMs should have a NIC in the given
  network. All mirrored subnetworks should belong to the given network.
  Structure is [documented below](#nested_network).

* `collector_ilb` -
  (Required)
  The Forwarding Rule resource (of type load_balancing_scheme=INTERNAL)
  that will be used as collector for mirrored traffic. The
  specified forwarding rule must have is_mirroring_collector
  set to true.
  Structure is [documented below](#nested_collector_ilb).

* `mirrored_resources` -
  (Required)
  A means of specifying which resources to mirror.
  Structure is [documented below](#nested_mirrored_resources).


<a name="nested_network"></a>The `network` block supports:

* `url` -
  (Required)
  The full self_link URL of the network where this rule is active.

<a name="nested_collector_ilb"></a>The `collector_ilb` block supports:

* `url` -
  (Required)
  The URL of the forwarding rule.

<a name="nested_mirrored_resources"></a>The `mirrored_resources` block supports:

* `subnetworks` -
  (Optional)
  All instances in one of these subnetworks will be mirrored.
  Structure is [documented below](#nested_subnetworks).

* `instances` -
  (Optional)
  All the listed instances will be mirrored.  Specify at most 50.
  Structure is [documented below](#nested_instances).

* `tags` -
  (Optional)
  All instances with these tags will be mirrored.


<a name="nested_subnetworks"></a>The `subnetworks` block supports:

* `url` -
  (Required)
  The URL of the subnetwork where this rule should be active.

<a name="nested_instances"></a>The `instances` block supports:

* `url` -
  (Required)
  The URL of the instances where this rule should be active.

- - -


* `description` -
  (Optional)
  A human-readable description of the rule.

* `region` -
  (Optional)
  The Region in which the created address should reside.
  If it is not provided, the provider region is used.

* `priority` -
  (Optional)
  Since only one rule can be active at a time, priority is
  used to break ties in the case of two rules that apply to
  the same instances.

* `filter` -
  (Optional)
  A filter for mirrored traffic.  If unset, all traffic is mirrored.
  Structure is [documented below](#nested_filter).

* `project` - (Optional) The ID of the project in which the resource belongs.
    If it is not provided, the provider project is used.


<a name="nested_filter"></a>The `filter` block supports:

* `ip_protocols` -
  (Optional)
  Possible IP protocols including tcp, udp, icmp and esp

* `cidr_ranges` -
  (Optional)
  IP CIDR ranges that apply as a filter on the source (ingress) or
  destination (egress) IP in the IP header. Only IPv4 is supported.

* `direction` -
  (Optional)
  Direction of traffic to mirror.
  Default value is `BOTH`.
  Possible values are `INGRESS`, `EGRESS`, and `BOTH`.

## Attributes Reference

In addition to the arguments listed above, the following computed attributes are exported:

* `id` - an identifier for the resource with format `projects/{{project}}/regions/{{region}}/packetMirrorings/{{name}}`


## Timeouts

This resource provides the following
[Timeouts](/docs/configuration/resources.html#timeouts) configuration options:

- `create` - Default is 20 minutes.
- `update` - Default is 20 minutes.
- `delete` - Default is 20 minutes.

## Import


PacketMirroring can be imported using any of these accepted formats:

```
$ terraform import google_compute_packet_mirroring.default projects/{{project}}/regions/{{region}}/packetMirrorings/{{name}}
$ terraform import google_compute_packet_mirroring.default {{project}}/{{region}}/{{name}}
$ terraform import google_compute_packet_mirroring.default {{region}}/{{name}}
$ terraform import google_compute_packet_mirroring.default {{name}}
```

## User Project Overrides

This resource supports [User Project Overrides](https://www.terraform.io/docs/providers/google/guides/provider_reference.html#user_project_override).
