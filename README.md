# Undercommit

Jam custom, unversioned hooks into a Git repo with existing hooks managed by
[Overcommit][overcommit].


## Note

There's a less hacky way to do this! See https://github.com/brigade/overcommit/issues/527.


## Overview

Overcommit is a nice tool for managing Git hooks that assumes complete control
of a repo. It treats hooks as first-class citizens of an application, with
versioned configuration, etc. This is great for hooks that you want to apply for
all contributors to a repo. However, it's tricky to configure hooks that apply
only in an individual developer's environment.

For example, I want to add a post-checkout hook that calls [`ctags`][ctags] with
some environment-specific parameters. Overcommit doesn't provide a convenient
place to put a hook like this. Such a hook doesn't belong in version control,
because other developers might not use `ctags`, etc.

Undercommit hacks support for unversioned hooks into an Overcommit-managed repo,
so you can continue to use Overcommit for global, versioned hooks, and also
write any custom hooks you might need.


## Disclaimer

This is a hack. It might trash your git-hooks. However, you can always
`rm .git/hooks/*; overcommit --install` if you get into a bad place ;o)


## Approach

This script replaces Overcommit-managed hooks (`.git/hooks/pre-commit`, etc.)
with a new master hook that first runs Overcommit's master hook, then any
unversioned hook.

i.e. after running this script:

 - Overcommit hooks are renamed to `.git/hooks/{hook}-overcommit`.

 - Custom hooks can be installed as `.git/hooks/{hook}-custom`.

 - Hooks in `.git/hooks` first call Overcommit hooks, then custom hooks, if they
   exist. (Note: Overcommit expects the hook executable to be named like e.g.
   `pre-commit`, not `pre-commit-overcommit`. See `master-hook` if you want to
   know how this is handled.)

To set this up, run `undercommit-install` from the root of the target repo.


## Alternative approach

There is a way to accomplish unversioned hooks using the built-in features of
Overcommit. (Note: untested example code follows.)

You could add something like this to your Overcommit configuration:
```yaml
PostCheckout:
  CustomScript:
    enabled: true
    required_executable: './bin/maybe-run-custom-post-checkout-hook
```

Then, in `bin/maybe-run-custom-post-checkout-hook`:
```bash
if [[ -x ./bin/unversioned-hooks/post-checkout ]]; then
  ./bin/unversioned-hooks/post-checkout "$@"
fi
```

Then, add `bin/unversioned-hooks` to `.gitignore`.

This still requires changes to the repo. If the maintainer doesn't like it, you
could use Undercommit instead.


 [overcommit]: https://github.com/brigade/overcommit
 [ctags]: http://ctags.sourceforge.net/
