## Create Compute Engine Instance

```sh
ansible-playbook playbook.yaml -vvv -e '{"state": "present"}'
```

## Delete Compute Engine Instance

When deleting the Compute Engine instance, you need to understand, if the persistent dependency will not be deleted automatically, you need to delete it manually. For example, if you have a persistent disk attached to the instance, you need to delete the disk manually after deleting the instance.

```sh
ansible-playbook playbook.yaml -vvv -e '{"state": "absent"}'
```