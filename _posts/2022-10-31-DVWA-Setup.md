Today, we are going to look at how you can set up DVWA in Windows. Before starting to look into our setup process, I wanna give an introduction to DWVA to those who do not know anything about it.

DVWA is a web application which have a lot of vulnerabilities in it. You can practice your bug hunting and penetration testing skills on it. This application has nearly all the bugs of OWASP 10. So it was a tiny introduction :) Without further wasting your time, I am going through the setup process.

## Setup:
The whole setup process is very straightforward. I am setting up DVWA on Windows 10, but this setup is also the same for previous Windows versions. So let’s start.

First of all, here's what we need:
1) Xampp (for hosting our local Apache Server)
2) DVWA (from Github)

### Downloading and setting up Xampp:
First of all, go to [https://www.apachefriends.org/](https://www.apachefriends.org/) and download XAMPP for Windows. An executable will be downloaded to your computer. Double click on it and setup XAMPP server while clicking on Next, Next … :)

### Downloading and setting up DVWA:
DVWA can be downloaded from GitHub through this link: [https://github.com/digininja/DVWA](https://github.com/digininja/DVWA)

After downloading the zip file, extract it. Copy the folder `DVWA-master` inside the extracted folder. Now go to `C:\xampp\htdocs\dashboard\` and paste your copied folder here.

You have to rename a file now, which is located in your copied folder. So here your copied folder is `DVWA-master` so go into this folder, you will find a folder named `config`. Inside this folder, you will find a file named `config.inc.php.dist`, remove `.dist` from the end of the file and make it a PHP file like `config.inc.php`. Now edit this PHP file in Notepad or any editor you like :) Change `db_user` and `db_password` fields like this:

BEFORE:
```
$_DVWA[ 'db_user' ] = 'dvwa';
$_DVWA[ 'db_password' ] = 'p@ssword';
```
AFTER:
```
$_DVWA[ 'db_user' ] = 'root';
$_DVWA[ 'db_password' ] = '';
```
At this point, setup is almost complete. Now launch the XAMPP control panel and start the Apache and MySQL services. Browse to the 127.0.0.1 address, and you will see the welcome page of Apache. Now go to `http://127.0.0.1/dashboard/DVWA-master/setup.php` and you will see a setup page of DVWA. Scroll to the bottom of the page and click on Create/Reset Database. You will see the login page of DVWA, and your Damn Vulnerable Web Application is now completely set up.

You can log in to DVWA using the default credentials (username="admin", password="password") mentioned on their GitHub.

VOILA! Now you can play around with this application and practice your web exploitation skills. By the time you master all types of bugs, you can also increase the security level of DVWA to challenge yourself.
