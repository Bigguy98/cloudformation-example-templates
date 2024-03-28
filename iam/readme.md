To create stack from this template.yml file in this folder, run this command:

> aws cloudformation create-stack \
  --capabilities CAPABILITY_NAMED_IAM \
  --stack-name iam-stack \
  --template-body file://template.yml \
  --parameters ParameterKey=paramUserName,ParameterValue=johndev ParameterKey=paramUserPassword,ParameterValue=johnPass@123 ParameterKey=paramManagedPolicy,ParameterValue=DeveloperPowerUser

To update user managed policy, create a change-set

> aws cloudformation create-change-set --change-set-name update-managed-policy\
  --capabilities CAPABILITY_NAMED_IAM \
  --stack-name iam-stack \
  --use-previous-template \
  --parameters ParameterKey=paramUserName,ParameterValue=johndev ParameterKey=paramUserPassword,ParameterValue=johnPass@123 ParameterKey=paramManagedPolicy,ParameterValue=View-Only

To apply change-set

> aws cloudformation execute-change-set \
    --change-set-name update-managed-policy \
    --stack-name iam-stack

To delete stack

> aws cloudformation delete-stack --stack-name iam-stack