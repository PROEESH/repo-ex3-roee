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
