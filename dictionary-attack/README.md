# Dictionary Attack (Password Cracking)

## Summary
This standalone lab shows an offline dictionary attack, a technique where an attacker attempts to crack a hashed password by testing it against a list of common passwords. It simulates a real-world scenario using a pre-generated hash and a public wordlist (`rockyou.txt`).

## Steps to Reproduce
The main steps in this process are:

1. Extract salt and hash
    - The script parses the database entries and separates the salt from the hash
2. Hash and compare
    - For each word in the wordlist, it concatenates the salt and the word, hashes the result, and then compares it to the stored hash
3. Iterate through the dictionary list until a match is found

Core logic:
```
# hashing function used in the application
def hash_pword(pword):
    hasher = sha256()
    hasher.update(salt.encode('utf-8'))
    hasher.update(pword.encode('utf-8'))
    return hasher.hexdigest()

# iterate through the wordlist, hashing each word and checking for a match
for w in wordlist:
    encoded = w.strip()
    hashed_password = hash_pword(encoded)
    if hashed_password == stored_password:
        print(f"Password is {w}")
        break
```

## Impact
Cracking passwords can lead to full account takeovers, giving attackers access to sensitive data or internal systems. In a real-world scenario, even one cracked password could be the entry point for a larger breach or system-wide compromise.

## Mitigation
A website could protect against this kind of attack by using a slow cryptographic hash function, such as `bycrpt` or `scrpt`. Bycrpt becomes slower as the iteration count increases, which makes it more resistant against brute-force cracking. Scrypt is a password-based key derivation function, which makes the hash take up a lot of memory and CPU time. The significant time and computation requirements make scrypt resistant against brute-force attacks. Additionally, using a unique, random salt for each password can help against reuse-based attacks, and locking accounts after repeated failed login attempts can help slow down brute-force attacks.

## Tools and Artifacts Used
- Python (to write the script)
- `rockyou.txt` wordlist (for brute-force attempts)
