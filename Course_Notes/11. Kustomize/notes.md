# 11. Kustomize 

Why Kustomize? 
* We need a way to customize our deployments on a per-environment basis.
* Allows templating and avoids code duplication 

Kustomize has two key terms:
* Base/Base-config - This is essentially a template - base config that will be the same across all environments and default values 
* Overlay - Allow us to customize the behavior on a per environment basis
  * overlays/dev, overlays/prod, overlays/stg - contain what you want to override on a per env basis

**File structure**: 
```
kustomize/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── patch-deployment.yaml
│   ├── stg/
│   │   ├── kustomization.yaml
│   │   └── patch-deployment.yaml
│   └── prod/
│       ├── kustomization.yaml
│       └── patch-deployment.yaml
├── components/
```

Kustomize comes builtin with kubectl. The one that comes shipped with kubectl isn't always the latest version.
Does not require learning how to use any complex or hard to read templating system (like helm)

## Kustomization.yaml file 
Kustomization tool looks for the file called kustomization.yaml which is in the root of the path specified and from there, works down into child directories.

File contains two sections:
* resources - All the resource files that you want kustomize to manage 
* customizations - that we need to apply to those files, such as a commonLabel 

How to actually execute kustomize, we run `kustomize build <path>` - it does not apply or deploy any resources, rather it just prints the customized configs to stdout 
In order to apply those commands, you need to pipe the output to kubectl apply `kustomize build k8s/ | k apply -f -` OR utilize `kubectl apply -k <path>` 

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - api/<path>/nginx_api.yaml # Specific path to file here 
    src/<path>                # Specific path of directory that contains kustomization file 
    db/<path>                 # Specific path of directory that contains kustomization file 

# Common Transformers 
commonLabels:
  org: KodeKloud
namespace: dev
namePrefix: myapp
nameSuffix: -dev
commonAnnotations:
  branch: main

# Image Transformer
images:
  - name: nginx # Looks through all k8s configs and replaces it with a new name haproxy - This replaces the image the deployments/pods use 
    newName: haproxy # If only changing image 
    newTag: "2.4" # not necessary but can be used standalone without newName if you just want to update the tag of the image

# Patches 
patches: 
  - target:
      kind: Deployment      # Match on deployments named api-deployment 
      name: api-deployment
    # JSON 6902 Inline Patch - operation, path (of the property in the yaml file) and value 
    patch: |-
      - op: replace  
        path: /spec/replicas
        value: 5 

```

---

## Managing Directories in Kustomize 
Kustomization.yaml file lives in the root of the directory, and specifies the path to subdirectories and resources within
As the Directories list grows, we can add in a separate kustomization.yaml file to each of the subdirectories. In the root kustomization.yaml you provide the dir paths where the next level of kustomization.yaml lives

---

## Common Transformers 
Let's say we want to apply a common transformation across all of our yaml files, such as a commonLabel 
They allow us to make common transformations to all k8s resources. other common transformers include:
* namePrefix/Suffix - adds a common prefix-suffix to all resource names 
* Namespace - adds a common namespace to all resources 
* commonAnnotations - adds a annotation to all resources 

## Patches 
Kustomize Patches provide another method to modifying K8s configs
Unlike transformers, patches provide more "surgical" approaches to targeting one or more specific sections in kubernetes resources
Patches can be applied in 2 different methods, either Merge Patch or JSON 6902 Patch
For JSON 6902 patches 3 parameters must be provided:
* operationType: add/remove/replace
* target: resource for patch to be applied 
  * kind
  * version/group
  * name
  * namespace
  * labelSelector
  * AnnotationSelector
* Value: what is the value that will be replaced or added. If removing then leave this param out

Patches can also be made into separate files, such as replica-patch.yaml, where:
```yaml
- op: replace
  path: /spec/replicas
  value: 5
```
Then in the kustomization.yaml:
```yaml
patches:
  - path: replica-patch.yaml # this is what links the above 
    target: 
      kind: Deployment
      name: nginx-deployment
```

---

## Overlays 
k8s/base - Default configs 
overlays/<env> - Per environment basis. Overlays are accomplished by providing transformations and patches 

In the child kustomization.yaml files, we add a param `bases: `, which provides a list of relative paths to the parent kustomization.yaml files 
We are not forced into any specific directory structure 

---

## Components 
Define the ability to provide reusable pieces of configuration logic (resources + patches) that can be included in multiple overlays 
Components are useful in situations where applications support multiple optional features that need to be enabled only in a subset of overlays 

Components are reusable blocks of kubernetes configs, and they can be enabled in overlays 
Components can live in another path from the root, such as `/components`. Here is an example component kustomization.yaml:

```yaml
# Path k8s/components/db 
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component
resources:
  - postgres-depl.yaml 
secretGenerator:
  - name: postgres-cred
    literals:
      - password=postgres123
patches:
  - deployment-patch.yaml 
```

Then to import a component, in Overlay/<env>/kustomization.yaml:
```yaml
bases:
  - ../../<path-to-base>/base
  
components:
  - ../../<path-to-component>/db
```