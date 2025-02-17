# IAMSpy

This is the repository containing IAMSpy, a library that utilises the Z3 prover to attempt to answer questions about AWS IAM. It can "load" a variety of IAM policies and convert them to generate Z3 constraints and a model, from which queries can be made on identifying whether actions are allowed or not. The aim of this library is to allow others to build new IAM tooling without having to worry about implementing their own IAM parsing and reasoning tools. Additionally, IAMSpy hopes to provide a focal point for the community to document observed IAM quirks allowing everyone to benefit from parsing that accounts for these oddities. 

NOTE: This is a work in progress. IAMSpy currently does not support all features/quirks within AWS IAM, mileage may vary for different cloud environments. We encourage others to contribute to the project. Please raise issues for any issues found, or general discussions to be had, and improvements are very much appreciated through pull requests.

## How To

NOTE: Should you pull newer changes for IAMSpy, it is recommended to re-generate any models that may have been pre-computed. This is because internal representations of data are likely to regularly change whilst IAMSpy is being built out to completion. 

IAMSpy can be installed through `poetry install` from a checked out repository. This will set up a python virtualenv to be used with IAMSpy. If you would like to install IAMSpy into another python environment, `pip install /path/to/iamspy`

### Library

The primary interface for applications to utilise IAMSpy is the `Model` class. This and an instantiated version called `model` can be found within the root of the module, and importable with `from iamspy import model`.

This model exposes the following to allow use of IAMSpy:

- `model.save(filename)`

  Saves the current model to filename

- `model.load_model(filename)`

  Loads a model from filename that was saved through `save`
  
- `model.load_gaad(filename)`

  Will load a GAAD stored within the provided filename. The GAAD will be parsed and added to the current model, the parsed GAAD is also returned

- `model.load_resource_policies(filename)`

  Loads and parses resource policies from the provided filename into the model

- `model.can_i(source, action, resource, conditions=[], condition_file=None, strict_conditions=False)`

  Checks whether the provided `source` can perform `action` on target `resource`. `source` and `resource` must be fully qualified ARNs.
  
  Optional parameters include:
  - `conditions` - List of `=` delimited key-value strings containing strings to set within conditions
  - `condition_file` - Filename of a conditions file following the format of conditions within a statement block to add further constraints on the request, such as specifying non string conditions or providing multiple values such as a date greater than a set time
  - `strict_conditions` - Boolean to enable inputted conditions are set as provided, and all other conditions explicitly as not provided. Setting this prevents IAMSpy enabling and setting conditions to get an action accepted without it being specified in the earlier statements.

### CLI

IAMSpy comes with a light-weight CLI wrapper around its main library entrypoints, once installed this is available as the `iamspy` application. `--help` should help with identifying available calls and parameters in a pinch. The various commands are detailed below:

Loading policies can be done with the various `load-*` subcommands, at the moment this is `load-gaad` and `load-resources` however more are anticipated to be added.

```bash
# Load an accounts GAAD obtained from "aws iam get-account-authorization-details"
iamspy load-gaad gaad.json

# Load resource policies following the format shown in resources.json.example
iamspy load-resources resources.json
```

The iamspy CLI will save generated constraints into a file on disk upon execution of these commands. Subsequent commands loads this file to load earlier generated data. By default this is stored within `model.smt2`, however this can be overridden with the `-f` flag to commands, for example `iamspy load-gaad -f different.smt2 gaad.json`. This flag would then need to be provided for each subsequent command.

Once all policies desired have been loaded, queries can be made with the `can-i` sucommand.

```bash
# At the very basic level, this takes a source ARN, an IAM action and a resource ARN. From this it will return True/False
iamspy can-i arn:aws:iam::123456789012:user/bob s3:GetObject arn:aws:s3:::bucket/object
```

Further arguments can be supplied to provide information and further constraints on conditions

```bash
# Providing string based condition values can be done through the "-c" parameter
iamspy can-i arn:aws:iam::123456789012:user/bob s3:GetObject arn:aws:s3:::bucket/object -c aws:referer=bobby.tables

# Other condition settings, such as IP or ARN types, or providing a range of values (for example saying the input condition is at least this value) can be done by creating a conditions file. This follows the same formation of the conditions block within a statement, and can be provided through the -C parameter. An example can be seen within conditions.json.example
iamspy can-i arn:aws:iam::123456789012:user/bob s3:GetObject arn:aws:s3:::bucket/object -C conditions.json

# By default, IAMSpy will attempt to set input conditions as needed to get a statement through the model and allowed. This may involve setting conditions automatically as required by policies observed. If this behavious is not desired, --strict-conditions should be set
iamspy can-i arn:aws:iam::123456789012:user/bob s3:GetObject arn:aws:s3:::bucket/object --strict-conditions
```

## Development

- Checkout the repo
- `poetry install`
- `poetry shell`

### Code Quality

flake8 is used for code quality checks, configured with `--max-line-length=120`. To ignore specific warnings, use `# noqa: E9876` with the appropriate E/W number from flake8 docs. At present, the approved ignores are:

- `# noqa: E712` for cases where flake8 is complaining about comparison statements inside z3 statements.

## Credits

This project stands on the shoulders of others. In particular, we'd like to highlight:

* The Z3 theorem prover from Microsoft Research - https://github.com/Z3Prover/z3
* The AWS Zelkova paper, available from https://www.cs.utexas.edu/users/hunt/FMCAD/FMCAD18/papers/paper3.pdf
