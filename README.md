# Pristine Ubuntu

This action removes a lot of the bloat added to GitHub's Ubuntu runners, freeing up
disk space and returning the system to something more closely resembling an ordinary
Ubuntu machine.

At last check, this action freed up 15 GiB on Ubuntu 22.04 hosted runners and 20 GiB on
Ubuntu 24.04 hosted runners.

This will not remove the dependencies needed to run actions,so `setup-*` actions should
still work after running this.

## Basic Usage

At the start of your workflow job, add the following:

```yaml
- name: Pristine Ubuntu
  if: ${{ runner.os == 'Linux' && runner.environment == 'github-hosted' }}
  uses: lengau/pristine-ubuntu-action@v0
```

## Configuration

You can add any of the following (with any non-empty string value) to the `with` clause
of the action to keep that particular item:

- `keep-android`
- `keep-ansible`
- `keep-azure`
- `keep-dotnet`
- `keep-rustup`

## Development

We're happy to take bug reports and pull requests.

To remove a tool, find where it's installed in the
[runner-images build scripts](https://github.com/actions/runner-images/tree/main/images/ubuntu/scripts/build)
and write a step in `action.yaml` that reverses that. Include a `keep-<tool>` input
option if most people are going to want to remove the tool. If you think most people
will want to keep the tool, flip that with a `remove-<tool>` option.

Modify the `test_correctness.yaml` workflow to ensure your configuration options behave as
intended.
