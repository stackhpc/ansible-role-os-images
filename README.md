OpenStack Images
================

This role generates guest instance images using disk-image-builder
and uploads them to OpenStack using the `os_image` module.

Requirements
------------

The OpenStack APIs should be accessible from the target host.
Client credentials should have been set in the environment, or
using the `clouds.yaml` format.

You must use a virtualenv with system site packages enabled
as this role relies on python packages installed by the package
manager, e.g:

```
virtualenv --system-site-packages ~/venvs/dib
```

Role Variables
--------------

`os_images_package_dependencies_extra`: List of additional packages to install
on the build host.

`os_images_cache`: a path to a directory in which to cache build artefacts.
It defaults to `~/disk_images`
`NOTE`: new images will NOT be built, even if changes are made in config, if an image is already cached.
Use `force_rebuild` flag in order to apply new config changes.

`os_images_auth_type`: OpenStack authentication endpoint and credentials.
Defaults to `password`.

`os_images_auth`: OpenStack authentication endpoint and credentials.  For
example, a dict of the form:
* `auth_url`: Keystone auth endpoint URL.  Defaults to `OS_AUTH_URL`.
* `project`: OpenStack tenant/project.  Defaults to `OS_TENANT_NAME`.
* `username`: OpenStack username.  Defaults to `OS_USERNAME`.
* `password`: OpenStack password.  Defaults to `OS_PASSWORD`.

`os_images_region`: Define a region to upload the images.  Default is None.

`os_images_cacert` is an optional path to a CA certificate bundle.

`os_images_interface` is the endpoint URL type to fetch from the service
catalog. Maybe be one of `public`, `admin`, or `internal`.

`os_images_list` is a list of YAML dicts, where `elements` and `image_url` are
mutually exclusive where each contain:
* `name`: the image name to use in OpenStack.
* `elements`: a list of diskimage-builder elements to incorporate into the image.
* `image_url`: the URL to image location on the Internet.
* `checksum`: Checksum to validate a downloaded image. Format: <algorithm>:<checksum|url>.
* `env`: (optional) environment variables to define for diskimage-builder parameters.
  This is a dict of the form of `KEY: VALUE`.
* `packages`: (optional) list of packages to install in the image.
* `size`: (optional) size to make the image filesystem.
* `properties`: (optional) dict of properties to set on the glance image.
  Common image properties are available
  [here](https://docs.openstack.org/glance/latest/user/common-image-properties.html).
* `type`: (optional) image type. Default in DIB is qcow2. Image formats are
  available [here](https://docs.openstack.org/glance/latest/user/formats.html).
* `force_rebuild`: (optional) boolean flag indicating whether or not the image should always
  be built (even if an existing image that name has been built before). The images on glance
  will be replaced if `os_images_upload` is set to `True`. This defaults to
  `os_images_force_rebuild`if left unset.
* `is_public`: (optional) whether the image should be set as visible to all
  projects or kept private.
* `owner`: (optional) ID of the project that should own the uploaded image.

`os_images_common`: A set of elements to include in every image listed.
Defaults to `cloud-init enable-serial-console stable-interface-names`.

`os_images_dib_pkg_name`: Optionally customise the name parameter passed 
to the ansible.builtin.pip module when installing diskimage-builder. This can
be used to install diskimage-builder from version control.

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

`os_images_public`: Whether uploaded images are public. Defaults to `True` - note this requires admin permissions.

`os_images_venv`: Path to virtualenv in which to install python dependencies to upload images.

`os_images_dib_venv`: Path to virtualenv in which to install DIB to build images.

`os_images_promote`: Whether or not to retire old and promote new images. Defaults to `False`.

`os_images_build`: Whether or not to build the images.

`os_images_name_suffix`: Image suffix which would be removed during image promotion, for exmple: -rc, -dev, -test etc. Mandatory for promotion functionality. Empty by default.

`os_images_hide`: Whether or not to hide the images in Glance list. Hiding images is available as an option in image retirement/promotion process. Defaults to `False`.

Dependencies
------------

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
            type: qcow2

Author Information
------------------

- Stig Telfer (<stig@stackhpc.com>)
