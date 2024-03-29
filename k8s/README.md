# Chkk Kubernetes Action

Here's an example of using Chkk K8s Action, in this case to run checks on a sample Helm chart `ingress-nginx`:


```yaml
name: CI
on:
  push:
jobs:
  deployment:
    runs-on: 'ubuntu-latest'
    steps:
      - uses: actions/checkout@v1

      - uses: chkk-io/actions/k8s@main
        id: precheck
        env:
          CHKK_ACCESS_TOKEN: ${{ secrets.CHKK_ACCESS_TOKEN }}
        with:
          continue-on-failure: false

      - name: Deploy
        uses: WyriHaximus/github-action-helm3@v2.1.3
        with:
          exec: |
            helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
            helm repo update
            helm install demo-nginx ingress-nginx/ingress-nginx \
                 --atomic \
                 --version 3.35.0 \
                 --post-renderer ./chkk-post-renderer
          kubeconfig: '${{ secrets.KUBECONFIG }}'

```


The Chkk GitHub Action has properties which are passed to the underlying chkk API. These are passed to the action using `with`.

| Parameter           | Default | Description                                                  |
| :------------------ | :------ | ------------------------------------------------------------ |
| checklists          |  []     | List of checks to run              |
| suppressions        |  []     | List of checks to suppress |
| filters        |   Secret.data,Secret.data.*      | List of filters to apply on Kubernetes resource           |
| continue-on-failure       |    false     | Do not raise error in case a check fails |




## Setup Access Token in GitHub repository secrets

1. Go to the `Settings` tab in your repository menu.

![settings.png](tutorial_images/settings.png)

2. From the side menu, select `Secrets`.

![secrets_menu.png](tutorial_images/secrets_menu.png)

3. Add your Chkk access token as a secret, as shown below.

![secret_setup.png](tutorial_images/secret_setup.png)

## Examples

### Run All Available Checks
To run all available checks, configure the Action with default parameters.

```yaml
name: CI
on:
  push:
jobs:
  deployment:
    runs-on: 'ubuntu-latest'
    steps:
      - uses: actions/checkout@v1

      - uses: chkk-io/actions/k8s@main
        id: precheck
        env:
          CHKK_ACCESS_TOKEN: ${{ secrets.CHKK_ACCESS_TOKEN }}

      - name: Deploy
        uses: WyriHaximus/github-action-helm3@v2.1.3
        with:
          exec: |
            helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
            helm repo update
            helm install demo-nginx ingress-nginx/ingress-nginx \
                 --atomic \
                 --version 3.35.0 \
                 --post-renderer ./chkk-post-renderer
          kubeconfig: '${{ secrets.KUBECONFIG }}'
```

### Run Specific CRRs

To run one or more specific checks on the sample Helm chart, configure the Action with following parameter:

`checklists` : <list of comma separated checklist IDs. e.g. chkls_200bff0f-ddca-43e0-ba11-9d387dfa0410,chkls_8u8bfer2-ddca-43e0-c2t-9d674d3fa0mn98>

```yaml
name: CI
on:
  push:
jobs:
  deployment:
    runs-on: 'ubuntu-latest'
    steps:
      - uses: actions/checkout@v1

      - uses: chkk-io/actions/k8s@main
        id: precheck
        env:
          CHKK_ACCESS_TOKEN: ${{ secrets.CHKK_ACCESS_TOKEN }}
        with:
          checklists: "chkls_200bff0f-ddca-43e0-ba11-9d387dfa0410,chkls_8u8bfer2-ddca-43e0-c2t-9d674d3fa0mn98"

      - name: Deploy
        uses: WyriHaximus/github-action-helm3@v2.1.3
        with:
          exec: |
            helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
            helm repo update
            helm install demo-nginx ingress-nginx/ingress-nginx \
                 --atomic \
                 --version 3.35.0 \
                 --post-renderer ./chkk-post-renderer
          kubeconfig: '${{ secrets.KUBECONFIG }}'
```

### Suppress CRRs

To suppress one or more specific checks, configure the Action with following parameter:

`suppressions` : <list of comma separated suppression IDs. e.g. sup_200bff0f-ddca-43e0-ba11-9d387dfa0410,sup_8u8bfer2-ddca-43e0-c2t-9d674d3fa0mn98>

```yaml
name: CI
on:
  push:
jobs:
  deployment:
    runs-on: 'ubuntu-latest'
    steps:
      - uses: actions/checkout@v1

      - uses: chkk-io/actions/k8s@main
        id: precheck
        env:
          CHKK_ACCESS_TOKEN: ${{ secrets.CHKK_ACCESS_TOKEN }}
        with:
          suppressions: "sup_200bff0f-ddca-43e0-ba11-9d387dfa0410,sup_8u8bfer2-ddca-43e0-c2t-9d674d3fa0mn98"

      - name: Deploy
        uses: WyriHaximus/github-action-helm3@v2.1.3
        with:
          exec: |
            helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
            helm repo update
            helm install demo-nginx ingress-nginx/ingress-nginx \
                 --atomic \
                 --version 3.35.0 \
                 --post-renderer ./chkk-post-renderer
          kubeconfig: '${{ secrets.KUBECONFIG }}'
```




### Suppress Check Failures

To prevent the overall Action from failing in case one or more CRRs fail, configure the Action with following parameter `continue-on-failure` to be `true`

```yaml
name: CI
on:
  push:
jobs:
  deployment:
    runs-on: 'ubuntu-latest'
    steps:
      - uses: actions/checkout@v1

      - uses: chkk-io/actions/k8s@main
        id: precheck
        env:
          CHKK_ACCESS_TOKEN: ${{ secrets.CHKK_ACCESS_TOKEN }}
        with:
          continue-on-failure: true

      - name: Deploy
        uses: WyriHaximus/github-action-helm3@v2.1.3
        with:
          exec: |
            helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
            helm repo update
            helm install demo-nginx ingress-nginx/ingress-nginx \
                 --atomic
                 --version 3.35.0
                 --post-renderer ./chkk-post-renderer
          kubeconfig: '${{ secrets.KUBECONFIG }}'
```

### Filter confidential data from Kubernetes resources

To filter out confidential data like Secrets from your Kubernetes resources, configure the Action with the following parameter:

`filters` : <list of comma separated filters. e.g. ClusterRole.*,ServiceAccount.metadata.*>

```yaml
name: CI
on:
  push:
jobs:
  deployment:
    runs-on: 'ubuntu-latest'
    steps:
      - uses: actions/checkout@v1

      - uses: chkk-io/actions/k8s@main
        id: precheck
        env:
          CHKK_ACCESS_TOKEN: ${{ secrets.CHKK_ACCESS_TOKEN }}
        with:
          filters: "Secret.data.*,ServiceAccount.metadata.*"

      - name: Deploy
        uses: WyriHaximus/github-action-helm3@v2.1.3
        with:
          exec: |
            helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
            helm repo update
            helm install demo-nginx ingress-nginx/ingress-nginx \
                 --atomic \
                 --version 3.35.0 \
                 --post-renderer ./chkk-post-renderer
          kubeconfig: '${{ secrets.KUBECONFIG }}'
```


Made with 🧡 by Chkk
