# rhc
![rhc](https://github.com/linux-system-roles/template/workflows/tox/badge.svg)

An ansible role which registers/connects RHEL systems to Red Hat.

## Requirements

subscription-manager and insights-client. Both are available from the standard
RHEL repositories, and subscription-manager is installed by default.

The role requires modules from `community.general`.  If you are using
`ansible-core`, you must install the `community.general` collection.  Use the
file `meta/collection-requirements.yml` to install it:
```
ansible-galaxy collection install -vv -r meta/collection-requirements.yml
```
If you are using Ansible Engine 2.9, or are using an Ansible bundle which
includes these collections/modules, you should have to do nothing.

## Role Variables

    rhc_state: present

Whether the system is connected to Red Hat; valid values are `present`
(to ensure registration/connection), `absent`, and `reconnect`.

When using `reconnect`, the system will be unregistered/disconnected in case
it was already registered/connected; because of this, the role will always
report a "changed" status.

    rhc_organization: "your-organization"

The organization of the user. This *must* be specified when registering if
either:
- the user belongs to more than one organization
- using activation keys (see `rhc_auth` below)

    rhc_auth: {}

The authentication method used to connect a system. This must be specified
in case a system may need to connect (e.g. in case it was not before).
There are few possible authentication methods; only one can be specified at
a time.

**NB**: the variables used for authentication are considered secrets, and thus
they ought to be secured. We recommend the usage of Ansible Vault as source
for them. The references below only describe which keys exists and what they
are for.

For username/password, authentication, specify the `login` dictionary:
```yaml
rhc_auth:
  login:
    username: "your-username"
    password: "your-password"
```

For activation keys authentication, specify the `activation_keys` dictionary,
together with `rhc_organization`:
```yaml
rhc_auth:
  activation_keys:
    keys: ["key-1", ...]
rhc_organization: "your-organization"
```

    rhc_server: {}

The details of the registration server to connect to:
```yaml
rhc_server:
  hostname: "registration-hostname"
  port: 443
  prefix: "registration-server-prefix"
  insecure: false
```
- `hostname` is the hostname of the registration server
- `port` is the port to which connect to on the server
- `prefix` is the prefix (starting with `/`) for the API calls to the
  registration server
- `insecure` specifies whether to disable the validation of the SSL certificate
  of the registration server

    rhc_baseurl: ""

The base URL for receiving content from the subscription server.

    rhc_repositories: []

A list with repositories to enable or disable in the system. Each item
is a dict containing two keys:
- `name` represents the name of a repository
- `state` is the state of it in the system, can be `enabled` or `disabled`
```yaml
rhc_repositories:
  - {name: "repo1", state: enabled}
  - {name: "repo2", state: disabled}
```

    rhc_release: "release"

A release to set for the system. Use `{"state":"absent"}` to actually unset the
release set for the system.
```
rhc_insights:
  state: present
```

Whether the system is connected to Insights; valid values are `present`
(to ensure registration/connection), and `absent`.


    rhc_insights:
      autoupdate: true

Whether the system automatically updates the dynamic configuration. By default is
`true`.

    rhc_insights:
      remediation: present

Whether the system is configured to run Insights remediation; valid values are
`present` (to ensure remediation) and `absent`.

    rhc_insights:
      tags: {}

A dictionary of tags that is added to the system record in Host Based Inventory (HBI).
Used to search for hosts. This tags has keys and values. Values for each key can be any 
data type. Posible values of `tags` parameter:
- `tags` is `null` or an empty value (e.g: `{}`) does not change the tags file content.
- `tags` is `{state: absent}` remove all the tags by removing the file.
- `tags` is non-empty, create the file with the specified tags

An example of the tags configured in the `insights-client` [documentation](https://access.redhat.com/documentation/en-us/red_hat_insights/2022/html/client_configuration_guide_for_red_hat_insights/con-insights-client-tagging-overview_insights-cg-adding-tags):

```yaml
rhc_insights:
  tags:
    group: _group-name-value_
    location: _location-name-value_
    description:
      - RHEL8
      - SAP
    key 4: value
```


    rhc_proxy: {}

The details of the proxy server to use for connecting:
```yaml
rhc_proxy:
  hostname: "proxy-hostname"
  port: 4321
  username: "proxy-hostname"
  password: "proxy-password"
```
- `hostname` is the hostname of the proxy server
- `port` is the port to which connect to on the proxy server
- `username` is the username to use for authenticating on the proxy server;
  it can be not specified if the proxy server does not require authentication
- `password` is the password to use for authenticating on the proxy server;
  it can be not specified if the proxy server does not require authentication

Use `{"state":"absent"}` to reset all the proxy configurations to empty
(effectively disabling the proxy server).

    rhc_environments: []

The list of environments to which register to when connecting the system.

*NB*: this only works when the system is being connected from an unconnected
state -- it cannot change the environments of already connected systems.

## Dependencies

None.

## Example Playbooks

Simple Registration to Red Hat including Insights, assuming authentication
using username & password:

```yaml
- hosts: all
  vars:
    rhc_auth:
      login:
        username: "your-username"
        password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          ....
  roles:
    - linux-system-roles.rhc
```

Ensure that certain RHEL 9 repositories are enabled, and another one is not:

```yaml
- hosts: all
  vars:
    rhc_repositories:
      - {name: "rhel-9-for-x86_64-baseos-rpms", state: enabled}
      - {name: "rhel-9-for-x86_64-appstream-rpms", state: enabled}
      - {name: "codeready-builder-for-rhel-9-x86_64-rpms", state: disabled}
  roles:
    - linux-system-roles.rhc
```

## License

MIT
