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

It is looking for a `MagicCookieKey` and `MagicCookieValue`. In [Terraform core](https://github.com/hashicorp/terraform/blob/8d2dffedb36a4cf34dc41828e24d1a2833c9c1f5/internal/plugin6/serve.go#L25) we can find the values
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

    $ export TF_PLUGIN_MAGIC_COOKIE=d602bf8f470bc67ca7faa0386276bbdd4330efaf76d1a219cb4d6991ca9872b2
    $ ./terraform-provider-hashicups 
    {"@level":"debug","@message":"plugin address","@timestamp":"2025-04-15T05:34:18.840723-04:00","address":"/var/folders/fn/w9s_vznd7qs71qz8rxv3mfb80000gn/T/plugin2463171971","network":"unix"}
    1|6|unix|/var/folders/fn/w9s_vznd7qs71qz8rxv3mfb80000gn/T/plugin2463171971|grpc|


This is a mix of stderr and stdout.  The first line is a log line sent to stderr.  The second line is a protocol descriptor
sent to stdout.  This line communicates to the core how it should communicate with this plugin.  Here we have:

* `1` - Probably a schema version for the message
* `6` - The plugin protocol version
* `/var/folders/fn/w9s_vznd7qs71qz8rxv3mfb80000gn/T/plugin2463171971` - A unix socket
* `grpc` - The remote procedure call protocol to use.  Here it is [grpc](https://grpc.io/).


GRPC is a really nice RPC protocol with libraries for it in nearly every language. This means
we can write a client in Python to explore how Terraform providers work in more detail.

Now we know how they communicate, but what do they communicate?   GRPC uses protocol buffers
which require that the endpoints agree on the schema of the binary communication in advance.
This is not as easy as inspecting the JSON communication that Ansible modules use.  We will
need to dive into the code a bit more to find the schema.  Protocol buffers have a schema
file that explain all the messages that can be sent between RPC peers.  We are looking
for the 6th version of this schema which we can find [here](https://github.com/hashicorp/terraform/blob/c161997dbfc7a5eeca10c465c5e1f347ceaecbd0/docs/plugin-protocol/tfplugin6.9.proto)


    // Copyright (c) HashiCorp, Inc.
    // SPDX-License-Identifier: MPL-2.0

    // Terraform Plugin RPC protocol version 6.9
    //
    // This file defines version 6.9 of the RPC protocol. To implement a plugin
    // against this protocol, copy this definition into your own codebase and
    // use protoc to generate stubs for your target language.
    //
    // This file will not be updated. Any minor versions of protocol 6 to follow
    // should copy this file and modify the copy while maintaing backwards
    // compatibility. Breaking changes, if any are required, will come
    // in a subsequent major version with its own separate proto definition.
    //
    // Note that only the proto files included in a release tag of Terraform are
    // official protocol releases. Proto files taken from other commits may include
    // incomplete changes or features that did not make it into a final release.
    // In all reasonable cases, plugin developers should take the proto file from
    // the tag of the most recent release of Terraform, and not from the main
    // branch or any other development branch.
    //

We can use this file to generate a client in Python to communicate with the hashicups provider using
protoc.  Download this file and call it `tfplugin6.proto`. Run protoc on it like this:

    protoc tfplugin6.proto --python_out=.


This produces a file name `tfplugin6_pb2.py` which will use in our Python client.


If we look at the `tfplugin6.proto` protobuf schema file we can see a service section that lists
all the RPC endpoints that we can connect to:


    327 service Provider {
    328     //////// Information about what a provider supports/expects
    329     
    330     // GetMetadata returns upfront information about server capabilities and
    331     // supported resource types without requiring the server to instantiate all
    332     // schema information, which may be memory intensive. This RPC is optional,
    333     // where clients may receive an unimplemented RPC error. Clients should
    334     // ignore the error and call the GetProviderSchema RPC as a fallback.
    335     rpc GetMetadata(GetMetadata.Request) returns (GetMetadata.Response);
    336 
    337     // GetSchema returns schema information for the provider, data resources,
    338     // and managed resources.
    339     rpc GetProviderSchema(GetProviderSchema.Request) returns (GetProviderSchema.Response);
    340     rpc ValidateProviderConfig(ValidateProviderConfig.Request) returns (ValidateProviderConfig.Response);
    341     rpc ValidateResourceConfig(ValidateResourceConfig.Request) returns (ValidateResourceConfig.Response);
    342     rpc ValidateDataResourceConfig(ValidateDataResourceConfig.Request) returns (ValidateDataResourceConfig.Response);
    343     rpc UpgradeResourceState(UpgradeResourceState.Request) returns (UpgradeResourceState.Response);


If we dig into the message types we can see that some of the request messages do not take any arguments. So we can
call these easily:


    379 message GetMetadata {
    380     message Request {
    381     }
    382 
    383     message Response {
    384         ServerCapabilities server_capabilities = 1;
    385         repeated Diagnostic diagnostics = 2;
    386         repeated DataSourceMetadata data_sources = 3;
    387         repeated ResourceMetadata resources = 4;
    388         // functions returns metadata for any functions.
    389         repeated FunctionMetadata functions = 5;
    390         repeated EphemeralMetadata ephemeral_resources = 6;
    391     }


Let's write a little Python script to call `GetMetadata` GRPC service:

    #!/usr/bin/env python3

    from subprocess import Popen, PIPE
    import io

    import grpc
    import tfplugin6_pb2
    import tfplugin6_pb2_grpc


    def main():

        env = {
            "TF_PLUGIN_MAGIC_COOKIE": "d602bf8f470bc67ca7faa0386276bbdd4330efaf76d1a219cb4d6991ca9872b2"
        }

        proc = Popen(
            "./terraform-provider-hashicups",
            stdout=PIPE,
            stderr=PIPE,
            shell=True,
            env=env,
        )

        message_version = None
        tf_proto_version = None
        socket_type = None
        socket_address = None
        protocol = None

        for line in io.TextIOWrapper(proc.stdout, encoding="utf-8"):
            line = line.rstrip("\n")
            print(line)
            data = line.split("|")
            message_version = data[0]
            tf_proto_version = int(data[1])
            socket_type = data[2]
            socket_address = data[3]
            protocol = data[4]
            break

        print(message_version, tf_proto_version, socket_type, socket_address, protocol)

        if message_version is None:
            for line in io.TextIOWrapper(proc.stderr, encoding="utf-8"):
                line = line.rstrip("\n")
                print(line)

        if tf_proto_version == 5:

            raise Exception("Not supported in version 5")

        elif tf_proto_version == 6:

            with grpc.insecure_channel(f"{socket_type}://{socket_address}") as channel:
                stub = tfplugin6_pb2_grpc.ProviderStub(channel)
                response = stub.GetMetadata(tfplugin6_pb2.GetMetadata.Request())
                print("received: " + str(response))

        else:
            raise Exception(f"Unsupported protocol version {tf_proto_version}")


    if __name__ == "__main__":
        main()



If we put the hashicups terraform provider into the directory with this script and run it we get:


    $ python get_metadata.py
    1|6|unix|/tmp/plugin2003463635|grpc|
    1 6 unix /tmp/plugin2003463635 grpc
    received: server_capabilities {
      plan_destroy: true
      get_provider_schema_optional: true
      move_resource_state: true
    }
    data_sources {
      type_name: "hashicups_coffees"
    }
    resources {
      type_name: "hashicups_order"
    }


Success!  We can retrieve some data from the provider.   We can see that it has
data sources and resources and that it provides some capabilities.

So we have learned that Terraform providers are self-contained programs that run as plugins
to the main Terraform process.  They communicate with the main process over unix sockets
using the GRPC protocol and protobuf messages to exchange data.

In future posts we will explore the other services that providers have to better understand
the Terraform provider lifecycle.


