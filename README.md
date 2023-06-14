# Paperless-NG Deprecated
Paperlessng install from source.

**Deprecated**. [Paperless-NG](https://github.com/jonaswinkler/paperless-ng) is
no longer maintained upstream. See [migration instructions to migrate](https://github.com/r-pufky/ansible_paperless_ngx#migration-from-r_pufkypaperlessng-paperless-ng-to-paperless-ngx)
to [Paperless-NGX](https://github.com/paperless-ngx/paperless-ngx); a community
maintained fork.

This ansible role has been migrated to [r_pufky.paperless_ngx](https://galaxy.ansible.com/r_pufky/paperless_ngx).

## Requirements
No additional requirements.

## Role Variables
Settings have been throughly documented for usage.
[defaults/main/main.yml](https://github.com/r-pufky/ansible_paperlessng/blob/main/defaults/main/main.yml).

## Dependencies
N/A

## Example Playbook
The following example will get an instance quickly up and running. Paperless
is highly customizable and you should fully read through defaults before using.

host_vars/paperlessng.example.com/vars/paperlessng.yml
``` yaml
# Paperless root user. This user will automatically be created and have root.
paperlessng_root_name:       'example_user'
paperlessng_root_email:      'user@example.com'
paperlessng_root_password:   '{{ vault_paperlessng_root_password }}'
paperlessng_filename_format: '{document_type}/{correspondent}/{created}-{title}-[{tag_list}]'
```

site.yml
``` yaml
- name:   'paperlessng server'
  hosts:  'paperlessng.example.com'
  become: true
  roles:
    - 'r_pufky.paperlessng'
```

## Suggested Archival Use
Suggested Use (based on archivst recommendations):

1. **Document Types** refer to the broad type of document in question. Is it
  a letter? Receipt? Bill? Every instance will be different, but this should
  be your broadest field. You just want to more of less get it in the
  ballpark. For example, my Receipts doctype holds receipts that I scan in,
  but it will also hold confirmations from my debtors that I paid a bill, or
  an email from Cash app that I sold Bitcoin.
2. **Correspondent** refers to the person/organization you are communicating
  with in the document. A bill from your credit card would have Capital One
  as correspondent for example, while a copy of your W2 might go under IRS.
  Again, you can be broad here, as trying to narrow it down is going to
  drive you crazy.
3. **Tags** are used to answer the below basic concepts:
  * **Who** is it referring to? In my case, I have tags for myself, my wife,
    the kids, and the dogs. They are all the same color to easily denote
    that. Note that this is NOT the same as correspondent.
  * **What** is it referring to? Is it related to your car loan? Is it
    related to your homes maintenance? Mark these tags in a different color
    to easily notice them.
  * **When** is the information in this document relevant? Was it a bill from
    2 years ago? Does it relate to your taxes for 2022? Personally, I make
    tags for the year it was received, as it makes it easier to sort. You can
    further break this down by month if needed.
4. I also make tags for special categories that I need to track. For example,
  I have a tag for any documents that we'll need for our taxes in the coming
  year, or critical documents (birth certs, etc). This helps to further
  break it down.
[Reference](https://old.reddit.com/r/selfhosted/comments/sdv0rr/paperless_ng_which_tags_document_types/hugenfp/)

## Using Management Utilities
You many need to temporarily enable the `paperlessng_user` user shell to login
and run managment utilties. It is **highly** recommended that the user shell be
disabled after use.

```bash
su - {{ paperlessng_user }}
cd /var/lib/paperlessng-X.X.X/scripts
. ../.venv/bin/activate
python3 manage.py document_renamer
```
[Reference](https://paperless-ng.readthedocs.io/en/latest/administration.html?highlight=management%20utilies#management-utilities)

`img2pdf` has been included to provide increased functionality for import
scripts. This will enable lossless conversion of images to pdf's, enabling
import into paperless. The following example will strip alpha channel data from
png's and convert to a lossless pdf for import.

```bash
convert test.png -background white -alpha remove -alpha off out.png
img2pdf out.png -o import.pdf
```
[Reference](https://github.com/josch/img2pdf)

## Migration From r_pufky.paperlessng (Paperless-NG to Paperless-NGX)
This ansible role has been migrated to [r_pufky.paperless_ngx](https://galaxy.ansible.com/r_pufky/paperless_ngx).

In place migrations from NG to NGX can be done using this role if the
following conditions are met:
1. Ensure the last release of paperless-ng is already deployed (1.5.0) and
   sucessfully launched. Shutdown.
2. Backup your data, and your databases. Recommend cloning your paperless
   instance as well.
3. Role variables are migrated and updated from `paperlessng_` to `pngx_`.
   Many variables have been added, removed, or changed. The easiest way is to
   start fresh with a new copy of `defaults/main.yml` and add your existing
   values into it for your host. Specifically watch out for:

   Variables Dropped:
   * `paperlessng_worker_retry` (removed).
   * `paperlessng_filename_parse_transforms` (removed).
   * `paperlessng_enable_update_check` (moved to UI toggle).

   Variables Explicitly Changed:
   * `paperlessng_root_name` --> `pngx_admin_name`
   * `paperlessng_root_email` --> `pngx_admin_email`
   * `paperlessng_root_password` --> `pngx_admin_password`
   * `paperlessng_ocr_rotate_pages_threshold` (float) --> `pngx_ocr_rotate_pages_threshold` (int)

   See [proxy instuctions below](#reverse-proxy-migration-changes) for reverse
   proxy changes.
4. Database backend changes are **not** supported. Database migrations to NGX
   are done automatically during role application.

[Reference](https://docs.paperless-ngx.com/setup/#migrating-from-paperless-ng)

### Reverse Proxy Migration Changes
Reverse proxy configuration has drastically **changed**. Be sure to set the
following configuration variables:

host_vars/paperless.example.com/vars/paperless.yml
``` yaml
pngx_use_x_forward_host: true
pngx_use_x_forward_port: true
pngx_url: 'https://paperless.example.com'
pngx_csrf_trusted_origins: 'https://paperless.example.com'
pngx_allowed_hosts: 'paperless.example.com'
pngx_cors_allowed_hosts: 'https://paperless.example.com'
```
See linked bugs detailing reasons for change:
* https://github.com/paperless-ngx/paperless-ngx/pull/674
* https://github.com/paperless-ngx/paperless-ngx/issues/817
* https://github.com/paperless-ngx/paperless-ngx/issues/712

Recieving a 403 after logging in explicitly after upgrading with:
```
Forbidden (403) CSRF verification failed. Request aborted.
```
See the [Proxy Rule Changes](https://github.com/paperless-ngx/paperless-ngx/wiki/Using-a-Reverse-Proxy-with-Paperless-ngx#nginx)
and be sure to add referrer-policy to allow requests through:

```
add_header Referrer-Policy 'strict-origin-when-cross-origin';
```

Restart both NGINX and Paperless and try again.

## Issues
Create a bug and provide as much information as possible.
Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://github.com/r-pufky/ansible_paperlessng/blob/main/LICENSE)

## Author Information
https://keybase.io/rpufky
