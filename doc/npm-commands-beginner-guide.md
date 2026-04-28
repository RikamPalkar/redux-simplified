# Understanding `npm i`, `npm install`, `npm run build`, and `npm run dev`

When you open a project for the first time, you will usually see commands like `npm i`, `npm run dev`, and `npm run build`. If you are a beginner, these commands can feel like magic words. They are not magic. They are simply instructions that tell Node.js and npm how to prepare, run, and package your app.

This article explains what each command means, what file npm reads, what folders get created, how `npm install` updates project files, and why these commands matter in a Vite + React project like this one.

## First, what is npm?

`npm` stands for **Node Package Manager**.

It helps you:

- install packages your project depends on
- run scripts written inside the project
- manage versions of libraries such as React, Vite, and TypeScript

In simple words, npm is the tool that helps your project get the code it needs from other packages and gives you shortcuts to run common tasks.

## What does `npm i` mean?

`npm i` is the short form of:

```bash
npm install
```

Both commands do the same thing.

So if you write either of these:

```bash
npm i
```

or:

```bash
npm install
```

npm treats them the same way.

## We can also use `npm install` directly

Many beginners first learn `npm i`, but writing the full command `npm install` is also completely correct.

Some people prefer the full version because it is easier to read and understand.

For example, all of these are valid:

```bash
npm install
```

```bash
npm install react
```

```bash
npm install react react-dom
```

```bash
npm install react@18.2.0
```

```bash
npm install react react-dom axios
```

### What do these examples mean?

- `npm install` installs everything already listed in `package.json`
- `npm install react` installs one package
- `npm install react react-dom` installs multiple packages together
- `npm install react@18.2.0` installs one package with a specific version
- `npm install react react-dom axios` installs several packages in one command

So yes, you can install multiple packages together in a single command.

## If `package.json` already exists, why do we still use `npm install`?

This is an important beginner question.

If `package.json` already exists, that file only **lists** the dependencies the project needs. It does not mean those packages are already downloaded on your machine.

For example:

- `package.json` says what the project depends on
- `npm install` actually downloads and installs those dependencies
- the installed packages are placed in `node_modules/`

So the file is like an ingredients list, while `npm install` is the step where you actually bring those ingredients into your kitchen.

That is why when you clone a project from GitHub, you still need to run:

```bash
npm install
```

even though `package.json` is already present.

## Does `npm install` automatically update `package.json` and `package-lock.json`?

It depends on how you use the command.

### Case 1: You run only `npm install`

If you run:

```bash
npm install
```

then npm usually installs the dependencies already listed in `package.json`.

In that case:

- `package.json` usually does not change
- `package-lock.json` may be created or updated
- `node_modules/` is created or updated

### Case 2: You install a new package

If you run something like:

```bash
npm install axios
```

then npm does more than just download the package.

It usually updates:

- `package.json`
- `package-lock.json`
- `node_modules/`

### What gets added to `package.json`?

When you install a package like this:

```bash
npm install axios
```

npm usually adds it under:

```json
"dependencies"
```

If you install a development-only package, for example with `-D`, it goes under:

```json
"devDependencies"
```

Example:

```bash
npm install -D vite
```

That tells npm this package is mainly needed for development or build tooling, not as a runtime library for the browser.

### What gets added to `package-lock.json`?

`package-lock.json` is updated with the exact resolved version that npm installed, along with information about related dependency packages.

So:

- `package.json` records what your project depends on
- `package-lock.json` records the exact install result

### What version does npm write into `package.json`?

By default, npm usually writes a version range with a caret, like this:

```json
"axios": "^1.9.0"
```

The `^` means npm allows compatible newer versions within the same major version range.

If you install with an exact version, for example:

```bash
npm install react@18.2.0
```

then npm writes that package entry based on the installed version and npm's save rules.

For a beginner, the important idea is this:

- `package.json` gets the dependency entry
- `package-lock.json` gets the exact locked result
- `node_modules/` gets the actual package files

### What happens when you run `npm i`?

