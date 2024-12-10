# With one setting create hub and spoke with network virtual appliances for GCP learning

This single-setting, non-modular script is intended for learning and demonstration, allowing creation of a hub and spoke environment with full GCP network setup and a fully configured pair of Network Virtual Appliances (NVAs). This is not intended to be a production level script. The NVAs used in the script are Cisco Cloud Services Routers (CSRs). Future versions of the script may add modularity if it is needed, though the intent of the script is to have limited configurability.


Capabilities that can be demonstrated:
* Internal network load balancer as a Next Hop
* Redundant pair of NVAs fully configured
* Hub and spoke environment with multiple hubs and multiple spokes
* Failing/rebooting an NVA while routing automatically recovers

See the diagram for more architectural details: https://github.com/kurtradecki/gcp-nva-ilb-demo/blob/main/diagram.png


Process to run the script:
* Create a project in GCP, if not already. For reference if needed, see [Creating and managing projects ](https://cloud.google.com/resource-manager/docs/creating-managing-projects)
* Access the Google Cloud command line (in Cloud Shell or gcloud on your machine). If needed, see https://cloud.google.com/shell/docs/launching-cloud-shell
* At the command prompt run gcloud config set project <project_id>
  * For how to find project ID, see https://cloud.google.com/resource-manager/docs/creating-managing-projects#identifying_projects
* In Google Cloud Shell Clone this repo.
  * See https://cloud.google.com/shell/docs/version-control#clone_a_repository
* In the Google Cloud Console (GUI) Enable APIs in the project: Compute Engine, Cloud Resource Manager
  * See https://cloud.google.com/endpoints/docs/openapi/enable-api
* If needed in the Google Cloud Console (GUI) set Org policies for the project: disableSerialPortAccess (not enforced) , vmCanIpForward (selectively allow or allow all) , requireShieldedVm (not enforced) , trustedImageProjects (selectively allow or allow all), vmExternalIpAccess (selectively allow or allow all)
  * See https://cloud.google.com/resource-manager/docs/organization-policy/creating-managing-policies#creating_and_editing_policies
  * You may need to override the parent policy in the org policy for the project. 
* (APIs and Org policies are also noted in terraform.tfvars)
* Set values in terraform.tfvars such as project-id.
  * For how to find project ID, see https://cloud.google.com/resource-manager/docs/creating-managing-projects#identifying_projects
* At the command prompt run the following terraform commands:
  * `terraform init`
  * `terraform plan`
  * `terraform apply -auto-approve`

Wait ~4 minutes and your environment will be ready to explore! Here's combined set of commands to run on VMs created by the script to test reachability while reloading NVAs: 
```
ping -c 5 172.16.1.10 && ping -c 5 10.1.1.10 && ping -c 5 10.1.2.10 && ping -c 5 172.16.2.10 && ping -c 5 10.2.1.10 && ping -c 5 10.2.2.10 && ping -c 5 192.168.10.10 && ping -c 5 192.168.1.10 && ping -c 5 8.8.8.8
```

Notes about the script and environment:
* Network Virtual Appliance (NVA) can be accessed via console. Select the VM and press the "CONNECT TO SERIAL CONSOLE" button.
* Network Virtual Appliance (NVA) credentials, username: consoleadmin, password: google123 
* No Cloud NAT for untrusted VPC (or any VPCs) and Routing not allowed through appliances from untrusted to any of the other VPCs, so traffic sourced from vm-in-untrusted will fail. This VM is to test reachability from other VPCs to it. 
* Management VPC is to reach NVAs so no reachability to hub VPCs, spoke VPCs or untrusted VPC. There is a VPC Peering to Transit VPC to reach management VPC from on-prem (Transit VPC simulates connection to on-prem). 
* No interface tracking configured on NVAs, so loss of a single interface could cause extended traffic loss.
* This script creates single-regional appliances. For how to configure multi-regional appliances, see see https://medium.com/google-cloud/need-dynamic-multi-region-failover-for-network-appliances-in-google-cloud-ea968e88ca0c and check back for a link to a possible future multi-regional script.
