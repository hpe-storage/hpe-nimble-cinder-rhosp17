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
    --timeout 240 \
    --config-download-timeout 240 \
    --overcloud-ssh-enable-timeout 3600 \
    --overcloud-ssh-port-timeout 3600
```

### 3.  Verify the configured changes

- Run "openstack volume service list" on the overcloud, and verify the Nimble backend is up
- Verify a volume can be created

