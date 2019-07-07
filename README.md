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