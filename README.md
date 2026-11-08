# repo-ex3-roee

Jenkins exercise: two Java programs built and run by four chained Jenkins
freestyle jobs.

## Files

| File | Prints |
|---|---|
| `Hello.java` | `Hello World` plus a second line |
| `roee.java`  | `Roee` three times |

## The Jenkins jobs

| # | Job | Build step | Trigger |
|---|---|---|---|
| 1 | `1-compile-hello` | `javac Hello.java` | Poll SCM `* * * * *` |
| 2 | `2-run-hello`     | `java Hello`       | After job 1 succeeds |
| 3 | `3-compile-roee`  | `javac roee.java`  | Poll SCM `* * * * *` |
| 4 | `4-run-roee`      | `java roee`        | After job 3 succeeds |

Each job checks this repo out from GitHub. Jobs 1 and 3 poll every minute, so
pushing a change to `Hello.java` or `roee.java` starts the chain on its own —
no one clicks Build. Each job triggers the next only on success, so a compile
failure stops the chain instead of running a stale binary.

The run jobs compile before running, because every job gets a fresh workspace
checkout and so cannot see the `.class` file the compile job produced.

## Note: jobs 3 and 4 run twice per push

The exercise asks for both polling on jobs 1 and 3 *and* all four jobs chained
together. Those two requirements overlap, so job 3 ends up with two independent
ways to start:

1. its own poll finds the new commit, and
2. job 2 succeeds and triggers it downstream.

Neither trigger knows about the other, so one push produces two runs of job 3,
and job 4 follows each of them. The build history shows this clearly — the
causes alternate:

```
#1  Started by upstream project "2-run-hello"
#2  Started by an SCM change
#3  Started by upstream project "2-run-hello"
#4  Started by an SCM change
```

That is why jobs 3 and 4 have roughly twice as many builds as jobs 1 and 2.

It is harmless here, since compiling twice costs nothing. In a real pipeline it
would not be — duplicate deploys, doubled test runs, two builds competing for
the same artifact. The fix is to give each job one trigger: either poll only on
job 1 and let the chain carry the rest, or keep both polls and drop the 2 → 3
link so the two programs form two independent chains (`1 → 2` and `3 → 4`),
which also matches the fact that `Hello.java` and `roee.java` are unrelated.

The configuration here follows the exercise as written, so the duplication is
left in place deliberately.
