# HPE Nimble Deployment Guide for RHOSP17

## Overview

This page provides steps to deploy & configure HPE Nimble backend driver for RHOSP17.

## Prerequisites

* Red Hat OpenStack Platform 17 with RHEL 9.

* Nimble array 6.0.0 or higher.

## Steps

### 1.  Prepare the Environment Files for cinder-volume container and nimble backend

#### 1.1 Environment File for cinder-volume container

Procedure

Create a new container images file for your overcloud:

todo: cert team to update below contents
```
openstack tripleo container image prepare default \
    --local-push-destination \
    --output-env-file containers-prepare-parameter-hpe.yaml
```

Edit the containers-prepare-parameter-hpe.yaml file.

todo: cert team to update below contents
```
parameter_defaults:
  CinderEnableIscsiBackend: false
  ControllerExtraConfig:
```


#### 1.2 Environment File for nimble backend

The environment file is an OSP director environment file. The environment file contains the settings for each backend you want to define.

Create the environment file “cinder-nimble-iscsi.yaml” under /home/stack/templates/ with below parameters and other backend details.

todo: cert team to update below contents
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

todo: cert team to update below contents
```
openstack overcloud deploy --templates /usr/share/openstack-tripleo-heat-templates \
    -e /home/stack/templates/node-info.yaml \
    -e /home/stack/containers-prepare-parameter-hpe.yaml \
    -e /home/stack/templates/cinder-nimble-iscsi.yaml \
    --ntp-server <ntp_server_ip> \
    --debug
```

### 3.  Verify the configured changes

3.1 SSH to controller node from undercloud and check the docker process for cinder-volume

todo: cert team to update below contents
```
(overcloud) [heat-admin@overcloud-controller-0 ~]$ sudo podman ps | grep cinder
56baa616ae2c  cld13b4.ctlplane.set.rdlabs.hpecorp.net:8787/rhosp-rhel9/openstack-cinder-volume:17           /bin/bash /usr/lo...  2 weeks ago  Up 2 hours ago         openstack-cinder-volume-podman-0
11777e9848c1  cld13b4.ctlplane.set.rdlabs.hpecorp.net:8787/rhosp-rhel9/openstack-cinder-scheduler:17        kolla_start           2 weeks ago  Up 2 hours ago         cinder_scheduler
5d6d06cacc19  cld13b4.ctlplane.set.rdlabs.hpecorp.net:8787/rhosp-rhel9/openstack-cinder-api:17              kolla_start           2 weeks ago  Up 2 hours ago         cinder_api_cron
4043187451d2  cld13b4.ctlplane.set.rdlabs.hpecorp.net:8787/rhosp-rhel9/openstack-cinder-api:17              kolla_start           2 weeks ago  Up 2 hours ago         cinder_api

```

3.2. Go to the controller node and execute "sudo podman exec -it openstack-cinder-volume-podman-0 bash" and verify that the backend details are visible in ```/etc/cinder/cinder.conf``` in the cinder-volume container

Given below is an example of iSCSI backend details.

todo: cert team to update below contents
```
[nimble]
image_volume_cache_enabled=True
nimble_pool_name=default
nimble_subnet_label=management
num_volume_device_scan_tries=10
san_ip=<nimble_ip>
san_login=<nimble_username>
san_password=<nimble_password>
use_multipath_for_image_xfer=True
volume_backend_name=nimble
volume_clear=zero
volume_driver=cinder.volume.drivers.hpe.nimble.NimbleISCSIDriver
backend_host=hostgroup
```

