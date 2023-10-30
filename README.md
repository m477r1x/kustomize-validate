# kustomize-validate

Kustomize-validate is a validation GitHub action, designed for those using kustomize or flux automation tools to deploy changes to Kubernetes clusters.

kustomise-validate is a bash script which compares changes to objects between your default branch and your PR branch. First, kustomize-validate will check to see if a `kustomization.yaml` file exists in each directory where changes have been made, and makes you aware if it does not exist. Next, it will check if any of the objects which have been modified, exist within the `kustomization.yaml` file for that particular directory, and advise you if they do not. 

You can purposefully ignore certain files, if you specifically do not want kustomize to automatically apply some files. To ignore a file you can add the following comment anywhere in the file:

`#!kustomize-validate ignore`
