# Kustomize Validate

A while ago I was working with kustomize files with different teams and it became very apparent that people (myself included) kept forgetting to add items to their kustomize.yaml files, or they would forget to add the kustomization.yaml file altogether. So I thought I would create a github action to remind me (or others). I am sure that there are probably better, more efficient ways of doing this than a bash file. But I wanted something which would work accross as many linux based systems as possible without having multiple additional dependencies.

So, here we have a GitHub Action that validates Kubernetes Kustomization files to ensure all resources referenced in `kustomization.yaml` files actually exist in the target directory, and vice versa. Perfect for teams using Kustomize or Flux automation tools.

## What It Does

- ✅ Scans your repository for all `kustomization.yaml` files
- ✅ Detects directories with YAML files that are missing a `kustomization.yaml`
- ✅ Verifies that all actual files in a directory are declared in the `kustomization.yaml`
- ✅ Verifies that all declared resources in `kustomization.yaml` actually exist
- ✅ Supports intentional file ignoring with a comment flag
- ✅ Provides clear, actionable error messages
- ✅ Fails the workflow if validation errors are found

## Usage

### Basic Setup

Create a workflow file `.github/workflows/kustomize-validate.yml`:

```yaml
name: Kustomize Validation

on:
  pull_request:
    branches:
      - main
      - develop

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Validate Kustomization Files
        uses: m477r1x/kustomize-validate@main
```

## How It Works

The action recursively searches for all `kustomization.yaml` files and performs comprehensive validation:

### 1. Missing kustomization.yaml Files
Before validating files, it checks for directories containing YAML files that are missing a `kustomization.yaml`:
```
❌ Error: Directory ./apps contains YAML files but is missing kustomization.yaml
```

### 2. Files in Directory → Listed in kustomization.yaml
If a file exists in the directory but isn't listed in `kustomization.yaml`, the action will report:
```
❌ Error: myfile.yaml not listed in ./apps/kustomization.yaml
```

### 3. Resources in kustomization.yaml → Exist in Directory
If a resource is declared in `kustomization.yaml` but doesn't exist, the action will report:
```
❌ Error: declared resource 'missing.yaml' not found in ./apps
```

### 3. Successful Validation
When validation passes for a directory:
```
✅ ./apps/kustomization.yaml validated successfully
```

## Ignoring Files

Sometimes you may want to include files in a directory that shouldn't be applied by Kustomize. Mark them with the ignore comment:

```yaml
#!kustomize-validate ignore
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
```

The action will then skip validation for that file and output:
```
⏭️🙈 Ignoring myfile.yaml (marked with #!kustomize-validate ignore)
```

## Example

### Directory Structure
```
apps/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── helpers.yaml
└── kustomization.yaml
```

### Valid kustomization.yaml
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - base
  - helpers.yaml
```

### Invalid kustomization.yaml
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - base
  # missing helpers.yaml - ERROR!
```

## Features

- **Recursive validation** - Automatically finds and validates all `kustomization.yaml` files in your repository
- **Missing file detection** - Catches directories with YAML files that are missing a `kustomization.yaml`
- **Bidirectional checks** - Ensures files and declarations stay in sync
- **File ignoring** - Mark files to skip validation with a comment
- **Clear reporting** - Green ✅ checkmarks for successful dirs, red ❌ marks for errors
- **Fast** - Direct bash implementation with no external dependencies (except yq)

## Requirements

- Ubuntu runner (default GitHub Actions environment)
- `yq` is installed automatically by the action

## Exit Codes

- **0** - All validation passed
- **1** - Validation errors found

This ensures your workflow fails appropriately when kustomization files are misconfigured.

## Contributing

Found a bug or have a feature request? Feel free to open an issue or submit a pull request!

## License

MIT License - see [LICENSE](LICENSE) file for details
