# Ansible Role Jenkins Backup

A specific implementation of a Jenkins backup using S3 to save encrypted backup files.
due to memory and performance issues some Ansible Modules were dropped. (Trying to avoid running commands through shell)
Ansible rsync module is still in use but performance might be a bit slow due to using it.

* OS: Ubuntu 16.04+

## Assumptions

- Using a Jenkins job to run the playbook
- Ansible runs in a Docker container
- Ansible playbook is executed on 'localhost'
- Docker container runs on the Jenkins master
- Secrets are stored in credstash
- Backup is uploaded to S3

### Notes

- AWK KMS Master Key with Customer generated key material
- Key policy to limit usage to specific roles/users
- Value for `{{ aws_cmk_id }}` is the literal ID of the CMK 
        example:  `arn:aws:kms:us-west-2:000000000000:key/0000-0000-0000-0000-000000`  
        in the example above the keyId is the part after `arn:aws:kms:us-west-2:000000000000:key/`

### Defaults

```shell
JENKINS_HOME: '/var/lib/jenkins'
s3_bucket_name: "tilix-jenkins-backup-{{ env }}"
tmp_path: "/tmp/{{ env }}_jenkins_backup"
aws_access_key: "aws_access_key"
aws_secret_key: "aws_secret_key"
aws_kms_region: 'us-west-2'
aws_cmk_alias: "{{ env }}-jenkins-backup"
```

>Errors tend to be generic, use -vvvv to debug

## Usage Example

```shell
ansible-playbook -i localhost -e 'env=dev' -e 's3_bucket_name=backup-bucket' -e 'JENKINS_HOME=/var/lib/jenkins'
```

```shell
ansible-playbook -i localhost jenkins-backup-playbook.yml -e 'env=dev' -e 's3_bucketname=jenkins-backup-dev' -e 'aws_kms_region=us-east-1' -e "{aws_kms_master_keys: ['key=arn:aws:kms:us-east-1:{aws_account}:key/{key_id}']}" -e 'JENKINS_HOME=/tmp'
```
