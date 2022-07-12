# golang-ansible-modules

Experimenting with writing ansible modules using golang.

## Code Structure

This repo consists of the following:

- [ansible-collection-helloworld](ansible-collection-helloworld) - source code for a basic "hello-world" ansible collection
- [test-playbook-git-source](test-playbook-git-source) - a test playbook that runs the `helloworld` module from the collection which is installed from the git repository source.
- [test-playbook-local-library](test-playbook-local-library) - a test playbook that runs the `helloworld` module which is downloaded to the ansible controller.
- [test-playbook-local-source](test-playbook-local-source) - a test playbook that runs the `helloworld` module from the collection which is installed from the local directory source.
- [test-playbook-role-helper](test-playbook-role-helper) - a test playbook that utilizes a role in a collection for building `helloworld` module executable

## Limitations

It does not seem possible to run .go files from within ansible (Barring usage of `shell` or `cmd` ansible modules, of course, which in this case defeats the whole purpose.)

Therefore, modules written in go need to be compiled to binaries (keeping in mind the target OS of course), which ansible can use since `2.2` because they added support for [binary modules](https://docs.ansible.com/ansible/latest/dev_guide/developing_program_flow_modules.html#binary-modules).

There are basically two possibilities to use standalone modules written in go, that I can see:

- pre-compiled binaries are distributed as part of a collection, which is installed with `ansible-galaxy`
- binaries are downloaded to the ansible controller before the "main" playbook runs
  - it could be desirable to download these to a `library` dir relative to the playbook: `{{ playbook_dir }}/library/`, in this case binaries should be picked up automatically by ansible
  - if binaries are downloaded to some "custom" location, then [`ANSIBLE_LIBRARY`](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#default-module-path) config needs to be modified to include that directory

Both of the above have their pros and cons, hence the best approach needs to be evaluated on a case-by-case basis.

There's another workaround which also works. We can add a "role" to the collection which will build the binaries at runtime. The benefits of this approach is that we do not need to store binaries in git and the binary is built automatically for the target system on which ansible is running. It does not look very pretty, but it works.

By adding a "dummy" file in place of the future executable, we also eliminate the need to run the "installation" in a separate playbook (like it's done in [test-playbook-local-library](test-playbook-local-library)). Ansible searches for the presence of the module file when the playbook is parsed, but doesn't actually run it until the module is called.
