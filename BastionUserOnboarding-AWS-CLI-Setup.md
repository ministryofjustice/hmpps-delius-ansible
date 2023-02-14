# AWS CLI setup
This page aims to give some useful guidance for setting up the AWS CLI with some automation around updating your AWS profile with a dynamically-generated session token.
Please consider this an example setup - we are conscious other engineers achieve the same result in different ways.

## Create your ~/.aws/config file
```
[default]
...
[hmpps]
output = json
region = eu-west-2
[hmpps_token]
output = json
region = eu-west-2
[profile eng-dev]
region = eu-west-2
output = json
[profile eng-prod]
region = eu-west-2
output = json
...
```

## Create your ~/.aws.credentials file
Generate an AWS CLI access Key from the AWS console, then replace <USERNAME>, <ACCESS_KEY_ID>,  <SECRET_ACCESS_KEY>, <ENG_DEV_ACCOUNT_ID> and <HMPPS_PROBATION_ACCOUNT_ID> in the following credentials file ($HOME/.aws/credentials):

```
[hmpps]
aws_access_key_id = <ACCESS_KEY_ID>
aws_secret_access_key = <SECRET_ACCESS_KEY>
mfa_serial = arn:aws:iam::<HMPPS_PROBATION_ACCOUNT_ID>:mfa/<USERNAME>
[hmpps_token]
aws_access_key_id = XXX
aws_secret_access_key = XXX
aws_session_token = XXX
[eng-dev]
role_arn = arn:aws:iam::<ENG_DEV_ACCOUNT_ID>:role/BastionDevSSMUsers
source_profile = hmpps_token
[eng-prod]
role_arn = arn:aws:iam::<ENG_PROD_ACCOUNT_ID>:role/BastionProdSSMUsers
source_profile = hmpps_token
...
```

## Create ~/.aws-mfa.sh
Note that this script depends on the installation of jq - the utility that processes and parses JSON.
See https://stedolan.github.io/jq/ or for MacOS users, it can be [installed using brew](https://formulae.brew.sh/formula/jq): ```brew install jq```

```
#!/usr/bin/env bash
set -eo pipefail

aws_token_profile_name="${AWS_PROFILE}_token"
aws_credentials_file="${HOME}/.aws/credentials"

MFA_SERIAL=arn:aws:iam::${AWS_ACCOUNT_ID}:mfa/${AWS_IAM_USERNAME}

read -p "Enter MFA code for ${MFA_SERIAL}: " MFA_TOKEN

# Get session token
SESSION_TOKEN=$(aws sts get-session-token --serial-number ${MFA_SERIAL} --token-code ${MFA_TOKEN})
echo "Updating AWS credentials for [${aws_token_profile_name}]"

# Update token profile
aws configure set aws_access_key_id $(echo $SESSION_TOKEN | jq -r .Credentials.AccessKeyId) --profile "${aws_token_profile_name}"
aws configure set aws_secret_access_key $(echo $SESSION_TOKEN | jq -r .Credentials.SecretAccessKey) --profile "${aws_token_profile_name}"
aws configure set aws_session_token $(echo $SESSION_TOKEN | jq -r .Credentials.SessionToken) --profile "${aws_token_profile_name}"
```

## Create an alias for the script
Ensure you replace <HMPPS_PROBATION_ACCOUNT_ID> and <USERNAME> with appropriate values
```
alias mfa='AWS_PROFILE=hmpps AWS_ACCOUNT_ID=<HMPPS_PROBATION_ACCOUNT_ID> AWS_IAM_USERNAME=<USERNAME> ~/aws-mfa.sh'
```

## Execute the script through the alias to authenticate to AWS
You will be prompted for your MFA token and the script will populate your session token into your AWS credentials files.
```
mfa

# You will be prompted with
# Enter MFA code for arn:aws:iam::<HMPPS_PROBATION_ACCOUNT_ID>:mfa/<USERNAME>:
```