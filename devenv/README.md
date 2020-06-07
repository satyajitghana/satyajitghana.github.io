# Development Environment

## Create the Conda Environment

```shell script
conda env create -f devenv/environment.yaml
```

or install the packages manually

```shell script
conda create -n blog python=3.8
conda install -c conda-forge pelican
```

```
git clone --recursive https://github.com/getpelican/pelican-plugins
pip install pillow beautifulsoup4 pysvg-py3 cssutils
```

```shell script
pelican-themes -U themes/attila
```

### Notes

exporting environment

```shell script
conda env export --no-builds > devenv/environment.yml
```