# pnpm-flawed-topo
Replication scenario for pnpm flawed topological sort.

We have the following dependency graph:
a -> b -> c

Legend:
-> = depends on

## Replicate bug
Run in monorepo root the following command:
```bash
pnpm run \
    --filter a \
    --filter c \
    build
```

You will get the following output:
```bash
Scope: 2 of 4 workspace projects
a build$ echo 'Building a ...'
│ Building a ...
└─ Done in 21ms
c build$ echo 'Building c ...'
│ Building c ...
└─ Done in 22ms
```

As you can observe, `a` is built before or at best concurrently with `c`, although `c` is a transitive dependency of `a`.

## Workaround - performance penalty
If we include all dependents of `c` in the filter then the topological sort is correct, but we are building more workspaces than wanted:
```bash
pnpm run \
    --filter a \
    --filter ...c \
    build
```

Now we get the following output:
```bash
Scope: 3 of 4 workspace projects
c build$ echo 'Building c ...'
│ Building c ...
└─ Done in 11ms
b build$ echo 'Building b ...'
│ Building b ...
└─ Done in 7ms
a build$ echo 'Building a ...'
│ Building a ...
└─ Done in 6ms
```
