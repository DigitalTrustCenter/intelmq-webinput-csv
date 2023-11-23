**Table of Contents**

1. [Requirements](#requirements)
2. [Installation](#installation)
   * [With pip](#with-pip)
     * [From PyPi](#from-pypi)
     * [From the repository](#from-the-repository)
3. [Afterwards](#afterwards)


Please report any errors you encounter at https://github.com/certat/intelmq-webinput-csv/issues

For upgrade instructions, see [UPGRADING.md](UPGRADING.md).

# Requirements

* An installed python3 [Flask](http://flask.pocoo.org/)
* An installed [intelmq](https://intelmq.org) installation on the same machine.

# Installation

Please note that the pip3 installation method does not (and cannot) create /opt/intelmq/etc/examples/webinput_csv.conf.
As workaround you need to move the file from the site-packages directory to /opt/intelmq manually.
The location of this directory varies, it could be `/usr/lib/python3.4/site-packages`, `/usr/local/lib/python3.5/dist-packages/` or similar.

## From PyPi

```bash
sudo -s

pip3 install intelmq-webinput-csv
```

## From the repository

Clone the repository if not already done:
```bash
git clone https://github.com/certat/intelmq-webinput-csv.git
```

If you have a local repository and you have or will do local modification, consider using an editable installation (`pip install -e .`).
```
pip3 install .
```

### File permissions

The backend will write temporary (CSV) files in the IntelMQ directory VAR_STATE_PATH (default value /opt/intelmq/var/lib/intelmq/bots). Therefore the webserver running IntelMQ Webinput CSV needs to have read and write access to that directory.  
You can change the used directory by adding VAR_STATE_PATH to the webinput_csv.conf file:
```json
{
	"destination_pipeline_queue": "...",
	"VAR_STATE_PATH": "/other/directory"
}
```

### Webserver configuration and permissions

The 0.4.0+ version of IntelMQ Webinput CSV uses [Flask-SocketIO](https://flask-socketio.readthedocs.io) for asynchronous functionality, for Flask-SocketIO deployement options see their [wiki](https://flask-socketio.readthedocs.io/en/latest/deployment.html).
  
An example of running gunicorn with the eventlet worker class:
```bash
gunicorn --worker-class eventlet -w 1 --chdir /usr/local/lib/python3.11/site-packages/ intelmq_webinput_csv.app:app --bind=0.0.0.0:8000
```
Change the site-packages path to the directory where your python/pip installation installs packages.
  
You can use nginx for example as reverse proxy. Note that Flask-SocketIO uses websockets, so we need to add an extra configuration block for the socket.
```
location /socket.io {
	proxy_set_header Host $http_host;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_set_header X-Forwarded-Proto $scheme;
	proxy_http_version 1.1;
	proxy_buffering off;
	proxy_set_header Upgrade $http_upgrade;
	proxy_set_header Connection "Upgrade";
	proxy_pass http://127.0.0.1:8000;
}

location / {
	proxy_set_header Host $http_host;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_set_header X-Forwarded-Proto $scheme;
	proxy_buffering off;

	# Ignore the default 1MB upload limit.
	client_max_body_size 0;
	
	proxy_pass http://127.0.0.1:8000;
}
```
  
IntelMQ Webinput CSV will redirect to / on errors, so if you run under a subdirectory, you must add a `proxy_redirect / /your-directory/;` statement to the second location block.

# Afterwards

Now continue with the [User Guide](User-Guide.md).
