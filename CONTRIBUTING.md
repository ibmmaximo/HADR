# Contributing

Note that during a build the version of the ansible collection is automatically adjusted to the correct version.  For local development will we always target the "next major" as the version, this is just a convenient placeholder string that works for local development; all released collections will automatically set this version to the correct value for the release.


## Building the collection locally
- Build and install the collection `make ansible-all` (default action)
- Build the collection `make ansible-build`
- Install the already built collection `make ansible-install`

### Testing a role
```bash
export ROLE_NAME=ibm_catalogs && make && ansible-playbook ibm.mas_devops.run_role
```

## Building the container image locally
- Build the container image `make docker-build`
- Run the already built container image `make docker-run`
- Build and run the container image `make docker-all`


## Using the docker image
This is a great way to test in a clean environment (e.g. to ensure the myriad of environment variables that you no doubt have set up are not impacting your test scenarios).  After you commit your changes to the repository a pre-release container image will be built, which contains your in-development version of the collection:

```bash
docker run -ti quay.io/ibmmas/ansible-devops:x.y.z-pre.mybranch bash
(app-root) oc login --token=xxxx --server=https://myocpserver
(app-root) export STUFF
(app-root) ansible localhost -m include_role -a name=ibm.mas_devops.ocp_verify
(app-root) ansible-playbook ibm.mas_devops.oneclick_core
```


## Style Guide
Failure to adhere to the style guide will result in a PR being rejected!

### YAML file extension
We want consistency and we want to follow the guidelines for the tools we are using, ideally everyone would agree on a standard extension for YAML files, but that has not been the case, so these are the guidelines for MAS:

- Within the context of ansible all YAML files should use the `yml` extension.
- Within the context of Operator SDK configuration, all YAML files should use the `yaml` extension

In practice, when working was MAS Gen2 applications this means:
- In the top level `/operator/name/` use `.yaml` (OSDK rules apply)
- Under `/operator/name/config/` use `.yaml` (OSDK rules apply)
- Under `/operator/name/molecule/` use `.yml` (Ansible rules override OSDK rules)
- Under `/operator/name/playbooks/` use `.yml` (Ansible rules override OSDK rules)
- Under `/operator/name/roles/` use `.yml` (Ansible rules override OSDK rules)

When working with an Ansible collection repository it is more straightforward.  Always use `.yml`

### Naming Jinja Templates
Within the `templates/` directory we place Jinja templates, as such the extension for these files should be `j2` rather than the extension of the format the file is a template for.  In other words use `templates/subscription.yml.j2` instead of `templates/subscription.yml`.

If you are using Visual Studio Code you may want to install the `Better Jinja` extension, it will provide syntax highlighting for Jinja templates.

### Structure tasks using numbered sections
To make the tasks easier to read use section headers as below:
- The dashed line should be eactly 80 characters long
- Leave two empty lines between each section
- Mutliple related tasks can be grouped into each section, you do not need to create a section header for every single task.  Leave a single empty line between tasks in the same section.

#### Example
```yaml
# 1. Install the CRD
# -----------------------------------------------------------------------------
- name: "Install MongoDBCommunity CRD"
  kubernetes.core.k8s:
    apply: yes
    definition: "{{ lookup('template', 'templates/community/crd.yml') }}"


# 2. Create namespace & install RBAC
# -----------------------------------------------------------------------------
- name: "Create namespace & install RBAC"
  kubernetes.core.k8s:
    apply: yes
    definition: "{{ lookup('template', 'templates/community/rbac.yml') }}"
```

### Align debug messages
To make debug easier to read when printed out by the default ansible display module all "property dump" debugging should be done as below:
- Column 1 is is Left aligned, padded to 41 characters using dots
- Using standard indentation, the "...." should end on column 50

#### Example
```yaml
- name: Debug properties
  debug:
    msg:
      - "MongoDb namespace ...................... {{ mongodb_namespace }}"
      - "MongoDb storage class .................. {{ mongodb_storage_class }}"
      - "MongoDb storage capacity (data) ........ {{ mongodb_storage_capacity_data }}"
      - "MongoDb storage capacity (logs) ........ {{ mongodb_storage_capacity_logs }}"
      - "MAS instance ID ........................ {{ mas_instance_id }}"
```

### Task naming
All tasks must be named.  For tasks that are not in main.yaml of the role, they should be prefixed with an indentifier for the file that they are part of so that the Ansible logs guide the user to the appropriate part of the role.

```yaml
# 7. Deploy the cluster
# -----------------------------------------------------------------------------
- name: "community : Create MongoDb cluster"
  kubernetes.core.k8s:
    apply: yes
    definition: "{{ lookup('template', 'templates/community/cr.yml') }}"
```

This will lead to logs like the following when the role is executed:
```
TASK [ibm.mas_devops.mongodb : community : Create MongoDb cluster]
```

### Failure condition checks
All roles must provide clear feedback about missing required properties that do not have a default built into the role.
- Use Ansible `assert/that` rather than `fail/when`.
- Be sure to check for empty string as well as not defined.  Properties that are resolved from environment variables which are not set will be passed into the role as an empty string (`""`) rather than undefined.

```yaml
# 1. Validate required properties
# -----------------------------------------------------------------------------
- name: "community : Fail if required properties are not provided"
  assert:
    that:
      - mongodb_storage_class is defined and mongodb_storage_class != ""
      - mongodb_storage_capacity_data is defined and mongodb_storage_capacity_data != ""
      - mongodb_storage_capacity_logs is defined and mongodb_storage_capacity_logs != ""
      - mas_instance_id is defined and mas_instance_id != ""
    fail_msg: "One or more required properties are missing"
```

## Maintain links between MAS documentation and github documentation
When creating a new ansible role or renaming an existing ansible role, please use the Review Manager button at the top of [internal MAS Knowledge Center](https://ibmdocs-test.mybluemix.net/docs/en/MAS-review_test?topic=installing-ansible-collection) and add a comment to the `Ansible Collection` topic describing the required change.  The idea is to maintain the links between the public MAS documentation and the github docs here.