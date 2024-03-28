## CloudFormation Template

There're 2 ways to create a tempalte:
- using Designer function on CloudFormation Web Service
- create a plain .yml template

This repository will forcus on creating CloudFormation template file

### Parameters

We can define template parameters as following:
```
Parameters:
    InstanceTypeParameter:
        Type: String
        Default: t2.micro
        AllowedValues:
        - t2.micro
        - m1.small
        - m1.large
        Description: Enter t2.micro, m1.small, or m1.large. Default is t2.micro.
```
InstanceTypeParameter: name of parameter
Type: parameter type, which can support types as String, Number, List<Number>, CommaDelimitedList, AWS-Specific Parameter Types or SSM Parameter Types (see this [link](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html) for more infomation)

Some requirements to takenote when using parameters in template:
- You can have a maximum of 200 parameters in an AWS CloudFormation template.
- Parameter name must be unique within the template
- Each parameter must be assigned a parameter type that is supported by AWS CloudFormation.
- Each parameter must be assigned a value at runtime for AWS CloudFormation to successfully provision the stack
- Parameters must be declared and referenced from within the same template.

You use the "Ref" intrinsic function to reference a parameter, for example
```
"Ec2Instance" : {
  "Type" : "AWS::EC2::Instance",
  "Properties" : {
    "InstanceType" : { "Ref" : "InstanceTypeParameter" },
    "ImageId" : "ami-0ff8a91507f77f867"
  }
}
```

We can also using dynamic references to specify template values. Currently, CloudFormation support these dynamic reference sources:
- ssm:  access plaintext in AWS Systems Manager Parameter Store
- ssm-secure: access secure strings in AWS Systems Manager Parameter Store
- secretsmanager: access entire secrets or secret values stored in AWS Secrets Manager


### Rule functions:

1. Fn::And

Syntax
> "Fn::And" : [{condition}, {...}]

Return true if all specified conditions is True.

Example
```
"Fn::And": [
  <!-- condition 1 -->
  {
    "Fn::Equals": [
      "sg-mysggroup",
      {"Ref": "ASecurityGroup"}
    ]
  },
  <!-- condition 2 -->
  {
    "Fn::Contains": [
      [
        "m1.large",
        "m1.small"
      ],
      {"Ref": "InstanceType"}
    ]
  }
]
```

2. Fn::Contains

Syntax:
> "Fn::Contains" : [[list_of_strings], string]
Return true if "list_of_strings" contains "string"

Example

```
"Fn::Contains" : [
  ["m1.large", "m1.small"], {"Ref" : "InstanceType"}
]
```
3. Fn::EachMemberEquals

Syntax:
> "Fn::EachMemberEquals" : [[list_of_strings], string]
Return true if all member of list_of_strings are string

Example
```
"Fn::EachMemberEquals" : [
  {"Fn::ValueOfAll" : ["AWS::EC2::VPC::Id", "Tags.Department"]}, "IT"
]
```

4. Fn::EachMemberIn

Syntax
> "Fn::EachMemberIn" : [[strings_to_check], [strings_to_match]]
Returns true if each member in a list of strings matches at least one value in a second list of strings.

Example
```
Returns true if each member in a list of strings matches at least one value in a second list of strings.
```

5. Fn::Equals
Syntax
> "Fn::Equals" : ["value_1", "value_2"]
Returns true if the two values are equal and false if they aren't.

Example
```
"Fn::Equals" : [{"Ref" : "EnvironmentType"}, "prod"]
```

6. Fn::Not

Syntax
> "Fn::Not" : [{condition}]

Returns true for a condition that evaluates to false.

Example
```
"Fn::Not" : [{"Fn::Equals" : [{"Ref" : "EnvironmentType"}, "prod"]}]
```

7. Fn::Or

Syntax:
> "Fn::Or" : [{condition}, {...}]
Returns true if any one of the specified conditions evaluates to true; returns false if all the conditions evaluate to false. Working as OR operator

Example
```
"Fn::Or" : [
  {"Fn::Equals" : ["sg-mysggroup", {"Ref" : "ASecurityGroup"}]},
  {"Fn::Contains" : [["m1.large", "m1.small"], {"Ref" : "InstanceType"}]}
]
```
8. Fn::RefAll

Syntax
> "Fn::RefAll" : "parameter_type"
Returns all values for a specified AWS parameter type.

Example
```
<!-- The following function returns a list of all VPC IDs for the Region and AWS account in which the stack is being created or updated: -->
"Fn::RefAll" : "AWS::EC2::VPC::Id"
```

9. Fn::ValueOf

Syntax:
> "Fn::ValueOf" : [ "parameter_logical_id", "attribute" ]

Returns an attribute value or list of values for a specific parameter and attribute.

10. Fn::ValueOfAll

Syntax:
> "Fn::ValueOfAll" : ["parameter_type", "attribute"]

(***) Section appendix:
The following list describes the attribute values that you can retrieve for specific resources and parameter types:

- The AWS::EC2::VPC::Id parameter type or VPC IDs: DefaultNetworkAcl, DefaultSecurityGroup, Tags.tag_key

- The AWS::EC2::Subnet::Id parameter type or subnet IDs: AvailabilityZone, Tags.tag_key, VpcId

