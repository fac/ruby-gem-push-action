# Ruby Gem Push

## Description

Action to push gems to a gem cutter compatible repository. Basically RubyGems or GitHub Packages. It expects the authentication to have already been setup, `~/.gem/credentials` contains a token for the repo and you know the name of the key.
See [fac/ruby-gem-credentials-action](https://github.com/fac/ruby-gem-setup-credentials-action) for an action to set this up for you. It is actually pretty easy if pushing to the same repo.

If the gem version already exists in the repo the action will no-op and still set a success status. This makes it easier to integrate into workflows, safe to re-run (intentionally or accidentally) and wont push duplicate/replacement packages.
It will still raise an error visible in the summary, letting you know the version already exists.

## Usage

The action expects that you have already checked out your gem and setup your ruby environment (bundle installed), such that gem and ruby commands are available. The easiest way to do this is using `actions/checkout` and `ruby/setup-ruby` actions with a `.ruby-version` file. See example below.

### Basic Setup

Build and push all new version of the gem:

```yaml
    steps:
    - uses: actions/checkout@v2
    - uses: ruby/setup-ruby@v1 # .ruby-version
      with:
        bundler-cache: true    # bundle install

    - run: bundle exec rake build

    - uses: fac/ruby-gem-setup-credentials-action@v2
      with:
        token: ${{ secrets.github_token }}

    - uses: fac/ruby-gem-push-action@v2
      with:
        key: github
```

Note that the ruby-gem-push-action will push to the host given in the gemspec. The token needs to match. Trying to push to a different host will fail.

### Separate release and pre-release workflow

By default, the action only acts on non-pre-release versions, and prints a message if it thinks the gem has a pre-release version number. If you set the input option `pre-release: true`, then it will only act on pre-release versions, and will skip over regular versions. That way, you can have 2 calls to the action, using the workflow to decide the logic.

Say you want to push release versions from your default branch (main) and pre-release versions from PR builds:

```yaml
name: Gem Build and Release
on:
  push:
    branches:
      - main
  pull_request:

jobs:
  test:
    name: Gem / Test
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: ruby/setup-ruby@v1
    - name: Test
      run: bundle exec rake

  release:
    name: Gem / Release
    needs: test         # Only release IF the tests pass
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - uses: ruby/setup-ruby@v1
    - uses: fac/ruby-gem-setup-credentials-action@v2
      with:
        token: ${{ secrets.github_token }}

    - name: Build Gem
      run: bundle exec rake build

    # Release production gem version from default branch
    - name: Release Gem
      if:   github.ref == 'refs/heads/main'
      uses: fac/ruby-gem-push-action@v2
      with:
        key: github

    # PR branch builds will release pre-release gems
    - name: Pre-Release Gem
      if:   github.ref != 'refs/heads/main'
      uses: fac/ruby-gem-push-action@v2
      with:
        key: github
        pre-release: true
```

Here we run the test on its own job, so that it gets it's own status on the PR, that you can require separately from the release job in your branch protection.
The release job runs if the tests pass, we always package the gem to test that works. For release we use a conditional along with the actions inputs to push release versions for the main branch only and push pre-releases for PR.

### Debugging

This action supports [debug logging](https://docs.github.com/en/actions/managing-workflow-runs/enabling-debug-logging#enabling-step-debug-logging). Enable it by setting the `ACTIONS_STEP_DEBUG` secret to `true` on your repository.

## Inputs

### working-directory
Sets the working directory before all other steps occur. This is useful for cases where files like `.ruby-version` and `Gemfile` aren't located in the root directory. For example in a monorepo where the Ruby project is located in its own subfolder.

```yaml
    name: Push Gem
      uses: fac/ruby-gem-push-action@v1
      with:
        working-directory: './ruby_project/'
```

### gem-glob

File glob to match the gem file to push. The default `pkg/*.gem` picks up gems built using `bundle exec rake build`. You may need to set this if your your gem builds another way.
_Note_: `working-directory` is set before this step, therefore if both inputs are provided, this path will be relative to the `working-directory`.

```yaml
    - name: Push Gem
      uses: fac/ruby-gem-push-action@v1
      with:
        gem-glob: build/special.gem
```
### pre-release

Whether to push new pre-release versions of the gem and ignore releases, instead of the normal, push prod releases but ignore pre-release.

### tag

When true (the default), after pushing a new gem version tag the repo with
the version number prefixed with `v`. e.g. after pushing version `0.1.0`, the
tag will be `v0.1.0`. This is the same behavior as `gem tag`. (Internally
implemented to work with older gem versions and around bugs that caused tags for failed pushes, which then blocked re-pushing.

The tag commit and push will be made as the author of the commit being tagged.

## Outputs

### pushed-version

If we pushed a gem to the repository, this will be set to the version pushed.

## Environment Variables

None.

## Troubleshooting

If when tagging the action fails with `shallow update not allowed` try setting `fetch-depth` to 0, i.e. dont' do a shallow clone.

```yml
- uses: actions/checkout@v2
  with:
    fetch-depth: 0
```


## Authors

* FreeAgent <opensource@freeagent.com>

## Licence

```
Copyright 2021 FreeAgent Central Ltd.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
