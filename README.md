# deploy

`deploy` deploys project files on a local or remote host, based on a per-project configuration file.

## General

`deploy` works on a per-project basis. A project is defined as a directory, including all the files and sub-directories it contains. `deploy` will deploy all files and directories of a project, except the following, which will be ignored:

* the `.gitignore` file
* the `.deploy` file
* the `README.md` file
* the `.idea` directory
* the `.git` directory
* any files and directories contained in the `.gitignore` file

Please note that for any directory paths contained in the `.gitignore` file, for local and remote-rsync deployment a directory can be listed by just giving the directory name, while for remote-s3 deployments, a directory must be listed by using the `directory/*` syntax. 

Unless the `delete` parameter is set no `no` or `false` (see below), `deploy` will delete files and directories that are present in the deployment path but not in the source directory. Therefore, the deployment path must owned by the project, and cannot be a shared path that is used by other projects as well.

## Usage

The syntax to use deploy is:

    deploy [--config=<config>] [<directory>]

The `--config` parameter can be used to instruct `deploy` which of multiple deployment configurations that are defined in the configuration file is to be used. It is optional if only one deployment configuration has been defined there, but it is mandatory if more than one deployment configuration has been defined there. If provided, `<config>` must be the name of a configuration that is present in the configuration file. 

For example, if the configuration file has a configuration section that begins with `[develop]`, the syntax is:

    deploy --config=develop [<directory>]

The `directory` paramater is optional and instructs `deploy` to switch to the provided directory before starting to deploy.

## Deployment Methods

`deploy` supports deploying to the following targets:

* local, where `rsync` will be used;
* remote, where `rsync` will be used;
* remote, where `aws s3 sync` will be used and optionally a CloudFront distribution can be invalidated after upload.

For remote deployments, `deploy` will attempt to use ssh or AWS credentials of the user executing it.

## Configuration

Configurations are defined in a `.deploy` configuration file that must be in a project's root directory. 

This configuration file can have multiple sections to define multiple deployment configurations. If multiple sections are present, each section must begin with one line that has the name of the deployment configuration in square brackets. For example:

    [develop]
    type=..

If the configuration file has only one section, this line containing the name of the deployment configuration can be skipped.

The following rules apply for the format for each section of the configuration file: 

* Each configuration parameter must be in its own line
* format is `key=value`
* line order inside a section is not relevant
* whitespace doesn't matter

Valid configuration parameters are the following:

### type

The `type` parameter defines which deployment method to use. Valid values are `local`, `remote-rsync` and `remote-s3`. If no value is provided, `local` will be used.

### path

The `path` parameter defines the (local or remote) deployment path. It is mandatory and can be absolute or relative, but must comply with the path requirements of `rsync` or `aws s3 sync`. In case of a s3 deployment, it must begin with the `s3://` prefix.

### host

The `host` parameter defines the deployment host. It must be set if the type parameter is set to `remote-rsync`.

### user

The `user` parameter defines the deployment user. It must be set if the type parameter is set to `remote-rsync`.

### cdn

The `cdn` parameter is optional and, if set, must contain an AWS CloudFront cdn distribution id. If it is set and if the type parameter is set to `remote-s3`, `deploy` will attempt to invalidate the distribution after uploading files.

### source

The `source` parameter is optional. If set, it must point to a relative subdirectory inside the project path which will then be used as deployment source, instead of the project's root directory. 

### delete

The `delete` parameter is optional. If it is provided and set to `no` or `false`, `deploy` will not delete any files or directories that are present in the deployment path but not in the source path. If it is not provided, `deploy` will delete such files and directories (see above).

## Dependencies

If deployment to AWS S3 is to be used, the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-linux.html) must be present on the system. 

For remote deployment, ssh keys (for `remote-rsync`) or AWS credentials (for `remote-s3`) must have been configured.