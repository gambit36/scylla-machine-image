## Installation
```
python3 -m venv .venv3
source .venv3/bin/activate
install -r requirements.txt
```

## Testing 
``` bash
# regenerate the teample
jinja2 scylla.yaml.j2  > scylla.yaml

# lint the template
cfn-lint --template scylla.yaml --region us-east-1

# running the test
pytest --log-cli-level info
```

# Using the template
```
# run the template from command line
aws cloudformation create-stack --region eu-west-1 --stack-name fruch-test-05 --template-body file://scylla.yaml \
    --parameters  ParameterKey=KeyName,ParameterValue=scylla-qa-ec2 \
    ParameterKey=InstanceType,ParameterValue=i3.large \
    ParameterKey=AvailabilityZone,ParameterValue=eu-west-1a \
    ParameterKey=ClusterName,ParameterValue=fruch \
    ParameterKey=InstanceCount,ParameterValue=3 \
    ParameterKey=ScyllaAmi,ParameterValue=ami-0ececa5cacea302a8

```

   

