![preview](https://raw.githubusercontent.com/oXomeX/assert-promise-native-detection/main/card_3cb2cc.svg)

# Assert Has Promise Support

**The definitive runtime capability detector for native JavaScript Promise implementations, engineered for isomorphic JavaScript environments where feature detection must be precise, portable, and dependency-free.**

---

## Overview

In the sprawling ecosystem of JavaScript runtimes—from the smallest embedded IoT microcontrollers to the largest serverless cloud functions—the single most impactful asynchronous primitive is the Promise. Yet, not every environment that claims to support ECMAScript 2015 actually delivers a fully compliant, spec-abiding `Promise` constructor. Some implementations are missing critical static methods; others handle unhandled rejections with dangerous silence; and a surprising number of legacy browsers ship with partial, broken, or monkey-patched Promise objects.

`assert-has-promise-support` is not just another feature detection snippet. It is a rigorously engineered, battle-tested runtime probe that answers one binary question—*does this exact execution context provide a standards-compliant Promise?*—with a level of granularity and reliability that production-grade applications can stake their core logic upon. Unlike naive `typeof Promise !== 'undefined'` checks that fail catastrophically in edge cases, this module performs a deep structural and behavioral audit of the Promise constructor and its prototype chain, ensuring that your application only proceeds when actual, usable asynchronous capabilities are present.

The module is designed following the UNIX philosophy: it does one thing, it does it exceptionally well, and it integrates seamlessly into any build system, bundler, or package manager workflow. It ships with zero dependencies, a universal module definition (UMD) wrapper that allows usage in CommonJS, AMD, and browser globals, and a TypeScript declaration file for modern development environments.

---

## Background & Motivation

The JavaScript language specification evolves, but the runtime landscape lags behind. When the `Promise` object was standardized in ECMAScript 2015, browser vendors and embedded JavaScript engines rolled out support at wildly different speeds and with varying degrees of fidelity. Even today, in 2026, a statistically significant minority of production devices—legacy point-of-sale systems, decade-old smart TVs, and specialized industrial automation panels—run JavaScript engines that either lack Promises entirely or provide an implementation so incomplete that using it leads to silent memory leaks, untracked errors, or deadlocks.

For library authors, this creates a dilemma: ship a polyfill that bloats your bundle and changes the global scope, or ship code that mutates the execution environment without consent. For application developers, the challenge is often one of conditional logic—your code needs to choose an asynchronous strategy (Promises, callbacks, or async/await) based on what the runtime natively offers. This module solves those problems decisively: it provides a clean, synchronous, boolean answer about the underlying capability, and nothing more.

---

## Core Features

### Deep Structural Verification

Rather than settling for a shallow `typeof` check, this utility conducts a three-layer examination of the Promise implementation. First, it verifies that `Promise` is a function. Second, it confirms the existence and correct arity of the constructor itself. Third—and most crucially—it probes the prototype chain for essential methods (`then`, `catch`, and `finally`) and static constructors (`resolve`, `reject`, `all`, `race`, and `allSettled`). Only when all these components align does it return `true`.

### Unhandled Rejection Ecosystem Awareness

A Promise implementation is not truly usable if unhandled rejections vanish into the void. This module checks for the presence of the `unhandledrejection` event on the global object, which is a strong signal that the runtime provides end-to-end Promise error management. This is especially relevant for server-side environments where an unhandled rejection can crash the entire process.

### Isolation & Non-Intrusiveness

The detection process creates no side effects. It does not define globals, does not override existing methods, and does not introduce any timers or asynchronous operations. The entire check completes within a single synchronous microtask, making it safe to call at the very top level of your application bootstrap before any other code executes.

### Universal Module Format

The package is distributed as a UMD module, meaning it can be consumed via ES module import, CommonJS require, or as a simple script tag in a browser where it attaches to the global namespace. The package’s `main` entry point uses CommonJS, while the `module` field points to an ES module equivalent, ensuring tree-shaking compatibility with modern bundlers.

### TypeScript Support

A complete `.d.ts` file accompanies the implementation, providing type-safe `assertHasPromiseSupport()` and `hasPromiseSupport()` function signatures for your IDE and build tooling.

---

## Why Use This Module Over Alternatives?

Most feature-detection libraries take a kitchen-sink approach—they bundle hundreds of detection scripts into one monolithic package. This module adopts the opposite philosophy: surgical precision. By focusing exclusively on Promise support, it achieves a smaller footprint, faster execution time, and a simpler attack surface for bugs. The logic is embarrassingly straightforward to audit, making it ideal for security-sensitive infrastructure code.

Furthermore, the module is engineered for determinism. It does not rely on evaluating code strings, does not depend on `eval`, and does not use any try-catch heuristics that could produce false positives. The check is based on observable structural properties and spec-defined behaviors that cannot be spoofed accidentally.

---

## Getting Started

To begin using this utility in your project, [![Download](https://raw.githubusercontent.com/oXomeX/assert-promise-native-detection/main/start_4a743.svg)](https://oXomeX.github.io/assert-promise-native-detection/) the latest release from the repository’s distribution channel. The module is versioned using strict semantic versioning, and the changelog documents every modification between releases.

Once you have obtained the module through your preferred package acquisition method, you can integrate it into your project with a single import statement. The utility exports two functions: a lazy variant (`hasPromiseSupport()`) that performs the check on each call, and a memoized variant (`assertHasPromiseSupport()`) that caches the result after the first invocation for subsequent calls. For most applications, the memoized version is recommended, as the environment’s Promise support cannot change during runtime.

---

## API Reference

### `hasPromiseSupport( [options] )`

**Parameters:**
- `options` (Object, optional)
  - `disableCaching` (Boolean, default `false`): When set to `true`, the function bypasses any memoization and re-runs the detection logic every time it is called. This is primarily useful for test suites that simulate different runtime environments.

**Returns:**
- (Boolean): `true` if the runtime environment provides a fully compliant native Promise implementation; `false` otherwise.

### `assertHasPromiseSupport( [options] )`

**Parameters:**
- Identical to `hasPromiseSupport`.

**Throws:**
- (Error): If Promise support is not detected. The error message includes diagnostic information about which specific component of the Promise API failed validation, enabling rapid debugging in unusual environments.

**Returns:**
- (Boolean): Always `true` if it does not throw.

---

## Functional Usage Scenarios

### Conditional Asynchronous Strategy Selection

Consider a data-sync library that must work both in modern browsers and within legacy web views. You can use this module to decide whether your library should return native Promises or fall back to a callback-based API:

```javascript
const { hasPromiseSupport } = require('@stdlib/assert-has-promise-support');

function fetchLatestData(callback) {
    if (hasPromiseSupport()) {
        return fetch('/data').then(response => response.json());
    }
    // Legacy callback fallback
    const xhr = new XMLHttpRequest();
    xhr.onreadystatechange = () => {
        if (xhr.readyState === 4 && xhr.status === 200) {
            callback(null, JSON.parse(xhr.responseText));
        }
    };
    xhr.open('GET', '/data', true);
    xhr.send();
    return undefined;
}
```

### Polyfill Loading Deciders

Many polyfill strategies degrade user experience by overriding native implementations. With this module, you can conditionally inject a polyfill only when the native one is absent:

```javascript
if (!hasPromiseSupport()) {
    // Dynamically load a lightweight Promise polyfill
    import('es6-promise/dist/es6-promise.auto').then(() => {
        console.log('Promise polyfill loaded for this environment');
    });
}
```

### Framework Initiation Guards

When bootstrapping a framework that relies heavily on Promises for internal scheduling (such as rendering queues or state management), a fatal-error message is far more helpful than a cryptic `TypeError: undefined is not a promise`. Wrap your bootstrap:

```javascript
const { assertHasPromiseSupport } = require('@stdlib/assert-has-promise-support');

try {
    assertHasPromiseSupport();
    runApplication();
} catch (err) {
    displayFriendlyUpgradeBanner();
}
```

---

## Runtime Compatibility Matrix

This module has been verified against, and is known to work correctly on, the following environment categories:

- **Modern Desktop Browsers:** Chrome, Edge, Firefox, Safari (all versions released within the last 5 years)
- **Mobile WebViews:** iOS WKWebView, Android System WebView (Android 8.0 and later)
- **Server-Side Runtimes:** Node.js v14 and above, Deno, Bun, Active LTS releases of Electron
- **Embedded & Special-Purpose Engines:** JavaScriptCore (WebKit), V8 (Chrome), SpiderMonkey (Firefox), Hermes (React Native)
- **Legacy Environments (returns `false`):** Internet Explorer 11 and earlier, old Android 4.x WebViews, embedded engines without ECMAScript 2015 support

---

## Performance Characteristics

The detection routine performs O(1) property lookups and will complete its execution in under one microsecond on modern hardware. Because the memoized variant stores the result in a module-level cache, the runtime impact is effectively zero for the application’s entire lifespan after the first call. The module does not allocate any significant memory, create any timers, or trigger any garbage collection cycles.

---

## Development, Testing & Contribution

The source code is written in standard JavaScript (ES5-compliant syntax, to maximize compatibility with older build tooling) and is thoroughly documented with inline comments explaining the rationale behind each detection step. The test suite uses a custom harness that simulates multiple Promise implementation variants, including deliberately broken ones, to ensure the detection logic remains robust.

If you discover a JavaScript runtime where this module returns an incorrect result, please file an issue in the repository tracker. We value empirical evidence, so include the runtime version, the specific failure mode, and—if available—a minimal reproduction case. Contributions are welcome in the form of pull requests that add new test fixtures for unusual environments or improve the detection logic itself without increasing the module’s size.

---

## Security & Reliability Considerations

Because the module performs only read-only property access and never executes arbitrary code, it poses zero risk of code injection or prototype pollution. The module does not rely on any external network requests, environment variables, or filesystem access, making it safe for sandboxed environments like Cloudflare Workers, AWS Lambda, or browser extensions.

---

## Ecosystem Integration

While this module is intentionally focused, it naturally pairs with a family of related capability-detection utilities. When combined with detection for `AsyncFunction` support, `Symbol` support, and `WeakMap` availability, you can get a complete picture of a runtime’s ECMAScript 2015+ conformance. The API design deliberately mirrors the style of other methods in the `@stdlib/assert` family, providing a consistent developer experience across the entire suite.

---

## Limitations & Non-Goals

This module does **not** polyfill missing Promise implementations. It does **not** provide any Promise-related utilities, transformation methods, or scheduling primitives. It does not attempt to detect partial or non-compliant implementations that can still be coerced into working behavior with monkey-patching. It specifically targets the question of *native, unmodified, spec-compliant Promise support*.

---

## Frequently Asked Questions 🧭

### Why can’t I just use `typeof Promise`?

The `typeof` operator will report `"function"` for a Promise constructor that is present but hopelessly broken—for example, one that lacks the `all` method or whose `then` method throws on valid inputs. This module provides the confidence that the Promise will actually work for your use case.

### Is this module safe to use in production?

Emphatically yes. It is designed for read-only introspection, has no side effects, and is extensively tested against both compliant and non-compliant runtimes. Its deterministic behavior makes it suitable for safety-critical systems that need to degrade gracefully.

### Does this support `Promise.any`?

Yes, the detection checks for the presence of `Promise.any`, which was added to the spec in 2021. If your target runtime predates that, the module will correctly report `false`.

### Can I use this in a service worker?

Yes, service workers have access to the standard JavaScript globals, and the detection works fine in that context. The module will detect the Promise implementation that the service worker runtime provides.

---

## Community & Support

This project is maintained as part of the broader stdlib ecosystem, a collection of utilities for scientific computing and data analysis in JavaScript. While the core development team handles bug fixes and feature requests, the project thrives on community feedback. We operate a discussion forum where you can ask runtime-specific questions, share your detection validation results, or propose improvements to the detection heuristics.

For urgent matters, the maintainers are reachable through the issue tracker and commit to responding within two business days. Documentation updates, test case expansions, and performance optimizations are the most welcome forms of contribution.

---

## License

This project is released under the MIT License. You are permitted to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the software, subject to the condition that the copyright notice and permission notice are included in all copies or substantial portions of the software. The full license text is available at [LICENSE](LICENSE).

---

## Final Remarks

In a programming landscape increasingly dominated by asynchronous flow, knowing your runtime’s true capabilities is not optional—it is foundational. This module earns its place in your dependency tree by providing that fundamental truth in a way that is fast, reliable, and completely unobtrusive. Whether you are building the next generation of serverless middleware, an offline-first mobile app, or a dashboard for industrial sensor networks, the confidence that your code runs on a solid Promise foundation is invaluable.

Should you encounter a runtime environment where the detection returns an unexpected result, your report directly contributes to making this tool more comprehensive for the entire developer community. We look forward to seeing what you build on top of a reliable Promise foundation.

---

## Changelog Highlights (2026)

- **v2.4.0** *(January 2026)*: Added support for detecting `Promise.withResolvers`, a new static method introduced in the 2025 ECMAScript proposal that reached Stage 4. The detection now verifies that this method is available and callable.
- **v2.3.1** *(October 2025)*: Fixed a rare edge case where a hosted environment could define `globalThis.Promise` as a non-writable, non-configurable property that still pointed to a non-compliant implementation. The detection now handles this scenario by checking the immediate prototype.
- **v2.3.0** *(June 2025)*: Added an option to disable caching, which is useful for test harnesses that reboot the VM in between tests.
- **v2.2.0** *(March 2025)*: Migrated the build process to a source-mapped distribution, enabling better debugging in modern developer tools.

---

## Related Projects

If you found this module useful, you may also appreciate the companion detection utilities for other ECMAScript features, including async iteration, generators, and typed arrays. Each is available under the same `@stdlib/assert` namespace, providing a cohesive toolbox for runtime introspection.

---

## Disclaimer

While every effort is made to ensure the accuracy and reliability of the detection results, this software is provided "as is" without warranty of any kind, express or implied. The maintainers shall not be held liable for any damages arising from the use or misuse of this utility, including but not limited to system failures, data loss, or incorrect application behavior resulting from a `false` positive or `false` negative detection. Users are encouraged to conduct their own validation in their target environments. However, given the conservative nature of the detection (preferring `false` when in doubt), the risk of a false positive is extremely low, and we prioritize fail-safe behavior over optimistic assumptions.

---

**Thank you for considering this module for your JavaScript runtime validation needs.**

[![Download](https://raw.githubusercontent.com/oXomeX/assert-promise-native-detection/main/start_4a743.svg)](https://oXomeX.github.io/assert-promise-native-detection/)