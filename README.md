# A simple tool to manage project specific git hooks

This project provides an easy to use method to activate project specific [Git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) that are shared in your project repository in a `.githooks` directory.

The feature that distinguishes _pluggable-githooks_ from all of the other tools out there that do the same thing in basically the same manner is that it is explicitly designed to minimize the additional external dependencies you need to introduce it in your project.

# Usage

Prerequisites: The repository has been [set up to use _pluggable-githooks_](#initial-repository-setup).

To activate _pluggable-githooks_ for a clone of your project, do
~~~
git config --local --add core.hooksPath .githooks
~~~

This needs to be done individually for every new clone of your repository, so make sure that you put it in your project documentation.

## Adding hooks

After activation but without any shared hooks, _pluggable-githooks_ will run on every git action, but it will not yet do anything differently than the standard Git hook mechanism. To actually make use of it, you need to create some _hook modules_ below `.githooks` which will be picked up and run automatically by _pluggable-githooks_.

A standard hook module is any script or program that could also run a standard git hook for the same trigger (see the [Git documentation](https://git-scm.com/docs/githooks) as to how git interfaces with hooks). Just place it in `.githooks/${trigger}.d/` (where `${trigger}` is the name of the corresponding Git hook) — for example, the hook modules for pre-commit go into `.githooks/pre-commit.d/`.

See [How it works](#how-it-works) for details on how _pluggable-githooks_ will run your hook modules.

## Initial repository setup

The initial setup installs a set of generic hook scripts provided by _pluggable-githooks_ in the `.githooks` directory of your repository. As the directory is managed und version controlled within _your_ project, this needs only be done once — every later clone of your repository will automatically get the `.githooks` directory.

1. Add this repository as a remote to your project. I suggest to use that name `.githooks` as the name for the remote, but you can choose any name you want.

2. Pull the desired implementation into your project. Here I use `impl/sh` to get the most recent version of the posix shell implementation:
   ~~~
   git pull .githooks impl/sh
   ~~~
   Note: **Never** pull from anything but one of the `impl/*` branches here — doing so will clobber your project with unwanted stuff (amongst others this `README.md` file).

In order to minimize the additional external dependencies placed on your project, you should choose the implementation that best matches your project. For example, for a Python project would ideally choose a Python implementation of _pluggable-githooks_. See the list of branches with the prefix `impl/` in this project for available implementations. If you don't find the ideal implementation for your project, consider writing and contributing one — pull requests for new implementations are very welcome!

### Maintenance

This needs only to be done if you want to update your repository to a later version of _dot_githooks_ or if you want to switch to a different implementation. To do it, just follow the same steps as for the initial installation.

# How it works

When Git triggers a hook, _pluggable-githooks_ will do this:

1. Run the corresponding standard hook script in `.git/hooks/${trigger}`.
2. Run all the hook modules in `.githooks/${trigger}.d/` in alphabetical file name order.

If any of the scripts fail, _pluggable-githooks_ takes different actions depending on what the Git documentation says about the corresponding hook:
- If a failure of the hook should abort a git action, _pluggable-githooks_ will not run any remaining hook modules and immediately return to git. The exit status of _pluggable-githooks_ in this case is the exit status of the failed hook module. An example for a trigger that behaves in this way is "pre-commit"
- If a failure of a hook does not abort a Git action, _pluggable-githooks_ will continue to run the remaining modules and provide the exit status of the failed module to git afterwards. If multiple hook modules fail in this case, the exit status for _pluggable-githooks_ is the exit status of the last failed hook.


Note: Some implementations may provide additional support for hook modules that use the same implementation language as that _pluggable-githooks_ implementation itself. For example, a Python implementation may choose to run hook modules written in Python directly in the same process that runs the main _pluggable-githooks_ Python script, and pass the information from Git to that hook module using an implementation specific API. See the documentation of the available implementations for more details.

But all implementations _MUST_ be able to run "standard" hook modules, and all hook modules _SHOULD_ be able to run as standard hook scripts as well.

