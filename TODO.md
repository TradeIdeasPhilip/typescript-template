# TODO

Suggestions for the template's rewrite, from riemann-surfaces-3d (October 2026).
Each one is something a new project needed and had to discover the hard way.

## Publish with GitHub Actions, not `docs/`

Committing the build to `docs/` adds a new copy of every bundle to git history on every build, forever.
canvas-recorder's `.git` grew to 361 MB that way.
Instead, build on GitHub and publish straight to Pages.

- Add `.github/workflows/deploy.yml`.  Copy it from
  [riemann-surfaces-3d](https://github.com/TradeIdeasPhilip/riemann-surfaces-3d/blob/master/.github/workflows/deploy.yml).
  It runs on every push to `master`, plus a "Run workflow" button:  `npm ci`, `npm run build`, then
  `actions/upload-pages-artifact` and `actions/deploy-pages`.
- `vite.config.js`:  drop `outDir: "docs"`, so the build goes to `dist/`.  Keep `base: "./"`, so the site
  works from any subfolder, like `/<repo-name>/`.
- `.gitignore`:  add `dist`.  Delete the committed `docs/` folder.
- `npm run build` should stay `tsc && vite build`, so a type error fails the deploy, and the live site
  keeps the last good version.
- README:  a "Publishing" section.  riemann-surfaces-3d's
  [README](https://github.com/TradeIdeasPhilip/riemann-surfaces-3d#publishing) has one to copy.
  Include the **one-time step**:  in the GitHub repo, **Settings → Pages → Build and deployment →
  Source: GitHub Actions**.  Do that *before* the first push, or the first deploy fails with
  "Get Pages site failed … Not Found".  If that happens, change the setting and use **Re-run all jobs**.

Background:  canvas-recorder's
[development-plans/ci-pages-deploy.md](https://github.com/TradeIdeasPhilip/canvas-recorder/blob/master/development-plans/ci-pages-deploy.md).

## Link back to GitHub from the app

Every project on GitHub Pages should link from its page back to its repository.
On a crowded page, a small `❓` (or `?`) is enough.  "About", "help" or "source code" also work.

```html
<a href="https://github.com/TradeIdeasPhilip/<repo-name>#readme" target="_blank" rel="noopener"
  title="About this project, and its source code, on GitHub">❓</a>
```

Put a placeholder in the template's `index.html`, and a checklist item in the README:
"Point the ❓ link at this repository."

## No `file:` dependencies

`"some-library": "file:../some-library"` works on your computer and silently breaks the deploy.
GitHub's build machine has no `../some-library`.  `npm ci` doesn't complain, because the dependency is
just a broken symlink, so the first error is from `tsc`.  riemann-surfaces-3d's deploy failed this way
from 2026-10-14.

Instead, depend on the published package, and use `npm link ../some-library` to try out a local copy.
A later `npm install` or `npm ci` puts the published copy back.
See canvas-recorder-lib's
[development-plans/new-project-lessons.md](https://github.com/TradeIdeasPhilip/canvas-recorder-lib/blob/master/development-plans/new-project-lessons.md).

## Current versions

- TypeScript 6 and Vite 8.  The template has TypeScript 5.2 and Vite 5.
- TypeScript 6 checks side-effect imports, like `import "./style.css"`, and fails without
  `src/vite-env.d.ts` containing `/// <reference types="vite/client" />`.  Vite's own template has it.
- phil-lib:  update to the current version.

## Projects that use canvas-recorder-lib

Probably a separate section, or a link to the library's README, rather than part of every project.

- Pin the dev server's port (`server.port` and `server.strictPort` in `vite.config.js`).  The video editor
  saves its work in the browser, and the browser keeps a separate copy for each port.  If Vite moved to
  another port, the editor would seem to forget everything.  canvas-recorder-lib's README has the master
  list of ports.
- `server.fs.allow` for a linked library, which lives outside the project.
- More Vite settings from affine-transforms:  `resolve.dedupe: ["phil-lib"]`, and
  `rollupOptions.input` for more than one page.  See new-project-lessons.md.
- riemann-surfaces-3d's `src/full-screen-component.ts` is a copy-as-is example of putting an existing
  program into the video editor.

## Also

- [typescript-getting-started](https://tradeideasphilip.github.io/typescript-getting-started/) describes
  the template and the setup steps.  It needs the same updates.
- Consider a short new-project checklist in the README:  rename the project in `package.json`, set the
  page title and icon, point the ❓ link at the repo, do the one-time Pages setting, push.
