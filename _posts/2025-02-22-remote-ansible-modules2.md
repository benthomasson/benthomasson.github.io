---
title:  "Remote Ansible Modules Continued"
date:   2025-02-22 10:13:00 -0500
categories: ansible code-review
---

This post answers the question from the [last post]({% post_url 2025-02-22-remote-ansible-modules %}):

* How to modules import from dependencies?
* Do dependencies need to be installed on the remote host?
* Are dependencies shipped to the remote host?

Most Ansible modules follow the pattern that the
[slack](https://github.com/ansible-collections/community.general/blob/main/plugins/modules/slack.py)
module does of importing utilties from `ansible.module_utils`.


```python

from ansible.module_utils.basic import AnsibleModule
from ansible.module_utils.six.moves.urllib.parse import urlencode
from ansible.module_utils.urls import fetch_url

```


If we try to run this module remotely we get an error:


```bash
rsync -av slack.py remote:

ssh remote /home/user/slack.py
Traceback (most recent call last):
  File "/home/user/slack.py", line 242, in <module>
    from ansible.module_utils.basic import AnsibleModule
ModuleNotFoundError: No module named 'ansible'

```


We can copy over the `ansible.module_utils` files to fix this issue:


```bash
rsync -av ansible --include "ansible" --include "ansible/module_utils" --include "ansible/module_utils/**" --exclude "*" remote:
```

And now we see that it behaves the same way as the locally run module.
It waits for JSON input and then prints the result as JSON output.

```bash
ssh remote /home/user/slack.py
{}
^D

```

We get the same error that we had before with the local module.


```js
{"msg": "Error: Module unable to locate ANSIBLE_MODULE_ARGS in json data from stdin.  Unable to figure out what parameters were passed", "failed": true}
```


If we provide the token and message we can see that it works:

```bash
ssh remote /home/user/slack.py
{"ANSIBLE_MODULE_ARGS": {"token": "REDACTED", "msg": "hello from remote" }}
^D
```

```js
{
    "msg": "OK",
    "invocation": {
        "module_args": {
            "token": "VALUE_SPECIFIED_IN_NO_LOG_PARAMETER",
            "msg": "hello from remote",
            "username": "Ansible",
            "icon_url": "https://www.ansible.com/favicon.ico",
            "link_names": 1,
            "validate_certs": true,
            "color": "normal",
            "domain": null,
            "channel": null,
            "thread_id": null,
            "icon_emoji": null,
            "parse": null,
            "attachments": null,
            "blocks": null,
            "message_id": null
        }
    }
}
```

![App name](/assets/images/hello_from_remote.png)

Normally we wouldn't run slack remotely, but this example illustrates that
remote modules run the same way as local modules if the `ansible.module_utils`
files are present in the working directory.

To make this easier we can use the [zipapp](https://docs.python.org/3/library/zipapp.html) python package.
Zipapps are executable zip files that a program with all their dependencies packaged in the zip file. We can
use this to package up `ansible.module_utils` and a module together to ship to a remote host.

First we need to create a directory with all the files that we need for the module including the module itself
and `ansible.module_utils`.

```bash
mkdir slack
cp slack.py slack/
rsync -av ansible --include "ansible" --include "ansible/module_utils" --include "ansible/module_utils/**" --exclude "*" slack/
```

Then we can make the zipapp:

```bash
python -m zipapp slack -m "slack:main" -p "/usr/bin/env python3"
```

Running this zipapp works just like the module:

```bash
./slack.pyz
{"ANSIBLE_MODULE_ARGS": {"token": "REDACTED", "msg": "hello from zipapp" }}
^D
```


```js
{
    "msg": "OK",
    "invocation": {
        "module_args": {
            "token": "VALUE_SPECIFIED_IN_NO_LOG_PARAMETER",
            "msg": "hello from zipapp",
            "username": "Ansible",
            "icon_url": "https://www.ansible.com/favicon.ico",
            "link_names": 1,
            "validate_certs": true,
            "color": "normal",
            "domain": null,
            "channel": null,
            "thread_id": null,
            "icon_emoji": null,
            "parse": null,
            "attachments": null,
            "blocks": null,
            "message_id": null
        }
    }
}
```

![App name](/assets/images/hello_from_zipapp.png)


We can copy this over to a remote host for execution:

```bash

rsync -av slack.pyz remote:

```


And then run it:

```bash
ssh remote /home/user/slack.pyz
{"ANSIBLE_MODULE_ARGS": {"token": "REDACTED", "msg": "hello from zipapp remote"}}
^D
```

```js
{
    "msg": "OK",
    "invocation": {
        "module_args": {
            "token": "VALUE_SPECIFIED_IN_NO_LOG_PARAMETER",
            "msg": "hello from zipapp remote",
            "username": "Ansible",
            "icon_url": "https://www.ansible.com/favicon.ico",
            "link_names": 1,
            "validate_certs": true,
            "color": "normal",
            "domain": null,
            "channel": null,
            "thread_id": null,
            "icon_emoji": null,
            "parse": null,
            "attachments": null,
            "blocks": null,
            "message_id": null
        }
    }
}
```

![App name](/assets/images/hello_from_zipapp_remote.png)

What is nice about the zipapp method is that we can include other dependencies
besides `ansible.module_utils`. Any pure python package could be included.  Python
packages with C dependencies will not work unless the C library is available.

Ansible doesn't use the zipapp method of shipping modules, because it was not
available when Ansible was initially developed. It builds tarballs called
[Ansiballz](https://docs.ansible.com/ansible/latest/dev_guide/developing_program_flow_modules.html#ansiballz-framework)
instead.  Zipapp was used here as an illustration of its underlying method.

In conclusion, we see that Ansible modules are programs that can include some
Python dependencies and can run on remote machines without installing the
dependencies. Ansible cleans up all these files after execution.

This brings up a few new questions:

* How many modules can we run at the same time?
* How many hosts can we run them against?
* What are the bottlenecks to Ansible scaling?
* How much time is spent building Ansiballz?
* Can we build the Ansiballz/zipapps ahead of time or do we have to build them at runtime?

I'll answer those questions in the [next post]()
