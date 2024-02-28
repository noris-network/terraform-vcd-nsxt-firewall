# vcd-nsxt-firewall

Terraform module which manages the NSX-T Firewall on an Edge Gateway on VMWare Cloud Director.

<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.1.9 |
| <a name="requirement_vcd"></a> [vcd](#requirement\_vcd) | >= 3.9.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_vcd"></a> [vcd](#provider\_vcd) | 3.9.0 |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [vcd_nsxt_firewall.nsxt_firewall](https://registry.terraform.io/providers/vmware/vcd/latest/docs/resources/nsxt_firewall) | resource |
| [vcd_nsxt_app_port_profile.nsxt_app_port_profile](https://registry.terraform.io/providers/vmware/vcd/latest/docs/data-sources/nsxt_app_port_profile) | data source |
| [vcd_nsxt_dynamic_security_group.nsxt_dynamic_security_groups](https://registry.terraform.io/providers/vmware/vcd/latest/docs/data-sources/nsxt_dynamic_security_group) | data source |
| [vcd_nsxt_edgegateway.nsxt_edgegateway](https://registry.terraform.io/providers/vmware/vcd/latest/docs/data-sources/nsxt_edgegateway) | data source |
| [vcd_nsxt_ip_set.nsxt_ip_sets](https://registry.terraform.io/providers/vmware/vcd/latest/docs/data-sources/nsxt_ip_set) | data source |
| [vcd_nsxt_security_group.nsxt_security_groups](https://registry.terraform.io/providers/vmware/vcd/latest/docs/data-sources/nsxt_security_group) | data source |
| [vcd_vdc_group.vdc_group](https://registry.terraform.io/providers/vmware/vcd/latest/docs/data-sources/vdc_group) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_vdc_edgegateway_name"></a> [vdc\_edgegateway\_name](#input\_vdc\_edgegateway\_name) | The name for the Edge Gateway. | `string` | n/a | yes |
| <a name="input_vdc_group_name"></a> [vdc\_group\_name](#input\_vdc\_group\_name) | The name of the VDC group. | `string` | n/a | yes |
| <a name="input_vdc_org_name"></a> [vdc\_org\_name](#input\_vdc\_org\_name) | The name of the organization to use. | `string` | n/a | yes |
| <a name="input_app_port_profiles"></a> [app\_port\_profiles](#input\_app\_port\_profiles) | Map of app port profiles used in this ruleset with their corresponding scopes. These will be looked up with a data ressource. | `map(string)` | `{}` | no |
| <a name="input_dynamic_security_group_names"></a> [dynamic\_security\_group\_names](#input\_dynamic\_security\_group\_names) | List of vcd\_nsxt\_dynamic\_security\_group names being used in this ruleset. These will be looked up with a data ressource. | `list(string)` | `[]` | no |
| <a name="input_ip_set_names"></a> [ip\_set\_names](#input\_ip\_set\_names) | List of vcd\_nsxt\_ip\_set names being used in this ruleset. These will be looked up with a data ressource. | `list(string)` | `[]` | no |
| <a name="input_rules"></a> [rules](#input\_rules) | List of rules to apply. | <pre>list(object({<br>    name                 = string<br>    direction            = string<br>    ip_protocol          = string<br>    action               = string<br>    enabled              = optional(bool)<br>    logging              = optional(bool)<br>    source_ids           = optional(list(string))<br>    destination_ids      = optional(list(string))<br>    app_port_profile_ids = optional(list(string))<br>  }))</pre> | `[]` | no |
| <a name="input_security_group_names"></a> [security\_group\_names](#input\_security\_group\_names) | List of vcd\_nsxt\_security\_group names being used in this ruleset. These will be looked up with a data ressource. | `list(string)` | `[]` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_firewall_id"></a> [firewall\_id](#output\_firewall\_id) | The ID of the firewall |
| <a name="output_firewall_rule_names"></a> [firewall\_rule\_names](#output\_firewall\_rule\_names) | The names of the firewall rules |
<!-- END_TF_DOCS -->

## Examples

### Real world example

```
locals {
    edge_firewall_rule = [
    {
        name        = "any>vms_ssh"
        direction   = "IN_OUT"
        ip_protocol = "IPV4_IPV6"
        action      = "ALLOW"
        destination_ids = [
        "webserver",
        "db",
        ]
        app_port_profile_ids = ["SSH"]
    },
    {
        name                 = "any>loadbalancer_tcp443"
        direction            = "IN_OUT"
        ip_protocol          = "IPV4"
        action               = "ALLOW"
        destination_ids      = ["loadbalancer"]
        app_port_profile_ids = ["HTTPS", "myPort"]
    }
  ]
}

module "edge_firewall" {
  source               = "git::https://github.com/noris-network/terraform-vcd-nsxt-firewall?ref=1.0.0"
  vdc_org_name         = var.vdc_org_name
  vdc_group_name       = var.vdc_group_name
  vdc_edgegateway_name = var.vdc_edge_gateway_name
  rules                = var.edge_firewall_rule
  ip_set_names = [
    "loadbalancer_test",
    "loadbalancer_prod",
    "webserver",
  ]
  app_port_profiles = {
    "SSH"    = "SYSTEM",
    "HTTPS"  = "SYSTEM",
    "myPort" = "TENANT",
  }
  depends_on = [module.vcd_nsxt_app_port_profile, module.vcd_nsxt_ip_set]
}
```

## Changelog

  * `v1.0.1`  - Implement lifecycle ignore_change rule on vdc_group_id to prevent destruction and recreation of the entire rule set upon creation of ip_set, app_port_profile or security groups
  * `v1.0.0`  - Initial release