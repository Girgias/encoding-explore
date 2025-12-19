# Encoding explore

This is a W.I.P. project to analyze existing ecodings to determine differences and statistics about old encodings and OS locales.

## Motivation

The initial motivation is to see where we can forgo using the C `mblen()` function in PHP and reduce dependence on locales in general.

## Resources

- Linux locales are *always* ASCII compatible: https://superuser.com/questions/1753662/how-can-i-set-my-systems-default-encoding-to-utf-16
- IBM AIX "ASCII characters in the unique code-point range": https://www.ibm.com/docs/en/aix/7.3.0?topic=characters-ascii-in-unique-code-point-range
- IBM AIX "Supported languages and locales": https://www.ibm.com/docs/en/aix/7.3.0?topic=globalization-supported-languages-locales
- IBM "PASE for i Locales": https://www.ibm.com/docs/en/i/7.6.0?topic=ssw_ibm_i_76/apis/pase_locales.html
- Windows code page identifier list: https://learn.microsoft.com/en-us/windows/win32/intl/code-page-identifiers
- Windows code page data: https://www.microsoft.com/en-us/download/details.aspx?id=10921
- Windows C/C++ reference for `setlocale()` CRT function : https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/setlocale-wsetlocale?view=msvc-170
- Windows C/C++ reference for `setlocale()` pragma: https://learn.microsoft.com/en-us/cpp/preprocessor/setlocale?view=msvc-170
- History of encodings and Unicode: https://www.gammon.com.au/unicode/
- IANA Character sets: https://www.iana.org/assignments/character-sets/character-sets.xhtml
- Default system values for national language versions for IBM i: https://www.ibm.com/docs/en/i/7.6.0?topic=information-default-system-values-national-language-versions
- Apple/MacOS language and Locale IDs: https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPInternational/LanguageandLocaleIDs/LanguageandLocaleIDs.html#//apple_ref/doc/uid/10000171i-CH15-SW1
- Wikipidia index of all Mac OS character encodings: https://en.wikipedia.org/wiki/Category:Classic_Mac_OS_character_encodings

## ToDo:

- [ ]: Extract Windows codepage data to determine differences between locales and which are *not* ASCII compatible
- [ ]: Determine if IBM provides code page data in the similar way as Microsoft and Windows
- [ ]: Determine if Apple provides code page data in the similar way as Microsoft and Windows


