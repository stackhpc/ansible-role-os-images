OpenStack Images
================

This role generates guest instance images using disk-image-builder
and uploads them to OpenStack using the `os_image` module.

Requirements
------------

The OpenStack APIs should be accessible from the target host.
Client credentials should have been set in the environment, or
using the `clouds.yaml` format.

Role Variables
--------------

`os_image_cache`: a path to a directory in which to cache build artefacts.
It defaults to `~/disk_images`

`os_image_auth_type`: OpenStack authentication endpoint and credentials.
Defaults to `password`.

`os_image_auth`: OpenStack authentication endpoint and credentials.  For example, a dict of the form:
* `auth_url`: Keystone auth endpoint URL.  Defaults to `OS_AUTH_URL`.
* `project`: OpenStack tenant/project.  Defaults to `OS_TENANT_NAME`.
* `username`: OpenStack username.  Defaults to `OS_USERNAME`.
* `password`: OpenStack password.  Defaults to `OS_PASSWORD`.  

`os_image_list` is a list of YAML dicts, each containing:
* `name`: the image name to use in OpenStack.
* `elements`: a list of diskimage-builder elements to incorporate into the image.
* `params`: (optional) environment variables to define for diskimage-builder parameters.
  These take the form of `KEY=VALUE` strings.
* `size`: (optional) size to make the image filesystem.

`os_image_common`: A set of elements to include in every image listed.
Defaults to `vm cloud-init enable-serial-console stable-interface-names`.

Dependencies
------------

Example Playbook
----------------

The following playbook generates a guest image and uploads it to OpenStack:

    ---
    - name: Generate guest image and upload
      hosts: openstack
      roles:
        - role: os-image
          os_image_list:
          - name: FedoraCore
            elements:
              - fedora
              - selinux-permissive
              - alaska-extras
            params:
              - DIB_ALASKA_DELETE_REPO=y
              - DIB_ALASKA_PKGLIST="pam-python pam-keystone"

Author Information
------------------

- Stig Telfer (<stig@stackhpc.com>)
