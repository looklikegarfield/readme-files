# Sentinel Policies for "google_compute_interconnect_attachment"

Sentinel policy files in this directory can be used with Terraform Enterprise to ensure that provisioned GCP Direct Interconnect resource comply with the organization's provisioning rules. These policies restrict MTU value and the region into which interconnect resources can be provisioned.

The Policy restricts the following resources:
* google_compute_interconnect_attachment.mtu
* google_compute_interconnect_attachment.region

## Pre Requistes 
Below are pre-requistes 
* `sentinel`
* `python3`


## Imports

This policy uses the tfplan-functions import, strings and types import. 

Import common-functions/tfplan-functions/tfplan-functions.sentinel with alias "plan"
```
import "tfplan-functions" as plan
import "strings"
import "types"
```


# Usage

The code has two parts:

### 1. GCP_IC_MTU

This policy checks for the mtu value, that should be 1500 and could not be a null value.

### 2. GCP_IC_REGION

This policy checks for the interconnect region.


# Working Code to Enforce policy


## The check_for_mtu Function

The policy which will iterate over all the resource type "google_compute_interconnect_attachment" and check whether the resource has recommended 1500 MTU value or not. 

```
mtu_messages = {}
for allResources as address, rc {
	uk_mtu = plan.evaluate_attribute(rc.change.after_unknown, "mtu")
	if types.type_of(uk_mtu) is "null" {
		k_mtu = plan.evaluate_attribute(rc, "mtu")
		if k_mtu is not "1500" {
			mtu_messages[address] = "Resource " + address + " has mtu with value " + k_mtu + " which is not allowed"
		}
	} else {
		mtu_messages[address] = "Resource " + address + " has default mtu which is not allowed"
	}
}
```
## The check_for_region Function

The policy which will iterate over all the resource type "google_compute_interconnect_attachment". If the resource is belonging to US then the policy will be passed, otherwise it will return violations.

```
region_messages = {}
for allResources as address, rc {
	uk_region = plan.evaluate_attribute(rc.change.after_unknown, "region")
	if types.type_of(uk_region) is "null" {
		k_region = plan.evaluate_attribute(rc, "region")
		if not strings.has_prefix(k_region,"us-") {
			region_messages[address] = "Resource " + address + " has region with value " + k_region + " which is not allowed"
		}
	} else {
		region_messages[address] = "Resource " + address + " has default region which is not allowed"
	}
}


```
## The Main Function
This function returns "False" if length of violations is not 0.

```
main = rule { GCP_IC_MTU and GCP_IC_REGION }

```

## Testing a Policy

```
sentinel test <sentinel file>
```
#####  Example: 
```
$ sentinel test google_compute_interconnect_attachment.sentinel
  PASS - google_compute_interconnect_attachment.sentinel
  PASS - test/google_compute_interconnect_attachment/fail.hcl
  PASS - test/google_compute_interconnect_attachment/pass.hcl
```