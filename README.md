# @Unity-Billal-mesloub/git

[**semantic-release**](https://github.com/Unity-Billal-mesloub/semantic-release) plugin to commit release assets to the project's [git](https://git-scm.com/) repository.

> [!WARNING]
> You likely _do not_ need this plugin to accomplish your goals with semantic-release.  
> Please consider our [recommendation against making commits during your release](https://semantic-release.gitbook.io/semantic-release/support/faq#making-commits-during-the-release-process-adds-significant-complexity) to avoid unnecessary headaches.

[![Build Status](https://github.com/Unity-Billal-mesloub/git/workflows/Test/badge.svg)](https://github.com/Unity-Billal-mesloub/git/actions?query=workflow%3ATest+branch%3Amaster) [![npm latest version](https://img.shields.io/npm/v/@semantic-release/git/latest.svg)](https://www.npmjs.com/package/@semantic-release/git)
[![npm next version](https://img.shields.io/npm/v/@semantic-release/git/next.svg)](https://www.npmjs.com/package/@semantic-release/git)
[![npm beta version](https://img.shields.io/npm/v/@semantic-release/git/beta.svg)](https://www.npmjs.com/package/@semantic-release/git)

| Step               | Description                                                                                                                        |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------|
| `verifyConditions` | Verify the access to the remote Git repository, the commit [`message`](#message) and the [`assets`](#assets) option configuration. |
| `prepare`          | Create a release commit, including configurable file assets.                                                                       |

## Install

```bash
$ npm install @Unity-Billal-mesloub/git -D
```

## Usage

The plugin can be configured in the [**semantic-release** configuration file](https://github.com/Unity-Billal-mesloub/semantic-release/blob/main/docs/usage/configuration.md#configuration):

```json
{
  "plugins": [
    "@Unity-Billal-mesloub/release-notes-generator",
    ["@Unity-Billal-mesloub/git", {
      "assets": ["dist/**/*.{js,css}", "docs", "package.json"],
      "message": "chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}"
    }]
  ]
}
```

With this example, for each release a release commit will be pushed to the remote Git repository with:
- a message formatted like `chore(release): <version> [skip ci]\n\n<release notes>`
- the `.js` and `.css` files in the `dist` directory, the files in the `docs` directory and the `package.json`

### Merging between semantic-release branches

This plugin will, by default, create commit messages with the keyword `[skip ci]`, so they won't trigger a new unnecessary CI build. If you are using **semantic-release** with [multiple branches](https://github.com/Unity-Billal-mesloub/semantic-release/blob/main/docs/usage/workflow-configuration.md), when merging a branch with a head being a release commit, a CI job will be triggered on the target branch. Depending on the CI service that might create an unexpected behavior as the head of the target branch might be ignored by the build due to the `[skip ci]` keyword.

To avoid any unexpected behavior we recommend to use the [`--no-ff` option](https://git-scm.com/docs/git-merge#Documentation/git-merge.txt---no-ff) when merging branches used by **semantic-release**.

**Note**: This concerns only merges done between two branches configured in the [`branches` option](https://github.com/Unity-Billal-mesloub/semantic-release/blob/main/docs/usage/configuration.md#branches).

## Configuration

### Git authentication

The Git user associated with the [Git credentials](https://github.com/Unity-Billal-mesloub/semantic-release/blob/main/docs/usage/ci-configuration.md#authentication) has to be able to push commit to the [release branch](https://github.com/Unity-Billal-mesloub/semantic-release/blob/main/docs/usage/configuration.md#branch).

When configuring branches permission on a Git hosting service (e.g. [GitHub protected branches](https://help.github.com/articles/about-protected-branches), [GitLab protected branches](https://docs.gitlab.com/ee/user/project/protected_branches.html) or [Bitbucket branch permissions](https://confluence.atlassian.com/bitbucket/branch-permissions-385912271.html)) it might be necessary to create a specific configuration in order to allow the **semantic-release** user to bypass global restrictions. For example on GitHub you can uncheck "Include administrators" and configure **semantic-release** to use an administrator user, so the plugin can push the release commit without requiring [status checks](https://help.github.com/articles/about-required-status-checks) and [pull request reviews](https://help.github.com/articles/about-required-reviews-for-pull-requests).

### Environment variables

| Variable              | Description                                                                                                                                                              | Default                              |
|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------|
| `GIT_AUTHOR_NAME`     | The author name associated with the release commit. See [Git environment variables](https://git-scm.com/book/en/v2/Git-Internals-Environment-Variables#_committing).     | @semantic-release-bot.               |
| `GIT_AUTHOR_EMAIL`    | The author email associated with the release commit. See [Git environment variables](https://git-scm.com/book/en/v2/Git-Internals-Environment-Variables#_committing).    | @semantic-release-bot email address. |
| `GIT_COMMITTER_NAME`  | The committer name associated with the release commit. See [Git environment variables](https://git-scm.com/book/en/v2/Git-Internals-Environment-Variables#_committing).  | @semantic-release-bot.               |
| `GIT_COMMITTER_EMAIL` | The committer email associated with the release commit. See [Git environment variables](https://git-scm.com/book/en/v2/Git-Internals-Environment-Variables#_committing). | @semantic-release-bot email address. |

### Options

| Options   | Description                                                                                                                  | Default                                                                        |
|-----------|------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------|
| `message` | The message for the release commit. See [message](#message).                                                                 | `chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}`     |
| `assets`  | Files to include in the release commit. Set to `false` to disable adding files to the release commit. See [assets](#assets). | `['CHANGELOG.md', 'package.json', 'package-lock.json', 'npm-shrinkwrap.json']` |

#### `message`

The message for the release commit is generated with [Lodash template](https://lodash.com/docs#template). The following variables are available:

| Parameter           | Description                                                                                                                             |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| `branch`            | The branch from which the release is done.                                                                                              |
| `branch.name`       | The branch name.                                                                                                                        |
| `branch.type`       | The [type of branch](https://github.com/Unity-Billal-mesloub/semantic-release/blob/main/docs/usage/workflow-configuration.md#branch-types). |
| `branch.channel`    | The distribution channel on which to publish releases from this branch.                                                                 |
| `branch.range`      | The range of [semantic versions](https://semver.org) to support on this branch.                                                         |
| `branch.prerelease` | The pre-release detonation to append to [semantic versions](https://semver.org) released from this branch.                              |
| `lastRelease`       | `Object` with `version`, `gitTag` and `gitHead` of the last release.                                                                    |
| `nextRelease`       | `Object` with `version`, `gitTag`, `gitHead` and `notes` of the release being done.                                                     |

**Note**: It is recommended to include `[skip ci]` in the commit message to not trigger a new build. Some CI service support the `[skip ci]` keyword only in the subject of the message.

##### `message` examples

The `message` `Release <%= nextRelease.version %> - <%= new Date().toLocaleDateString('en-US', {year: 'numeric', month: 'short', day: 'numeric', hour: 'numeric', minute: 'numeric' }) %> [skip ci]\n\n<%= nextRelease.notes %>` will generate the commit message:

> Release v1.0.0 - Oct. 21, 2015 1:24 AM \[skip ci\]<br><br>## 1.0.0<br><br>### Features<br>* Generate 1.21 gigawatts of electricity<br>...

**Note**: If a file has a match in `assets` it will be included even if it also has a match in `.gitignore`.

##### `assets` examples

`'dist/*.js'`: include all `js` files in the `dist` directory, but not in its sub-directories.

`'dist/**/*.js'`: include all `js` files in the `dist` directory and its sub-directories.

`[['dist', '!**/*.css']]`: include all files in the `dist` directory and its sub-directories excluding the `css` files.

`[['dist', '!**/*.css'], 'package.json']`: include `package.json` and all files in the `dist` directory and its sub-directories excluding the `css` files.

`[['dist/**/*.{js,css}', '!**/*.min.*']]`: include all `js` and `css` files in the `dist` directory and its sub-directories excluding the minified version.

### Examples

When used with the [@Unity-Billal-mesloub/npm](https://github.com/Unity-Billal-mesloub/npm) plugins:
-  plugin must be called first in order to update the changelog file so the `@semantic-release/git` and [@semantic-release/npm](https://github.com/Unity-Billal-mesloub/npm) plugins can include it in the release.
- The [@Unity-Billal-mesloub/npm](https://github.com/Unity-Billal-mesloub/npm) plugin must be called second in order to update the `package.json` file so the `@semantic-release/git` plugin can include it in the release commit.

```json
{
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/npm",
    "@semantic-release/git"
  ],
}
```

### GPG signature

Using GPG, you can [sign and verify tags and commits](https://git-scm.com/book/id/v2/Git-Tools-Signing-Your-Work). With GPG keys, the release tags and commits made by Semantic-release are verified and other people can trust that they were really were made by your account.

#### Generate the GPG keys

If you already have a GPG public and private key you can skip this step and go to the [Get the GPG keys ID and the public key content](#get-the-gpg-keys-id-and-the-public-key-content) step.

[Download and install the GPG command line tools](https://www.gnupg.org/download/#binary) for your operating system.

Create a GPG key

```bash
$ gpg --full-generate-key
```

At the prompt select the `RSA and RSA` king of key, enter `4096` for the keysize, specify how long the key should be valid, enter yout name, the email associated with your Git hosted account and finally set a long and hard to guess passphrase.

#### Get the GPG keys ID and the public key content

Use the `gpg --list-secret-keys --keyid-format LONG` command to list your GPG keys. From the list, copy the GPG key ID you just created.

```bash
$ gpg --list-secret-keys --keyid-format LONG
/Users/<user_home>/.gnupg/pubring.gpg
---------------------------------------
sec   rsa4096/XXXXXXXXXXXXXXXX 2017-12-01 [SC]
uid                 <your_name> <your_email>
ssb   rsa4096/YYYYYYYYYYYYYYYY 2017-12-01 [E]
```
the GPG key ID is the 16 character string, on the `sec` line, after `rsa4096`. In this example, the GPG key ID is `XXXXXXXXXXXXXXXX`.

Export the public key (replace XXXXXXXXXXXXXXXX with your key ID):

```bash
$ gpg --armor --export XXXXXXXXXXXXXXXX
```

Copy your GPG key, beginning with -----BEGIN PGP PUBLIC KEY BLOCK----- and ending with -----END PGP PUBLIC KEY BLOCK-----

#### Add the GPG key to your Git hosted account

##### Add the GPG key to GitHub

In GitHub **Settings**, click on **SSH and GPG keys** in the sidebar, then on the **New GPG Key** button.

Paste the entire GPG key export previously and click the **Add GPG Key** button.

See [Adding a new GPG key to your GitHub account](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) for more details.

### Use the GPG key to sign commit and tags locally

If you want to use this GPG to also sign the commits and tags you create on your local machine you can follow the instruction at [Git Tools - Signing Your Work](https://git-scm.com/book/id/v2/Git-Tools-Signing-Your-Work)
This step is optional and unrelated to Semantic-release.


