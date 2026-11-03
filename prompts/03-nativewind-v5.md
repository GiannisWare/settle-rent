# 03 · NativeWind v5 setup

Status: approved

## Goal
Enable typed NativeWind `className` styling on native and web in the existing Expo Router app, with Tailwind CSS v4 processing and verified bundling.

## Decisions
- The user explicitly requested v5. The supplied attachment is a v4 documentation index; follow the official v5 installation guide instead: https://www.nativewind.dev/v5/getting-started/installation.
- NativeWind v5 is a prerelease. Use the documented exact pair `nativewind@5.0.0-rc.0` and `react-native-css@3.1.0-rc.0`; do not use floating preview tags.
- Keep Expo SDK 58 (~58.0.16), React Native 0.88.0 and React 19.3.0. The NativeWind guide's tested target is SDK 57, so verify SDK 58 rather than assuming compatibility. If incompatible, stop and explain before changing SDK or choosing another NativeWind release.
- Install with `npx expo install` only. Proposed runtime dependencies: the exact RC pair. Proposed development dependencies: `tailwindcss@4.1.12`, `@tailwindcss/postcss@4.1.12`, `postcss` (resolve and record the installed version), `lightningcss@1.30.1`. Add an npm override for `lightningcss` at 1.30.1. Plan approval covers these additions.
- Reanimated, Worklets, safe-area-context and expo-system-ui are already present; preserve SDK-compatible versions. `userInterfaceStyle` is already automatic.
- Reuse `src/global.css`; retain existing font variables and existing `src/constants/theme.ts` tokens. Move the stylesheet side-effect import from the theme module to the root layout so it loads once.
- Setup only: no screen redesign, theme migration, new brand tokens, formatter plugin, or v4 Babel/Tailwind configuration.
- Approved follow-up: add Expo's recommended `eslint@^9.0.0` and `eslint-config-expo@~58.0.4` plus its generated flat configuration to enable required linting. Resolve pre-existing CSS declarations, web-tab ref typing, and hydration-effect lint failures with minimal changes so required checks can pass.
- Read SDK 58 reference https://docs.expo.dev/versions/v58.0.0/ and Expo Metro guidance https://docs.expo.dev/guides/customizing-metro/ before implementation. Consult `.agents/skills/expo-design-system/SKILL.md` for extending the existing styling system.

## Out of scope
Auth, content, backend, app branding, navigation changes, SDK upgrades, and converting existing components to utility classes throughout the app.

## Steps
### 1. Install the documented toolchain
Inspect the lockfile and current dependency tree. Record baseline lint/type-check behavior. Install the proposed packages through Expo CLI using exact versions where specified and add the Lightning CSS override in `package.json`. Update `package-lock.json`. Inspect peer dependency results; do not bypass incompatibilities with force or legacy peer dependency flags. If lint requires installing ESLint tooling that is currently absent, request approval for the concrete extra packages before adding them.

### 2. Connect Metro, PostCSS, global CSS and TypeScript
Create `metro.config.js` using `getDefaultConfig` from `expo/metro-config`, wrapped by `withNativewind` from `nativewind/metro`. Create `postcss.config.mjs` enabling `@tailwindcss/postcss`. Prepend the documented Tailwind theme, preflight and utilities imports plus `nativewind/theme` to `src/global.css`, preserving its font variables and keeping utilities unlayered. Import that file once in `src/app/_layout.tsx`; remove its existing import from `src/constants/theme.ts`. Add `nativewind-env.d.ts` referencing `react-native-css/types` and ensure TypeScript includes it. Add a CSS module declaration only if necessary, without breaking the existing animated icon CSS-module types. No custom Babel configuration is needed by this guide.

### 3. Verify utility compilation and runtime behavior
Use a temporary minimal utility-styled React Native View/Text probe in the home screen to exercise layout and text styling. Confirm `className` type-checking and that utilities compile for native and web. Open the iOS simulator, capture a screenshot, and inspect the probe; check web styling too. Remove the temporary probe and restore the home screen after verification, then rerun affected checks. Existing screens retain their current styling. Report any unavailable runtime verification honestly.

### 4. Record lasting setup decisions
Update AGENTS.md with the pinned NativeWind/engine pair, Tailwind v4 setup and global CSS entry point, preserving unrelated edits and the file's 150-line limit. Add `## Result` here and set `Status: done` only after required checks pass. Follow the repository's Done / Verified / You need to / Test it yourself / Notes reporting format.

## Verification
- `npx expo lint` and `npx tsc --noEmit`: record baseline failures separately; both must pass before calling setup done.
- `npx expo install --check`: check SDK dependency alignment.
- `npm ls nativewind react-native-css tailwindcss @tailwindcss/postcss postcss lightningcss`: verify versions and no invalid peers.
- `npx expo export --platform ios --output-dir /tmp/settle-nativewind-ios` and equivalent web export to `/tmp/settle-nativewind-web`: exercise Metro and PostCSS for both targets.
- `npx expo start --clear`, boot/open an iOS simulator and capture a screenshot of a temporary className probe. Verify the rendered style and subsequent Fast Refresh; check equivalent web output in a browser when available.
- Remove the probe, rerun lint/type-check and affected bundle checks, and confirm the existing home and Explore screens still load.
- Final end-to-end outcome: the app starts with the new CSS pipeline and typed utilities work, while its existing screens keep their appearance.

## AGENTS.md updates
In Expo, record NativeWind v5 RC0 with its exact engine, Tailwind v4, root-layout CSS import and configuration entry points. Existing theme tokens remain authoritative; future utility-based theme extensions belong in the CSS theme and must avoid duplicate competing values.
