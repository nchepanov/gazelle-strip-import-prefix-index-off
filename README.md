# Reproducer

This repository is a reproducer for the issue related to using Gazelle `proto_strip_import_prefix` directive
with and without `--index=false`

# Context

In our repository all protos live in a special directory within the monorepo.
For historical reasons they reference each other relative to the directory, not the repository root.
We are trying to speed up gazelle by using `--index=false` but it seems 
to break the `proto_strip_import_prefix` directive rewriting of the `imp`orts during `language/proto/resolve.go:resolveProto()`

# Steps to reproduce

* clone the repository
* install Gazelle: `go install github.com/bazelbuild/bazel-gazelle/cmd/gazelle@latest`
* run `gazelle` in the repository root
* result: nothing changed, notice `prefix/foo/BUILD.bazel`, especially `deps = ["//prefix/moo:moo_proto"]`

    proto_library(
        name = "foo_proto",
        srcs = ["foo.proto"],
        strip_import_prefix = "/prefix",        <<<<<
        visibility = ["//visibility:public"],
        deps = ["//prefix/moo:moo_proto"],      <<<<<
    )

* run `gazelle --index=false` in the repository root
* notice the change in the `deps` attribute

    diff --git a/prefix/foo/BUILD.bazel b/prefix/foo/BUILD.bazel
    index fab6b89..037722e 100644
    --- a/prefix/foo/BUILD.bazel
    +++ b/prefix/foo/BUILD.bazel
    @@ -5,5 +5,5 @@ proto_library(
        srcs = ["foo.proto"],
        strip_import_prefix = "/prefix",
        visibility = ["//visibility:public"],
    -    deps = ["//prefix/moo:moo_proto"],
    +    deps = ["//moo:moo_proto"],
    )

# Expected behavior

The `strip_import_prefix` is present in both situations, so the root BUILD.bazel file
was read with or without indexing. The `deps` attribute should be the same in both cases.
