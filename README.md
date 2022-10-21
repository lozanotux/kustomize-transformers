# Kustomize Transformers

In this repository you can found an use case of `Kustomize Transformers` in action to deploy an application using **ArgoCD** with a Blue/Green strategy.

> **NOTE:** for more information about transformers you can see next link: [https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/). And you can check the file [openshift/overlays/prod/blue/kustomization.yaml](./openshift/overlays/prod/blue/kustomization.yaml) to see two key concepts.

## Pre-requisites

To replicate this exercise you need:
- A kubernetes/openshift cluster
- A running instance of ArgoCD
- This repository

## How to use it?

1. Open the **ArgoCD** web console and create the blue application first. For this, complete the form with the next values:
   - **Application Name**: argocolors-app-prod-blue
   - **Project Name**: default
   - **Sync Policy**: Automatic
   - **Marked Options**:
      - Apply Out Of Sync Only
   - **Repository URL**: https://github.com/lozanotux/kustomize-transformers.git
   - **Revision**: main
   - **Path**: openshift/overlays/prod/blue
   - **Cluster URL**: https://kubernetes.default.svc
   - **Namespace**: `Choose your own namespace`
2. Repeat the previous step, but create the green application instead. Complete the form with the next values:
   - **Application Name**: argocolors-app-prod-green
   - **Project Name**: default
   - **Sync Policy**: Automatic
   - **Marked Options**:
      - Apply Out Of Sync Only
   - **Repository URL**: https://github.com/lozanotux/kustomize-transformers.git
   - **Revision**: main
   - **Path**: openshift/overlays/prod/green
   - **Cluster URL**: https://kubernetes.default.svc
   - **Namespace**: `Choose your own namespace`
3. Then, is time to create the **Route** of the application. Maybe, in a CI/CD scenario this can achieved with a Tekton Task instead of manually. So, run the next `oc` command from a command line:
   ```bash
   $ oc create route edge argocolors-app-bluegreen --service=argocolors-app-blue
   ```

   > **NOTE:** you can now open the resulting URL in a web browser to see **argocolors-app** in action. To know wich is the resulting URL you can run the next command: `oc get route argocolors-app-bluegreen -o template --template=https://{{.spec.host}}`
4. To change from blue to green application version, you can edit your `Route` object to point to corresponding service. To do this with `oc` tool, you can the next command:
   ```bash
   $ oc set route-backends argocolors-app-bluegreen argocolors-app-green=100
   ```

   > **NOTE:** to revert the changes, simply modify the name of the service at the end of the command.