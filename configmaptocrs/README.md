# Custom Resources Generator

Up until release v0.12 MetalLB was configured via MetalLB's configmap.
Deprecating the configmap impacts on the users' current configurations.
To mitigate this, the MetalLB resource generator is an offline tool to
convert the configmap into the new CRDs.

## Using the generator

### Procedure

The conversion tool is shipped as a container image under `quay.io/metallb/configmaptocrs`.

Run the container, mapping the path with the source file `config.yaml` to `/var/input` inside the container:

```bash
docker run -d -v $(pwd):/var/input quay.io/metallb/configmaptocrs -source config.yaml 
```

## Example

For this MetalLB configmap in config.yaml:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    peers:
    - my-asn: 64512
      peer-asn: 64512
      peer-address: 10.96.0.100
    address-pools:
    - name: my-ip-space
      protocol: bgp
      addresses:
      - 198.51.100.0/24
      bgp-advertisements:
      - aggregation-length: 32
        localpref: 100
        communities:
        - 64512:1234
```

The generator will generate a `resources.yaml` file that looks like:

```yaml
# This was autogenerated by MetalLB's custom resource generator.
apiVersion: metallb.io/v1beta2
kind: BGPPeer
metadata:
  creationTimestamp: null
  name: peer1
  namespace: metallb-system
spec:
  holdTime: 1m30s
  keepaliveTime: 30s
  myASN: 64512
  passwordSecret: {}
  peerASN: 64512
  peerAddress: 10.96.0.100
status: {}
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  creationTimestamp: null
  name: my-ip-space
  namespace: metallb-system
spec:
  addresses:
  - 198.51.100.0/24
status: {}
---
apiVersion: metallb.io/v1beta1
kind: BGPAdvertisement
metadata:
  creationTimestamp: null
  name: bgpadvertisement1
  namespace: metallb-system
spec:
  aggregationLength: 32
  communities:
  - 64512:1234
  ipAddressPools:
  - my-ip-space
  localPref: 100
status: {}
---

```