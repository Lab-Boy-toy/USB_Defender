# USB Drive Auth

USB Drive Auth is a Windows-based Python project that creates and verifies authentication files on USB storage devices.

The project is split into two main programs:

* **USB Auth Writer** creates an authentication file for a USB drive.
* **USB Auth Background** watches for newly connected drives and checks whether they are registered with the local organization.

The goal is to make the process mostly automatic. The writer can be started and left running, then a USB can simply be plugged in to receive its authentication file. The background checker can monitor newly connected drives and determine whether they contain the correct authentication information.

## How It Works

Each installation uses a three-digit organization ID.

When a USB is registered, the writer:

1. Detects a newly connected drive.
2. Pulls the drive's Windows volume serial number.
3. Uses the organization ID to encode the serial number.
4. Writes the encoded result into `USB_ID.txt` on the USB.
5. Waits for the drive to be removed before preparing for another USB.

The background checker performs the opposite process.

When a new USB is connected, it:

1. Detects the newly connected drive.
2. Looks for `USB_ID.txt`.
3. Pulls the drive's current volume serial number.
4. Recreates the encoded value using the local organization ID.
5. Compares the recreated value against the value stored on the USB.
6. Treats the drive as approved if the values match.
7. Adds approved drives to the current native drive list so they are not repeatedly checked.
8. Warns the user if the authentication file is missing or incorrect.
9. Waits for an unknown drive to be removed and reports its drive letter and serial number.

## Current Flow

```text
Start Program
      |
      v
Record Existing Drives
      |
      v
Wait for New USB
      |
      v
Look for USB_ID.txt
      |
      +----------------------+
      |                      |
   Missing                 Found
      |                      |
      v                      v
   Flag USB          Read Drive Serial
                             |
                             v
                    Recreate Encoded ID
                             |
                             v
                       Compare Values
                         /        \
                        /          \
                    Match        No Match
                      |              |
                      v              v
                 Approve USB      Flag USB
                      |
                      v
             Update Native Drives
```

## Requirements

* Windows
* Python 3
* `pywin32`

Install the required Windows API package with:

```bash
python -m pip install pywin32
```

## Organization ID

The project uses a three-digit organization ID.

Example:

```text
457
```

The organization ID is stored locally in:

```text

Org_ID.txt

```

The writer can create this file during setup.

The background checker expects the local organization ID file to already exist. If it does not, the checker exits and asks for the organization ID to be configured first.

## USB Authentication File

Registered USB drives also contain a file named:

```text

USB_ID.txt

```

This file contains an encoded value generated from the USB drive's volume serial number and the local organization ID.

Because the encoded value depends on the drive serial, copying an authentication file from one USB drive to another should not produce a valid match.

## Automatic Drive Detection

The programs use the Windows drive list to determine which drives existed when the program started.

New drives are identified by comparing the current list of drives against that original list.

The writer waits for a new drive, writes the authentication file, then waits for that drive to be removed.

The checker continuously watches for new drives. Once a USB successfully passes authentication, it is added to the accepted/native drive state so the program does not repeatedly authenticate the same connected drive.

## Unknown USB Behavior

A drive is currently treated as unregistered if:

* `USB_ID.txt` is missing.
* The contents of `USB_ID.txt` do not match the value generated from the USB's current serial number and the local organization ID.

When this happens, the program warns the user and waits for the drive to be removed.

The USB's drive letter and serial number are saved before removal so they can still be reported afterward.

## Running at Windows Startup

The background checker can be configured to launch when the user signs into Windows.

Press:

```text
Win + R
```

and enter:

```text
shell:startup
```

A shortcut can then be placed in the Startup folder.

For a background process without a console window, the shortcut can use `pythonw.exe` instead of `python.exe`.

## Current Status

The current prototype supports:

* Automatic detection of newly connected drives
* Automatic USB authentication-file creation
* Three-digit organization IDs
* Drive serial based encoding
* Automatic reading of USB authentication files
* Authentication-file comparison
* Detection of missing authentication files
* Detection of authentication files that do not match the drive
* Whitelisting of successfully authenticated connected drives
* Warning behavior for unknown drives
* Drive serial reporting after removal
* Continuous background monitoring
* Windows startup support

## Security Limitations

This project is currently a prototype and should not be treated as hardened endpoint security.

The current system uses the Windows filesystem volume serial number as the USB identifier. This is useful for experimentation, but it is not the same as a secure hardware identity and can potentially change or be reproduced.

The current encoding system is also a custom transformation rather than cryptographic authentication.

Because of this, the project currently demonstrates the authentication workflow rather than providing strong protection against a determined attacker.


Future versions could improve this by using:

* Cryptographic hashes
* HMAC authentication
* Signed authentication files
* Protected organization secrets
* Hardware USB identifiers
* Windows device-control APIs
* Logging
* A graphical warning system
* Stronger device blocking before access is allowed

## Project Goal

The long-term goal is to build a system that can identify whether newly connected USB storage belongs to an organization and react differently to known and unknown devices.

This version establishes the basic registration and verification system needed to build toward that goal.

## Disclaimer

This project is intended for learning, experimentation, and defensive security research on systems and USB devices you own or are authorized to manage.
