# AppArmor Rules for ISPmanager

This repository contains a collection of AppArmor security profiles designed specifically for ISPmanager. ISPmanager is a popular web hosting control panel solution, and these profiles provide an additional layer of security when running ISPmanager on a server.

## Project Overview

The AppArmor security profiles in this repository are designed to restrict the capabilities of various services within ISPmanager, limiting their access to the file system and preventing them from performing unauthorized actions. This can help to mitigate the potential impact of software vulnerabilities in ISPmanager and the services it manages.

The profiles cover a range of services, including but not limited to Dovecot, ClamAV's freshclam, and Dovecot's configuration tool. Each profile is carefully crafted with the necessary permissions for each service to function correctly, while ensuring that they have the minimum privileges required.

## Usage

To use these profiles, you would typically install AppArmor on your server, place the profiles in the appropriate directory (usually `/etc/apparmor.d/`), and then reload AppArmor. Please refer to the AppArmor and ISPmanager documentation for more detailed instructions.

### Download the script from the provided URL
<code>wget https://raw.githubusercontent.com/unixweb-info/Automatic-configuration-of-AppArmor-profiles-in-ISPmanager/main/apparmor-install-and-setting-in-ispmanager6.sh</code>

### Make the downloaded script executable
<code>chmod +x apparmor-install-and-setting-in-ispmanager6.sh</code>

### Run the script
<code>./apparmor-install-and-setting-in-ispmanager6.sh</code>


## Contributions

Contributions to this project are welcome. If you have improvements to existing profiles, or have created a new profile for a service not currently covered, please feel free to submit a pull request.

## Disclaimer

While these profiles are designed to enhance security, they are provided as-is, and users should apply them at their own risk. Always test changes in a safe environment before applying them to a production system.

Please note that this project is not officially affiliated with or endorsed by the creators of ISPmanager. It is a community project, created to help users enhance the security of their ISPmanager installations.
