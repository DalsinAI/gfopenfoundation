# GFOpenFoundation

**Native GNUstep Base (Foundation) port for GFOS / AmigaOS 3.x.** Reuse the existing OpenAmigaGCC 16 Objective-C compiler/runtime and our repaired libnix instead of rewriting Objective-C collection, string, and filesystem classes. Primary first consumer: [AmigaOolite](https://github.com/DalsinAI/amigaoolite).

## Status

**18 November 2026: repository established; upstream GNUstep Base has NOT yet been cross-built or guest-tested.** This project is a real platform dependency lane, not a declaration of an implemented `foundation.library`. Start with `libgnustep-base.a` and the normal Foundation headers; qualify a shared Amiga library ABI only after basic class/runtime compatibility passes.

## Confirmed existing dependencies on daletop

| Layer | Existing source | Evidence/status |
| --- | --- | --- |
| C runtime and POSIX-like calls | [GFOpenGCC / libnix](https://github.com/DalsinAI/gfopengcc/tree/main/patches/libnix) | GCC16 stove contains `libnix*.a`; 11 documented fixes to upstream bebbo libnix `5707d0b` |
| Objective-C compilation/dispatch | GFOpenGCC 16.2 | Cross GCC configured `--enable-languages=c,c++,objc`; `libobjc.a` present and Objective-C class compiled to native m68k object |
| Threads | GFOpenGCC pthread library, OpenMulticore | Existing `libpthread.a` plus AROS-stuff pthread patches; correctness/capability audit required |
| Unicode | GFOpenBrowser Unicode/ICU library | Existing shared Unicode services; check ABI and GNUstep ICU requirements |
| JavaScript | [GFOpenBrowser `javascript.library`](https://github.com/DalsinAI/gfopenbrowser/tree/main/src/jslib) | JavaScriptCore + native JIT and host-class API, for Oolite's SpiderMonkey adapter (not a Foundation dependency) |
| Oolite | [AmigaOolite](https://github.com/DalsinAI/amigaoolite) | Native upstream galaxy RNG and planet-mesh object compile; main engine awaits Foundation |

The **libnix work is real**, not hypothetical. Eleven tracked upstream patch files address stack swaps, startup exit codes, `UtilityBase` math, FPCR, C++ destructors, thread-safe `setlocale`, base-relative C++ initialisation, `strtol`/`strtoul` end pointers, duplicate `libnix4.library` exports, POSIX socket `ioctl`/`fcntl`, and the socket-library user/group bridge without `usergroup.library`. See [patch inventory](https://github.com/DalsinAI/gfopengcc/blob/main/patches/libnix/README.md). These changes improve the GFOS/AmigaOS C runtime but **do not amount to a GNUstep Base port**.

## Engineering plan

1. Pin a GNUstep Base source release and audit licensing, configure requirements and file-level dependencies. Keep patches against upstream; avoid permanently vendoring and forking unrelated code.
2. Compile a minimal Objective-C Foundation `NSString`/`NSArray`/`NSDictionary` smoke test under the existing GCC16 `libobjc` ABI; prove reference counting and exception behaviour.
3. Map GNUstep's POSIX function usage to libnix and record feature detection, missing calls, errno, path/case rules and filesystem semantics. Do not shim functions that already work on native GFOS.
4. Map Objective-C dispatch, threads/locks/condition variables, TLS, locale, time and Unicode to libobjc, pthreads/OpenMulticore and existing GFOS services; test in guest.
5. Build GNUstep Base as a static target archive plus Foundation headers. Test collection/object lifetimes, property lists, error handling, filesystem, resource bundles and memory under 68040/68060.
6. Integrate with Oolite's real source (not just a standalone demo), then evaluate `foundation.library` ABI/resource ownership. Add CI receipts, guest UAT and an installed SDK.

## Ground rules

Use native libnix and the existing OpenUp series. Do not bundle independent POSIX emulation, duplicate JavaScriptCore, or create a new Objective-C runtime. Keep objective-C ABI and GNUstep Base's upstream licensing intact. Record native compilations separately from guest proof. No completed library or RC is claimed until Foundation class tests and Oolite integration pass.
