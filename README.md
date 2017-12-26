#### Sqlcipher Extension for Windows PHP (DLL)

*Disclaimer*: The built DLL's may be old and have vulnerabilities. Use at your own risk.

This repo provides a built version of [sqlcipher](https://github.com/sqlcipher/sqlcipher) as a Windows PHP module (DLL).

Currently there are compiled versions for:
 - [PHP 7.1](dist/php71) (built from PHP 7.1.11)
 - [PHP 5.6](dist/php56) (built from PHP 5.6.15)

##### Usage
In general you can use much like `pdo_sqlite` extension. Following [abbat/pdo_sqlcipher](https://github.com/abbat/pdo_sqlcipher/blob/master/README.en.md) approach, the extensions in this repo have copied the core pdo_sqlite and update their namespace from `sqlite` to `sqlcipher`.

A typical connection to the pdo_sqlcipher extension may look something like:

```php
$this->_connection = new PDO('sqlcipher:my-sqlite.db');

# decrypt database
$this->_connection->exec("PRAGMA key = 'my-sqlite-key';");
```

This project has not fully utilized all of the tools built into sqlite or sqlcipher, and as such, all APIs have not been tested to work (or not work).

TODO add steps to build the module and dependencies on Windows

Credit to [sqlcipher/sqlcipher](https://github.com/sqlcipher/sqlcipher) project for their great work.

Credit to [abbat/pdo_sqlcipher](https://github.com/abbat/pdo_sqlcipher/blob/master/README.en.md) for supplying a nice alternative to directly modifying the pdo_sqlite extension.

Credit to [CovenantEyes](https://github.com/CovenantEyes/sqlcipher-windows) for some of the Windows-specific flags/modifications of the sqlcipher codebase in order to build on windows.

License from Sqlcipher as it is their software.
TODO add applicable license info for PHP

