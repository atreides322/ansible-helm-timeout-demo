# ansible-helm-timeout-demo

This is a simple Helm chart and Ansible playbook to demonstrate Helm `--timeout` being used without `--wait` to accommodate long-running pre-chart hooks.

The chart simply consists of a pre-chart batch job that sleeps for six minutes and then terminates.
This is long enough to trigger Helm's default five minute timeout, causing it to fail.
If `--timeout 6m` is provided, the chart successfully installs.

<https://helm.sh/docs/intro/using_helm/#helpful-options-for-installupgraderollback>

## Requirements

You will need:

 * Ansible
 * Helm
 * Kubernetes cluster

If the `kubernetes.core` collection is not already installed, run `ansible-galaxy collection install -r requirements.yml`.

## Assumptions

This demo assumes the Kubernetes cluster has been configured in `~/.kube/config` and will install the chart into the `default` namespace.

## Tests

### Negative Helm Test

This will fail because the job takes six minutes to terminate and Helm's default duration is `5m`.

```sh
sh helm-defaults-fails.sh
```

### Positive Helm Test

This will succeed because the timeout has been set to `10m`, but the job finishes in six.

```sh
sh  helm-w-timeout-succeeds.sh
```

### Negative Ansible Test

This will fail because `kubernetes.core.helm` only adds `--timeout`, based on the `wait_timeout` module option, if `wait: yes` is specified.

```sh
sh ansible-helm-timeout-ignored.sh 
```

### Positive Ansible Test

This will succeed `wait` has been enabled.
However, there are situations where `wait` is undesirable, such as an application that has a very long startup process.
Helm's `--wait` will block waiting for all resources to become ready.

```sh
sh ansible-helm-timeout-honored.sh 
```
