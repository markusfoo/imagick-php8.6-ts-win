# PHP 8.6 Windows Builds & Imagick Extension

Automated Windows builds for PHP 8.6 development snapshots and a patched Imagick extension compatible with PHP 8.6.

## What This Repository Provides

### 1. PHP 8.6 Dev Builds (Windows)

Pre-built **complete PHP 8.6 distributions** for Windows from the latest `php/php-src` master branch.

| Variant | Architecture | Optimization |
|---------|-------------|--------------|
| x64 | 64-bit | Standard |
| x86 | 32-bit | Standard |
| x64-AVX2 | 64-bit | AVX2 (Intel Haswell 2013+, AMD Ryzen 2017+) |
| x86-AVX2 | 32-bit | AVX2 |

Each zip is a ready-to-use PHP distribution:
- `php.exe`, `php-cgi.exe` (CLI + FastCGI)
- `php8ts.dll` (Thread Safe engine)
- `php8apache2_4.dll` (Apache 2.4 module)
- `ext/*.dll` — 40+ extensions (mysqli, opcache, curl, gd, intl, mbstring, soap, xsl, sockets, etc.)
- Dependency DLLs (libcrypto, libssl, icu, brotli)
- `php.ini-development`, `php.ini-production`

**Compiler:** Visual Studio 2024 (VC18) on `windows-2025` runner.
**Build type:** Thread Safe (ZTS) — for Apache mod_php and XAMPP.

**XAMPP Installation:**
1. Download `x64-AVX2.zip` for best performance on modern CPUs
2. Stop Apache
3. Replace `php8ts.dll`, `php8apache2_4.dll`, `php.exe`, `ext/` in `C:\xampp\php\`
4. Start Apache
5. Verify: `C:\xampp\php\php.exe -v`

Trigger via **Actions** → **Build PHP 8.6.0-dev TS Windows VS18** → **Run workflow**.

---

### 2. Imagick Extension for PHP 8.6 (Windows)

Pre-built `php_imagick.dll` for PHP 8.6 on Windows (x64, Thread Safe).

#### The Problem

Starting with PHP 8.6, the Zend Engine changed `zend_is_callable()` from an exported API function to a `static zend_always_inline` function. It is no longer exported by `php8ts.dll`.

Older Imagick DLLs (compiled for PHP 8.5 and earlier) import `zend_is_callable` directly, causing a fatal error on startup:

> The procedure entry point `zend_is_callable` could not be located in the dynamic link library `php8ts.dll`.

#### The Fix

A patched version of [Imagick/imagick](https://github.com/Imagick/imagick) that replaces the call to the inlined `zend_is_callable` with `zend_is_callable_ex`, which remains an exported `ZEND_API` function in PHP 8.6.

```c
// Before (PHP 8.5):
if (!user_callback || !zend_is_callable(user_callback, 0, NULL TSRMLS_CC)) {

// After (PHP 8.6):
if (!user_callback || !zend_is_callable_ex(user_callback, NULL, 0, NULL, NULL, NULL)) {
```

Trigger via **Actions** → **Build Imagick PHP 8.6.0beta1 VS18 x64** → **Run workflow**.

---

## PHP 8.6 Features

- Partial Function Application (PFA) — `?` placeholder
- `clamp()` built-in function
- Readonly Property Defaults — `public readonly array $x = [];`
- `SortDirection` enum
- And all other [PHP 8.6 RFCs](https://wiki.php.net/rfc#php_86)

## Workflows

| Workflow | What it builds | Output |
|----------|---------------|--------|
| `build-php86-dev-latest` | Full PHP 8.6 distro from master | GitHub Release (zip) |
| `build-imagick` | php_imagick.dll (patched) | GitHub Release (zip) |

## Source

- PHP source: [php/php-src](https://github.com/php/php-src) (master branch)
- PHP SDK: [php/php-sdk-binary-tools](https://github.com/php/php-sdk-binary-tools)
- Imagick source: [Imagick/imagick](https://github.com/Imagick/imagick) (patched)
- Ubuntu .deb builds: [markusfoo/php86-apt-builder](https://github.com/markusfoo/php86-apt-builder)
