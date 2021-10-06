# Produce cluster-connectivity reports
## About
This action produces cluster-connectivity reports for your K8s-based application. It will first extract the cluster's connectivity graph by scanning your repository for YAML files containing endpoint resources (e.g., Deployments) or connectivity resources (Kubernetes NetworkPolicies). It will then summarize the cluster connectivity in either a consice textual report or a graphical representation.

An example connectivity report (in md format):
|query|src_ns|src_pods|dst_ns|dst_pods|connection|
|---|---|---|---|---|---|
|||||||
||[default]|[app in (checkoutservice,frontend,recommendationservice)]|[default]|[productcatalogservice]|TCP 3550|
||[default]|[app in (checkoutservice,frontend)]|[default]|[shippingservice]|TCP 50051|
||[default]|[frontend]|[default]|[checkoutservice]|TCP 5050|
||[default]|[cartservice]|[default]|[redis-cart]|TCP 6379|
||[default]|[app in (checkoutservice,frontend)]|[default]|[currencyservice]|TCP 7000|
||[default]|[app in (checkoutservice,frontend)]|[default]|[cartservice]|TCP 7070|
|||ip block: 0.0.0.0/0|[default]|[frontend]|TCP 8080|
||[default]|[checkoutservice]|[default]|[emailservice]|TCP 8080|
||[default]|[frontend]|[default]|[recommendationservice]|TCP 8080|
||[default]|[loadgenerator]|[default]|[frontend]|TCP 8080|
||[default]|[frontend]|[default]|[adservice]|TCP 9555|

This action is part of a wider attempt to provide [shift-left automation for generating and maintaining Kubernetes Network Policies](https://shift-left-netconfig.github.io/).

## Inputs
### deployment-path
(Optional) The path in the GitHub workspace where deployment yamls are. Default is . (scanning the whole repository).
### netpol-path
(Optional) The path in the GitHub workspace where the NetworkPolicy yamls are stored. Default is . (scanning the whole repository).
### output-format
(Optional) Connectivity report format: either "md" (default), "yaml", "csv", "dot" or "txt".
## Outputs
### conn-results-artifact
The name of the artifact containing the connectivity report
### conn-results-file
The name of the actual file in the artifact, which contains the connectivity report
## Usage examples
### A manually-triggered action for creating a csv report (result is stored in an Action artifact)
```yaml
name: report-network-connectivity
on:
  workflow_dispatch:

jobs:
  report-connectivity:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: shift-left-netconfig/netpol-reports-gh-action@v1
        with:
          output-format: csv
```
### Automatically add a connectivity report as a PR comment
```yaml
name: report-network-connectivity
on:
  pull_request:

jobs:
  report-connectivity:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Produce connectivity report
        id: conn-report
        uses: shift-left-netconfig/netpol-reports-gh-action@v1
      - uses: actions/download-artifact@v2
        with:
          name: ${{ steps.conn-report.outputs.conn-results-artifact }}
      - name: comment PR
        uses: machine-learning-apps/pr-comment@1.0.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          path: ${{ steps.conn-report.outputs.conn-results-file }}
```
