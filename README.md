# HPE Alletra 6000 and Nimble Deployment Guide for RHOSP17

## Overview

This page provides steps to deploy & configure HPE Alletra 6000 and Nimble backend driver for RHOSP17.

## Prerequisites

* Red Hat OpenStack Platform 17 with RHEL 9.

* Nimble array 6.1.0 or higher.

## Steps

### 1.  Prepare the Environment Files for cinder-volume container and nimble backend

#### 1.1 Environment File for cinder-volume container

Procedure

Create a new container images file for your overcloud:

```
sudo openstack tripleo container image prepare \
    -e containers-prepare-parameter.yaml \
    --output-env-file templates/overcloud-images.yaml
```

Edit the containers-prepare-parameter.yaml file.

Sample file for iSCSI backend is available in [templates](https://github.com/hpe-storage/hpe-nimble-cinder-rhosp17/blob/master/templates) folder for reference.


#### 1.2 Environment File for nimble backend

The environment file is an OSP director environment file. The environment file contains the settings for each backend you want to define.

Create the environment file “cinder-nimble-iscsi.yaml” under /home/stack/templates/ with below parameters and other backend details.

```
parameter_defaults:
  CinderEnableIscsiBackend: false
  ControllerExtraConfig:
```

Sample file for iSCSI backend is available in [templates](https://github.com/hpe-storage/hpe-nimble-cinder-rhosp17/blob/master/templates) folder for reference.

#### Additional Help

For further details of Nimble cinder driver, kindly refer documentation [here](https://docs.openstack.org/cinder/wallaby/drivers.html#nimbleiscsidriver)

### 2.  Deploy the overcloud and configure backends

After creating ```cinder-nimble-iscsi.yaml``` file with appropriate backends, deploy the backend configuration by running the openstack overcloud deploy command using the templates option.
Use the ```-e``` option to include the environment file ```cinder-nimble-iscsi.yaml```.

The order of the environment files (.yaml) is important as the parameters and resources defined in subsequent environment files take precedence.

```
openstack overcloud deploy \
    --templates /usr/share/openstack-tripleo-heat-templates \
    --stack overcloud \
    -e /usr/share/openstack-tripleo-heat-templates/environments/cinder-backup.yaml \
    -n /home/stack/templates/network_data.yaml \
    -r /home/stack/templates/roles_data.yaml \
    -e /home/stack/overcloud-baremetal-deployed.yaml \
    -e /home/stack/overcloud-networks-deployed.yaml \
    -e /home/stack/templates/overcloud-vip-deployed.yaml \
    -e /home/stack/containers-prepare-parameter.yaml \
    -e /home/stack/templates/overcloud-images.yaml \
    -e /home/stack/templates/inject-trust-anchor-hiera.yaml \
    -e /home/stack/templates/network-environment.yaml \
    -e /home/stack/templates/custom-domain.yaml \
    -e /home/stack/templates/cinder-nimble-iscsi.yaml \
    --ntp-server <ntp_server_ip> \
    --timeout 240 \
    --config-download-timeout 240 \
    --overcloud-ssh-enable-timeout 3600 \
    --overcloud-ssh-port-timeout 3600
```

### 3.  Verify the configured changes

3.1 SSH to controller node from undercloud and check the docker process for cinder-volume

```
[root@node01 cinder]# podman ps | grep cinder
0c265e7cdc9b  c3-dl360g10-462.ctlplane.cxo.storage.hpecorp.net:8787/rhosp-rhel9/openstack-cinder-api:17.0                               kolla_start           5 days ago    Up 5 days ago (healthy)              cinder_api
85f0281ba1f7  c3-dl360g10-462.ctlplane.cxo.storage.hpecorp.net:8787/rhosp-rhel9/openstack-cinder-api:17.0                               kolla_start           5 days ago    Up 5 days ago (healthy)              cinder_api_cron
a132ae577cd7  c3-dl360g10-462.ctlplane.cxo.storage.hpecorp.net:8787/rhosp-rhel9/openstack-cinder-scheduler:17.0                         kolla_start           5 days ago    Up 5 days ago (healthy)              cinder_scheduler
a9d3ad125cba  c3-dl360g10-462.ctlplane.cxo.storage.hpecorp.net:8787/rhosp-rhel9/openstack-cinder-backup:pcmklatest                      /bin/bash /usr/lo...  5 days ago    Up 5 days ago                        openstack-cinder-backup-podman-0
8249ee9c10cf  c3-dl360g10-462.ctlplane.cxo.storage.hpecorp.net:8787/hpe3parcinder/openstack-cinder-volume-hpe3parcinder17-0:pcmklatest  /bin/bash /usr/lo...  43 hours ago  Up 43 hours ago                      openstack-cinder-volume-podman-0
```

3.2. Go to the controller node and execute "sudo podman exec -it openstack-cinder-volume-podman-0 bash" and verify that the backend details are visible in ```/etc/cinder/cinder.conf``` in the cinder-volume container

Given below is an example of iSCSI backend details.

```
[nimble]
Image_volume_cache_enabled=True
Nimble_pool_name=default
enable_unsupported_driver=True
nimble_subnet_label=Data1
nimble_iscsi_ips = <Discovery Address>
num_volume_device_scan_tries=10
san_ip=<Management IP>
san_login=admin
san_password=<admin password>
use_multipath_for_image_xfer=True
volume_backend_name=nimble
volume_clear=zero
volume_driver=cinder.volume.drivers.nimble.NimbleISCSIDriver
backend_host=hostgroup
```

