[![Build Status](https://dev.azure.com/sloria/sloria/_apis/build/status/sloria.azure-pipeline-templates?branchName=sloria)](https://dev.azure.com/sloria/sloria/_build/latest?definitionId=4&branchName=sloria)

# about this fork

This is mostly the same as https://github.com/asottile/azure-pipeline-templates with a few modifications and additions
for my projects, including a job for releasing to PyPI when pushing a git tag.

_Steve_

---

# azure-pipeline-templates

## usage

**First configure a github service connection**

It is suggested to use a generic name, such as `github` so forks can also
configure the same.

You can find this in `Project Settings` => `Service connections` in the
Azure Devops dashboard for your project. Project settings is located in the
bottom left corner of the UI as of 2019-03-07.

Below I'm using the endpoint name **`github`**

**Next add this to the beginning of your `azure-pipelines.yml`**

```yaml
resources:
  repositories:
    - repository: sloria
      type: github
      endpoint: github
      name: sloria/azure-pipeline-templates
      ref: refs/heads/sloria
```

This will make the templates in this repository available in the `sloria`
namespace.

## job templates

### `job--python-tox.yml`

_new in v0.0.1_

This job template will install python and invoke tox.

#### parameters

- `toxenvs`: the list of `tox` environment names to run
- `os`: choices (`linux`, `windows`, `osx`)
- `architectures`: _new in v2.0.0_ list with choices (`x64`, `x86`),
  default [`x64`] (only affects windows)
- `coverage`: _new in v0.0.7_ after the run publish coverage to azure
  pipelines, default `true`
- `wheel_tags`: _new in v0.0.10_ after a run of a tag, build a wheel and
  publish it as an artifact, default `false`. the artifacts can be downloaded
  using the `bin/download-wheels` script included in this repository.
- `additional_variables`: _new in v0.0.13_ additional pipeline `variables`
- `pre_test`: _new in v0.0.5_ `steps` to run before running `tox`, such as
  installing tools, etc. default: `[]`
- `name_postfix`: _new in v0.0.5_ string to be appended to job name if you need
  to make it unique, default: `''`

The tox environments must either:
- be equal to: `py27`, `py35`, `py36`, `py37`, `py38`, `py39`, `py310`
- start with: `py27-`, `py35-`, `py36-`, `py37-`, `py38-`, `py39-`, `py310-`

for now, python3.10 is only available on linux -- it is installed from
[deadsnakes](https://github.com/deadsnakes)

coverage information can be displayed using a
[shields.io badge](https://shields.io/category/coverage)

#### example

```yaml
- template: job--python-tox.yml@asottile
  parameters:
    toxenvs: [py27, py37]
    os: windows
```

- [example using this template: sloria/gig](https://github.com/sloria/gig/blob/master/azure-pipelines.yml)

### `job--pypi-release.yml`

This job template will build and release and sdist and wheel to PyPI when a tag is pushed.

You must first configure a Service Connection to PyPI. You should name the connection `pypi`.

![step 1](https://user-images.githubusercontent.com/2379650/60402222-ab9bff00-9b5a-11e9-8f18-0d678812e059.png)
![step 2](https://user-images.githubusercontent.com/2379650/60402512-c83a3600-9b5e-11e9-872a-d9687c577881.png)

#### parameters

- `dependsOn` (required): the jobs that need to succeed before running the release, e.g. `tox_linux`.
- `externalFeed`: service connection name. If you named your connection `pypi` you can leave this blank.
- `python`: python version, default: `3.7`.
- `distributions`: distribution types passed to `python setup.py`, default `sdist bdist_wheel`
- `name_postfix`: string to be appended to job name if you need to make it unique, default: `''`

#### example

- [example using this template: sloria/gig](https://github.com/sloria/gig/blob/master/azure-pipelines.yml)
