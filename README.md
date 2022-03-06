# Paperless-ng
Paperlessng install from source.

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

## Issues
Create a bug and provide as much information as possible.
Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://github.com/r-pufky/ansible_paperlessng/blob/main/LICENSE)

## Author Information
https://keybase.io/rpufky
