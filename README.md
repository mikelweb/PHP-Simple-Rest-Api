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