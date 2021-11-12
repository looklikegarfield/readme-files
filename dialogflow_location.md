# Sentinel Policies for "gcp_dialogflow_dlp"

Sentinel policy files in this directory can be used with Terraform Enterprise to ensure that provisioned GCP Dialogflow agent comply with the organization's provisioning rules. This policy enforce resources to be created in the US. Global also has data-at-rest in the US.

The Policy restricts the following resources:

* google_dialogflow_cx_agent.location

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


## # Get all dialogflow Resources
```
all_df_Resources = plan.find_resources("google_dialogflow_cx_agent")

```
# Working Code to Enforce policy
The code finds the resource type "google_dialogflow_cx_agent" and enforces resources to be created in the US. Global also has data-at-rest in the US..

```
location_messages = {}
for all_df_Resources as address, rc {
	df_location = plan.evaluate_attribute(rc.change.after, "location")

	print(df_location)

	if types.type_of(df_location) is null {

		location_messages[address] = " region with value " + df_location + " which is not allowed"
		print(location_messages)

	} else {

		if df_location == global_region or strings.has_prefix(df_location, prefix) {

		} else {

			location_messages[address] = "Resource " + address + " has region other than US or GLOBAL"
			print(location_messages)

		}
	}
}

GCP_DIALOGFLOW_REGION = rule { length(location_messages) is 0 }

```
## The Main Function
This function returns "False" if length of violations is not 0.

```
main = rule { GCP_DIALOGFLOW_REGION }

```

## Testing a Policy

```
sentinel test <sentinel file>
```
#####  Example: 
```
$ sentinel test gcp_dialogflow_region.sentinel
  PASS - gcp_dialogflow_region.sentinel
  PASS - test/gcp_dialogflow_region/fail.hcl
  PASS - test/gcp_dialogflow_region/pass.hcl
```
