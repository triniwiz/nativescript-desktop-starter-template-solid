# NativeScript Desktop Starter (Solid)

A SolidJS starter that runs on Windows, iOS and Android. Layout uses
[MasonKit](https://github.com/triniwiz/nativescript-mason) (Flexbox, CSS Grid
and HTML-style elements), and styling uses Tailwind CSS v4. Solid renders
through [`@nativescript-community/solid-js`](https://github.com/nativescript-community/solid-js)
into a [DOMiNATIVE](https://github.com/SudoMaker/DOMiNATIVE) DOM, so you write
ordinary Solid JSX: `<div class="flex gap-4">`, `<button on:tap={…}>`.

This is the Solid version of
[nativescript-desktop-starter-template](https://github.com/triniwiz/nativescript-desktop-starter-template)
(NativeScript-Vue) and
[nativescript-desktop-starter-template-react](https://github.com/triniwiz/nativescript-desktop-starter-template-react).
The screens are the same.

## Prerequisites

> NativeScript for Windows is in alpha. Expect rough edges.

To run on Windows you need the following. `npx ns doctor windows` checks them.

| Requirement | Notes |
| --- | --- |
| Windows 10 1809 (build 17763) or later | The app targets `net10.0-windows10.0.26100.0`, with a minimum of `10.0.17763.0`. |
| [Node.js](https://nodejs.org) LTS | Provides `npm` and `npx`. |
| [.NET 10 SDK](https://dotnet.microsoft.com/download) | The build runs `dotnet build`. Check it with `dotnet --version`. |
| Developer Mode | Settings → Privacy & security → For developers. Windows needs it to install and launch the unsigned debug build. |

For iOS and Android, follow the [NativeScript environment setup](https://docs.nativescript.org/setup/).

You don't need to install the NativeScript CLI globally. It's a dev dependency, and the npm scripts run it.

## Getting started

```bash
git clone https://github.com/triniwiz/nativescript-desktop-starter-template-solid my-app
cd my-app
npm install
npm run windows   # or: npm run ios / npm run android
```

`npm run windows` builds the app, installs and launches it, then updates it
with HMR as you save. `npm run clean` removes the build output, and
`npm run typecheck` runs `tsc`. To rebuild and restart on every save instead,
run `npx ns run windows --no-hmr`.

Use npm. `package.json` relies on npm `overrides`.

## Project layout

| Path | What it's for |
| --- | --- |
| `src/app.tsx` | App entry. Sets up the DOM, then starts Solid with the home screen. |
| `src/dom.ts` | Loads Solid's DOM, turns on `View.preflight` and registers MasonKit's elements. |
| `src/navigation.tsx` | `navigate(Screen)` and `goBack()`. Each screen is its own Solid root on a NativeScript `Page`. |
| `src/app.css` | Tailwind, plus the `dark:`, `ios:`, `android:`, `windows:`, `phone:` and `tablet:` variants. |
| `src/components/Home.tsx` | The home screen. |
| `src/components/Playground.tsx` | Flexbox and Grid examples. |
| `types/jsx.d.ts` | JSX typings for what NativeScript adds to Solid's DOM ones: `on:tap`, `on:loaded` and other NativeScript events, and `<scroll>`. |
| `postcss-masonkit.mjs` | Keeps the Tailwind layout utilities (`flex`, `grid`, `gap-*`, `max-w-*`, `absolute`, ...) that NativeScript's Tailwind plugin would otherwise remove. |
| `solid-workarounds.mjs`, `src/solid-dom.ts` | Keep the Solid renderer from replacing the runtime's globals (see [Temporary workarounds](#temporary-workarounds)). |
| `android-workarounds.mjs` | Keeps Android rebuilds from loading a stale activity chunk (see [Temporary workarounds](#temporary-workarounds)). |

## Tips

- **JSX:** write HTML-style tags in lowercase: `<div>`, `<section>`, `<h1>`,
  `<p>`, `<span>`, `<button>`, `<code>`, `<scroll>`. Use `class`.
- **Events:** use Solid's `on:` namespace with the NativeScript event name:
  `on:tap={…}`, `on:loaded={…}`. `onClick` and other `on<Event>` props don't
  work. Event names are case sensitive, events don't bubble, and there is no
  delegation. `types/jsx.d.ts` lists the events JSX knows about; add others
  to `CustomEvents` there.
- **Text:** `{a} · {b}` makes three text nodes. Pass one string instead,
  `` {`${a} · ${b}`} ``, as the footer on the home screen does.
- **Navigation:** `navigate(Screen)` pushes a page and `goBack()` pops it.
  A screen's root is disposed when you navigate back from it, so `onCleanup`
  runs then. Inside a screen, `usePage()` returns its `Page`.
- **Class names** must appear in full in your source so Tailwind can find them.
  Pick between complete strings (`'flex-row'`, `'flex-col'`) rather than
  building them (`` `flex-${dir}` ``), as the playground does.
- **Responsive layouts:** there are no `sm:` or `md:` breakpoints. Use layouts
  that fit the space they're given instead. They also reflow when the window
  is resized:

  ```tsx
  <div class="grid grid-cols-[repeat(auto-fill,minmax(240,1fr))] gap-4">…</div>
  <div class="flex flex-row flex-wrap gap-4"><div class="flex-1 basis-40">…</div></div>
  <main class="mx-auto w-full max-w-5xl px-5">…</main>
  ```

- **Units:** unitless values are dp, and `px` means physical pixels, so write
  arbitrary values without a unit, for example `auto-rows-[96]`.
- **Avoid** `tracking-*`, `leading-*` and `bg-linear-*`. For gradients, write
  a `@utility` instead, like `bg-hero` in `src/app.css`.
- **Keep** `@import 'tailwindcss/theme'` and `@import 'tailwindcss/utilities'`
  in `src/app.css`, not `@import 'tailwindcss'`. Tailwind's preflight would
  hide every view.

## Known issues

- HMR remounts the whole app: `src/app.tsx` boots through `startSolidApp`,
  which re-renders the home screen after a `.tsx` save, so signals start
  over. A screen opened with `navigate()` keeps showing the old code until
  you go back and open it again. CSS changes apply in place.
- With HMR, `@nativescript/vite` serves packages from its own dev bundle,
  which doesn't go through `solid-workarounds.mjs`, and on Android DOMiNATIVE
  and Solid load twice ("You appear to have multiple instances of Solid").
  The app and the renderer still share one Solid, so this is only noise.

## Troubleshooting

- **Something's missing from your setup:** `npx ns doctor windows` lists what it can't find and how to fix it.
- **The app builds but won't launch** ("deployment failed"): turn on Developer Mode.
- **A stale or broken build:** run `npm run clean`, then `npm run windows` again.

## Temporary workarounds

These can be removed once the fixes are released:

- `@nativescript/core` and `@nativescript/vite` are installed through pkg.pr.new from
  [NativeScript/NativeScript#11468](https://github.com/NativeScript/NativeScript/pull/11468),
  which is the `feat/windows` branch plus percentage sizes inside MasonKit layouts.
- `@nativescript/windows` is pinned to an exact alpha, because a caret range
  would also match older, incompatible betas.
- `solid-workarounds.mjs` swaps the Solid renderer's `src/dom.js` import for
  `src/solid-dom.ts`. The original copies every DOMiNATIVE class onto
  `globalThis` (`document`, `Event`, `EventTarget`, `HTMLElement`, ...),
  replacing the iOS runtime's `Event` and `EventTarget`. Because the renderer
  creates elements with the global `document`, the plugin also imports
  DOMiNATIVE's `document` into it.
- `android-workarounds.mjs` names core's Android activity chunk without a
  hash. The generated Java activity loads it by the name it had at the last
  native build, so after a rebuild that renamed it, Android started the old
  chunk and crashed ("isEmbedded is not a function").
- `postcss-masonkit.mjs` also keeps `touch-action` and `:hover` rules, which
  `@nativescript/tailwind` strips even though MasonKit supports them.
- `masonkit-hmr.mjs` stops MasonKit from loading twice during HMR.
- `postcss-selector-commas.mjs` makes classes with a comma, like
  `grid-cols-[repeat(auto-fill,minmax(240,1fr))]`, match
  ([NativeScript/NativeScript#11463](https://github.com/NativeScript/NativeScript/pull/11463)).
