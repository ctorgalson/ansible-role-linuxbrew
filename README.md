# Linuxbrew (`ctorgalson.linuxbrew`)

Manually installs linuxbrew, brew packages, and taps on Ubuntu/Debian to avoid
piping a shell script to `sh` :)

The role assumes that it could be run in a playbook that uses `become: true`,
so it requires the name of a non-root user to run `brew` commands safely. For
other tasks--such as installing dependencies using `apt`--it uses become to
escalate privileges. The result is, it should work regardless of the value of
`ansible_user`, _as long as the `{{ lb__owner }}` user exists_.

Credit to [markosamuli](https://github.com/markosamuli) for [a good Linuxbrew
role](https://github.com/markosamuli/ansible-linuxbrew) that didn't quite match
up with my needs. I've used that role as the basis for this one (and probably
introduced my own mistakes).

## Tasks

The role is broken down into three tasks files:

### `main.yml`

This file:

  - checks if `brew` already exists,
  - includes `install.yml` when `brew` does not yet exist,
  - includes `packages.yml` when the `lb__packages` variable is not empty, _or_
    either of the two variables `lb__update_homebrew_when_installing_packages`,
    or `lb__upgrade_all_when_installing_packages` is `true`.
  - includes any number of shell-configuration (or other) task files provided
    in `lb__shell_configuration_tasks`.

### `install.yml`

This file:

  - installs dependencies using `apt`,
  - creates required Linuxbrew directories,
  - clones the main and core Homebrew repos,
  - symlinks the `brew` binary,
  - installs the `portable-ruby` package.

### `packages.yml`

This file:

  - updates `brew` itself when `lb__update_homebrew_when_installing_packages`
    is `true`,
  - updates all `brew` packages when `lb__upgrade_all_when_installing_packages`
    is `true`,
  - installs any `brew` packages defined in `lb__packages`,
  - installs any `brew` taps defined in `lb__taps`.

## Requirements

No special requirements.

## Role Variables

### Vars

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `lb__prefix`            | string | `/home/linuxbrew/.linuxbrew`                           | Location for all `brew` related files. |
| `lb__brew`              | string | `{{ lb__prefix }}/bin/brew`                            | Path to the `brew` binary. |
| `lb__homebrew_dir`      | string | `{{ lb__prefix }}/Homebrew`                            | Path to the Homebrew repo directory. |
| `lb__homebrew_core_dir` | string | `{{ lb__prefix }}/Homebrew/Library/Taps/homebrew-core` | Path to the Homebrew core repo directory. |
| `lb__directories      ` | list   | See `vars/main.yml`                                    | List of directories to be created in `lb__prefix` dir. |
| `lb__repos`             | list   | See `vars/main.yml`                                    | List of repos to be cloned during install. Each item must have `repo`, `dest`, and `version` properties suitable for `ansible.builtin.git`. |
| `lb__dependencies`      | list   | See `vars/main.yml`                                    | List of apt packages required for `brew` install and use. |

### Defaults

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `lb__owner`                                    | string  | `{{ ansible_user }}` | The name of the owner for the `{{ lb__prefix }}` directory and contents. |
| `lb__group`                                    | string  | `{{ ansible_user }}` | The name of the group for the `{{ lb__prefix }}` directory and contents. |
| `lb__shell_configuration_tasks`                | list    | `[]`                 | A list of paths to Ansible task include files to be run after the basic installation. |
| `lb__update_homebrew_when_installing_packages` | boolean | `true`               | Whether or not to update `brew` when installing new pacakges. |
| `lb__upgrade_all_when_installing_packages`     | boolean | `true`               | Whether or not to upgrade Linuxbrew package when installing new pacakges. |
| `lb__packages`                                 | list    | `[]`                 | A list of Linuxbrew packages to install. Each item must specify a `name` property, and can have optional `state`, `path`, and `install_options` properties suitable for `ansible.community.homebrew`. | 
| `lb__tabs`                                     | list    | `[]`                 | A list of Linuxbrew taps to install. Each item must specify a `name` property, and can have `state`, `path`, and `url` properties suitable for `ansible.community.homebrew_tap`. |

## Dependencies

This role relies on the `ansible.community` collection for the `homebrew` and
`homebrew_tap` modules.


## Example Playbook

    - hosts: servers
      become: true
      
      vars:
        lb__owner: "ctorgalson"
        lb__group: "{{ lb__owner }}"
        lb__packages:
          - name: "bottom"
            state: "present"
          - name: "starship"
            state: "present"

      tasks:
         - name: "Install and configure Linuxbrew, packages, and taps."
           ansible.builtin.import_role:
             name: "ctorgalson.linuxbrew"

## License

GPL-3.0-only
