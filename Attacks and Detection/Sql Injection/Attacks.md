# Injection attacks

Occurs when attacker is able to send command through a web server to a backend system

Most common – sql injection

## SQL injection

- A normal web application -> gets user input -> convert them into database query -> send to the server. The response from the server -> sent to the users
- Sql injection -> attacker send unauthorized query as input to the backend server
- SQL Injections are critical attack vectors

## To prevent this:

- Input validation
  - Blocks the use of ‘ or –
  - It allows only valid email id or username or text

- Least privilege
  - Restricts the tables which the webserver can access
  - Eg: a shopping app’s searchbar can access only the product table and not the customer db table

## Why it matters:

- SQLi is in the OWASP Top 10 every year
- Leads to data exfiltration, account takeover, even OS-level code execution
- SQLi is often visible in HTTP logs, WAF logs, SIEM alerts, database errors, and unusual response behavior.

---

# How SQL injection works

A web app takes your input and builds a SQL query with it.

The problem is when it just converts your input directly into the query string without checking it first.

If the user enters normal input:

```text
username: aparnaa
password: test123
```

The query becomes:

```sql
SELECT * FROM users WHERE username = 'aparnaa' AND password = 'test123';
```

But If the attacker gives the input as:

```text
username: admin' OR '1'='1
password: anything
```

the query becomes:

```sql
SELECT * FROM users WHERE username = 'admin' OR '1'='1' AND password = 'anything';
```

This creates a syntax error because the quote breaks the query.

That is why attackers often start by testing with:

```text
'
"
`
)
'))
```

---

# Types of SQL injection

## Union Based SQL injection:

- Uses 2 or more select statement that I already available in the application to create a single result in the application response
- Useful in – getting info from parts of the db that is not supposed to be interacted by the user
- To successfully use a union attack both select query must
  - Have same datatype
  - Same number of columns
- Another way – sql map automated
  - Using null in union until the condition matches and the error stops

```sql
' UNION SELECT NULL,NULL,NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL—
```

This method is popular because you can also use it to figure out the data types of each column by trying a different value to NULL in each column:

```sql
' UNION SELECT 'x',NULL,NULL,NULL--
' UNION SELECT NULL,'x',NULL,NULL,NULL--
' UNION SELECT NULL,NULL,'x',NULL,NULL--
' UNION SELECT NULL,NULL,'x',NULL,NULL,NULL,NULL,NULL--
```

## Error based SQL injection:

- When the attacker gives an unauthorized query and the web application returns the a error message or a application response, attacker can get program logic from this and conduct an attack
- Eg: Warning: mysql_query(): Unable to save result set in C:\xampp\htdocs\dvwa\vulnerabilities\sqli\source\low.php on line 10

## Blind Sql injection:

- When web server gives out a generic error the attacker might have a invalid query and attacker might need to troubleshoot it
- So they use blind SQL in this case
- Way to exfiltrate data from db when you cant see the output

### Boolean based:

- Ask the db true or false and determine the answer based on the response given
  - Response in general would be a content error or a blank page
- Eg: ID=1 AND 1=1 -> true
- Eg: ID=2 AND 1=2 -> false

### Time based:

- Asks the db to pause for some time and gives out the results.
- If the result is produced after the stated time then it means the SQL is executed successfully
- Eg: SLEEP(5) – sleep for 5 sec. if the webapplication is sustible to blind sql then there will be a delay of 5 sec while loading
