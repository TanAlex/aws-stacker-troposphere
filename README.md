# Stacker (Troposphere) to generate CloudFormation templates

The config.yml contains 2 stacks:
vpc and webserver

There is a "target" which is like a stacks group
```yaml
targets:
  - name: all
    requires: [ vpc, myWeb ]
```

The followings are commands samples:
```bash
stacker build -r us-west-2 --stacks all -v -d stacks_dump -t config/staging.env config/config.yml

stacker build -r us-west-2 --stacks vpc -v -d stacks_dump -t config/staging.env config/config.yml

stacker destroy -r us-west-2 --stacks vpc -v -d stacks_dump -t config/staging.env config/config.yml
```