## Create Storage Bucket

```sh
ansible-playbook playbook.yaml -vvv -e '{"state": "present"}'
```

## Delete Storage Bucket

```sh
ansible-playbook playbook.yaml -vvv -e '{"state": "absent"}'
```