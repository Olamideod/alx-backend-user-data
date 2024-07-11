`# Personal Data Handling Project

## Table of Contents
- [Description](#description)
- [Resources](#resources)
- [Learning Objectives](#learning-objectives)
- [Requirements](#requirements)
- [Tasks](#tasks)

## Description
This project focuses on handling personal data securely, including obfuscating sensitive information, encrypting passwords, authenticating users, and connecting securely to a database.

## Resources
Read or watch:
- [What Is PII, non-PII, and Personal Data?](https://example.com)
- [Logging Documentation](https://docs.python.org/3/howto/logging.html)
- [bcrypt Package](https://pypi.org/project/bcrypt/)
- [Logging to Files, Setting Levels, and Formatting](https://docs.python.org/3/howto/logging.html#logging-to-a-file)

## Learning Objectives
By the end of this project, you should be able to:
- Identify examples of Personally Identifiable Information (PII)
- Implement a log filter that obfuscates PII fields
- Encrypt a password and check the validity of an input password
- Authenticate to a database using environment variables

## Requirements
- All files will be interpreted/compiled on Ubuntu 18.04 LTS using python3 (version 3.7)
- All files should end with a new line
- The first line of all files should be exactly `#!/usr/bin/env python3`
- A `README.md` file at the root of the project folder is mandatory
- Code should follow the `pycodestyle` style (version 2.5)
- All files must be executable
- Length of files will be tested using `wc`
- All modules, classes, and functions should have documentation explaining their purpose
- All functions should be type annotated

## Tasks

### Task 1: Log Formatter
Update the `RedactingFormatter` class to accept a list of strings `fields` constructor argument. Implement the `format` method to filter values in incoming log records using `filter_datum`.

```python
import logging

class RedactingFormatter(logging.Formatter):
    """ Redacting Formatter class """

    REDACTION = "***"
    FORMAT = "[HOLBERTON] %(name)s %(levelname)s %(asctime)-15s: %(message)s"
    SEPARATOR = ";"

    def __init__(self, fields):
        super(RedactingFormatter, self).__init__(self.FORMAT)
        self.fields = fields

    def format(self, record: logging.LogRecord) -> str:
        return filter_datum(self.fields, self.REDACTION, super().format(record), self.SEPARATOR)

### Task 2: Create Logger
Implement a get_logger function that returns a logging.Logger object.

The logger should be named "user_data" and only log up to logging.INFO level.
It should not propagate messages to other loggers.
It should have a StreamHandler with RedactingFormatter as formatter.

```python
import logging

PII_FIELDS = ("name", "email", "phone", "ssn", "password")

def get_logger() -> logging.Logger:
    logger = logging.getLogger("user_data")
    logger.setLevel(logging.INFO)
    logger.propagate = False
    handler = logging.StreamHandler()
    handler.setFormatter(RedactingFormatter(PII_FIELDS))
    logger.addHandler(handler)
    return logger


### Task 3: Connect to Secure Database
Implement a get_db function that returns a connector to the database (mysql.connector.connection.MySQLConnection object).

Use the os module to obtain credentials from the environment.
Use the mysql-connector-python module to connect to the MySQL database.

```python
import mysql.connector
import os

def get_db() -> mysql.connector.connection.MySQLConnection:
    user = os.getenv("PERSONAL_DATA_DB_USERNAME", "root")
    password = os.getenv("PERSONAL_DATA_DB_PASSWORD", "")
    host = os.getenv("PERSONAL_DATA_DB_HOST", "localhost")
    database = os.getenv("PERSONAL_DATA_DB_NAME")
    return mysql.connector.connect(
        user=user,
        password=password,
        host=host,
        database=database
    )

### Task 4: Read and Filter Data
Implement a main function that retrieves all rows in the users table and displays each row under a filtered format.

```python

def main():
    db = get_db()
    cursor = db.cursor()
    cursor.execute("SELECT * FROM users;")
    fields = cursor.column_names
    for row in cursor:
        message = '; '.join(f"{field}={value}" for field, value in zip(fields, row))
        log_record = logging.LogRecord("user_data", logging.INFO, None, None, message, None, None)
        formatter = RedactingFormatter(fields)
        print(formatter.format(log_record))
    cursor.close()
    db.close()

if __name__ == "__main__":
    main()
