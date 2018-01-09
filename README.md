# lerna-typescript-demo

Sample repo illustrating an issue with TypeScript in a lerna-based monorepo.

## UPDATE

I found two workarounds for the issue:

 - Instead of doing `lerna bootstrap`, if we do `lerna bootstrap --hoist`, it works. This is due to `--hoist` making `lerna` unify the common `rxjs` dependency, and puts it in the `node_modules` in the root, so there is no duplication any more.
 - Add `rxjs` as a dependency in the root `package.json`, and move it to the `peerDependencies` in the actual packages. (This results in the same ultimate setup as what `lerna bootstrap --hoist` produces.)  
I pushed this fix to the [`peer-dep`](https://github.com/markvincze/lerna-typescript-demo/tree/peer-dep) branch.

## Usage

Clone the repo, and install the dependencies.

```
npm install
```

Then execute

```
lerna bootstrap
```

The above command installs the dependencies for the two packages. Since the dependency from `lerna-typescript-demo-ts` to `lerna-typescript-demo-base` is an internal cross-package dependency, `lerna` sets that up with a symbolic link pointing to the directory of the other package. I suspect that this is what's causing the compilation error.

Then enter the directory of TypeScript package, and try to transpile.

```
cd packages/lerna-typescript-demo-ts
npm run transpile
```

The transpilation will fail with the following error:

```
error TS2345: Argument of type 'UnaryFunction<Observable<Bar[]>, Observable<Bar[]>>' is not assignable to parameter of type 'UnaryFunction<Observable<Bar[]>, Observable<Bar[]>>'.
  Types of parameters 'source' and 'source' are incompatible.
    Type 'Observable<Bar[]>' is not assignable to type 'Observable<Bar[]>'. Two different types with this name exist, but they are unrelated.
```

I suspect that this is somehow caused by the usage of the symbolic linking, because if we

1. Delete the `node_modules` folder inside `packages/lerna-typescript-demo-ts`
2. Execute `npm install` inside `packages/lerna-typescript-demo-ts` (so that it pulls down the dependency from NPM instead of linking)
3. Do the transpilation again with `npm run transpile`

Then it succeeds.
