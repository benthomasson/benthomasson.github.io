---
title:  "Remote Ansible Modules"
date:   2025-02-22 8:33:00 -0500
categories: ansible code-review
---

In this post I'll answer the questions that arose from the [last post]({% post_url  2025-02-22-how-does-ansible-work%}):

* How do modules work on a remote host?
* Does the module need to run on the remote host or locally?
* How do we get the module to the remote host?
* Does python need to be installed there?


Remote modules work the same way on remote machines as the do locally.  They
run as a separate process.  There are a few ways to read the inputs listed in
the documentation
[here](https://docs.ansible.com/ansible/2.7/dev_guide/developing_program_flow_modules.html).
In the example below we use an
[old-style](https://docs.ansible.com/ansible/2.7/dev_guide/developing_program_flow_modules.html#old-style-modules)
module which expects the inputs to be read from a file in the `key=value` form.


Here is a simple old-style module named `argtest` that I will use in the
exploration of remote module execution:


```python

#!/usr/bin/python

# This is an old style module

import os
import sys
import json
import glob

with open(sys.argv[0]) as f:
    executable = f.read()
with open(sys.argv[1]) as f:
    more_args = f.read()
files = glob.glob(os.path.join(os.path.dirname(sys.argv[0]), '*'))
env = dict(os.environ)

print(json.dumps({
    "args" : sys.argv,
    "executable": executable,
    "more_args": more_args,
    "files": files,
    "env": env
}))


```


This module reads its arguments from a file and here is an example input file named `args`:

```
key=value


```


We can call that module locally with python and the args file.  If we pipe it through json.tool we get
nicer output.

```bash
python3 argtest.py args | python -m json.tool
```

This returns:

```js
{
    "args": [
        "argtest.py",
        "args"
    ],
    "executable": "#!/usr/bin/python\n\n# This is an old style module\n\nimport os\nimport sys\nimport json\nimport glob\n\nargs = sys.argv\nwith open(sys.argv[0]) as f:\n    executable = f.read()\nwith open(sys.argv[1]) as f:\n    more_args = f.read()\nfiles = glob.glob(os.path.join(os.path.dirname(sys.argv[0]), '*'))\nenv = dict(os.environ)\n\nprint(json.dumps({\n    \"args\" : args,\n    \"executable\": executable,\n    \"more_args\": more_args,\n    \"files\": files,\n    \"env\": env\n}))\n\n\n",
    "more_args": "key=value\n\n",
    "files": [
        "argtest.py",
        "args",
    ],
    "env": { REDACTED }
}


```


Let's try that remotely over ssh.  First we need to send the module and the arguments file to the remote host.
Then we can run the module directly since it starts with a `#!/usr/bin/python` line.


```bash
rsync -av argtest.py args remote:
ssh remote /home/user/argtest.py args | python -m json.tool
```


The output is similar to the locally run module.  The only exception are the location of the files.

```js
{
    "args": [
        "/home/user/argtest.py",
        "args"
    ],
    "executable": "#!/usr/bin/python\n\n# This is an old style module\n\nimport os\nimport sys\nimport json\nimport glob\n\nargs = sys.argv\nwith open(sys.argv[0]) as f:\n    executable = f.read()\nwith open(sys.argv[1]) as f:\n    more_args = f.read()\nfiles = glob.glob(os.path.join(os.path.dirname(sys.argv[0]), '*'))\nenv = dict(os.environ)\n\nprint(json.dumps({\n    \"args\" : args,\n    \"executable\": executable,\n    \"more_args\": more_args,\n    \"files\": files,\n    \"env\": env\n}))\n\n\n",
    "more_args": "key=value\n\n",
    "files": [
        "/home/user/argtest.py",
        "/home/user/args"
    ],
    "env": {REDACTED}
}

```

The remote system needs to have python installed if it is running python based
modules.  If they are binary modules this is not needed, but the file system
needs to be writable by the user running the module.

Here we learned that Ansible modules run remotely the same way that they do locally:
they are executed as stand-alone processes.

Notice that we did not install Ansible on the remote machine.  This simple
module did not import anything from Ansible.  Other modules do import from
ansible.module_utils.  

This raises a few more questions:

* How to modules import from dependencies?
* Do dependencies need to be installed on the remote host?
* Are dependencies shipped to the remote host?

In the [next post]({% post_url 2025-02-22-remote-ansible-modules2 %}) I'll answer these questions and cover
how other Ansible modules run remotely.

