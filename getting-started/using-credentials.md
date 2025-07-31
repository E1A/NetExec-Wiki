---
description: Using credentials with NetExec
---

# Using Credentials

## Using Credentials

Every protocol supports using credentials in one form or another. For details on using credentials with a specific protocol, see the appropriate wiki section.

Generally speaking, to use credentials, you can run the following commands:

```bash
nxc <protocol> <target(s)> -u username -p password
```

{% hint style="success" %}
Code execution results in a (**Pwn3d!**) added after the login confirmation. With the SMB protocol, your compromised users are most likely in the (local) administrators group.
{% endhint %}

| Protocol | See Pwn3d! in output                                   |
| -------- | ------------------------------------------------------ |
| FTP      | No check                                               |
| SSH      | root (otherwise specific message) :white\_check\_mark: |
| WINRM    | Code execution at least :space\_invader:               |
| LDAP     | Path to domain admin :crown:                           |
| SMB      | Most likely (local) admin :white\_check\_mark:         |
| RDP      | Code execution at least :space\_invader:               |
| VNC      | Code execution at least :space\_invader:               |
| WMI      | Most likely local admin :white\_check\_mark:           |

{% hint style="info" %}
When using usernames or passwords that contain special symbols (especially exclaimation points!), wrap them in single quotes to make sure your shell interprets them as a string.
{% endhint %}

Example:

```bash
nxc <protocol> <target(s)> -u username -p 'October2022!'
```

{% hint style="info" %}
Due to a [bug](https://bugs.python.org/issue9334) in Python's argument parsing library, credentials beginning with a dash (`-`) will throw an `expected at least one argument` error message. To get around this, specify the credentials by using the 'long' argument format (note the `=` sign):
{% endhint %}

```bash
nxc <protocol> <target(s)> -u='-username' -p='-October2022'
```

## Using a Credential Set From the Database

By specifying a credential ID (or multiple credential IDs) with the `-id` flag, nxc will automatically pull that credential from the back-end database and use it to authenticate (saves a lot of typing):

```bash
nxc <protocol> <target(s)> -id <cred ID(s)>
```

## Multi-Domain Environment

You can use nxc with mulitple domain environment

```bash
nxc <protocol> <target(s)> -u FILE -p password
```

Where **FILE** is a file with usernames in this format

```bash
DOMAIN1\user
DOMAIN2\user
```

## Brute Forcing & Password Spraying

All protocols support brute-forcing and password spraying. For details on brute-forcing/password spraying with a specific protocol, see the appropriate wiki section.

By specifying a file or multiple values nxc will automatically brute-force logins for all targets using the specified protocol:

Examples:

```bash
nxc <protocol> <target(s)> -u username1 -p password1 password2
```

```bash
nxc <protocol> <target(s)> -u username1 username2 -p password1
```

```bash
nxc <protocol> <target(s)> -u ~/file_containing_usernames -p ~/file_containing_passwords
```

```bash
nxc <protocol> <target(s)> -u ~/file_containing_usernames -H ~/file_containing_ntlm_hashes
```

## Password Spraying Without Bruteforce

Can be useful for protocols like WinRM and MSSQL. This option avoids bruteforcing when you use files (-u file -p file).

```bash
nxc <protocol> <target(s)> -u ~/file_containing_usernames -H ~/file_containing_ntlm_hashes --no-bruteforce
```

```bash
nxc <protocol> <target(s)> -u ~/file_containing_usernames -p ~/file_containing_passwords --no-bruteforce
```

```bash
user1 -> pass1
user2 -> pass2
```

{% hint style="info" %}
By default nxc will exit after a successful login is found. Using the --continue-on-success flag will continue spraying even after a valid password is found. Useful for spraying a single password against a large user list. The --continue-on-success flag is incompatible with command execution.
{% endhint %}

```bash
nxc <protocol> <target(s)> -u ~/file_containing_usernames -H ~/file_containing_ntlm_hashes --no-bruteforce --continue-on-success
```

### Throttling Authentication Requests

{% hint style="warning" %}
Authentication throttling works on a per-host basis! Keep this in mind if you are spraying credentials against multiple hosts.
{% endhint %}

If there is a need to throttle authentications during brute forcing, you can use the jitter functionality. The length of the timeout (in seconds) between requests is randomly selected from an interval unless otherwise specified. If you want to hardcode the timeout, set the upper and lower bounds of the interval to the same value. The syntax is as follows:

```bash
nxc <protocol> <target> --jitter 3 -u ~/file_containing_usernames -p ~/file_containing_passwords
nxc <protocol> <target> --jitter 2-5 -u ~/file_containing_usernames -p ~/file_containing_passwords
nxc <protocol> <target> --jitter 4-4 -u ~/file_containing_usernames -p ~/file_containing_passwords
```
