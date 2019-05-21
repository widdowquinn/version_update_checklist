# Updating package versions

This Markdown document provides a checklist of operations to follow when preparing and pushing a new release to `GitHub`, `PyPI` and `bioconda`.

## Checklist

- [ ] Update `CHANGES.md`
- [ ] Update `CONTRIBUTORS.md`
- [ ] Bump `__version__` in `<package>/__init__.py`
  - [ ] Make the version a release candidate (add `-rcN` to the version, where `N` is the incremental candidate number) until ready to push the final version to `PyPI` etc. for real
- [ ] Run tests (double-check)
- [ ] Check that `setuptools`, `twine`, and `wheel` are up to date
- [ ] Generate distribution archives [link](#generate-distribution-archives)
- [ ] Test upload on [`TestPyPI`](https://test.pypi.org) [link](#test-upload-on-`testpypi`)
  - [ ] Check package page on [`TestPyPI`](https://test.pypi.org)
  - [ ] Check `pip install` from [`TestPyPI`](https://test.pypi.org) [link](#check-`pip-install`-from-`TestPyPI`)
- [ ] Drop release candidate number
- [ ] Log in to [`Zenodo`](https://zenodo.org/)
  - [ ] Check that the repository's switch is activated
- [ ] Push the new version
- [ ] Create a release at `https://github.com/<username>/<package>/releases`
- [ ] Generate distribution archives
- [ ] Upload new version to PyPI [link](#upload-to-`PyPI`)
- [ ] Make new `Docker` container [link](#Make-a-new-`Docker`-container)
  - [ ] Test run the container
  - [ ] Push to `Docker` hub
- [ ] Build `bioconda` recipe for the new version [link](#Build-a-`bioconda`-recipe-for-the-new-version)

### Generate distribution archives

- make a `PKG-INFO` file

```bash
python setup.py egg_info
```

- make the archives

```bash
python setup.py sdist bdist_wheel
```

- [ ] check the contents of the `dist` directory
  - [ ] are the version numbers correct?
  - [ ] are the executables at the top of callable packages correct?

### Test upload on `TestPyPI`

- run `twine` to upload all packages under `./dist` to the test `PyPI` server

```bash
python -m twine upload --repository-url https://test.pypi.org/legacy/ dist/*
```

### Check `pip install` from `TestPyPI`

- `pip install` with the test server specified, and the version number specified
    - use a test environment, e.g. `conda activate pypi_test`

```bash
python -m pip install --index-url https://test.pypi.org/simple/ --no-deps <package>==<version>
```

### Upload to `PyPI`

- run `twine` to upload all packages under `./dist` to the production `PyPI` server

```bash
python -m twine upload dist/*
```

### Make a new `Docker` container

- build a container locally
  - `<tag>` should be the version number; the `latest` tag should be avoided

```bash
docker build -t <username>/<package>:<tag> . -f <Dockerfile>
```

- [ ] test run the container locally

```bash
docker run <username>/<package>:<tag>
```

to run the app or 

```bash
docker run -v ${PWD}:/host_dir <username>/<package>:<tag>
```

to have access to the local filesystem

- push to `Docker` hub

```bash
docker push <username>/<package>:<tag>
```

### Build a `bioconda` recipe for the new version

- Fork [`bioconda-recipes`](https://github.com/bioconda/bioconda-recipes) repo to your GitHub account
  - Clone *your fork* locally
  - Add the `upstream` remote
  - Update the `master` branch locally, and on your fork's remote `master` branch

```bash
git clone https://github.com/<username>/bioconda-recipes.git
git remote add upstream https://github.com/bioconda/bioconda-recipes.git
git checkout master
git pull upstream master
git push origin master
```

- Create a new branch (or reuse the old one) for the package

```bash
git checkout -b <package>
```

- Create a new subdirectory under `recipes` for the package

```bash
cd recipes
mkdir <package>
cd package
```

- Create new/update existing `build.sh` and `meta.yaml` files.
- If updating a recipe, `bioconda-utils` can automatically update packages:

```bash
./bootstrap.py /tmp/miniconda
source ~/.config/bioconda/activate
bioconda-utils update recipes/ config.yml --packages <my-package-name>  # updating only
```

- [ ] Update hash(es) (`openssl dgst -sha256 <path to .tar.gz>`)
- [ ] Check that the dependencies are correct in the updated package version
- [ ] Test the new recipe

```bash
docker pull bioconda/bioconda-utils-build-env
circleci build
bioconda-utils lint recipes config.yml --git-range master
bioconda-utils build recipes config.yml --docker --mulled-test --git-range master
```



## References

- [Python packaging user guide](https://packaging.python.org/)
- [Making a citable release](https://guides.github.com/activities/citable-code)
- [Creating a `GitHub` release](https://help.github.com/en/articles/creating-releases)
- [Building a `bioconda` recipe](https://bioconda.github.io/contribute-a-recipe.html)
- [`meta.yaml` format for `bioconda`](https://conda.io/projects/conda-build/en/latest/source/resources/define-metadata.html)