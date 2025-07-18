# Reused Salt

## Summary
This section covers a reused salt vulnerability in a web-based version of the gift card application implemented using Django. Unlike some of the other vulnerabilities, which targeted the original C-based implementation, this issue arises within the Python-based web interface and its HTML components.

To identify this vulnerability, I checked the stored password hashes in the `users` table and noticed that they all started with the same prefix. This shows that a static salt was being reused for all passwords.

**Note:** The original source files were provided as part of the course and are not included in this repository.

## Affected Component
The affected component was the password hashing function which used a static seed for salt generation.

## Vulnerability Details
In the function below, the same static seed is being used to salt every password. As a result, all the passwords stored in the database began with the same prefix.

```
SEED = settings.RANDOM_SEED

def generate_salt(length, debug=True):
    random.seed(SEED)
    return hexlify(random.randint(0, 2**length-1).to_bytes(length, byteorder='big'))
```

## Steps to Reproduce
To reproduce this issue, I created multiple user accounts with different passwords. I then queried the `users` table in the database to view the hashed passwords. Even though all the passwords were different, each hash began with the same prefix (`000000000000000000000000000078d2`). This confirmed that a static salt was being reused for every password.

Below is a sample output of the stored password hashes showing the identical salt prefix:
```
user1 | 000000000000000000000000000078d2$18821d...
user2 | 000000000000000000000000000078d2$a8dfe9...
user3 | 000000000000000000000000000078d2$9649d0...
```

## Impact
Reusing the same salt makes it easier for attackers to reuse rainbow tables to crack multiple passwords at once. In a real-world scenario, if the database were leaked or a data breach occurred, it could lead to a large-scale credential exposure and account takeovers.

## Mitigation
To fix this vulnerability, I removed the static `SEED` argument in the `generate_salt()` function. By default Python's `random.seed()` uses the current timestamp as the seed. This means that the seed value changes automatically each time, resulting in a new salt value for each password. This ensures that even if the passwords are the same, the salts and hashed passwords will be different.
```
def generate_salt(length, debug=True):
    random.seed()
    return hexlify(random.randint(0, 2**length-1).to_bytes(length, byteorder='big'))
```

## Tools and Artifacts Used
- SQL (used to query the `users` table and view the stored hashes)
- Django-based gift card application (target application used for testing)
- Manual source code review
