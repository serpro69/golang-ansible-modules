# golang-ansible-modules

Examples of writing ansible modules using golang.

I've never been a fan of dynamic languages in general and Python in particular. But working a lot with Ansible, I do, however, feel that sometimes an *ansible role* doesn't cut it, and it would be better to write a *module*. I've recently started playing around with Golang, and I feel it could potentially be utilized for such a task. Hence, I've made this project to experiment and provide examples on how ansible modules could be written in golang.

*It is worth noting that in many cases python is still much more preferable to use when developing ansible modules.*

We are going to utilize ansible's [binary modules](https://docs.ansible.com/ansible/latest/dev_guide/developing_program_flow_modules.html#binary-modules), and hence, the same principles can be used to write ansible modules in any other programming language that can build code as native executables.For example, one could in theory write ansible modules in java, build them as native images with Graal, and use those images as ansible binary modules. In practice, though, it's probably an overkill to write ansible modules in java.

You might also be interested in [writing ansible modules in kotlin](https://github.com/serpro69/kotlin-ansible-modules)
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

Both of the above have their pros and cons:

* pre-compiled binaries committed to source code and distributed as part of the collection
  * pros:
    * simple installation (via `ansible-galaxy`)
    * easy to distribute since binaries (effectively - modules) are part of a collection
    * better suited for public usage
  * cons:
    * binaries committed with the code
    * could lead to a more complicated build/release process since you need to ensure to always update and commit the binary when releasing new version of a collection; otherwise source code of a module and executable might differ
    * could be harder to debug; but this applies to any kind of binary module really
* downloaded binaries
  * pros:
    * no binaries in git
    * not hard to distribute if collection contains only go-based modules but might but still more suitable for internal (non-public) code
    * better suited for internal usage
  * cons:
    * installation requires a separate playbook, so you'll most likely end up with 1 extra command:
      ```
      ansible-galaxy install ... # installs "regular" dependencies
      ansible-playbook libraries.yml # downloads go-based modules locally <- extra step
      ansible-playbook main.yml # main play
      ```
    * more complicated release for artifacts if they're part of a collection
      * need to build only Go-based modules and filter binaries to publish
    * as a consequence of the above, it might not make much sense to store modules as part of an ansible collection that also contains modules written in python and/or other ansible roles

Hence, the best approach needs to be evaluated on a case-by-case basis.

There's another workaround which also works. We can add a "role" to the collection which will build the binaries at runtime. The benefits of this approach is that we do not need to store binaries in git and the binary is built automatically for the target system on which ansible is running. It does not look very pretty, but it works.

By adding a [dummy plugin file](ansible-collection-helloworld/io_github_serpro69/another_helloworld_go/plugins/modules/helloworld) (which can be just an empty text file) in place of the future executable, we also eliminate the need to run the "installation" in a separate playbook (like it's done in [test-playbook-local-library](test-playbook-local-library)). Ansible searches for the presence of the module file when the playbook is parsed, but doesn't actually run it until the module is called.

If the module isn't pre-built before it's used, ansible will throw an error that the module is missing interpreter and can't be executed - quite obvious, since we're trying to get ansible to run an empty text file. We can also do some better error handling for such cases, and convert the "placeholder module" into a shell script, which, if executed by ansible, would always return `failed: true` and a meaningful error message. See the [helloworld](ansible-collection-helloworld/io_github_serpro69/helloworld_go_better_error_handling/plugins/modules/helloworld) script and the [test playbook](test-playbook-role-helper/test_three.yml).
