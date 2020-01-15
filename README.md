# server-side-apply-exploration

This repository contains example manifests to exercise the newly added topology markers, that are used by server side apply.

For each of the markers, there's a CRD manifest that configures it, and one or more object manifests to exercise compatibility between different configurations.


## Setup
I used a 1.17 cluster to have access to all markers.

## Process
There's 2 main paths we want to explore per marker:
* How do objects behave when they're created with one configuration for a marker (e.g. `x-kubernetes-list-type: set`) and then updated with a different one (e.g. with `x-kubernetes-list-type: atomic`)?
  * To exercise that:
      * Apply the respective CRD (e.g. `list/crd-scalar-list.yml`) with the initial value (e.g. `x-kubernetes-list-type: set`)
      * Apply the initial version of the object (e.g. `list/scalar-blues.yml`)
      * Update the topology configuration of the object in the CRD (e.g. by setting `x-kubernetes-list-type: atomic`) and applying the CRD
      * See if anything has changed via `kubectl get <crd/object> -o yaml`
        * nothing did for the majority of cases
      * Apply the same or edited object spec (depending on what you're trying to exercise)

## Testing upgrade from client- to server-side apply and back
To issue client-side apply commands:
`kubectl apply -f <file>`

To issue server-side apply commands:
`kubectl apply -f <file> --server-side --field-manager=<name>`

## Examples
### Updating `x-kubernetes-list-type` from `set` to `map`?
