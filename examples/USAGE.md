
#### General Usage Example
In general you can use much like `pdo_sqlite` extension. Following [abbat/pdo_sqlcipher](https://github.com/abbat/pdo_sqlcipher/blob/master/README.en.md) approach, the extensions in this repo have copied the core pdo_sqlite and update their namespace from `sqlite` to `sqlcipher`.

A typical connection to the pdo_sqlcipher extension may look something like:

```php
$key = 'my-sqlite-key';
$sql = 'SELECT * from mytable';

$dbh = new PDO('sqlcipher:db.sqlite', null, null, null)﻿or die("cannot open the database");;

# decrypt database
$dbh->exec('PRAGMA key = "' . $key . '";');

$ret = $dbh->query($sql);

if($ret) 
{
  foreach($ret as $i) {
    echo $i[0];
  }
} 
else
{
  echo 'Error: ';
  print_r($this_connection->errorInfo());
}

```