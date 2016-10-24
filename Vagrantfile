# -*- mode: ruby -*-
# # vi: set ft=ruby :

require 'vagrant-triggers'

Vagrant.require_version ">= 1.6.0"

protonet_ami_owner = "225955013674"

def get_latest_ami(channel, owner)
	client = Aws::EC2::Client.new()
	name_glob = "protonet-bootstick-" + channel + "-*"
	filters = [
		{ name: "owner-id", values: [owner] },
		{ name: "virtualization-type", values: ["hvm"] },
		{ name: "name",	values: [name_glob] }
	]

	client.describe_images({filters: filters}).images.sort_by{ |x| x[:creation_date] }.last[:image_id]
end

# returns group's ID
def create_security_group(group_name, open_ports)
	client = Aws::EC2::Client.new()
	id = client.create_security_group({
	  group_name: group_name,
	  description: "Protonet SOUL port 22 & 80 ingress"
	})[:group_id]

	puts "Created security group '#{group_name}' with ID '#{id}'"

	begin
		open_ports.each do |port|
			client.authorize_security_group_ingress({
				group_name: group_name,
				ip_protocol: 'tcp',
				from_port: port,
				to_port: port,
				cidr_ip: '0.0.0.0/0'
			})

			puts "Authorized ingress on port #{port} for group '#{group_name}'"
		end
	rescue Exception => e
		client.delete_security_group({
		  group_name: group_name
		})
		throw e
	end

	id
end

def create_sg_if_not_exists(group_name)
	client = Aws::EC2::Client.new()
	begin
		response = client.describe_security_groups({
			group_names: [group_name]
		})
	rescue Aws::EC2::Errors::InvalidGroupNotFound => e
		create_security_group(group_name, [22, 80])
	end

	group_name
end

Vagrant.configure("2") do |config|
  config.ssh.username = "platform"
  config.vm.synced_folder '.', '/vagrant', :disabled => true
  config.vm.box = 'dummy'
  config.vm.provider :aws do |aws, override|
    aws.user_data = IO.read('cloud_config.yaml')
    aws.security_groups = [create_sg_if_not_exists('protonet-soul-ingress')]
    aws.block_device_mapping = [
      { 'DeviceName' => '/dev/xvda', 'Ebs.VolumeSize' => 8, 'Ebs.VolumeType' => 'gp2' },
      { 'DeviceName' => '/dev/xvdb', 'Ebs.VolumeSize' => 100, 'Ebs.VolumeType' => 'gp2' },
    ]
    aws.instance_type = 'm4.large'
    aws.ami = get_latest_ami('soul3', protonet_ami_owner)
		aws.tags = {:Name => 'Protonet SOUL'}
 end
end