- The AWS::EC2::SecurityGroup::Id parameter type or security group IDs: Tags.tag_key


### Mappings

Example of a mapping named "RegionMap"
```
Mappings: 
  RegionMap: 
    us-east-1: 
      "HVM64": "ami-0ff8a91507f77f867"
      "HVMG2": "ami-0a584ac55a7631c0c"
    us-west-1: 
      "HVM64": "ami-0bdb828fd58c52235"
      "HVMG2": "ami-0a584ac55a7631c0c"
    eu-west-1: 
      "HVM64": "ami-047bb4163c506cd98"
      "HVMG2": "ami-0a584ac55a7631c0c"
    ap-southeast-1: 
      "HVM64": "ami-08569b978cc4dfa10"
      "HVMG2": "ami-0a584ac55a7631c0c"
    ap-northeast-1: 
      "HVM64": "ami-06cd52961ce9f0d85"
      "HVMG2": "ami-0a584ac55a7631c0c"
```


Mapping referrence example
```
Resources: 
    myEC2Instance: 
        Type: "AWS::EC2::Instance"
        Properties: 
        ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", HVM64]
        InstanceType: m1.small
```



### Conditions

Conditions associate with a resource or output so that AWS CloudFormation only creates the resource or output if the condition is true.


We have some condition functions:

- Fn::And
- Fn::Equals
- Fn::ForEach
- Fn::If
- Fn::Not
- Fn::Or

Syntax:
```
Conditions:
    Logical ID:
        Intrinsic function
```

Example:
```
Conditions:
    CreateProdResources: !Equals 
        - !Ref EnvType
        - prod
    IsProduction: !Equals 
        - !Ref EnvType
        - prod
    CreateBucket: !Not 
        - !Equals 
        - !Ref BucketName
        - ''
    CreateBucketPolicy: !And 
        - !Condition IsProduction
        - !Condition CreateBucket
```

### Transform


### Resources

This's the most important part, which declare AWS resources you want to include in the stack.

Syntax 
```
Resources:
    Logical ID:
        Type: Resource type
        Properties:
            et of properties
```

For details:
- Logical ID: unique name in template. Sometimes, you have to use physicalID - ID of resources had been created in AWS before.
- Type: AWS resource type, refer this (link)[https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html] for more informations.
- Properties: properties of resource, each resource type require a set of different propreties

### Outputs

- The (optional) Outputs section declares output values that you can import into other stacks (to create cross-stack references), return in response (to describe stack calls), or view on the AWS CloudFormation console. 

Syntax

```
Outputs:
    Logical ID:
        Description: Information about the value
        Value: Value to return
        Export:
            Name: Name of resource to export
```
For details:
- Logical ID: an identifier for the output
- Value: The value of the property returned by the aws cloudformation describe-stacks command.
- Export: The name of the resource output to be exported for a cross-stack reference. This name must be unique within AWS Region.






## Ways to create CloudFormation stack using template

### Option 1: using Cloud Formation webservice


### Option 2: using AWS CLI

To create a stack using command:
> aws cloudformation create-stack 

For example:
```
aws cloudformation create-stack \
  --stack-name myteststack \
  --template-body file:///home/testuser/mytemplate.json \
  --parameters ParameterKey=Parm1,ParameterValue=test1 ParameterKey=Parm2,ParameterValue=test2
```

To list stacks
> aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE

To view stack details
> aws cloudformation describe-stacks --stack-name myteststack

View stack event history
> aws cloudformation describe-stack-events --stack-name myteststack

Listing all stack resouces
> aws cloudformation list-stack-resources --stack-name myteststack

Get stack's template to reuse
> aws cloudformation get-template --stack-name myteststack

Check validate syntax error
> aws cloudformation validate-template --template-url https://s3.amazonaws.com/cloudformation-templates-us-east-1/S3_Bucket.template
> aws cloudformation validate-template --template-body file:///home/local/test/sampletemplate.json

Upload local artifacts to S3 bucket


Delete stack
> aws cloudformation delete-stack --stack-name myteststack

Update stack

```
aws cloudformation update-stack --stack-name mystack \
  --use-previous-template \
  --parameters ParameterKey=VPCID,UsePreviousValue=true ParameterKey=SubnetIDs,ParameterValue=SampleSubnetID1\\,UpdatedSampleSubnetID2
```

Create change-set
```
aws cloudformation create-change-set --change-set-name update-private-subnet-id\
  --stack-name custom-vpc \
  --use-previous-template \
  --parameters ParameterKey=paramPrivateSubnet2CIDR,ParameterValue="10.192.41.0/24"
```

## Nested stack

- Nested stacks are stacks created as part of other stacks. You create a nested stack within another stack by using the AWS::CloudFormation::Stack resource.
Use this for sharing resources between templates.
- With change sets for nested stacks you can preview the changes to your application and infrastructure resources across the entire nested stack hierarchy and proceed with updates when you've confirmed that all the changes are as intended

- Use the resource import feature to nest an existing stack within another existing stack with resource type AWS::CloudFormation::Stack


## Stackset
- AWS CloudFormation StackSets extends the capability of stacks by enabling you to create, update, or delete stacks across multiple accounts and AWS Regions with a single operation.
