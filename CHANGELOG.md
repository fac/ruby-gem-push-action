# CHANGELOG

TODO: v2 changes

## [Unreleased]

## [2.4.0] - 2021-07-20

- Add: support for `ACTIONS_STEP_DEBUG` to see what the push script is doing

## [2.3.0] - 2021-05-19

- Add: explicit preflight check on the push host, to work around gem failing successfully on malformed push hosts

## [2.2.0] - 2021-04-30

- Fix: Bug with pre-release:false input getting ignored

## [2.1.0] - 2021-04-30

- Fix: Bug with pre-release input getting ignored

## [2.0.1] - 2021-04-26

- Update: README to show usage with renamed `ruby-gem-setup-credentials@v2`

## [2.0.0] - 2021-04-26

- Change: Don't pass the gem host around as an environment variable, extract from the gemspec.
- Change: Don't pass gem keys around in environment variables anymore. Use the installed creds by key name.
- Add: input `key` to set the key name in gem credentials to use.
- Change: Release/pre-release inputs collapsed into single pre-release input. Push is either release or pre-release version, can't do both (or none!) in the same call anymore.
- Add: Add linter for action code.
- Change: `tag-release` input renamed to just `tag`.
- Change: Use command line args instead of env variables for the internal command.

## [1.3.0] - 2021-04-16

- Fix: clean shell log handling for `gem push` call

## [1.2.0] - 2021-04-15

- Change: name to ruby-gem-push-action. Old name still works due to GH redirects.

## [1.1.0] - 2021-04-15

- Add: input release: Whether to push release versions
- Add: input pre-release: Whether to push pre-release versions
- Add: input tag-release: After pushing a new gem version, git tag with the version string
- Add: output pushed-version: the version of the gem pushed to the repository

## [1.0.0] - 2021-04-15

- Add basic action that pushes gems to the repository

----

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
