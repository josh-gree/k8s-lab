- create an ssh key and upload

```bash
>> ssh-keygen

>> aws ec2 import-key-pair --key-name "eksworkshop" --public-key-material file://~/.ssh/id_rsa.pub
```

- create AWS KMS CUSTOM MANAGED KEY

```bash
>> aws kms create-alias --alias-name alias/eksworkshop --target-key-id $(aws kms create-key --query KeyMetadata.Arn --output text)
>> export MASTER_ARN=$(aws kms describe-key --key-id alias/eksworkshop --query KeyMetadata.Arn --output text)
```

- create cluster deployment file

```bash
cat << EOF > eksworkshop.yml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: eksworkshop-eksctl
  region: ${AWS_REGION}

managedNodeGroups:
- name: nodegroup
  desiredCapacity: 3
  iam:
    withAddonPolicies:
      albIngress: true

secretsEncryption:
  keyARN: ${MASTER_ARN}
EOF
```

- create cluster - to use a specific AWS_PROFILE set this env var

```bash
eksctl create cluster -f eksworkshop.yml
```
