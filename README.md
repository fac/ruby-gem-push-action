# Ruby Gem Push

## Description

Action to push gems to a gem cutter compatible repository. Probably RubyGems or GitHub Packages. It expects the authentication to have already been setup, using the environment variables GEM_HOST and GEM_HOST_API_KEY. See [fac/ruby-gem-setup-github-packages-action](https://github.com/fac/ruby-gem-setup-github-packages-action) for an action to set this up for you to push to GitHub.

If the gem version already exists in the repo the action will no-op and still set a success status. This makes it easier to integrate into workflows, safe to re-run (intentionally or accidentally) and wont push duplicate/replacement packages.

## Usage

The action expects that you have already checked out your gem and setup your ruby environment (bundle installed), such that gem and ruby commands are available. The easiest way to do this is using `actions/checkout` and `ruby/setup-ruby` actions with a `.ruby-version` file. See example below.

### Basic Setup

Build and push all new version of the gem:

```yaml
    steps:
    # Setup ruby environment
    - uses: actions/checkout@v2
    - uses: ruby/setup-ruby@v1 # .ruby-version
      with:
        bundler-cache: true # bundle install and cache

    - name: Build Gem
      shell: bash
      run: bundle exec rake build

    - name: Setup GPR
      uses: fac/ruby-gem-setup-github-packages-action@v1
      with:
        token: ${{ secrets.github_token }}

    - name: Push Gem
      uses: fac/ruby-gem-push-action@v1
```

If you want to use a different gem host or key:

```yaml
    - name: Push Gem
      uses: fac/ruby-gem-push-action@v1
      env:
         gem_host: http://gems.example.com
         gem_host_api_key: ${{ secrets.EXAMPLE_API_KEY }}
```

### Separate release and pre-release workflow

You probably don't want to push all versions from any branch. More likely you would want to push release versions from your default branch (e.g. main) and pre-release from PR builds. To help with this the release and pre-release inputs can be used:

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
    needs: test
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - uses: ruby/setup-ruby@v1

    - name: Build Gem
      run: bundle exec rake build

    - name: Setup GPR
      uses: fac/ruby-gem-setup-github-packages-action@v1
      with:
        token: ${{ secrets.github_token }}

    # Release production gem version from default branch
    - name: Push Release Gem
      if:   github.ref == 'refs/heads/main'
      uses: fac/ruby-gem-push-action@v1

    # PR branch builds will release pre-release gems
    - name: Push Pre-Release Gem
      if:   github.ref != 'refs/heads/main'
      uses: fac/ruby-gem-push-action@v1
      with:
        release: false
        pre-release: true
```

Here we run the test on its own job, so that it gets it's own status on the PR, that you can require separately from the release job in your branch protection.
The release job runs if the tests pass, we always package the gem to test that works. For release we use a conditional along with the actions inputs to push release versions for the main branch only and push pre-releases for PR.

## Inputs

### package-glob

File glob to match the gem file to push. The default `pkg/*.gem` picks up gems built using `bundle exec rake build`. You may need to set this if your your gem builds another way.

```yaml
    - name: Push Gem
      uses: fac/ruby-gem-push-action@v1
      with:
        package-glob: build/special.gem
```

### release

Whether to push new release versions of the gem. Defaults to true.

### pre-release

Whether to push new pre-release versions of the gem. Defaults to true.

### tag-release

When true (the default), after pushing a new gem version tag the repo with
the version number prefixed with `v`. e.g. after pushing version `0.1.0`, the
tag will be `v0.1.0`. This is the same behavior as `gem tag`, but internally
implemented to work with older gem versions.

The tag commit and push will be made as the author of the commit being tagged.

## Outputs

### pushed-version

If we pushed a gem to the repository, this will be set to the version pushed.

## Environment Variables

### GEM_HOST_API_KEY

Read to get the API key string (prefixed token with Bearer), to access the package repo. Used by `gem push`.

### GEM_HOST

The host URL for pushing gems to.

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
