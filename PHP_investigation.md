# Analysis regarding usage of locales and encodings in PHP and impact of php-src

> [!NOTE]
> This is still a Work In Progress

Limited aspects of PHP depend on the locale or character encoding (or _code page_)
of the filesystem of the underlying OS.

In general, we believe that C locales are a terrible way to handle internationalization (i18n).
An infamous [MPV maintainer rant](github.com/mpv-player/mpv/commit/1e70e82baa91)
explains part of what makes C locales ill-suited to this task.
This extends to PHP code which should not need to alter the locale or think about its existence,
as it impacts portability of PHP code and affects system libraries.[^1]
As put by an RFC author "Locale-sensitivity is almost always a bug."[^4]

[^4]: https://externals.io/message/85638

The objective of this analysis is to determine which aspects of PHP are dependent on the locale,
and what can be done to reduce or eliminate this dependence.

## Aspects of PHP no longer depending on locale or character encoding

- PHP 8.0 removed support for EBCDIC [^2]
  - Note: this seems to not fully be correct on Windows as one can use `sapi_windows_cp_set()` to set an EBCDIC code page
- PHP 8.0 made float to string casts locale-independent [^3]
- PHP 8.2 made various case conversions locale-independent [^4]
  - This affected the following functions:
    - `strtolower()`
    - `strtoupper()`
    - `stristr()`
    - `stripos()`
    - `strripos()`
    - `lcfirst()`
    - `ucfirst()`
    - `ucwords()`
    - `str_ireplace()`
    - `array_change_key_case()`
    - Sort functions with the `SORT_FLAG_CASE` mode
      - TODO: confirm `k{r}sort()` was properly upgraded as there is usage of `zend_binary_strcasecmp_l()`

[^2]: https://github.com/php/php-src/pull/5390
[^3]: https://wiki.php.net/rfc/locale_independent_float_to_string
[^4]: https://wiki.php.net/rfc/strtolower-ascii

## Aspects of PHP capable of modifying the locale or character encoding 

PHP INI setting defining character encoding:

- Core:
  - `default_charset`
  - `input_encoding`
  - `output_encoding`
  - `internal_encoding`
  - `zend.multibyte`
  - `zend.script_encoding`
  - `zend.detect_unicode`
- MBString
  - `mbstring.language` (? Implied by documentation it sets internal encoding)
  - `mbstring.encoding_translation`
  - `mbstring.internal_encoding`
  - `mbstring.http_input`
  - `mbstring.http_output`
  - `mbstring.http_output_conv_mimetypes` (? No documentation)
- Iconv
  - `iconv.input_encoding`
  - `iconv.output_encoding`
  - `iconv.internal_encoding`

PHP functions capable of changing the locale or character encoding:

- `setlocale()`
- `sapi_windows_cp_set()`

## Aspects of PHP affected by the locale or process character encoding

### Userland functions

- `printf()` family of functions
- `strcoll()` and `SORT_LOCALE_STRING` flag for `sort()` functions
- `strnatcmp()`, `strnatcasecmp()`, `substr_compare()`, `SORT_NATURAL` and  
  - Reason: relies on `php_strnatcmp`, `zend_binary_strcasecmp_l`, `zend_binary_strncasecmp_l`
- `basename()`
  - Reason: relies on `php_basename` PHPAPI which uses `php_mblen()`
  - TODO: Other userland functions affected
- `fgetcsv()`, `str_getcsv()`, `SplFileObject->fgetcsv()`, and `SplFileObject` iterator methods when `SplFileObject:: READ_CSV` mode is used
  - Reason: Relies on `php_fgetcsv` PHPAPI which uses `php_mblen()` 
- `escapeshellcmd()`, `mail()`, and `mb_send_mail()`
  - Reason: Relies on `php_escape_shell_cmd` PHPAPI which uses `php_mblen()`
- `escapeshellarg()`
    - Reason: Relies on `php_escape_shell_arg` PHPAPI which uses `php_mblen()`
      - TODO: Make this API private
- ext/ctype functions
  - Reason: Relies on C lib `is{*}` functions

### php-src

- Any C function affected by locale
  - `toupper()`
  - `tolower()`
  - `isspace()`
  - `isdigit()`
  - etc. see https://en.cppreference.com/w/c/locale/setlocale
- Usage of C multibyte functions such as `mblen()` (wrapped as `php_mblen()`)
  - `php_basename()`
  - `php_fgetcsv()`
  - `php_escape_shell_cmd()`
  - `php_escape_shell_arg()`
- `strnatcmp_ex()`, `zend_binary_strcasecmp_l()`, `zend_binary_strncasecmp_l()` APIs
