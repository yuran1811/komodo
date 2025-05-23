# Builders

A builder is a machine running the Komodo Periphery agent (and usually docker), which is able to handle a RunBuild / BuildRepo command from Komodo core. Any server connected to Komodo can be chosen as the builder for a build.

Building on a machine running production software is usually not a great idea, as this process can use a lot of system resources. It is better to start up a temporary cloud machine dedicated for the build, then shut it down when the build is finished. Komodo supports AWS EC2 for this task.

## AWS builder

Builders are now Komodo resources, and are managed via the core API / can be updated using the UI.
To use this feature, you need an AWS EC2 AMI with docker and Komodo Periphery configured to run on system start.
Once you create your builder and add the necessary configuration, it will be available to attach to builds.

### Setup the instance

Create an EC2 instance, and install Docker and Periphery on it.

The following script is an example of installing Docker and Periphery onto a Ubuntu/Debian instance:
```sh
#!/bin/bash
apt update
apt upgrade -y
curl -fsSL https://get.docker.com | sh
systemctl enable docker.service
systemctl enable containerd.service
curl -sSL https://raw.githubusercontent.com/moghtech/komodo/main/scripts/setup-periphery.py | HOME=/root python3
systemctl enable periphery.service
```

:::note
AWS provides a "user data" feature, which will run a provided script as root. The above can be used with AWS user data
to provide a hands free setup.
:::

### Make an AMI from the instance

Once the instance is up and running, ssh in and confirm Periphery is running using: 

```sh
sudo systemctl status periphery.service
```

If it is not, the install hasn't finished and you should wait a bit. It may take 5 minutes or more (all in updating / installing Docker, Periphery is just a 12 MB binary to download).

Once Periphery is running, you can navigate to the instance in the AWS UI and choose `Actions` -> `Image and templates` -> `Create image`. Just name the image and hit create.

The AMI will provide a unique id starting with `ami-`, use this with the builder configuration.

### Configure security groups / firewall
The builders will need inbound access on port 8120 from Komodo Core, be sure to add a security group with this rule to the Builder configuration.
