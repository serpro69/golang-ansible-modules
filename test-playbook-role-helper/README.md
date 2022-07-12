# test-playbook-role-helper

Runs the `helloworld` module which is installed by a "helper" role which is distributed as part of the collection.

## Requirements

* Go must be installed on the ansible controller
  * The role checks for the presence of `go` executable by running `go version`

## Usage

```
ansible-galaxy install -r requirements.yml --force
ansible-playbook test_one.yml -v
ansible-playbook test_two.yml -v
```