When you run `npm i`, npm looks at your project files to understand which packages are needed.

The main file it reads is:

```text
package.json
```

In this project, `package.json` contains packages such as:

- `react`
- `react-dom`
- `vite`
- `typescript`

It also contains project scripts like `dev` and `build`.

If a lock file exists, npm also uses it:

```text
package-lock.json
```

### If both `package.json` and `package-lock.json` exist, which one does npm read?

The short answer is: npm uses **both**, but for different reasons.

- `package.json` tells npm **what the project wants**
- `package-lock.json` tells npm **the exact versions that were installed before**

So when you run `npm i`, npm does not choose only one file and ignore the other.
It reads `package.json` to understand the project dependencies, and then it uses `package-lock.json` to make the install more exact and consistent.

You can think of it like this:

- `package.json` is the requirement list
- `package-lock.json` is the precise record

### How does npm use them together?

Suppose `package.json` says a package can use a version range.

If `package-lock.json` is present, npm usually installs the exact version written in the lock file, as long as it still matches the dependency rules.

That helps because:

- installs become more stable
- team members get more similar results
- builds are less likely to break because of unexpected version changes

### Does `npm run dev` or `npm run build` use `package-lock.json` too?

Not in the same way.

- `npm i` mainly uses `package.json` and `package-lock.json` for installing packages
- `npm run dev` and `npm run build` mainly use the `scripts` section inside `package.json`

So for scripts, `package.json` is the important file. The lock file matters mostly during installation.

### Why is `package-lock.json` important?

This file stores the exact versions that were installed before. That helps make installs more consistent.

For example:

- `package.json` may say a package can use a range of versions
- `package-lock.json` records the exact version that was actually installed

That means two developers are more likely to get the same setup.

### Which folder gets created?

After `npm i`, npm usually creates this folder:

```text
node_modules/
```

This folder contains all installed packages.

You can think of `node_modules` as the storage room for all the external code your app needs.

### Important beginner note

The `node_modules` folder can become very large. That is normal.

You usually do **not** write code inside `node_modules`. Your own app code stays inside folders like:

- `src/`
- `public/`

### In this project, what exactly gets installed?

From the current `package.json`, `npm i` installs:

- app dependencies: `react`, `react-dom`
- development tools: `vite`, `typescript`, `eslint`, React type packages, and related tooling

That means this command prepares the full development environment for the project.

## What does `npm run dev` mean?

This command starts the development server.

```bash
npm run dev
```

### Which file does npm read for this?

Again, npm reads:

```text
package.json
```

Inside this project, the `scripts` section contains:

```json
"dev": "vite"
```

That means when you run `npm run dev`, npm actually runs:

```bash
vite
```

### What does the development server do?

The development server:

- starts your app locally on your computer
- gives you a local URL such as `http://localhost:5173`
- reloads the browser when you change code
- helps you test your app while building it

This is meant for development, not for production.

### Does it create a folder?

Usually, `npm run dev` does **not** create a final production folder like `dist/`.

Instead, it starts a live local server so you can work on the app in real time.

### When should you use it?

Use `npm run dev` when:

- you are writing code
- you want to see changes instantly
- you are testing the app during development

## What does `npm run build` mean?

This command creates a production-ready version of your app.

```bash
npm run build
```

### Which file does npm read?

It reads `package.json`, then looks for the `build` script.

In this project, the script is:

```json
"build": "tsc && vite build"
```

This means two things happen in order.

### Step 1: `tsc`

`tsc` is the TypeScript compiler.

It checks your TypeScript code for type errors.

If there is a serious TypeScript problem, the build can fail here before the app is packaged.

### Step 2: `vite build`

After TypeScript checking passes, Vite builds the app for production.

This process:

- bundles your files together
- optimizes the code
- prepares static files that can be uploaded to a hosting platform

### Which folder gets created?

After a successful build, Vite creates:

```text
dist/
```

This is the production output folder.

It contains the final files that are ready to be deployed, such as:

