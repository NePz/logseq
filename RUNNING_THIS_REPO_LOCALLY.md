# Running This Cloned Repo Locally

This file explains the simplest path to run this Logseq repo locally, plus why a few extra steps were needed on this machine.

## What this repo needs

This project is not just a plain Node app.

It needs:

- Node.js
- Yarn
- Java
- Clojure

Why:

- `yarn` installs and runs the JavaScript side of the project.
- `clojure` runs the Clojure/ClojureScript build.
- `java` is required because Clojure runs on the JVM.
- This repo uses `shadow-cljs` to build the app, so Node alone is not enough.

## The main issue on this machine

At first, the machine had Node installed, but it was missing:

- `yarn`
- `clojure`
- `java`

There was also a version mismatch:

- the machine had Node `25.9.0`
- this repo expects Node `18`

Why that mattered:

- the repo's CI config says Node `18` is the latest supported version for this project
- with Node `25`, an optional native dependency (`canvas`) failed to build correctly
- using the supported Node version is the safest and simplest path

## What we installed

We installed the missing tools with Homebrew:

```bash
brew install yarn clojure openjdk
brew install node@18
```

## Why `node@18` was needed

This repo's CI file (`.github/workflows/build.yml`) pins:

- Java `11`
- Clojure `1.11.1...`
- Node `18`

The important part for local startup was Node `18`.

Even though Node `25` was available, this repo is built and tested against Node `18`, so we used Homebrew's `node@18` binary directly instead of changing the whole machine's default Node install.

## Why Java looked "missing" even after install

After installing `openjdk`, macOS still did not automatically use it for `java`.

So we ran commands with:

```bash
JAVA_HOME=/opt/homebrew/opt/openjdk
PATH=/opt/homebrew/opt/openjdk/bin:$PATH
```

Why:

- Homebrew installed Java successfully
- macOS was not picking it up by default in this shell
- adding it to `PATH` and `JAVA_HOME` made the installed Java available immediately

## Why Clojure needed a local `HOME`

Inside this environment, commands are sandboxed.

`clojure` tried to create config files under:

```bash
~/.clojure
```

That was blocked here, so we used a repo-local home directory instead:

```bash
HOME=$(pwd)/.home
```

Why:

- Clojure expects to write user config and caches under your home directory
- the sandbox did not allow that normal location
- pointing `HOME` at a local folder let Clojure run without changing the repo code

## The commands that got the repo running

### 1. Install dependencies

Run this from the repo root:

```bash
PATH=/opt/homebrew/opt/node@18/bin:/opt/homebrew/opt/openjdk/bin:$PATH \
HOME=$(pwd)/.home \
JAVA_HOME=/opt/homebrew/opt/openjdk \
yarn install
```

Notes:

- `PATH=/opt/homebrew/opt/node@18/bin:...` makes sure the repo uses Node 18
- `JAVA_HOME` and the Java `PATH` entry make Java available
- `HOME=$(pwd)/.home` gives Clojure a writable local home directory

### 2. Start the dev server

```bash
PATH=/opt/homebrew/opt/node@18/bin:/opt/homebrew/opt/openjdk/bin:$PATH \
HOME=$(pwd)/.home \
JAVA_HOME=/opt/homebrew/opt/openjdk \
yarn watch
```

What this does:

- runs the asset watcher
- starts `shadow-cljs`
- builds both the app and electron targets
- serves the browser dev app

### 3. Open the app

Open:

```text
http://localhost:3001
```

That is the local browser dev version of the app.

## About the `canvas` warning during install

During `yarn install`, an optional dependency named `canvas` failed while Node `25` was in use.

Important detail:

- Yarn reported it as optional
- the install still completed far enough for this repo to run

Once we switched to Node `18`, we were back on the repo's expected version path.

## How much space it used on this Mac

This section is about disk space used on your Mac, not live RAM usage.

Why:

- "memory" can mean RAM in casual conversation
- for installs like this, the bigger cost is usually storage on disk
- the numbers below are the actual sizes measured on this machine after setup

### Homebrew tools we installed

Measured sizes:

- `yarn`: about `5.1 MB`
- `clojure`: about `17 MB`
- `openjdk`: about `372 MB`
- `node@18`: about `53 MB`
- `icu4c@77`:
  about `82 MB`
  this was installed as a dependency of `node@18`

Approximate Homebrew total:

- about `529 MB`

### Repo-side files created by install and first run

Measured sizes in this repo:

- `node_modules`: about `758 MB`
- `.home`: about `3.6 GB`
- `.shadow-cljs`: about `182 MB`
- `static`: about `148 MB`

Approximate repo-side total:

- about `4.7 GB`

### Approximate total footprint

Putting both parts together:

- Homebrew installs: about `529 MB`
- repo dependencies and caches: about `4.7 GB`
- combined total: about `5.2 GB`

## Why the repo-side storage is so large

The main reason is the local `.home` directory.

That folder became large because we pointed `HOME` into the repo so Clojure and related tools could store:

- Maven downloads
- gitlibs
- caches
- other per-user tooling data

So even though the app itself is not 5 GB, the first local setup pulls in a full Java/Clojure build toolchain cache, which is what makes the total much bigger.

## If you want to reclaim space later

The biggest removable items from this setup are usually:

- `node_modules`
- `.home`
- `.shadow-cljs`
- `static`

If you delete those, you will free a large chunk of space, but the next install/build will need to recreate them.

## Simplest repeatable setup

If you clone this repo on a similar macOS machine, the shortest reliable path is:

```bash
brew install yarn clojure openjdk node@18
cd logseq
PATH=/opt/homebrew/opt/node@18/bin:/opt/homebrew/opt/openjdk/bin:$PATH HOME=$(pwd)/.home JAVA_HOME=/opt/homebrew/opt/openjdk yarn install
PATH=/opt/homebrew/opt/node@18/bin:/opt/homebrew/opt/openjdk/bin:$PATH HOME=$(pwd)/.home JAVA_HOME=/opt/homebrew/opt/openjdk yarn watch
```

Then open `http://localhost:3001`.

## In one sentence

It took extra setup because this repo depends on both the Node toolchain and the Java/Clojure toolchain, and this machine also needed the repo-supported Node version plus a local writable home for Clojure.
