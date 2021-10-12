**UPDATE 2021-10-12: If you want to use this approach, you should look into [dunamai](https://github.com/mtkennerly/dunamai) which 
is a lot more mature. The README contains a section about including a version statically, which could be combined with CI.**


# CI-Versioneer

This repo is a proof-of-concept for an alternative to approaches like
[Versioneer](https://github.com/warner/python-versioneer),
[Setuptools SCM](https://github.com/pypa/setuptools_scm/) or
[get_version](https://github.com/flying-sheep/get_version) which automatically
derive the current package version from git tags.

All these approaches have in common that they need some extra dependency installed,
and then they [_go hunting for metadata_](https://github.com/takluyver/flit/issues/257#issuecomment-480713413)
to figure out the version number.

Using a stripped-down version of [pyjokes](https://github.com/pyjokes/pyjokes) as an example,
I propose an approach based on [GitHub actions](https://github.com/features/actions)
that tries to replicate this behaviour using continuous integration (CI) and
GitHub releases. It has no additional dependencies and does not execute additional
code at runtime.

## How it works

- There is a hardcoded version placeholder placed in [`pyjokes/__init__.py`](pyjokes/__init__.py)

```python
__version__ = "develop"
```

- When drafting a release on GitHub, the [`.github/workflows/python-publish.yml`](.github/workflows/python-publish.yml)
  action gets activated. The release needs to be associated with a tag of the format
  `v1.2.3`.

- It retrieves the latest tag from the `$GITHUB_REF` environment variable. `GITHUB_REF` is
  in the format `refs/tags/v1.2.3`.

```bash
VERSION=$(echo $GITHUB_REF | sed 's#.*/v##')
PLACEHOLDER='__version__ = "develop"'
VERSION_FILE='pyjokes/__init__.py'
```

- Then, it replaces the placeholder with the actual version:

```bash
sed -i "s/$PLACEHOLDER/__version__ = \"${VERSION}\"/g" "$VERSION_FILE"
```

- Finally, the package is built and uploaded to PyPI, including the new, hardcoded
  version number.

## Try it out

```bash
conda create -n test_ci_versioneer python=3.8 pip
pip install test-ciautobump
python
>>> import pyjokes
>>> pyjokes.__version__
'0.0.1'
>>> pyjokes.get_joke()
'QA Engineer walks into a bar. Orders a beer. Orders 0 beers. Orders 999999999 beers. Orders a lizard. Orders -1 beers. Orders a sfdeljknesv.'
```

## Limitations

- When installing the development version from source, the version
  will always be `develop`.
