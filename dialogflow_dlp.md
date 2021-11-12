# Sentinel Policies for "gcp_dialogflow_dlp"

Sentinel policy files in this directory can be used with Terraform Enterprise to ensure that provisioned GCP Dialogflow agent comply with the organization's provisioning rules. This policy enforces use of DLP (security_settings) for Dialogflow agent.

The Policy restricts the following resources:

* google_dialogflow_cx_agent.security_settings

## Pre Requistes 
Below are pre-requistes 
* `sentinel`
* `python3`


## Imports

This policy uses the tfplan-functions import, strings and types import. 

Import common-functions/tfplan-functions/tfplan-functions.sentinel with alias "plan"
```
import "tfplan-functions" as plan
import "types"
```


## # Get all dialogflow Resources
```
all_df_Resources = plan.find_resources("google_dialogflow_cx_agent")

```
# Working Code to Enforce policy
The code finds the resource type "google_dialogflow_cx_agent" and ensure security_settings for Dialogflow agent should not be null.

```
dlp_messages = {}
for all_df_Resources as address, rc {
	dlp_security = plan.evaluate_attribute(rc.change.after, "security_settings")

	is_dlp_security_null = rule { types.type_of(dlp_security) is "null" }
	#print (is_dlp_security_null )

	if is_dlp_security_null is true {

		dlp_messages[address] = rc
		print(" security_settings with value "null" is not allowed ")

	} else {

		if dlp_security is not null {
            print("The value for google_dialogflow_cx_agent.security_settings is " + dlp_security )

		} else {

			dlp_messages[address] = rc
			print("Please enter correct value for parameter security_settings in google_dialogflow_cx_agent !!!")

		}
	}

}

GCP_DIALOGFLOW_DLP = rule { length(dlp_messages) is 0 }


```
## The Main Function
This function returns "False" if length of violations is not 0.

```
main = rule { GCP_DIALOGFLOW_DLP }

```

## Testing a Policy

```
sentinel test <sentinel file>
```
#####  Example: 
```
$ sentinel test gcp_dialogflow_dlp.sentinel
  PASS - gcp_dialogflow_dlp.sentinel
  PASS - test/gcp_dialogflow_dlp/fail.hcl
  PASS - test/gcp_dialogflow_dlp/pass.hcl
```
