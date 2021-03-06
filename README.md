# glance-irods: iRODS Storage Back-end for OpenStack Glance

## Dependencies
* [Python-irodsclient](https://pypi.python.org/pypi/python-irodsclient)
* OpenStack Glance

## Manual Installation

This procedure is intended for OpenStack Newton release; procedure may differ for other versions of OpenStack.

These steps assume you are installing glance-irods on an OpenStack Glance server (or LXC as would be set up by OpenStack-Ansible).

There's certainly a better way to package this (e.g. as a fork of glance_store), but this will get you going for now.

### Install python-irodsclient

Activate your Glance's virtual environment and:
```
pip install git+git://github.com/irods/python-irodsclient.git
```

If you're working with an OpenStack deployed by OpenStack-Ansible, then try installing with the `--isolated` switch.

### Configure Glance

Add or change the following configuration to the appropriate sections of your `glance-api.conf`; this file is stored in `/etc/glance/glance-api.conf` for glance-api as packaged by OpenStack-Ansible and Ubuntu Cloud Archive.

```
[default]
# Replace the following with connection details for your own iRODS deployment
irods_store_host = exampleirodshost.org
irods_store_port = 1247
irods_store_zone = tempZone
irods_store_path = /tempZone/home/openstack_images
irods_store_user = openstack_images
irods_store_password = somepassword

[glance_store]
stores = file, irods  # You may have a comma-separated list of multiple back-ends
default_store = irods
```

### Patch glance_store

The glance_store source code needs some tweaks to support the new iRODS storage backend.

First, find the library path for your glance_store Python package. In an LXC deployed via OpenStack-Ansible, this may be something like `/openstack/venvs/glance-14.0.4/lib/python2.7/site-packages/glance_store`.

Add `irods` to the `choices` tuple in the following block of `backend.py` in that folder:

```
    cfg.StrOpt('default_store',
               default='file',
               choices=('file', 'filesystem', 'http', 'https', 'swift',
                        'swift+http', 'swift+https', 'swift+config', 'rbd',
                        'sheepdog', 'cinder', 'vsphere', 'irods'),
```
See example `backend.py` in this repo.

Copy irods_store.py to the `_drivers` subfolder of glance_store.

Next to the `my-venv/lib/python2.7/site-packages/glance_store` folder, you should see a dist-info folder, e.g. `/openstack/venvs/glance-14.0.4/lib/python2.7/site-packages/glance_store-0.18.0.dist-info`. Find `entry_points.txt` in this folder and add the following text to the `[glance_store.drivers]` section:

```
[glance_store.drivers]
...
glance.store.irods.Store = glance_store._drivers.irods_store:Store
irods = glance_store._drivers.irods_store:Store
```

See example `entry_points.txt` in this repo.

### Restart glance-api

`systemctl restart glance-api`


## Questions?

Feel free to contact the developers:
- Edwin Skidmore (edwin@cyverse.org)
- Amit Juneja (amitj@cyverse.org)
- Chris Martin (cmart@cyverse.org)

## License

It's a standard BSD License. See LICENSE.txt
