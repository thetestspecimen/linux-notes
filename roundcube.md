# Roundcube

Roundcube is (as per the official [website](https://roundcube.net/)):

> a free and open source webmail solution with a desktop-like user interface which is easy to install/configure and that runs on a standard LAMPP server.

The important things for me are:

- It has a responsive design, and is well laid out
- Support for PGP/GPG encryption, signing etc.

## Changing the logos

**Please note the following requirements:**
- **Roundcube version > 1.4**
- **Base skin: Elastic (i.e. not larry or classic)**

You may want to update the standard logos that roundmail uses to match your website.

Although this is possible, it is not particularly well documented, and I ran into a lot of problems trying to get it to work
without just cloning the whole of one of the skins.

(Note: Blanket cloning one of the skins will work but you will miss out on any changes to the base skin in subsequent updates, or
have to do the same work again on every update!)

## Basic information

I am assuming at this point that you have successfully installed roundcube, and have the skin 'Elastic' in use. If that is not the 
case please go and install roundcube, and come back when you are done.

The directory paths mentioned in this guide are relative to where roundcube is installed on your server. For example your install
may be at:

	/var/www/roundcube

or

	/var/www/html/roundcube

...or any other variant. so something living in the 'base' directory would be refering to your equivalent of the above.

## What will change

After you have been through this guide, the following will have been updated with your own logo:

1) The login screen logo
2) The main logo in the top right once logged in
3) The watermark in the email section
4) The favicon

## Creating the favicon and new logo files

You will need the following files:

- custom-favicon.ico
- custom-logo.svg

I would recommend that your logo is close to haveing the same height and width dimensions, it doesn't need to be **precise**, 
but the further away from this you are the more the end result will be scaled. 

### Favicon

The favicon is fairly standard and can be generated using numerous websites. For example:

[favicon.io](https://favicon.io/favicon-converter/)

### Main logo

The main logo should preferably be in svg format. This is mainly due to the fact that it is scalable, as implied in the name.
It is also the format used by the 'elastic' skin by default.

Again there are various ways to go about converting png or jpg to svg. Just have a look what is available.

## Creating a new child skin

We now need to create a 'child' skin, which inherits from the 'parent' skin. The parent in this case is the 'elastic' skin.

Go to your base roundcube directory. There should be a folders called 'skins', go inside the folder.

Within this folder you will create a new folder with the name of your skin. It doesn't matter what you use, as long as:

- it is all lowercase
- it does not contain any spaces
- it is not 'larry', 'elastic' or 'classic' (i.e. the existing skins)

I will call mine 'newskin'

Inside the 'newskin' folder create a folder called 'images'. You will now put your favicon and logo files in this folder. It is recommended that
you call them 'custom-favicon.ico' and 'custom-logo.svg', although in theory you could use whatever you like.

**However, do not call them 'favicon.ico' and 'logo.svg'**

As this is what they are called in the base theme, and it will not work.

Move back into the newskin folder and create the following files:

- meta.json
- watermark.html

Open 'meta.json', and enter the following text:

```json
{
	"name": "newskin",
	"extends": "elastic",
	"author": "Aleksander Machniak",
	"license": "Creative Commons Attribution-ShareAlike",
	"license-url": "http://creativecommons.org/licenses/by-sa/3.0/",
	"config": {
		"layout": "widescreen",
		"jquery_ui_colors_theme": "bootstrap",
		"embed_css_location": "/styles/embed.css",
		"editor_css_location": "/styles/embed.css",
		"media_browser_css_location": "none",
		"favicon": "/images/custom-favicon.ico"
	},
	"meta": {
		"viewport": "width=device-width, initial-scale=1.0, shrink-to-fit=no, maximum-scale=1.0",
		"theme-color": "#f4f4f4",
		"msapplication-navbutton-color": "#f4f4f4"
	}
}
```

Save the file, and open 'watermark.html'. Paste the following text:

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			html, body { height: 100%; overflow: hidden; }
			body {
  				background: url(images/custom-logo.svg) center no-repeat #fff;
  				background-size: 30%;
  				background-blend-mode: luminosity;
			}
			body:before { content: ""; position: absolute; top: 0; bottom: 0; left: 0; right: 0; background: rgba(255, 255, 255, .85); }
		</style>
	</head>
	<body></body>
</html>
```

Save the file.

## Create config file

We now need to create a configuration file for the new skin.

Within the folder 'config' in the base roundcube directory, create the file 'newskin.inc.php'.

Paste in the following and save:

```php
// provide an URL where a user can get support for this Roundcube installation
// PLEASE DO NOT LINK TO THE ROUNDCUBE.NET WEBSITE HERE!
$config['support_url'] = 'mailto:info@mywebsite.com';

// Name your service. This is displayed on the login screen and in the window title
$config['product_name'] = 'My Website Webmail';

// $config['skin_logo'] = array("*" => "/images/mylogo.svg", "messageprint" => "/images/mylogo.svg", "elastic:*" => "/images/mylogo.svg");
$config['skin_logo'] = array("*" => "/images/custom-logo.svg");
$config['skin'] = 'newskin';
```

The above will allow the use of the new logos, as well as allowing you to change:

- the contact email on the login screen
- the name of the email / website service on the login screen

## Update the main config file

We now need to tell roundcube where to find the config file, so we reference it in the main config file 'config.inc.php':

```php
$config['include_host_config'] = array(
	'mail.mywebsite.com' => 'newskin.inc.php'
); 
```

where 'mail.mywebsite.com' is the URL of the webmail login.

You can also define multiple skins and mail URLs as follows:

```php
$config['include_host_config'] = array(
	'mail.mywebsite.com' => 'newskin.inc.php',
	'mail.mywebsite2.net' => 'newskin2.inc.php'
); 
```

That should be it!

**Be sure to check that folder and file permissions / users are as they should be (i.e. you haven't got the logos, favicon or config files as unreadable).**