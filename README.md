
## Overview

The examples in this repository are focused on helping new users understand what is possible with `open-cluster-management`.

These are here for reference and can be modified within the terms of the Apache2 License.

## Deploying the examples

### Policies

Most policies demonstrate how to apply various kinds of Configuration across your fleet of clusters.
There are 2 policies that have placeholder values that will need to be updated for your specific usage.

To update the `.htpasswd` configuration for the custom Authentication provider, edit the `kustomization.yaml`
and replace `TO_BE_WITH_OVERRIDDEN_BASE64_ENCODED_HTPASSWD_FILE_CONTENTS`. Protect the contents of the file
once edited as the secret is used to add authenticated users to your cluster.

Also, update `TO_BE_OVERRIDDEN_WITH_RHACM_WEB_CONSOLE_URL` with the URL for your RHACM console, for example
`https://multicloud-console.apps.kilo.demo.red-chesterfield.com/multicloud/clusters`

```yaml
patchesStrategicMerge:
- |-
  apiVersion: policy.open-cluster-management.io/v1
  kind: Policy
  metadata:
    name: policy-consolelink
    namespace: open-cluster-management-policies
  spec:
    policy-templates:
    - objectDefinition:
        apiVersion: policy.open-cluster-management.io/v1
        kind: ConfigurationPolicy
        metadata:
          name: policy-securitycontextconstraints-example
        spec:
          object-templates:
          - complianceType: musthave
            objectDefinition:
              apiVersion: console.openshift.io/v1
              kind: ConsoleLink
              metadata:
                name: acm-console-link
              spec:
                applicationMenu:
                  imageURL: https://www.vectorlogo.zone/logos/openshift/openshift-icon.svg
                href: TO_BE_OVERRIDDEN_WITH_RHACM_WEB_CONSOLE_URL
- |-
  apiVersion: policy.open-cluster-management.io/v1
  kind: Policy
  metadata:
    name: policy-auth-provider
    namespace: open-cluster-management-policies
  spec:
    object-templates:
      - complianceType: musthave
        objectDefinition:
          data:
            htpasswd: "OVERRIDDEN_BASE64_ENCODED_HTPASSWD_FILE_CONTENTS"
          kind: Secret
          metadata:
            name: htpass-secret
            namespace: openshift-config
```

```bash
kustomize build policies/ | oc apply -f -
```