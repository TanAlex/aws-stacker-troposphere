# Stacker (Troposphere) to generate CloudFormation templates

The config.yml contains these stacks:
vpc, bastion and webserver

I created a target called "all" so I can group multiple stacks to a single one  
This is not necessary as in Stacker build command, if you don't provide --stacks option,  
it will build all the stacks.  

I put it here just for reference  
```yaml
targets:
  - name: all
    requires: [ vpc, myWeb ]
```

I created a custom blueprint in the blueprints/asg.py  
The things I added to it are:  
- new UserData to install nginx and custom index.html to display hostname
- AutoScalingPolicies to scale up and down based on CloudWatch Alarms for CPUUtilization
  All in the create_autoscaling_policies() function

Before running the stacker build process  
- Install proper libs by `pipenv install`
- Create a ssh-key in AWS using your public key and call it 'default'
  If you use your existing key, change the key-name in config.yml to yours
- Modify the AMI in config.yml to meet your AMI version requirement, I use CentOS7
- Change `OfficeNetwork: 162.156.1.187/32` in config.yml to your PC public IP so only you can access the bastion server. I used whatismyip service to find my own public IP


NOTE:
In the original Stacker blueprint, the ASG doesn't have 'MetricsCollection' config  
Missing that will disable CloudWatch metrics collection then it won't trigger the policy  
I added that line to the original function like below  

```python
    def get_autoscaling_group_parameters(self, launch_config_name, elb_name):
        return {
            'AvailabilityZones': Ref("AvailabilityZones"),
            'LaunchConfigurationName': Ref(launch_config_name),
            'MinSize': Ref("MinSize"),
            'MaxSize': Ref("MaxSize"),
            'MetricsCollection': [MetricsCollection(Granularity="1Minute")],
            'VPCZoneIdentifier': Ref("PrivateSubnets"),
            'LoadBalancerNames': If("CreateELB", [Ref(elb_name), ], []),
            'Tags': [ASTag('Name', self.name, True)],
        }
```

The followings are commands samples:
```bash
stacker build -r us-west-2 --stacks all -v -t config/staging.env config/config.yml
# or just
stacker build -r us-west-2 -v -t config/staging.env config/config.yml

# dump vpc stack's Cloudformation json file to a local folder called 'stacks_dump'
stacker build -r us-west-2 --stacks vpc -v -d stacks_dump -t config/staging.env config/config.yml

# To distroy
stacker destroy -r us-west-2 --stacks vpc -v -d stacks_dump -t config/staging.env config/config.yml
```