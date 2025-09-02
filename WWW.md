# Web hosting on cs.purdue.edu

First see this [help page](./Help pages.md): [Creating personal homepage][hp].

[hp]: https://www.cs.purdue.edu/resources/facilities/help/Creating-personal-homepage.html

Undergraduate students don't have a "directory page," but you do have a "personal
page."

## WWW server

While your [home directory](./Home symlink.md) is on an NFS drive and is mounted
on all of the CS servers, only one of them hosts the web server. At the time of
writing, this is `pythia.cs.purdue.edu`:
```
$ dig +short www.cs.purdue.edu
pythia.cs.purdue.edu.
128.10.19.120
```

## More advanced features

The web server running on pythia.cs is [Apache HTTPd]. It's using [mod_userdir
to serve per-user directories][mod_userdir]. See the *Allowing users to alter
configuration* section of that page; it mentions that we can use `.htaccess`
files to modify web server configuration for our directory. (The help pages
[also mention this][password].)

For example, you can disable directory listings so clients can access files in
a directory but not list the contents (like removing the read permission from
a UNIX directory):
```apache
# Don't provide directory listings
Options -Indexes
```

See [the tutorial on `.htaccess` files][.htaccess] for more information on what
you can do with these files.

[Apache HTTPd]: https://httpd.apache.org/
[mod_userdir]: https://httpd.apache.org/docs/2.4/howto/public_html.html
[.htaccess]: https://httpd.apache.org/docs/2.4/howto/htaccess.html
[password]: https://www.cs.purdue.edu/resources/facilities/help/Password-Protected-Directories.html

### Reverse proxy

You can use your personal page as a reverse proxy using a configuration like
this:
```apache
# Enable rewrite rules
RewriteEngine On
# Set the base path that rewrites will match relative to
RewriteBase "/homes/kkasad/"
# Redirect the path to always include a trailing slash
RewriteRule "^flip$" "/homes/kkasad/flip/"
# Reverse proxy to the given site
RewriteRule "^flip/(.*)$" "http://data.cs.purdue.edu:20202/$1" [proxy]
```

> [!NOTE]
> Remember that the WWW server is running on `pythia.cs`, so you can't use
> loopback addresses like `localhost` or `127.0.0.1`. Make sure your upstream
> service is listening on a non-local IP (like `0.0.0.0`).
