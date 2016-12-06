# Vagrantfile for Protonet SOUL

## Requirements

 - [Vagrant](https://www.vagrantup.com/downloads.html) needs to be installed on your machine
 - You must install the necessary Vagrant plugins with the following command:
   - `vagrant plugin install vagrant-aws aws-sdk`
 - Your AWS credentials need to be [configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) using the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html)


## Usage

Simply run `vagrant up --provider=aws` in the main directory. Vagrant will provision a machine on your AWS account.

After it's finished, you can get the machine's address by running `vagrant ssh-config | grep HostName`.

Wait for the installation to finish (typically 20-40) and browse to that address in orded to begin the setup wizard.
