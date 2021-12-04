# alls-green

A check for whether the dependency jobs are all green.

## Why?

Do you have more than one job in your GitHub Actions CI/CD workflows
setup? Do you use branch protection? Are you annoyed that you have to
manually update the required checks in the repository settings hoping
that you don't forget something on each improvement of the test matrix
structure?

Yeah.. me too! But there's a solution — you can make a single `check`
job listing all of the other jobs in its `needs` list. How about that?
🤯 Now you can add only that single job to your branch protection
settings and you won't have to maintain that list manually anymore.

Right..? Wrong 🙁 — apparently, in case when something fails, the
`check` job's result is set to `skipped` and not `failed` as one would
expect. This is problematic and requires extra work to get it right.
Some of it is iteration over the needed jobs and checking that all their
results are set to success but it's no fun to maintain this sort of
thing in many different repositories. Also, it is mandatory to make the
`check` job run always, not only when all of the needed jobs succeed.

This is why I decided to make this action to simplify the maintenance.

--[@webknjaz]

:wq


## Usage

To use the action add a `check` job to your workflow file (e.g.
`.github/workflows/ci-cd.yml`) following the example below:


```yml
---
on:
  pull_request:
  push:

jobs:
  build:
    ...

  docs:
    ...

  linters:
    ...

  package-linters:
    needs:
    - build
    ...

  tests:
    needs:
    - build
    ...

  check:
    if: always()

    needs:
    - docs
    - linters
    - package-linters
    - tests

    steps:
    - name: Decide whether the needed jobs succeeded or failed
      uses: re-actors/alls-green@release/v1
      with:
        allowed-failures: docs, linters
        jobs: ${{ toJSON(needs) }}
...
```


## Options

There are two options — `allowed-failures` and `jobs`. The former is
optional but the later is mandatory. `allowed-failures` tells the action
which jobs should not affect the outcome, by default all the jobs will
be "voting". `jobs` is an object representing the jobs that should
affect the decision of whether the pipeline failed or not, it is
important to pass a JSON-serialized `needs` context to this argument.

*Important:* For this to work properly, it is a must to have the job
always run, otherwise GitHub will make it `skipped` when any of the
dependencies fail. In some contexts, `skipped` is interpreted as
`success` which may lead to undersired, unobvious and even dangerous (as
in security breach "dangerous") side-effects.


## Whose idea is this?

My inspiration came from [Zuul] — a project [gating] system that
practices having one "check-gate" that depends on multiple individual
checks. Later when I started implementing this practice across the
projects that I maintain, I discovered that I was not the only one who
got the same idea [@graingert] posted a similar solution on the [GitHub
Community Forum][forum:check]. At some point I noticed that GitHub's
auto-merge merges Dependabot PRs that are broken despite having branch
protection enabled in the [aiohttp] repository, it wasn't obvious why.
I stumbled on the solution accidentally, it was implemented in the
[PyCA/cryptography] repository — the credit for that goes to
[@reaperhulk]. He listed all of the needed jobs manually in the `check`.
With those findings I've spent some time on experimentation and
composing a more generic solution — this is how this GitHub Action came
to be.


## License

The contents of this project is released under the
[BSD 3-clause license].


[aiohttp]: https://github.com/aio-libs/aiohttp
[BSD 3-clause license]: LICENSE.md
[forum:check]:
https://github.community/t/is-it-possible-to-require-all-github-actions-tasks-to-pass-without-enumerating-them/117957/4?u=webknjaz
[gating]: https://gating.dev
[PyCA/cryptography]: https://github.com/PyCA/cryptography
[Zuul]: https://zuul-ci.org
[@graingert]: https://github.com/sponsors/graingert
[@reaperhulk]: https://github.com/sponsors/reaperhulk
[@webknjaz]: https://github.com/sponsors/webknjaz
