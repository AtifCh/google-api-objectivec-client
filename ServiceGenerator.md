```xml

```
Usage: ServiceGenerator [FLAGS] [ARGS]

  Required Flags:

    --outputDir PATH
        The destination directory for writing the generated files.

  Optional Flags:

    --discoveryService URL
        Instead of discovery’s default URL, use the specified URL as the
        location to send the JSON-RPC requests. This is useful for running
        against a custom or prerelease server.

    --apiLogDir DIR
        Write out a file into DIR for each JSON API description processed. These
        can be useful for reporting bugs if generation fails with an error.

    --httpLogDir PATH
        Turn on the HTTP fetcher logging and set it to write to PATH. This can
        be useful for diagnosing errors on discovery fetches.

    --generatePreferred
        Causes the list of services to be collected, and all preferred services
        to be generated.

    --httpHeader NAME:VALUE
        Causes the given NAME/VALUE pair to be added as an HTTP header on *all*
        HTTP requests made by the generator. Can be used repeatedly to provide
        additional header pairs.

    --formattedName SERVICE:VERSION=NAME
        Causes the given SERVICE:VERSION pair to override its service name in
        files, classes, etc. with NAME. If :VERSION is omitted the override is
        for any version of the service. Can be used repeatedly to provide
        several maps when generating a few things in a single run.

    --addServiceNameDir yes|no  Default: no
        Causes the generator to add a directory with the service name in the
        outputDir for the files. This is useful for generating multiple
        services.

    --generatedDir yes|no  Default: no
        Causes a directory in outputDir called "Generated" to be created and
        used to contain the generated files.

    --removeUnknownFiles yes|no  Default: no
        By default, the generator will report unknown files in the output
        directory, as commonly happens when classes go away in a new API
        version. This option causes the generator to also remove the unknown
        files.

    --rootURLOverrides yes|no  Default: yes
        Causes any API root URL for a Google sandbox server to be replaced with
        the googleapis.com root instead.

    --verbose
        Generate more verbose output. Can be used more than once.

  Arguments:

    Multiple arguments can be given on the command line.

    service:version
        The description of the given [service]/[version] pair is fetched and the
        files for it are generated. When using --generatePreferred version can
        be '-' to skip generating the name service.

    http[s]://url/to/rpc_description_json
        A URL to download containing the descripiton of a service to generate.

    path/to/rpc_description.json
        The path to a text file containing the description of a service to
        generate.
```
```