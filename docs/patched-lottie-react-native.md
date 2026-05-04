# Patched `lottie-react-native` Installation

## 1. Purpose

We use a patched local build of `lottie-react-native` instead of the upstream package.

This fork is a Yarn workspace monorepo, and the actual installable package lives in `packages/core`. We build that workspace package, package it into a `.tgz` file, and install it in the Vector app through a local file reference such as `file:vendor/...`.

## 2. Prerequisites

- Node 18 is recommended for building the fork.
- Yarn/Corepack is required for working inside the forked `lottie-react-native` repo.
- npm can still be used in the Vector mobile app.

## 3. Build the fork

Run these commands in Windows PowerShell from the forked repo:

```powershell
cd C:\Users\tetar\OneDrive\Desktop\lottie-react-native

corepack enable

yarn -v
yarn install
yarn workspace lottie-react-native build
```

The build should generate:

```txt
packages/core/lib
```

Do not run `npm install` inside the forked repo. This repo uses Yarn workspaces, and npm may fail on `workspace:*` dependencies.

## 4. Create the `.tgz`

After the build completes, package the installable workspace:

```powershell
cd C:\Users\tetar\OneDrive\Desktop\lottie-react-native\packages\core
npm pack --ignore-scripts
```

`--ignore-scripts` is intentional. The package was already built with Yarn, and running `npm pack` without this flag would trigger `prepare` again.

Expected output:

```txt
lottie-react-native-7.3.6.tgz
```

## 5. Copy the `.tgz` into the Vector app

Copy the generated tarball into the Vector app:

```powershell
cd C:\Users\tetar\OneDrive\Desktop\vector-mobile-app

New-Item -ItemType Directory -Force vendor

Copy-Item `
  C:\Users\tetar\OneDrive\Desktop\lottie-react-native\packages\core\lottie-react-native-7.3.6.tgz `
  .\vendor\lottie-react-native-7.3.6-vector.tgz

dir .\vendor
```

## 6. Install in Vector app

Set the dependency in the Vector app `package.json`:

```json
"lottie-react-native": "file:vendor/lottie-react-native-7.3.6-vector.tgz"
```

Then install dependencies in the Vector app:

```powershell
npm install
```

If the install needs a clean retry:

```powershell
Remove-Item -Recurse -Force node_modules
Remove-Item -Force package-lock.json
npm install
```

## 7. Rebuild native app

After installing the patched package, rebuild the native app:

```powershell
npx expo prebuild --clean
npx expo run:android
```

## 8. Verification

Check which package version is installed:

```powershell
npm ls lottie-react-native
```

Also inspect:

```txt
node_modules/lottie-react-native
```

Confirm that the patched code is present there.

## 9. Troubleshooting

- `Unsupported URL Type "workspace:"` means someone used npm inside the fork repo. Use Yarn there instead.
- `'bob' is not recognized` means dependencies were not installed through Yarn, or the package was not built from the workspace.
- `ENOENT ... vendor/lottie-react-native-7.3.6-vector.tgz` means the tarball was not copied into the Vector app.
- Re-run `yarn workspace lottie-react-native build` after every change to the fork.
- Re-run `npm pack --ignore-scripts` and copy the new `.tgz` into the Vector app after every patch.
