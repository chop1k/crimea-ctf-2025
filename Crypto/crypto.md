# Crypto

Task from cryptography category with forgotten name

## Input

Text [file](../Payloads/crypto.txt) with some encrypted(lol) data.

## Goal

Decrypt data and get the flag.

## Solving

Looks like first part of the file is actually a flag.

Let's write a bash script to decrypt the flag.

```bash
#!/bin/bash

chars=("." "-" "&" "<" ">" "'" "*" "@" "{" "}" "\^" "$" "=" "!" "-" "_")
for letter in {A..Z} {a..z} {0..9} "${chars[@]}"; do
	hash=$(echo -n "$letter" | md5sum | awk '{print $1}')
	sed -i "s/$hash/$letter/g" task.hs
done
```

After executing the script, we will get most part of the flag.

## The flag

```text
crimea_ctf{KEYloGgER$_BOtmASter_In_ANda
```

Last two symbols we guessed manually.
