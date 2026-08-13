# Imagick for PHP 8.6.0 (TS and NTS) - Windows x64

Pre-built `php_imagick.dll` for PHP 8.6.0beta1 on Windows (x64, Thread Safe).

## The Problem
Starting with PHP 8.6, the Zend Engine changed `zend_is_callable()` from an exported API function to a `static zend_always_inline` function. This means it is no longer exported by `php8ts.dll`. 

Older Imagick DLLs (compiled for PHP 8.5 and earlier) import `zend_is_callable` directly, causing a fatal Windows error on startup:
> The procedure entry point `zend_is_callable` could not be located in the dynamic link library `php8ts.dll`.

## The Solution
This repository contains a patched version of the Imagick source code (originally from [Imagick/imagick](https://github.com/Imagick/imagick)). 

The patch replaces the call to the inlined `zend_is_callable` with a call to `zend_is_callable_ex`, which remains an exported `ZEND_API` function in PHP 8.6.

**File patched:** `imagick_class.c` (line 12143)
```c
// Before (PHP 8.5):
if (!user_callback || !zend_is_callable(user_callback, 0, NULL TSRMLS_CC)) {

// After (PHP 8.6):
if (!user_callback || !zend_is_callable_ex(user_callback, NULL, 0, NULL, NULL, NULL)) {
