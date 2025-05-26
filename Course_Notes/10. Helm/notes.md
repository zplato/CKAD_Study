# Helm
Section 10 notes on Helm for CKAD exam. 

Helm allow us to manage the application deployment as a whole. Helm looks at the application as a package to manage.
Based on the package name, Helm knows how to operate on those objects within that package.

Helm is analogous to an "installer" for an application or computer video game. 
Example helm command: `helm install wordpress`

Helm contains customizations for different environments:
* values.yaml file for environment specific values and variables

Helm also handles release management, version and upgrades or rollbacks 

Helm Dependencies: kubectl and kubernetes installed. 

---

## Helm Concepts
Some Deployments may have different values between different environments. 
First step is to convert the files into "templates" that are generic
values are stored in a values.yaml file. Anyone who wants to deploy the application will use values.yaml + template files

**Charts** - The combination of an environment values.yaml + template files is what makes up a helm "chart".
There is also a chart.yaml file which has information about the chart itself. The chart.yaml is mostly metadata that describes the chart. 

A common place for public helm charts is https://artifacthub.io
Can also search through cli via `helm search hub <chart>`

Add other repositories via `helm repo add <bitnami>`, where bitnami is a repo used as an example.

To install a chart `helm install <release-name> <chart-name>`, each chart has a "release", and each release has a release name. 
You can install the same chart, just with different release-name specified. 

Additional Helm Commands:
* `helm list` - to list downloaded helm packages 
* `helm uninstall <my-release>` - to uninstall a helm package of a specified release
