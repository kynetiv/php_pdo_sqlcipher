
#### General Usage Example for pdo_sqlcipher
In general you can use this extension much like `pdo_sqlite` extension.  Only difference is that we've modified the namespace from `sqlite` to `sqlcipher`.

So a typical connection to the `pdo_sqlcipher` extension may look something like:

```php
$key = 'my-sqlite-key';
$sql = 'SELECT * FROM mytable';

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
  print_r($dbh->errorInfo());
}

```