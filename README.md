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
### Updating `x-kubernetes-list-type` from `set` to `atomic`
`$ kubectl apply -f list/crd-scalar-list.yml --server-side --field-manager=first`

`$ kubectl apply -f list/object-scalar-blues.yml`

`$ kubectl get colourlists.colours.example.com/colour-list -o yaml` shows that `first` is managing the individual entries in the list:
```
apiVersion: colours.example.com/v1
kind: ColourList
metadata:
  creationTimestamp: "2020-01-16T12:08:11Z"
  generation: 1
  managedFields:
  - apiVersion: colours.example.com/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:spec:
        f:colours:
          v:"navy": {}
          v:"turquoise": {}
    manager: first
    operation: Apply
    time: "2020-01-16T12:08:11Z"
  name: colour-list
  namespace: default
  resourceVersion: "219240"
  selfLink: /apis/colours.example.com/v1/namespaces/default/colourlists/colour-list
  uid: a9a29378-2ef5-435f-bc5e-0aaf13af6375
spec:
  colours:
  - turquoise
  - navy
```
Update `x-kubernetes-list-type: atomic` in `list/crd-scalar-list.yml`

`$ kubectl apply -f list/crd-scalar-list.yml`

`$ kubectl apply -f list/object-scalar-greens.yml --server-side --field-manager=second`

`$ kubectl get colourlists.colours.example.com/colour-list -o yaml` shows that the list has been entirely replaced and `second` is the new field manager, of the entire list:
```
apiVersion: colours.example.com/v1
kind: ColourList
metadata:
  creationTimestamp: "2020-01-16T12:01:04Z"
  generation: 2
  managedFields:
  - apiVersion: colours.example.com/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:spec:
        f:colours:
          v:"navy": {}
          v:"turquoise": {}
    manager: first
    operation: Apply
    time: "2020-01-16T12:01:04Z"
  - apiVersion: colours.example.com/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:spec:
        f:colours: {}
    manager: second
    operation: Apply
    time: "2020-01-16T12:01:49Z"
  name: colour-list
  namespace: default
  resourceVersion: "218400"
  selfLink: /apis/colours.example.com/v1/namespaces/default/colourlists/colour-list
  uid: c5f85b88-fc2f-4074-a8dc-13ef6016198e
spec:
  colours:
  - sage
  - lime
  - emerald
  - olive
```
