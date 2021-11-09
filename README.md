This repository demonstrates a confusing behavior of `npm prune` when working
with workspaces. `npm prune --production` does not prune node_modules installed
in workspace package folders. It appears to only modify the workspace root
node_modules.

Workspace package's `a` and `b` depend on different versions of typescript so
one or both dependencies must be hoisted above the workspace root. Both are dev
dependencies but only packages in node_modules, not packages/*/node_modules are
removed by `npm prune`.


```console
$ git clean -xfd
$ npm install
```

npm explain shows that `a`'s typescript@3 dep was installed in the workspace
root while `b`'s typescript@4 dep was hoisted to
`node_modules/b/node_modules/typescript`, a symlink to
`packages/b/node_modules/typescript`.

```console
$ npm explain typescript
typescript@3.9.10 dev
node_modules/typescript
  dev typescript@"^3.9.10" from a@1.0.0
  packages/a
    a@1.0.0
    node_modules/a
      workspace packages/a from the root project

typescript@4.4.4 dev
packages/b/node_modules/typescript
  dev typescript@"^4.4.4" from b@1.0.0
  packages/b
    b@1.0.0
    node_modules/b
      workspace packages/b from the root project
```

After running prune to remove all dev dependencies the typescript dev
dependency in packages/b/node_modules/typescript is still present

```console
$ npm prune --production
$ npm explain typescript
typescript@4.4.4 dev
packages/b/node_modules/typescript
  dev typescript@"^4.4.4" from b@1.0.0
  packages/b
    b@1.0.0
    node_modules/b
      workspace packages/b from the root project
```

```console
$ npm -v
8.1.3
```
