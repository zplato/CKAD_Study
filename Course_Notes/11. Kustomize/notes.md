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

---

## Managing Directories in Kustomize 
Kustomization.yaml file lives in the root of the directory, and specifies the path to subdirectories and resources within
