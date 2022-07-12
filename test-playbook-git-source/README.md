# test-playbook-git-source

Runs the `helloworld` module from the collection which is installed from the remote git repository.

## Requirements

* none

## Usage

```
ansible-galaxy install -r requirements.yml --force
ansible-playbook test.yml -v
```
