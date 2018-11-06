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

`os_images_cache`: a path to a directory in which to cache build artefacts.
It defaults to `~/disk_images`

`os_images_auth_type`: OpenStack authentication endpoint and credentials.
Defaults to `password`.

`os_images_auth`: OpenStack authentication endpoint and credentials.  For
example, a dict of the form:
* `auth_url`: Keystone auth endpoint URL.  Defaults to `os_images_auth_env[OS_AUTH_URL]`.
* `project`: OpenStack tenant/project.  Defaults to `os_images_auth_env[OS_TENANT_NAME]`.
* `username`: OpenStack username.  Defaults to `os_images_auth_env[OS_USERNAME]`.
* `password`: OpenStack password.  Defaults to `os_images_auth_env[OS_PASSWORD]`.

`os_images_auth_env`:
* `OS_PROJECT_DOMAIN_NAME`: Keystone domain name containing the project.
  Defaults to `"{{ lookup('env', 'OS_PROJECT_DOMAIN_NAME') }}"`
* `OS_USER_DOMAIN_NAME`: Keystone user's domain name.
  Defaults to `"{{ lookup('env', 'OS_USER_DOMAIN_NAME') }}"`
* `OS_PROJECT_NAME`: Keystone project name.
  Defaults to `"{{ lookup('env', 'OS_PROJECT_NAME') }}"`
* `OS_USERNAME`: Keystone user name.
  Defaults to `"{{ lookup('env', 'OS_USERNAME') }}"`
* `OS_PASSWORD`: Keystone password.
  Defaults to `"{{ lookup('env', 'OS_PASSWORD') }}"`
* `OS_AUTH_URL`: Keystone authentication URL.
  Defaults to `"{{ lookup('env', 'OS_AUTH_URL') }}"`
* `OS_INTERFACE`: Interface type, can be one of: [admin, public, internal].
  Defaults to `"{{ lookup('env', 'OS_INTERFACE') }}"`
* `OS_IDENTITY_API_VERSION`: Keystone identity API version.
  Defaults to `"{{ lookup('env', 'OS_IDENTITY_API_VERSION') }}"`

`os_images_list` is a list of YAML dicts, where `elements` and `image_url` are
mutually exclusive where each contain:
* `name`: the image name to use in OpenStack.
* `elements`: a list of diskimage-builder elements to incorporate into the image.
* `image_url`: the URL to image location on the Internet.
* `env`: (optional) environment variables to define for diskimage-builder parameters.
  This is a dict of the form of `KEY: VALUE`.
* `size`: (optional) size to make the image filesystem.
* `properties`: (optional) dict of properties to set on the glance image.
  Common image properties are available
  [here](https://docs.openstack.org/glance/latest/user/common-image-properties.html).
* `type`: (optional) image type. Default in DIB is qcow2. Image formats are
  available [here](https://docs.openstack.org/image-guide/image-formats.html).
* `force_rebuild`: (optional) boolean flag indicating whether or not the image should always
  be built (even if an existing image that name has been built before). The images on glance
  will be replaced if `os_images_upload` is set to `True`. This defaults to 
  `os_images_force_rebuild`if left unset.

`os_images_common`: A set of elements to include in every image listed.
Defaults to `cloud-init enable-serial-console stable-interface-names`.

`os_images_dib_version`: Optionally set a version of diskimage-builder to install.
By default this is not constrained.

`os_images_git_elements`: An optional list of elements to pull from github, deploy
locally for incorporation into the images.  Supply a list of dicts with the
following parameters:
* `repo`: URL to a git repo for cloning (if not already present)
* `local`: local path for git cloning
* `version`: optional git reference (branch, tag, hash) for cloning.  Defaults
  to `HEAD`
* `elements_path`: optional relative path to elements within the repository.

`os_images_elements`: An optional list of paths for site-specific DIB elements.

`os_images_upload`: Whether to upload built images to Glance. Defaults to `True`.

`os_images_force_rebuild`: Whether or not to force a rebuild of the DIB image. The images on Glance
will be replaced with the newly built image if `os_images_upload` is set to `True`. Defaults to
`False`.

`os_images_overwrite_policy`: What to do when an image with the same name already exists. The value
must be one of the following strings:
* `rename`: rename old image to append creation timestamp
* `overwrite`: remove old image

Defaults to `rename`.


Dependencies
------------

This has a dependency on `jmespath` as we use the `json_query` filter in the ansible playbooks.

Example Playbook
----------------

The following playbook generates a guest image and uploads it to OpenStack:

    ---
    - name: Generate guest image and upload
      hosts: openstack
      roles:
        - role: stackhpc.os-images
          os_images_auth:
            auth_url:     "{{ lookup('env','OS_AUTH_URL') }}"
            username:     "{{ lookup('env','OS_USERNAME') }}"
            password:     "{{ lookup('env','OS_PASSWORD') }}"
            project_name: "{{ lookup('env','OS_TENANT_NAME') }}"
          os_images_list:
          - name: FedoraCore
            elements:
              - fedora
              - selinux-permissive
              - alaska-extras
            env:
              DIB_ALASKA_DELETE_REPO: "y"
              DIB_ALASKA_PKGLIST: "pam-python pam-keystone"
          - name: FedoraAtomic27
            image_url: https://ftp.icm.edu.pl/pub/Linux/dist/fedora-alt/atomic/stable/Fedora-Atomic-27-20180326.1/CloudImages/x86_64/images/Fedora-Atomic-27-20180326.1.x86_64.qcow2
            properties:
              os_distro: fedora-atomic
            type: qcow

Author Information
------------------

- Stig Telfer (<stig@stackhpc.com>)
