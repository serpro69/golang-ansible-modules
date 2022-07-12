# test-playbook-local-source

Runs the `helloworld` module from the collection which is installed from the local directory source.

## Usage

```
ansible-galaxy install -r requirements.yml --force
ansible-playbook test_one.yml -v
ansible-playbook test_two.yml -v
```
