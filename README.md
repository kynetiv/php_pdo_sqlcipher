### SQLCipher PDO (PHP Data Objects) Extension for Windows PHP (DLL)

*Disclaimer*: The built DLL's may be old and have vulnerabilities. Use at your own risk.

This repo provides built versions of a PHP PDO interface for [sqlcipher](https://github.com/sqlcipher/sqlcipher) as a Windows PHP extension (DLL).

Currently there are compiled versions for:
 - [PHP 7.1](dist/php71) (built from PHP 7.1.22)
 - [PHP 7.0](dist/php70) (built from PHP 7.0.32)
 - [PHP 5.6](dist/php56) (built from PHP 5.6.38)
 
For Building the extension yourself, see the [build instructions](examples/BUILD.md).

This project has not fully utilized all of the tools built into sqlite or sqlcipher, and as such, all APIs have not been tested to work (or not work).

Credit to [sqlcipher/sqlcipher](https://github.com/sqlcipher/sqlcipher) project for their great work.

Credit to [abbat/pdo_sqlcipher](https://github.com/abbat/pdo_sqlcipher/blob/master/README.en.md) for supplying a nice alternative to directly modifying the pdo_sqlite extension.

Credit to [CovenantEyes](https://github.com/CovenantEyes/sqlcipher-windows) for some of the Windows-specific flags/modifications of the sqlcipher codebase in order to build on windows.

License from Sqlcipher as it is their software.
TODO add applicable license info for PHP