- built HTML
- minified JavaScript
- optimized assets

In this project, the `dist/` folder includes things like:

- `index.html`
- `assets/index-...js`
- `assets/index-...css`
- image files inside `assets/`
- `logo.png` copied to the root of `dist/`

### What is the difference between `src/` and `dist/`?

- `src/` contains the source code you write
- `dist/` contains the final optimized files generated from that source code

So `src/` is your working area, while `dist/` is the finished result.

## Why does `dist/` contain an `assets/` folder?

This is a very common beginner question.

During build, Vite collects files your app needs in the browser and places many of them inside:

```text
dist/assets/
```

This folder often contains:

- JavaScript files
- CSS files
- images imported by your app
- fonts or other static files used by the app

The reason React apps need this is simple: the browser does not run your original project structure directly.

Your app starts as source files such as:

- React components
- TypeScript files
- CSS files
- imported images

Before deployment, those files are processed and converted into files the browser can load efficiently. The `assets/` folder is where many of those processed files are stored.

## Where do the files in `dist/assets/` come from?

They come from files used by your project during the build.

For example, they may come from:

- files imported from `src/`
- CSS used by your app
- images imported into components or other source files

In this project, image files inside `dist/assets/` are generated from source images that are part of the app build. Vite copies them into the output so the browser can load them from the production version.

## Why are the names slightly different in `dist/assets/`?

You may notice names like these:

- `index-8a07d668.js`
- `index-c0eb800f.css`
- `denim-pioneer-6adab778.jpg`

These names are different on purpose.

The extra part, such as `8a07d668`, is usually a **hash**.

### Why does Vite add a hash?

Hashes help with browser caching.

Here is the idea:

- if the file content changes, the hash changes too
- if the hash changes, the browser knows it should fetch the new file
- if the file did not change, the browser can safely reuse the cached version

This improves performance and helps users get the latest version when you deploy updates.

So the file name changes are not random. They are useful for speed and reliability.

## Why is `logo.png` in `dist/` but other files are inside `dist/assets/`?

That usually depends on where the original file came from.

In Vite projects:

- files from `src/` that are imported into the app are often processed and placed in `dist/assets/`
- files from `public/` are usually copied more directly into `dist/`

That is why a public file like `logo.png` can appear as:

```text
dist/logo.png
```

while imported app files can appear inside:

```text
dist/assets/
```

So even though both are static files, they are handled differently depending on how they enter the build.

## A simple way to remember the three commands

You can remember them like this:

- `npm i` = install what the project needs
- `npm run dev` = run the app for coding and testing
- `npm run build` = create the final production version

## In what order should a beginner use these commands?

Usually, the flow is:

```bash
npm i
npm run dev
```

When you are ready to create a production version:

```bash
npm run build
```

That is the most common beginner workflow.

## What about deployment?

Deployment means putting your app online so other people can use it.

Because this project is a Vite app, the thing you deploy is the `dist/` folder created by `npm run build`.

There is no deploy script in this project right now, so a common beginner-friendly deployment command is:

```bash
npx vercel --prod
```

### Why this command?

- `npx` lets you run a package without installing it globally first
- `vercel` is a popular platform for deploying frontend apps
- `--prod` tells Vercel to create a production deployment

### Important note before deploying

Before you deploy, run:

```bash
npm run build
```

That ensures your app is ready and that the `dist/` folder exists.

## Final summary

If you are just starting, here is the easiest mental model:

- `npm i` reads `package.json`, installs packages, and creates `node_modules/`
- if `package-lock.json` exists, npm also uses it to install exact locked versions
- `npm run dev` reads the `dev` script in `package.json` and starts a local development server
- `npm run build` reads the `build` script in `package.json`, checks TypeScript, and creates `dist/`
- `dist/assets/` contains processed build files such as JavaScript, CSS, and imported images, often with hashed names for caching
- `npx vercel --prod` is one common way to deploy the finished app online

Once you understand these four commands, most beginner React and Vite projects start to make much more sense.