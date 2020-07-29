
## Overview

The examples in this repository are focused on helping new users understand what is possible with `open-cluster-management`.

All content provided in this repository is provided reference and can be modified within the terms of the Apache2 License.

## Deploying the examples

### Policies

Most policies demonstrate how to apply various kinds of Configuration across your fleet of clusters.
There are 2 policies that have placeholder values that will need to be updated for your specific usage.

#### Creating default users for your clusters

The example policies are defined to allow you to create and distribute users consistently across all of your clusters.

1. To define your desired users, follow these steps for each user:

```bash
touch htpasswd.txt
htpasswd -b -B htpasswd.txt username password
```

2. Generate the base64 encoded contents to include in the `kustomization` template (below).
```bash
cat htpasswd.txt | base64
```

#### Update your kustomization template for your environment

After you have defined your users and have the base64 encoded form of the `htpasswd` file, you can then update the `kustomization` template to parameterize your policies.

1. Copy `kustomization.yaml.template` to `kustomization.yaml`
```bash
cp policies/kustomization.yaml.template policies/kustomization.yaml
```
2. To update the `.htpasswd` configuration for the custom Authentication provider, edit the `kustomization.yaml`
and replace `TO_BE_UPDATED_BASE64_ENCODED_HTPASSWD_FILE_CONTENTS`. Protect the contents of the file
once edited as the secret is used to add authenticated users to your cluster.
3. Also, update `TO_BE_UPDATED_WITH_RHACM_WEB_CONSOLE_URL` with the URL for your RHACM console, for example
`https://multicloud-console.apps.clusterName.baseDomain/multicloud/clusters`

```yaml
...
patches:
- target:
    group: policy.open-cluster-management.io
    version: v1
    kind: Policy
    name: policy-consolelink
  patch: |-
    - op: replace
      path: /spec/policy-templates/0/objectDefinition/spec/object-templates/0/objectDefinition/spec
      value:
        applicationMenu:
          imageURL: https://www.vectorlogo.zone/logos/openshift/openshift-icon.svg
        href: TO_BE_UPDATED_WITH_RHACM_WEB_CONSOLE_URL
- target:
    group: policy.open-cluster-management.io
    version: v1
    kind: Policy
    name: policy-auth-provider
  patch: |-
    - op: replace
      path: /spec/policy-templates/0/objectDefinition/spec/object-templates/1/objectDefinition/data/htpasswd
      value: TO_BE_UPDATED_BASE64_ENCODED_HTPASSWD_FILE_CONTENTS
```

#### Apply the policies to your hub cluster

```bash
export KUBECONFIG=/path/to/hub/kubeconfig
kustomize build policies/ | oc apply -f -
```