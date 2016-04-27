# OpenProject version for UCS

This repo contains the scripts required to make OpenProject work on the UCS
distribution.

We're following the Docker approach, outlined at
<http://docs.software-univention.de/app-tutorial.html#docker:example:prerequisites>.

## Prerequisites

First, you need to launch a UCS instance on EC2. At the time of writing
(2016/04/27), the latest AMI is `ami-08be0c7b`. If it no longer exists, search
in the **Community AMIs** for `ucs`.

Once the instance has launched, you need to ssh into it using the `root` user,
and then follow the instructions displayed in the message of the day (i.e.
change the root password, and connect to the UCS web management UI to finish
the installation).

After that, you need to make sure you have the necessary packages installed for
local development:

```bash
ucr set repository/online/unmaintained='yes'
apt-get install -y univention-appcenter-dev univention-appcenter-docker univention-appcenter
univention-app dev-setup-local-appcenter
```

Finally, you need to setup the OpenProject APT repository that holds the
packages you want to release on UCS:

```bash
wget -qO - https://deb.packager.io/key | sudo apt-key add -
echo "deb https://deb.packager.io/gh/opf/openproject-ce wheezy stable/5" | sudo tee /etc/apt/sources.list.d/openproject-ce.list
sudo apt-get update
```

## Testing the UCS installation locally

Register the app (do it every time you change stuff):

```bash
./bin/test-local
```

Install the app:

```bash
univention-app install openproject
```

Remove the app:

```bash
univention-app remove openproject
```

## Releasing a new version

Update the `.ini` file with the proper OpenProject version.

Make sure the `VERSION` numbers have been incremented if you've made changes to
the `inst`/`uinst` files.

Use the provided `Makefile` to generate the required tarball with the right
structure as per the documentation at
<http://docs.software-univention.de/app-tutorial.html#provide>.

```
make all
```

This will package everything into an `openproject.tar.gz` file. You can then
upload it using the form at <https://upload.univention.de/upload.php>.

## Important

* Only increment the `VERSION` numbers of the `inst`/`uinst` whne you've made
  changes.

* The `inst` join script MUST BE IDEMPOTENT! See
  https://docs.software-univention.de/developer-reference.html#join:write. Make
sure that you're not overwriting any configuration file that may have been
updated by the package or the user.

