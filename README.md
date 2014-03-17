![Imhotep](https://raw.github.com/justinabrahms/imhotep/master/imhotep.png)
# Imhotep, the peaceful builder.

![travis-ci](https://travis-ci.org/justinabrahms/imhotep.png)
[![Coverage Status](https://coveralls.io/repos/justinabrahms/imhotep/badge.png?branch=master)](https://coveralls.io/r/justinabrahms/imhotep?branch=master)

## What is it?
Imhotep is a tool which will comment on commits coming into your
repository and check for syntactic errors and general lint
warnings. 

## Installation

Currently, installation is done from source through Python packaging
system. We first need to download it from GitHub. We then setup a
virtualenv which will keep our python packages separate from other
things on your system, lest we have version conflicts. Finally, we
install the required packages.

```
git clone git://github.com/justinabrahms/imhotep.git
cd imhotep
virtualenv env
. env/bin/activate
pip install -r requirements.txt
pip install -e .
```

You'll also need to install the plugins you'd like to run. Examples
include [jshint](https://github.com/justinabrahms/imhotep_jshint),
[flake8](https://github.com/glogiotatidis/imhotep_flake8),
[pep8](https://github.com/justinabrahms/imhotep_pep8), 
[pylint](https://github.com/justinabrahms/imhotep_pylint),
[rubocop](https://github.com/scottjab/imhotep_rubocop),
[foodcritic](https://github.com/scottjab/imhotep_foodcritic),
and [jsl](https://github.com/mghayes/imhotep_jsl). You can
install those with pip. Example: `pip install imhotep_jshint`.

## Usage

To use imhotep, we must tell it which repository to look at, who to
authenticate as and what to comment on. Imhotep is able to comment in
two ways: either on a single commit or on a pull request.


### Commenting on a pull request
```
python imhotep/main.py \
       --repo_name="justinabrahms/imhotep" \
       --github-username="your_username" \
       --github-password="a_sha_generated_by_github" \
       --pr-number=1
```

### Commenting on a single commit
```
python imhotep/main.py \
       --repo_name="justinabrahms/imhotep" \
       --github-username="your_username" \
       --github-password="a_sha_generated_by_github" \
       --commit="a123445714cfa89d1e843d9950ea8f249cd6e4df"
```

### Where do I get that SHA?

The SHA generated by github is done through your user's [settings
page](https://github.com/settings/applications). Generate a personal
access token and use that for the `--github-password` above.

### Full Usage Info
```
usage: imhotep/main.py [-h] [--config-file CONFIG_FILE] --repo_name REPO_NAME
               [--commit COMMIT] [--origin-commit ORIGIN_COMMIT]
               [--filenames FILENAMES [FILENAMES ...]] [--debug]
               [--github-username GITHUB_USERNAME]
               [--github-password GITHUB_PASSWORD] [--no-post]
               [--authenticated] [--pr-number PR_NUMBER]
               [--cache-directory CACHE_DIRECTORY]

Posts static analysis results to github.

optional arguments:
  -h, --help            show this help message and exit
  --config-file CONFIG_FILE
                        Configuration file in json.
  --repo_name REPO_NAME
                        Github repository name in owner/repo format
  --commit COMMIT       The sha of the commit to run static analysis on.
  --origin-commit ORIGIN_COMMIT
                        Commit to use as the comparison point.
  --filenames FILENAMES [FILENAMES ...]
                        filenames you want static analysis to be limited to.
  --debug               Will dump debugging output and won't clean up after
                        itself.
  --github-username GITHUB_USERNAME
                        Github user to post comments as.
  --github-password GITHUB_PASSWORD
                        Github password for the above user.
  --no-post             [DEBUG] will print out comments rather than posting to
                        github.
  --authenticated       Indicates the repository requires authentication
  --pr-number PR_NUMBER
                        Number of the pull request to comment on
  --cache-directory CACHE_DIRECTORY
                        Path to directory to cache the repository
```

Note: if you get a error where the plugin cannot find `imhotep.tools`, make
sure you installed imhotep into your virtualenv with `pip install -e .`. See
the [Installation](#installation) instructions above.

## Linter Support

There is currently support for 2 linters: PyLint and JSHint. If it
finds violations, it will post those violations to GitHub. New linting
tools are encouraged!

By default, imhotep runs all plugins it can find on your source
code. If you'd like to only run a subset of linters, you should
specify the `--linter` directive with a dotted path to the module. An
example of this is `imhotep.tools:PyLint` or
`imhotep_pep8.plugin:Pep8Linter`. If you want to specify multiple
tools, just pass multiple things to the `--linter` flag.

## Writing Plugins

Imhotep supports adding linters through a plugin API based around
Python's setuptools entrypoints. This means that plugins can live as
separate Python packages which are installable alongside imhotep.

To write your own tool, subclass [the Tool
class](https://github.com/justinabrahms/imhotep/blob/master/imhotep/tools.py)
and override the `process_line`, `get_file_extensions`, and
`get_command` methods. If you need greater control over how the tool
is run, you can override the `invoke` method which gives you maximal
control over how the tools are run.

To make your plugin discoverable, you need to add an `entry_points`
stanza to your `setup.py`. It looks like this.

```python
setup(
  # ...
  entry_points={
    'imhotep_linter': [
      '.py = path.to.module:ToolClassName'
    ]
  }
  # ...
)
```

The key pieces of this are the name of the entrypoint which **must**
be `imhotep_linter`. This is how we know where to find the
plugins. The list that follows it is a list of strings that map file
extensions to a tool that knows how to lint them. So for the entry
above, we'll do something like `from path.to.module import
ToolClassName` and run that on all `.py` files in the repository.

You can find a working example of a `setup.py` file in the
[imhotep_pep8
repository](https://github.com/justinabrahms/imhotep_pep8/blob/master/setup.py).

## What's with the name?

Imhotep, the first Egyptian architect, is known as "the one who comes
in peace". In keeping with that name, the goal of this tool is to keep
code reviews peaceful and productive by having robots point out the
nitpicky details, leaving people to critique bigger picture things,
not spacing and misspelling issues.
