# .htaccess
In Apache HTTP servers, `.htaccess` (hypertext access) is the configuration file that allows for web server configuration.

This document includes a number of best practice server rules for making web pages fast and secure. Adapted from [HTML5 Boilerplate](http://html5boilerplate.com/).

**In This Documentation**

1. [How It Works](#how-it-works)
2. [Server Settings](#server-settings)
	* [CORS-Enabled Images](#cors-enabled-images)
	* [Webfont Access](#webfont-access)
	* [MIME Types](#mime-types)
	* [GZIP Compression](#gzip-compression)
	* [Expires Headers](#expires-headers)
	* [Prevent Mobile Network Provider Modifications](#prevent-mobile-network-provider-modifications)
	* [ETag Removal](#etag-removal)
	* [UTF-8 Encoding](#utf-8-encoding)
	* [A Little Extra Security](#a-little-extra-security)
3. [How to Contribute](#how-to-contribute)
4. [License](#license)
5. [Changelog](#changelog)


## How It Works
If you're hosting a site on an Apache server, you can add any or all of the following snippets of code to the `.htaccess` file.

If no `.htaccess` file exists, create one. If you're hosting a WordPress site, make sure that you add these snippets *after* any existing WordPress configurations.

**Note:** If you're on a Mac, files beginning with a "." are hidden by default. To view your `.htaccess` file, [follow this tutorial](http://osxdaily.com/2009/02/25/show-hidden-files-in-os-x/).


## Server Settings

### CORS-Enabled Images
Send CORS headers if browsers request them. Enabled by default for images.

```ApacheConf
# ----------------------------------------------------------------------
# CORS-enabled images (@crossorigin)
# ----------------------------------------------------------------------

# Send CORS headers if browsers request them; enabled by default for images.
# developer.mozilla.org/en/CORS_Enabled_Image
# blog.chromium.org/2011/07/using-cross-domain-images-in-webgl-and.html
# hacks.mozilla.org/2011/11/using-cors-to-load-webgl-textures-from-cross-domain-images/
# wiki.mozilla.org/Security/Reviews/crossoriginAttribute

<IfModule mod_setenvif.c>
	<IfModule mod_headers.c>
		# mod_headers, y u no match by Content-Type?!
		<FilesMatch "\.(gif|ico|jpe?g|png|svg|svgz|webp)$">
			SetEnvIf Origin ":" IS_CORS
			Header set Access-Control-Allow-Origin "*" env=IS_CORS
		</FilesMatch>
	</IfModule>
</IfModule>
```


### Webfont Access
Allow access from all domains for webfonts.

```ApacheConf
# ----------------------------------------------------------------------
# Webfont access
# ----------------------------------------------------------------------

# Allow access from all domains for webfonts.
# Alternatively you could only whitelist your
# subdomains like "subdomain.example.com".

<IfModule mod_headers.c>
	<FilesMatch "\.(eot|font.css|otf|ttc|ttf|woff)$">
		Header set Access-Control-Allow-Origin "*"
	</FilesMatch>
</IfModule>
```


### MIME Types
Proper MIME type for all files

```ApacheConf
# ----------------------------------------------------------------------
# Proper MIME type for all files
# ----------------------------------------------------------------------

# JavaScript
# Normalize to standard type (it's sniffed in IE anyways)
# tools.ietf.org/html/rfc4329#section-7.2
AddType application/javascript         js jsonp
AddType application/json               json

# Audio
AddType audio/mp4                      m4a f4a f4b
AddType audio/ogg                      oga ogg

# Video
AddType video/mp4                      mp4 m4v f4v f4p
AddType video/ogg                      ogv
AddType video/webm                     webm
AddType video/x-flv                    flv

# SVG
# Required for svg webfonts on iPad
# twitter.com/FontSquirrel/status/14855840545
AddType     image/svg+xml              svg svgz
AddEncoding gzip                       svgz

# Webfonts
AddType application/vnd.ms-fontobject  eot
AddType application/x-font-ttf         ttf ttc
AddType application/x-font-woff        woff
AddType font/opentype                  otf

# Assorted types
AddType application/octet-stream            safariextz
AddType application/x-chrome-extension      crx
AddType application/x-opera-extension       oex
AddType application/x-shockwave-flash       swf
AddType application/x-web-app-manifest+json webapp
AddType application/x-xpinstall             xpi
AddType application/xml                     rss atom xml rdf
AddType image/webp                          webp
AddType image/x-icon                        ico
AddType text/cache-manifest                 appcache manifest
AddType text/vtt                            vtt
AddType text/x-component                    htc
AddType text/x-vcard                        vcf
```


### GZIP Compression
Dramatically reduce the size of the files your server sends to browsers by enabling gzip compression.

```ApacheConf
# ----------------------------------------------------------------------
# Gzip compression
# ----------------------------------------------------------------------

<IfModule mod_deflate.c>

	# Force deflate for mangled headers developer.yahoo.com/blogs/ydn/posts/2010/12/pushing-beyond-gzipping/
	<IfModule mod_setenvif.c>
		<IfModule mod_headers.c>
			SetEnvIfNoCase ^(Accept-EncodXng|X-cept-Encoding|X{15}|~{15}|-{15})$ ^((gzip|deflate)\s*,?\s*)+|[X~-]{4,13}$ HAVE_Accept-Encoding
			RequestHeader append Accept-Encoding "gzip,deflate" env=HAVE_Accept-Encoding
		</IfModule>
	</IfModule>

	# Compress all output labeled with one of the following MIME-types
	# (for Apache versions below 2.3.7, you don't need to enable `mod_filter`
	# and can remove the `<IfModule mod_filter.c>` and `</IfModule>` lines as
	# `AddOutputFilterByType` is still in the core directives)
	<IfModule mod_filter.c>
		AddOutputFilterByType DEFLATE application/atom+xml \
		application/javascript \
		application/json \
		application/rss+xml \
		application/vnd.ms-fontobject \
		application/x-font-ttf \
		application/xhtml+xml \
		application/xml \
		font/opentype \
		image/svg+xml \
		image/x-icon \
		text/css \
		text/html \
		text/plain \
		text/x-component \
		text/xml
	</IfModule>

</IfModule>
```

### Expires Headers
A first-time visitor to your page may have to make several HTTP requests, but by using the Expires header you make those components cacheable. This avoids unnecessary HTTP requests on subsequent page views. Expires headers are most often used with images, but they should be used on all components including scripts, stylesheets, etc.

```ApacheConf
# ----------------------------------------------------------------------
# Expires headers (for better cache control)
# ----------------------------------------------------------------------

# These are pretty far-future expires headers.
# They assume you control versioning with filename-based cache busting
# Additionally, consider that outdated proxies may miscache
# www.stevesouders.com/blog/2008/08/23/revving-filenames-dont-use-querystring/

# If you don't use filenames to version, lower the CSS and JS to something like
# "access plus 1 week".

<IfModule mod_expires.c>
	ExpiresActive on

	# Perhaps better to whitelist expires rules? Perhaps.
	ExpiresDefault                          "access plus 1 month"

	# cache.appcache needs re-requests in FF 3.6 (thanks Remy ~Introducing HTML5)
	ExpiresByType text/cache-manifest       "access plus 0 seconds"

	# Your document html
	ExpiresByType text/html                 "access plus 0 seconds"

	# Data
	ExpiresByType application/json          "access plus 0 seconds"
	ExpiresByType application/xml           "access plus 0 seconds"
	ExpiresByType text/xml                  "access plus 0 seconds"

	# Feed
	ExpiresByType application/atom+xml      "access plus 1 hour"
	ExpiresByType application/rss+xml       "access plus 1 hour"

	# Favicon (cannot be renamed)
	ExpiresByType image/x-icon              "access plus 1 week"

	# Media: images, video, audio
	ExpiresByType audio/ogg                 "access plus 1 month"
	ExpiresByType image/gif                 "access plus 1 month"
	ExpiresByType image/jpeg                "access plus 1 month"
	ExpiresByType image/png                 "access plus 1 month"
	ExpiresByType video/mp4                 "access plus 1 month"
	ExpiresByType video/ogg                 "access plus 1 month"
	ExpiresByType video/webm                "access plus 1 month"

	# HTC files  (css3pie)
	ExpiresByType text/x-component          "access plus 1 month"

	# Webfonts
	ExpiresByType application/vnd.ms-fontobject "access plus 1 month"
	ExpiresByType application/x-font-ttf    "access plus 1 month"
	ExpiresByType application/x-font-woff   "access plus 1 month"
	ExpiresByType font/opentype             "access plus 1 month"
	ExpiresByType image/svg+xml             "access plus 1 month"

	# CSS and JavaScript
	ExpiresByType application/javascript    "access plus 1 year"
	ExpiresByType text/css                  "access plus 1 year"

</IfModule>

# ----------------------------------------------------------------------
# Built-in filename-based cache busting
# ----------------------------------------------------------------------
# <IfModule mod_rewrite.c>
#   RewriteCond %{REQUEST_FILENAME} !-f
#   RewriteCond %{REQUEST_FILENAME} !-d
#   RewriteRule ^(.+)\.(\d+)\.(js|css|png|jpg|gif)$ $1.$3 [L]
# </IfModule>
```

#### Cache Busting
Traditionally, if you use a far future Expires header you have to change the component's filename whenever the component changes.

The snippet above has built-in filename cache busting. To use it, uncomment the relevant lines.

Doing so will route all requests for /path/filename.20120101.ext to /path/filename.ext. To use this, just add a time-stamp number (or your own numbered versioning system) into your resource filenames in your HTML source whenever you update those resources.

#### Example:

```html
<script src="/js/myscript.20120305.js"></script>
<script src="/js/jqueryplugin.45.js"></script>
<link rel="stylesheet" href="css/somestyle.49559939932.css">
<link rel="stylesheet" href="css/anotherstyle.2.css">
```

You do not have to rename the resource on the filesystem. All you have to do is add the timestamp number to the filename in your HTML source. The `.htaccess` directive will serve up the proper file.


### Prevent Mobile Network Provider Modifications
Prevents modification of your code over 3G on some European providers.

```ApacheConf
# ----------------------------------------------------------------------
# Prevent mobile network providers from modifying your site
# ----------------------------------------------------------------------

# The following header prevents modification of your code over 3G on some
# European providers.
# This is the official 'bypass' suggested by O2 in the UK.

<IfModule mod_headers.c>
	Header set Cache-Control "no-transform"
</IfModule>
```


### ETag Removal
Entity tags (ETags) are a mechanism that web servers and browsers use to determine whether the component in the browser's cache matches the one on the origin server.

Learn more on the [Yahoo! Developer Network](http://developer.yahoo.com/performance/rules.html#etags).

```ApacheConf
# ----------------------------------------------------------------------
# ETag removal
# ----------------------------------------------------------------------

# FileETag None is not enough for every server.
<IfModule mod_headers.c>
	Header unset ETag
</IfModule>

# Since we're sending far-future expires, we don't need ETags for
# static content.
# developer.yahoo.com/performance/rules.html#etags
FileETag None
```


### UTF-8 Encoding

```ApacheConf
# ----------------------------------------------------------------------
# UTF-8 encoding
# ----------------------------------------------------------------------

# Use UTF-8 encoding for anything served text/plain or text/html
AddDefaultCharset utf-8

# Force UTF-8 for a number of file formats
AddCharset utf-8 .atom .css .js .json .rss .vtt .xml
```


### A Little Extra Security
Prevent people from accessing stuff on your server that they shouldn't.

```ApacheConf
# ----------------------------------------------------------------------
# A little more security
# ----------------------------------------------------------------------

# "-Indexes" will have Apache block users from browsing folders without a
# default document Usually you should leave this activated, because you
# shouldn't allow everybody to surf through every folder on your server (which
# includes rather private places like CMS system folders).
<IfModule mod_autoindex.c>
	Options -Indexes
</IfModule>

# Block access to "hidden" directories or files whose names begin with a
# period. This includes directories used by version control systems such as
# Subversion or Git.
<IfModule mod_rewrite.c>
	RewriteCond %{SCRIPT_FILENAME} -d [OR]
	RewriteCond %{SCRIPT_FILENAME} -f
	RewriteRule "(^|/)\." - [F]
</IfModule>

# Block access to backup and source files. These files may be left by some
# text/html editors and pose a great security danger, when anyone can access
# them.
<FilesMatch "(\.(bak|config|dist|fla|inc|ini|log|psd|sh|sql|swp)|~)$">
	Order allow,deny
	Deny from all
	Satisfy All
</FilesMatch>

# Increase cookie security
<IfModule mod_php5.c>
	php_value session.cookie_httponly true
</IfModule>
```


## How To Contribute

In lieu of a formal style guide, take care to maintain the existing coding style.


## License

Licensed under the [MIT License](http://gomakethings.com/mit/).


## Changelog

* v1.0 - February 9, 2013
	* Initial release.