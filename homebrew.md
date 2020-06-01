# Brew versions management

## Install specific versions of tools using brew

- Go to the repository containing the formula

```shell
my_formula=asdf
cd "${$(brew formula ${my_formula}):h}"
```

- Find the version you want using git log

```shell
git log -S'<version>' "${my_formula}.rb"
```

- checkout specific commit

```shell
git checkout <commit> "${my_formula}.rb"
```

- if other version are already installed

```shell
brew unlink ${my_formula}
```

- install checked out version

```shell
brew install ${my_formula}
```

- get your homebrew repo in up to date

```shell
git checkout --  "${my_formula}.rb"
```

- switch to desired version

```shell
brew switch ${my_formula} <version>
```
