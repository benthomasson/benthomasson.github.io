---
title:  "How does Terraform work?"
date:   2025-04-15 05:08:00 -0500
categories: terraform code-review
---

Terraform is an automation tool that excels at provisioning infrastructure. How do Terraform providers work?  Let's
take a close look at the example [hashicups provider](https://github.com/hashicorp/terraform-provider-hashicups).

Terraform providers are Go programs.  After checking out this repo and setting up a Go environment they can be compiled
as normal:

    git checkout https://github.com/hashicorp/terraform-provider-hashicups.git
    cd terraform-provider-hashicups
    go build -o terraform-provider-hashicups

If we try to run the provider by itself we get this message:


    $ ./terraform-provider-hashicups
    This binary is a plugin. These are not meant to be executed directly.
    Please execute the program that consumes these plugins, which will
    load any plugins automatically
    Exit code 1

Terraform providers are called as subprocesses to the main Terraform process.
There must be something in the environment that lets the subprocess know it is
being called by Terraform.

Terrform has a nice plugin architecture where the main type of plugin is a
provider. Plugin architectures are great for large number of applications.
Design a simple core program that is capable of loading, executing, and
communicating with plugins and then extend the core in whatever domain you
are working with.  This is especially good for teams of developers because
most of them will be writing plugins to the core.

Hashicorp wrote a plugin library for go called [go-plugin](https://github.com/hashicorp/go-plugin)
which Terraform providers use. Looking throught this code we see a hint in [go-plugin/server.go](https://github.com/hashicorp/go-plugin/blob/cfdf485783602a2ca85502dedebf441be7bcbc8d/server.go#L248):


    248         // Validate the handshake config
    249         if opts.MagicCookieKey == "" || opts.MagicCookieValue == "" {
    250             fmt.Fprintf(os.Stderr,
    251                 "Misconfigured ServeConfig given to serve this plugin: no magic cookie\n"+
    252                     "key or value was set. Please notify the plugin author and report\n"+
    253                     "this as a bug.\n")
    254             exitCode = 1
    255             return
    256         }
    257

It is looking for a MagicCookieKey and MagicCookieValue. In [Terraform core](https://github.com/hashicorp/terraform/blob/8d2dffedb36a4cf34dc41828e24d1a2833c9c1f5/internal/plugin6/serve.go#L25) we can find the values
for these:

    26 var Handshake = plugin.HandshakeConfig{
    27     // The ProtocolVersion is the version that must match between TF core
    28     // and TF plugins.
    29     ProtocolVersion: DefaultProtocolVersion,
    30
    31     // The magic cookie values should NEVER be changed.
    32     MagicCookieKey:   "TF_PLUGIN_MAGIC_COOKIE",
    33     MagicCookieValue: "d602bf8f470bc67ca7faa0386276bbdd4330efaf76d1a219cb4d6991ca9872b2",
    34 }

Now if we set these as environment variables and run the provider again we get:

    $ ./terraform-provider-hashicups 
    {"@level":"debug","@message":"plugin address","@timestamp":"2025-04-15T05:34:18.840723-04:00","address":"/var/folders/fn/w9s_vznd7qs71qz8rxv3mfb80000gn/T/plugin2463171971","network":"unix"}
    1|6|unix|/var/folders/fn/w9s_vznd7qs71qz8rxv3mfb80000gn/T/plugin2463171971|grpc|


This is a mix of stderr and stdout.  The first line is a log line sent to stderr.  The second line is a protocol descriptor
sent to stdout.  This line communicates to the core how it should communicate with this plugin.  Here we have:

* 1 - Probably a schema version for the message
* 6 - The plugin protocol version
* /var/folders/fn/w9s_vznd7qs71qz8rxv3mfb80000gn/T/plugin2463171971 - A unix socket
* grpc - The remote procedure call protocol to use.  Here it is [grpc](https://grpc.io/).


GRPC is a really nice RPC protocol with libraries for it in nearly every language. This means
we can write a client in Python to explore how Terraform providers work in more detail.

Now we know how they communicate, but what do they communicate?   GRPC uses protocol buffers
which require that the endpoints agree on the schema of the binary communication in advance.
This is not as easy as inspecting the JSON communication that Ansible modules use.  We will
need to dive into the code a bit more to find the schema.











* Outline

[link](https://benthomasson.com)

```python
print('Hello world')
```
