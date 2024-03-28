To create stack from this template.yml file in this folder, run this command:

> aws cloudformation create-stack \
  --capabilities CAPABILITY_NAMED_IAM \
  --stack-name iam-stack \
  --template-body file://template.yml \
  --parameters ParameterKey=paramUserName,ParameterValue=johndev ParameterKey=paramUserPassword,ParameterValue=johnPass@123 ParameterKey=paramManagedPolicy,ParameterValue=DeveloperPowerUser