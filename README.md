
# Cloudify Environment Blueprint

Provisions infrastructure and starts a Cloudify Manager to be used with other Cloudify Examples.

_Note: Without bootstrap, deployment should take 5 minutes. With bootstrap, up to 40 minutes._

To ask a question or report an issue, please use [github issues](https://github.com/cloudify-examples/cloudify-environment-setup/issues) or visit the [Cloudify users groups](https://groups.google.com/forum/#!forum/cloudify-users).

### Purpose

Cloudify Manager can be used in any environment, whether cloud, baremetal, or a hybrid of the two. This blueprint will deploy and configure the reference environment that is used by some other examples.

### Pre-requisites

- IaaS Cloud provider and API credentials and sufficient permissions to provision network and compute resources:
  - [AWS Credentials](http://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html)
  - [Openstack Credentials](https://docs.openstack.org/user-guide/common/cli-set-environment-variables-using-openstack-rc.html) - *skip step 5 in those instructions -- do not "source" the file*.
  - [Azure Credentials](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-api-authentication)
- A virtual environment application such as [virtualenv](https://virtualenv.pypa.io/en/stable/) installed on your computer.
- [Cloudify CLI](http://docs.getcloudify.org/4.1.0/installation/from-packages/) installed in a virtual environment.

### Preparation

- You will create an inputs yaml file. Examples are provided in the `inputs` directory already if you prefer.
- Decide if you will use a pre-baked image or if you will bootstrap.

**Using a pre-baked image.**

You will find a list of pre-bootstrapped images on [Cloudify's Downloads page](http://cloudify.co/download).

  - AWS: AMIs are listed in the "aws-blueprint.yaml" under the `cloudify_ami` input.
    You may also find links [here](http://cloudify.co/thank_you_aws_ent).
    Also, change the "bootstrap: True" to "False" in your inputs file.
  - Openstack: Follow [these instructions](https://docs.openstack.org/user-guide/dashboard-manage-images.html) to upload the [Openstack QCOW image](http://cloudify.co/download) to Openstack.
    You will also need to find the correct values for cloudify_image, centos_core_image, ubuntu_trusty_image, small_image_flavor, large_image_flavor. Ask your Openstack Admin for more info on these.
  - Azure: There is not currently a pre-bootstrapped image for Azure, so bootstrap is the only option.

**Bootstrap**

To execute bootstrap, add "bootstrap: True" as a single line to your "*.yaml" file. This is the default already in the sample input files.

## Instructions

1. Install [Cloudify CLI](http://docs.getcloudify.org/4.1.0/installation/from-packages/).

2. Download and extract this blueprint archive ([link](https://github.com/cloudify-examples/cloudify-environment-setup/archive/latest.zip)) to your current working directory.

3. To install your environment's infrastructure, execute one of the example commands below, inserting your account credentials in the _*.yaml_ file located in the _inputs_ directory for your IaaS.

_Note: This command should be run from the same directory in which you extracted the blueprint in the previous step._

#### For AWS run:

```shell
$ cfy install cloudify-environment-setup-4.1/aws-blueprint.yaml -i cloudify-environment-setup-latest/inputs/aws.yaml --install-plugins
```

#### For Azure run:

```shell
$ cfy install cloudify-environment-setup-4.1/azure-blueprint.yaml -i cloudify-environment-setup-latest/inputs/azure.yaml --install-plugins
```

#### For Openstack run:

```shell
$ cfy install cloudify-environment-setup-4.1/openstack-blueprint.yaml -i cloudify-environment-setup-latest/inputs/openstack.yaml --install-plugins
```

4. Configure or Bootstrap (and then configure) your Cloudify Manager.

When the install execution has finished, you will run this command to get the deployment outputs:

```shell
$ cfy deployments outputs -b cloudify-environment-setup-latest/
```

The output should look similar to:

```json
{
  "1-Instructions": "/path/to/file/instructions.txt", 
  "2-Demo": "cfy install https://github.com/cloudify-examples/nodecellar-auto-scale-auto-heal-blueprint/archive/4.1.zip -b demo -n aws-blueprint.yaml"
}
```

### Open the "Instructions" file.

This file has been generated by the workflow and contains all of the commands you need to execute in order to configure, or bootstrap and then configure, your Cloudify Manager.

Look inside:

```shell
Step 1)
  SSH into the manager VM:

ssh -i ~/.ssh/cfy-manager-key-os centos@000.000.000.000
....
```

_The first three steps must be completed if you are executing bootstrap. The rest of the steps are followed regardless._

5. Bootstrap your manager:

_Only run this step if you are not using a pre-baked image._

#### SSH into the manager VM:

```shell
$ ssh -i ~/.ssh/cfy-manager-key-os centos@10.239.0.241
```

#### From within the manager VM, install Cloudify CLI:

```shell
$ sudo rpm -i http://repository.cloudifysource.org/cloudify/4.1.0/ga-release/cloudify-enterprise-cli-4.1.rpm
```

#### From within the manager VM, boostrap the Cloudify manager:

```shell
$ cfy bootstrap /opt/cfy/cloudify-manager-blueprints/simple-manager-blueprint.yaml -i public_ip=10.239.0.241 -i private_ip=192.168.121.5 -i ssh_user=centos -i ssh_key_filename=/home/centos/.ssh/key.pem -i agents_user=ubuntu -i ignore_bootstrap_validations=true -i admin_username=admin -i admin_password=admin
```

6. Configure your manager:

At this stage, it is suggested to wait 5 minutes for all of the services to synchronize. Both bootstrapped and pre-bootstrapped managers need a few moments to stabilize after starting.

#### Initialize the management profile:

_If you bootstrapped, exit the manager VM and execute this command on your workstation CLI._

```shell
$ cfy profiles use -s centos -k ~/.ssh/cfy-manager-key-os -u admin -p admin -t default_tenant 10.239.0.241
```

_Your manager is now ready. Proceed to the example blueprints!_

Start with [Nodecellar Auto-scale Auto-heal](https://github.com/cloudify-examples/nodecellar-auto-scale-auto-heal-blueprint/tree/4.0.1).

6. When you are ready to uninstall your environment, run:

```shell
$ cfy profiles use local
$ cfy uninstall --allow-custom-parameters -p ignore_failure=true --task-retries=30 --task-retry-interval=5
```

# Troubleshooting

## 502 Bad Gateway

- If `cfy profiles use ...` fails with this output, the service has started in error state. Try restarting the VM.

```shell
<head><title>502 Bad Gateway</title></head>
<body bgcolor="white">
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.8.0</center>
</body>
</html>
```

## [Errno 61] Connection refused

- If `cfy profiles use ...` fails with the following output, then the server is refusing your connection because of too many requests. Most likely this is an issue with your network. Expect this issue to come up a lot until your network service improves.

```shell
Attempting to connect...
HTTPConnectionPool(host='**.***.***.***', port=80): Max retries exceeded with url: /api/v3/provider/context (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x10d9d0590>: Failed to establish a new connection: [Errno 61] Connection refused',))
```

## Connection aborted, BadStatusLine

- If `cfy profiles use ...` fails with the following output, check the internet connection.

```shell
Attempting to connect...
Can't use manager 34.226.3.116. ('Connection aborted.', BadStatusLine("''",))
```

## InfluxDB wont start

- If `cfy bootstrap...` will not progress beyond influxdb, there is an issue with accessing the InfluxDB service on that port. This can be either an issue with a security group rule, or the IP may not be the primary interface.

```shell
2017-07-19 06:03:35.359  CFY <manager> [influxdb_yrqzak.create] Task started 'fabric_plugin.tasks.run_script'
[10.239.1.41] out: /tmp2017-07-19 06:03:36.959  LOG <manager> [influxdb_yrqzak.create] INFO: Installing InfluxDB...
2017-07-19 06:03:37.048  LOG <manager> [influxdb_yrqzak.create] INFO: Checking whether SELinux in enforced...
2017-07-19 06:03:37.425  LOG <manager> [influxdb_yrqzak.create] INFO: Downloading resource influxdb_NOTICE.txt to /opt/cloudify/influxdb/resources/influxdb_NOTICE.txt
2017-07-19 06:03:39.387  LOG <manager> [influxdb_yrqzak.create] INFO: Checking whether /opt/cloudify/influxdb/resources/influxdb-0.8.8-1.x86_64.rpm is already installed...
2017-07-19 06:03:39.923  LOG <manager> [influxdb_yrqzak.create] INFO: yum installing /opt/cloudify/influxdb/resources/influxdb-0.8.8-1.x86_64.rpm...
2017-07-19 06:03:41.116  LOG <manager> [influxdb_yrqzak.create] INFO: Deploying InfluxDB configuration...
2017-07-19 06:03:41.200  LOG <manager> [influxdb_yrqzak.create] INFO: Deploying blueprint resource components/influxdb/config/config.toml to /opt/influxdb/shared/config.toml
2017-07-19 06:03:42.565  LOG <manager> [influxdb_yrqzak.create] INFO: Deploying blueprint resource components/influxdb/config/cloudify-influxdb to /etc/sysconfig/cloudify-influxdb
2017-07-19 06:03:43.561  LOG <manager> [influxdb_yrqzak.create] INFO: Deploying blueprint resource components/influxdb/config/cloudify-influxdb.service to /usr/lib/systemd/system/cloudify-influxdb.service
2017-07-19 06:03:45.063  LOG <manager> [influxdb_yrqzak.create] INFO: Deploying blueprint resource components/influxdb/config/logrotate to /etc/logrotate.d/influxdb
2017-07-19 06:03:46.568  LOG <manager> [influxdb_yrqzak.create] INFO: Waiting for 192.168.121.5:8086 to become available...
2017-07-19 06:05:53.842  LOG <manager> [influxdb_yrqzak.create] INFO: 192.168.121.5:8086 is not available yet, retrying... (1/24)
[10.239.1.41] out:
[10.239.1.41] out:
[10.239.1.41] out:
```
## Invalid Block Device Mapping

- If `cfy install...` results in the below error, update the cloudify_host_block_device_mapping section to '/dev/sda1' in your input file.

```shell
<Response><Errors><Error><Code>InvalidBlockDeviceMapping</Code><Message>Invalid device name /dev/sda</Message></Error></Errors><RequestID>c1419ad6-5a27-457d-a8b7-d70c40ac8093</RequestID></Response>
```
