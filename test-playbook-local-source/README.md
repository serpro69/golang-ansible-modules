# test-playbook-local-source

Runs the `helloworld` module from the collection which is installed from the local directory source.

## Requirements

* none

## Usage

```
ansible-galaxy install -r requirements.yml --force
ansible-playbook test.yml -v
```
