# PHP-Simple-Rest-Api

The purpose of this script is to provide a basic REST API that interacts with a MySQL database. It handles four CRUD operations (SELECT, INSERT, UPDATE & DELETE).


The API is flexible, allowing you to specify the table and, in some cases, the record ID in the URL. It takes care of preventing SQL injection, properly handling different HTTP methods, and returning data in JSON format.


Here's a simple explanation of what each part of this PHP script does:


```include('dbcon.php');```

Includes an external file called which contains the database connection.

```$method = $_SERVER['REQUEST_METHOD'];```

This gets the HTTP method of the request (e.g., GET, POST, PUT, DELETE) to know which database operation to perform.

```$request = explode('/', str_replace("/api.php/", "", $_SERVER['REQUEST_URI']));```

Extracts the part of the URL after /api.php/ and splits it into an array. For example, if the URL is /api.php/users/1, it would create an array like ['users', '1'].

```$input = json_decode(file_get_contents('php://input'), true);```

This reads the body of the request (if any) and decodes it from JSON format into a PHP associative array. This is useful for POST or PUT requests that send data in JSON.

```mysqli_set_charset($conexion, 'utf8');```

This sets the character encoding for the database connection to UTF-8, ensuring special characters (like accents or emojis) are handled correctly.

```$table = preg_replace('/[^a-z0-9_]+/i', '', array_shift($request));```

This grabs the first part of the URL (which should be the table name) and removes any characters that aren’t letters, numbers, or underscores, to make sure the table name is safe.

```$key = array_shift($request) + 0;```

This takes the next part of the URL (which should be the ID or primary key of a record) and converts it into an integer. If no key is given, it defaults to 0.

```$columns = preg_replace('/[^a-z0-9_]+/i', '', array_keys($input));```

This gets the keys from the $input array (which are the column names) and makes sure they only contain valid characters (letters, numbers, and underscores).

```$values = array_map(function ($value) use ($conexion) { ... }, array_values($input));```

This cleans the values in the $input array. If a value is null, it leaves it as null; otherwise, it escapes the value to prevent any special characters from messing up the SQL query.

```
$set = '';
for ($i=0;$i<count($columns);$i++) {
  $set.=($i>0?',':'').'`'.$columns[$i].'`=';
  $set.=($values[$i]===null?'NULL':'"'.$values[$i].'"');
}
```

This creates a string like column1='value1', column2='value2' for the SET part of an SQL query, which will be used in UPDATE operations.

```
switch ($method) {
  case 'GET':
    $sql = "select * from `$table`".($key?" WHERE ID=$key":''); break;
  case 'PUT':
    $sql = "update `$table` set $set where id=$key"; break;
  case 'POST':
    $sql = "insert into `$table` set $set"; break;
  case 'DELETE':
    $sql = "delete `$table` where id=$key"; break;
}
```

This switch statement checks the HTTP method and generates the appropriate SQL query.

```$result = mysqli_query($conexion, $sql);```

This runs the SQL query on the database using the connection $conexion. The result of the query (if any) is stored in $result variable.

```
if(!$result) {
    http_response_code(404);
    die(mysqli_error());
}
```

If the SQL query fails, it sends an HTTP 404 response (Not Found) and shows the error message from MySQL.

```
if ($method == 'GET') {
  header('Content-Type: application/json');
  if (!$key) echo '[';
  for ($i=0;$i<mysqli_num_rows($result);$i++) {
    echo ($i>0?',':'').json_encode(mysqli_fetch_object($result));
  }
  if (!$key) echo ']';
} elseif ($method == 'POST') {
  echo mysqli_insert_id($conexion);
} else {
  echo mysqli_affected_rows($conexion);
}
```

If the method is GET (for getting data), it sets the response header to indicate that the response will be in JSON format and iterates through all the rows from the query result converting each row into JSON format.

If no key is provided in the URL, it wraps the results in square brackets [] to return them as an array of JSON objects.

If the method is POST (for inserting data), it returns the ID of the newly inserted record using mysqli_insert_id.

For other methods (like PUT or DELETE), it returns the number of rows affected by the query (e.g., how many rows were updated or deleted).

```mysqli_close($conexion);```

This closes the database connection after the query is finished.