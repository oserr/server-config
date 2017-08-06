# Server Config

A brief description of server configurations to setup a an Apache-PostgreSQL web server on [AmazonLightsail][1] for a [Udacity][2] project.

## Browsing the app

```
http://34.212.34.224/
```

## Accessin the app via ssh

```bash
ssh -p 2200 -i grader_key grader@34.212.34.224
```

## Software installed

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install postgres apache2 libapache2-mod-wsgi python-pip
sudo pip install --upgrade pip
sudo pip install oauth2client psycopg2 SQLAlchemy Flask
```

Before installing the python packages with `pip`, I installed anaconda so I could use the `conda` environment I used in my catalog project, however, I ran into an error installing `mod_wsgi` within the conda environment, so I decided to give `pip` a try.

## Configuration changes made

* created user *omar*: `sudo adduser --disabled-password omar`
    * added my public ssh key to the list of authorized keys
    * granted sudo permissions to *omar*
* created user *grader*: `sudo adduser --disabled-password grader`
    * created ssh key pair for *grader* and added the public key to the list of authorized keys
    * granted sudo permissions to *grader*
* modified `/etc/ssh/sshd_config`:

    | Setting                | From              | To  |
    | ---------------------- | ----------------- | --- |
    | Port                   | 22                | 2200|
    | PasswordAuthentication | yes               | no  |
    | PasswordRootLogin      | prohibit-password | no  |

* restarted ssh service: `sudo service ssh restart`
* created database *itemcatalog*: `sudo -u postgres createdb itemcatalog`
* created database user *catalog*: `sudo -u postgres createuser --pwprompt catalog`
* configured firewall with `ufw`
    ```bash
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw allow 2200/tcp
    sudo ufw allow http
    sudo ufw allow ntp
    sudo ufw enable
    sudo ufw status
    ```
* configured firewall ports in AmazonLightsail console to match settings configured via `ufw`
* cloned git repo `https://github.com/oserr/itemcatalog.git` to `/var/www/html`
* created *app.wsgi* in `/var/www/html/itemcatalog`:
    ```python
    import sys
    sys.path.insert(0, '/var/www/html/itemcatalog')
    from project import app as application
    ```
* modified the user and database arguments to `create_engine`
* created file *client_secret.json* in `/var/www/html/itemcatalog`
* modified the path to *client_secret.json* in *project.py* to be absolute
* added line `WSGIScriptAlias / /var/www/html/itemcatalog/app.wsgi` to `/etc/apache2/sites-available/000-default.conf`
* restarted apache2: `sudo apache2ctl restart`

## Third party resources

* [Stack Overflow][3]
* [PostgresSQL][4] [Documentation][5]
* [Apache HTTP Server][6]
* [Flask][7]
* linux `man` pages
* apache2 error log: `/var/log/apache2/error.log`

[1]: https://amazonlightsail.com/
[2]: https://www.udacity.com/
[3]: https://stackoverflow.com/
[4]: https://www.postgresql.org/
[5]: https://www.postgresql.org/docs/current/static/index.html
[6]: https://httpd.apache.org/
[7]: http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi
