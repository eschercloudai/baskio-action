# Baskio - Build And Scan Kubernetes Images Openstack

A composite Action for remotely building an image using
the [eschercloud-image-builder](https://github.com/eschercloudai/image-builder) repo.
It uses [Baskio](https://github.com/eschercloudai/baskio) under the hood to build the images and scan them.

The version specified ties in with the Baskio release version.

# Scope

⚠️Currently in beta at the moment.

# Prerequisites

It is expected that you have a network and sufficient security groups in place to run this action.<br>
The action will not create the network or security groups for you.

For example:

```
openstack network create image-builder
openstack subnet create image-builder --network image-builder --dhcp --dns-nameserver 1.1.1.1 --subnet-range 10.10.10.0/24 --allocation-pool start=10.10.10.10,end=10.10.10.200
openstack router create image-builder --external-gateway public1
openstack router add subnet image-builder image-builder

OS_SG=$(openstack security group list -c ID -c Name -f json | jq '.[]|select(.Name == "default") | .ID')
openstack security group rule create "${OS_SG}" --ingress --ethertype IPv4 --protocol TCP --dst-port 22 --remote-ip 0.0.0.0/0 --description "Allows SSH access"
openstack security group rule create "${OS_SG}" --egress --ethertype IPv4 --protocol TCP --dst-port -1 --remote-ip 0.0.0.0/0 --description "Allows TCP Egress"
openstack security group rule create "${OS_SG}" --egress --ethertype IPv4 --protocol UDP --dst-port -1 --remote-ip 0.0.0.0/0 --description "Allows UDP Egress"
```

You will also require a source image to reference for the build to succeed.


<details>
<summary>Calling action example</summary>

Below is a complete example of calling this action for multiple versions.

```yaml
jobs:
  build_image:
    runs-on: ubuntu-22.04
    name: Runs the kubernetes image-builder against Openstack
    strategy:
      max-parallel: 3
      matrix:
        build-os:
          - "ubuntu-2204"
        flavor:
          - "image-builder"
          - "image-builder-with-gpu"
        version:
          - crictl: "1.25.0"
            k8s: "1.23.14"
          - crictl: "1.25.0"
            k8s: "1.24.8"
          - crictl: "1.25.0"
            k8s: "1.25.4"
        include:
          - flavor: "image-builder-with-gpu"
            gpu:
              enabled: "true"
          - flavor: "image-builder"
            gpu:
              enabled: "false"
    steps:
      - uses: actions/checkout@v3
      - id: openstack-image-builder
        uses: eschercloudai/baskio-action@v0.0.1-beta.3
        with:
          os-auth-url: ${{ secrets.os_host }}
          os-username: ${{ secrets.os_user }}
          os-password: ${{ secrets.os_password }}
          os-project-name: ${{ secrets.os_pname }}
          os-project-id: ${{ secrets.os_pid }}
          os-region-name: ${{ secrets.os_region }}
          network-id: ${{ secrets.os_network_id }}
          attach-config-drive: false
          build-os: ${{ matrix.build-os }}
          source-image-id: "SOURCE_IMAGE_ID"
          flavor-name: ${{ matrix.flavor }}
          crictl-version: ${{ matrix.version.crictl }}
          k8s-version: ${{ matrix.version.k8s }}
          enable-nvidia-support: ${{ matrix.gpu.enabled }}
          nvidia-installer-url: ${{ secrets.nvidia_url }}
          nvidia-driver-version: "510.73.08"
          grid-license-server: ${{ secrets.nvidia_grid_license_server }}
          gh-token: ${{ secrets.GITHUB_TOKEN }}
          gh-pages-branch: "gh-pages"

          # Only set these if defaults are no good
          #os-application-credential-id:
          #os-application-credential-secret:
          #os-project-domain-name:
          #os-user-domain-name:
          #os-region:
          #os-identity-api-version:
          #os-interface:
          #image-repo:
          #use-floating-ip:
          #floating-ip-network-name:
          #image-visibility:
          #gh-account:
```

</details>

See [Baskio](https://github.com/eschercloudai/baskio) for variable options.

# Usage

See [action.yml](action.yml) for required and optional inputs.

# GH-Pages

This action will generate some static website files that can be used in GitHub Pages.
Simply setup GitHub pages to point to the generated docs folder and off you go.

# TODO

* Add additional drivers etc to speed up boot time of image
* Got some images into the image to reduce image pull latency
* Probably much more!

# License

The scripts and documentation in this project are released under the [Apache v2 License](LICENSE).