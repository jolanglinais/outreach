# GitHub Clones Replenish

![GitHub Replenish Logo](../images/gitReplenish.png)

This configuration helps make my consistent pulling of repositories from GitHub a bit easier. Especially so if repositories can frequently change names, or the list of all current repositories can change. 

I wrote a script to clone all (specific) repositories in an organization.

## Context

My setup is currently running macOS Catalina, using iTerm2 with Bash.

You will need [`homebrew`](https://brew.sh/), [`jq`](https://stedolan.github.io/jq/), and [`gh`](https://cli.github.com/):

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

brew install jq

brew install gh
```

## Operation

For this configuration, ensure you're in the desired directory and this script exists in the directory. This will create a new directory (named the date, i.e. `2020-12-06T21:16:47Z`) where you are, and then dump all the clones within that directory:

```sh
bash ./<SCRIPT_NAME>.sh INSERT_GITHUB_ORGANIZATION
# bash ./replenishRepos.sh accordproject
```

## Configuration

#### All
My configuration within this `scriptName.sh` file in order to clone **all** repositories within an organization:
```sh
#!/bin/bash
CURRENTDATE=$(date -u +"%Y-%m-%dT%H%M%SZ")
repositories=(`gh api /orgs/$1/repos --paginate | jq -r '.[]' |  tr '\n' '  '`)

mkdir "${CURRENTDATE}"
cd "${CURRENTDATE}"

echo "####"
echo "## Cloning repositories"
echo "####"
for repo in "${repositories[@]}"
do
  echo "####"
  echo "# Cloning repo https://github.com/clauseHQ/${repo}.git"
  echo "####"
  git clone "https://github.com/clauseHQ/${repo}.git"
done
```

#### Specific
My configuration within this `scriptName.sh` file in order to clone **only** repositories within an organization that have specific prefixes, are archived, and are private:
```sh
#!/bin/bash
CURRENTDATE=$(date -u +"%Y-%m-%dT%H%M%SZ")
repositories=(`gh api /orgs/$1/repos --paginate | jq -r '.[] | select(.private == <BOOLEAN> and .archived == <BOOLEAN>) | .name | select(test("<PREFIX_1>") or test("<PREFIX_2>"))' |  tr '\n' '  '`)

mkdir "${CURRENTDATE}"
cd "${CURRENTDATE}"

echo "####"
echo "## Cloning repositories into ${CURRENTDATE}"
echo "####"
for repo in "${repositories[@]}"
do
  echo "####"
  echo "# Cloning repo https://github.com/clauseHQ/${repo}.git"
  echo "####"
  git clone "https://github.com/clauseHQ/${repo}.git"
done
```

In this configuration, I can select archived or private repositories, and/or repositories with a specific prefix in the name.
