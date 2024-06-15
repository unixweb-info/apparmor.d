# AppArmor Rules for ISPmanager

This repository contains a collection of AppArmor security profiles designed specifically for ISPmanager. ISPmanager is a popular web hosting control panel solution, and these profiles provide an additional layer of security when running ISPmanager on a server.

## Project Overview

The AppArmor security profiles in this repository are designed to restrict the capabilities of various services within ISPmanager, limiting their access to the file system and preventing them from performing unauthorized actions. This can help to mitigate the potential impact of software vulnerabilities in ISPmanager and the services it manages.

The profiles cover a range of services, including but not limited to Dovecot, ClamAV's freshclam, and Dovecot's configuration tool. Each profile is carefully crafted with the necessary permissions for each service to function correctly, while ensuring that they have the minimum privileges required.

## Usage

To use these profiles, you would typically install AppArmor on your server, place the profiles in the appropriate directory (usually `/etc/apparmor.d/`), and then reload AppArmor. Please refer to the AppArmor and ISPmanager documentation for more detailed instructions.

### Download the script from the provided URL
```bash
wget https://raw.githubusercontent.com/unixweb-info/Automatic-configuration-of-AppArmor-profiles-in-ISPmanager/main/apparmor-install-and-setting-in-ispmanager6.sh

```
### Make the downloaded script executable
```bash
chmod +x apparmor-install-and-setting-in-ispmanager6.sh
```
### Run the script
```bash
./apparmor-install-and-setting-in-ispmanager6.sh

```
#### Bash Script
```bash
#!/bin/bash

#  Check if the script is being run as root or by a user in the sudo group
if [[ $EUID -ne 0 ]]; then
  if groups $USER | grep &>/dev/null '\bsudo\b'; then
    echo "Script is being run by a user in the sudo group."
  else
    echo "This script must be run as root or by a user in the sudo group. Exiting."
    exit 1
  fi
fi

#  Check if the operating system is supported
if [[ "$(lsb_release -is)" != "Ubuntu" && "$(lsb_release -is)" != "Debian" ]]; then
  echo "This script is only supported on Ubuntu and Debian. Exiting."
  exit 1
fi

#  Check if sudo is installed and install it if not
if ! command -v sudo &>/dev/null; then
  echo "sudo is not installed. Installing..."
  apt-get install sudo
fi

#  Check if git is installed and install it if not
if ! command -v git &>/dev/null; then
  echo "git is not installed. Installing..."
  sudo apt-get install git
fi

#  Define packages to install
packages=("apparmor-utils" "libapache2-mod-apparmor" "auditd")

#  Install packages if they are not already installed
for package in "${packages[@]}"; do
  if dpkg -s "$package" >/dev/null 2>&1; then
    echo "$package is already installed."
  else
    echo "Installing $package..."
    if ! sudo apt install -y "$package"; then
      echo "Error installing $package. Exiting."
      exit 1
    fi
  fi
done

#  Backup existing AppArmor profiles
if [ -d "/etc/apparmor.d" ]; then
  echo "Backing up existing AppArmor profiles..."
  if ! sudo cp -r /etc/apparmor.d /root/apparmor.d; then
    echo "Error backing up existing AppArmor profiles. Exiting."
    exit 1
  fi
else
  echo "No existing AppArmor profiles found."
fi

#  Define patterns to match files to remove
patterns=("/etc/apparmor.d/tunables/home" "/etc/apparmor.d/usr.bin.doveconf" "/etc/apparmor.d/usr.bin.freshclam" "/etc/apparmor.d/usr.lib.dovecot.*" "/etc/apparmor.d/usr.sbin.apache2" "/etc/apparmor.d/usr.sbin.apache2.dpkg-dist" "/etc/apparmor.d/usr.sbin.clamd" "/etc/apparmor.d/usr.sbin.dovecot" "/etc/apparmor.d/usr.sbin.exim4" "/etc/apparmor.d/usr.sbin.mysqld" "/etc/apparmor.d/usr.sbin.nginx" "/etc/apparmor.d/usr.sbin.php-fpm8.1" "/etc/apparmor.d/usr.sbin.proftpd")

#  Remove files if they exist
for pattern in "${patterns[@]}"; do
  files=$(find / -path "$pattern" 2>/dev/null)
  for file in $files; do
    if [ -f "$file" ]; then
      echo "Removing $file..."
      if ! sudo rm -rf "$file"; then
        echo "Error removing $file. Exiting."
        exit 1
      fi
    else
      echo "$file does not exist."
    fi
  done
done

#  Clone AppArmor profiles from GitHub
#  Create a temporary directory
temp_dir=$(mktemp -d)

#  Clone AppArmor profiles from GitHub into the temporary directory
echo "Cloning AppArmor profiles from GitHub..."
if ! sudo git clone https://github.com/unixweb-info/apparmor.d.git "$temp_dir"; then
  echo "Error cloning AppArmor profiles from GitHub. Exiting."
  exit 1
fi

#  Move the cloned files into /etc/apparmor.d
echo "Moving AppArmor profiles into /etc/apparmor.d..."
sudo rsync -a "$temp_dir/" /etc/apparmor.d/

#  Remove the temporary directory
sudo rm -r "$temp_dir"

#  Removing the README.md, .gitattributes files and the .git directory from the /etc/apparmor.d folder.
sudo rm -rf /etc/apparmor.d/{README.md,.gitattributes,.git} 


#  Restart services
echo "Restarting services..."
if ! sudo systemctl restart apparmor auditd; then
  echo "Error restarting services. Exiting."
  exit 1
fi

#  Enforce AppArmor profiles
profiles=("/etc/apparmor.d/usr.sbin.dovecot" "/etc/apparmor.d/usr.bin.freshclam" "/etc/apparmor.d/usr.sbin.clamd" "/etc/apparmor.d/usr.sbin.exim4" "/etc/apparmor.d/usr.sbin.apache2" "/etc/apparmor.d/usr.sbin.mysqld" "/etc/apparmor.d/usr.sbin.nginx" "/etc/apparmor.d/usr.sbin.php-fpm8.1" "/etc/apparmor.d/usr.sbin.proftpd")

for profile in "${profiles[@]}"; do
  if [ -f "$profile" ]; then
    echo "Enforcing $profile..."
    if ! sudo aa-enforce "$profile"; then
      echo "Error enforcing $profile. Exiting."
      exit 1
    fi
  else
    echo "$profile does not exist."
  fi
done

#  Restart services
echo "Restarting services..."
if ! sudo systemctl restart clamav-daemon clamav-freshclam mysql nginx dovecot proftpd php8.1-fpm apache2; then
  echo "Error restarting services. Exiting."
  exit 1
fi

# Display AppArmor status
echo "Displaying AppArmor status..."
if ! sudo aa-status; then
  echo "Error displaying AppArmor status. Exiting."
  exit 1
fi
```
## Contributions

Contributions to this project are welcome. If you have improvements to existing profiles, or have created a new profile for a service not currently covered, please feel free to submit a pull request.

## Disclaimer

While these profiles are designed to enhance security, they are provided as-is, and users should apply them at their own risk. Always test changes in a safe environment before applying them to a production system.

Please note that this project is not officially affiliated with or endorsed by the creators of ISPmanager. It is a community project, created to help users enhance the security of their ISPmanager installations.
