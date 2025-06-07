# Archiwizard (Reverse Engineering)

Task from reverse engineering category named "Архимаг".

## Input

A zip archive containing a directory with another archive inside, which in turn contains yet another directory with an archive, and so on recursively.

## Goal

Unzip the archive and get the flag.

## Solving

Attempting to unpack the archive manually proved pointless (I tried, lol), so we decided to write an automation script to handle the extraction for us.

The script was written in Python because a Bash script would be too slow to process all the data. With Bash, the extraction would take over two hours - completely unacceptable during a CTF competition. Bash's slowness stems from its need to execute a new process for each command, which creates significant overhead for the operating system. Python, on the other hand, doesn't spawn new processes for each operation, making it substantially faster than Bash.

For comparison: on my laptop, Python running in /tmp (remember that tmpfs operates in memory) unpacks the archive in about five minutes or less. Meanwhile, Bash working on an ext4 filesystem (in my home's Downloads directory) fails to completely unpack the archive even after two hours or more.

```python
import os
import sys
import zipfile
import shutil

def is_encrypted(zip_file):
    with zipfile.ZipFile(zip_file) as zf:
        for zinfo in zf.infolist():
            if zinfo.flag_bits & 0x1:
                return True
    return False

def extract_zip(zip_file, extract_to):
    if is_encrypted(zip_file):
        raise ValueError(f"Archive '{zip_file}' is encrypted. Cannot extract.")

    with zipfile.ZipFile(zip_file) as zf:
        zf.extractall(extract_to)

def find_first_zip(directory):
    for root, _, files in os.walk(directory):
        for file in files:
            if file.lower().endswith('.zip'):
                return os.path.join(root, file)
    return None

def clean_directory_except(directory, keep_file, protect_file):
    for entry in os.listdir(directory):
        entry_path = os.path.join(directory, entry)
        if entry_path != keep_file and os.path.abspath(entry_path) != os.path.abspath(protect_file):
            if os.path.isdir(entry_path):
                shutil.rmtree(entry_path)
            else:
                os.remove(entry_path)

def recursive_unpack(zip_file, protect_file):
    work_dir = os.path.abspath("workdir")

    if os.path.exists(work_dir):
        shutil.rmtree(work_dir)
    os.makedirs(work_dir)

    # Copy initial zip into working dir
    zip_copy = os.path.join(work_dir, os.path.basename(zip_file))
    shutil.copy(zip_file, zip_copy)

    current_zip = zip_copy

    while current_zip:
        print(f"Extracting: {current_zip}")
        extract_zip(current_zip, work_dir)

        # Remove the zip we just extracted
        os.remove(current_zip)

        # Find next zip archive
        next_zip = find_first_zip(work_dir)

        if next_zip:
            # Clean everything except the next zip and the script itself
            clean_directory_except(work_dir, next_zip, protect_file)
            current_zip = next_zip
        else:
            print("✅ No more zip archives found.")
            current_zip = None

    print("🎉 Done. Final extracted content is in 'workdir/'.")

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print(f"Usage: {sys.argv[0]} <zip_file>")
        sys.exit(1)

    input_zip = sys.argv[1]
    script_file = os.path.abspath(__file__)

    try:
        recursive_unpack(input_zip, script_file)
    except Exception as e:
        print(f"❌ Error: {e}")
        sys.exit(1)
```

The unpacking process completed in about 5 minutes, resulting in an encrypted archive.

We could use hash cracking tools like hashcat or John the Ripper. As it turns out, the archive password was simply "1".

## Flag

```text
crimea_ctf{PrIvaTe_tE_h@CKeR}
```
