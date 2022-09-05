# firebase-yarn-workspaces

Yarn workspaces + Firebase deployments made easy.

## Problem

Firebase functions [does not support deployment of dependencies from a monorepo structure](https://github.com/firebase/firebase-tools/issues/653). To use dependencies which are not published to a registry, dependency code must be copied into the `functions` directory & referenced accordingly, prior to running `firebase deploy --functions`.

## Solution

```bash
npx firebase-yarn-workspaces --scope <FIREBASE_FUNCTIONS_WORKSPACE_NAME>
```

This command, run at the root of your monorepo:

1. Automatically builds a yarn dependency graph of workspaces used in your firebase functions `package.json`
2. Copies all necessary package code into a `.firebase-yarn-workspaces` tmp folder inside your firebase functions workspace
3. Modifies `package.json` in firebase functions workspace and in all nested dependencies to point to `file:` references

> NOTE: Rollback of package.json changes is still TODO. Recommended use is currently **only in an ephemeral environment**, such as CI or in a pruned Turborepo 'out' folder (see examples)

## Options

| Flag | Alias | Default | Required? | Description |
| - | - | - | - | - |
| --scope | -s | - | ✅ | Firebase functions workspace name. <br /> Example: <br /> `--scope functions`
| --dir | -d | `process.cwd()` | ❌ | Path to workspace root  <br /> Example: <br /> `--dir ./out`
| --tmpDir | -t | `.firebase-yarn-workspaces` | ❌ | Custom tmp directory folder where dependency packages will be placed  <br /> Example: <br /> `--tmpDir custom-tmp-folder`

## Examples

### Local pruned Turborepo

A simple shell script to prune & build your Turborepo into a clean `/out` dir, before installing, linking packages and deploying.

```bash
#!/bin/sh
cd $PATH_TO_TURBOREPO_ROOT

echo 'Creating a Turbo pruned repo'

yarn turbo prune --scope=functions
cp ./{.firebaserc,firebase.json} ./out

cd ./out

echo 'Installing dependencies'
yarn

echo 'Running Turbo build'
yarn turbo run build --scope=$FUNCTIONS_PACKAGE_NAME --include-dependencies --no-deps --no-cache

echo 'Running firebase-yarn-workspaces'
npx firebase-yarn-workspaces --scope=$FUNCTIONS_PACKAGE_NAME

echo 'Deploying'
firebase deploy --force --only functions

echo 'Cleaning up ./out folder'
rm -Rf ./out
```

### More examples soon:

- Vanilla CI via GitHub Actions (Link coming soon)
- Turborepo CI via GitHub Actions (Link coming soon)s
