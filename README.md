# Baskio - Build And Scan Kubernetes Images Openstack

A composite Action for remotely building an image using the [eschercloud-image-builder](https://github.com/eschercloudai/image-builder) repo.
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

When referencing this action you only need to provide the following - changing any variables as required.
```
{
  "source_image": "SOURCE_IMAGE_ID",
  "flavor": "INSTANCE_FLAVOR",
  "floating_ip_network": "IP_NETWORK_NAME",
}
```
See [Baskio](https://github.com/eschercloudai/baskio) for more variable options.

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