---
title: Server and Endpoints
---

Implementing your API
=====================


## Server template

A standard PyMacaron 'server.py' looks like:

```python
import os
import sys
import logging
from flask import Flask
from flask_cors import CORS
from pymacaron import API, letsgo


log = logging.getLogger(__name__)


# WARNING: you must declare the Flask app as shown below,
# keeping the variable name 'app' and the file name 'server.py',
# since gunicorn is configured to lookup the variable 'app'
# inside the code generated from 'server.py'.

app = Flask(__name__)
CORS(app)
# Here you could add custom routes, etc.


def start(port=80, debug=False):

    # Your swagger api files are under ./apis, but you could
    # have them anywhere else really.

    here = os.path.dirname(os.path.realpath(__file__))
    path_apis = os.path.join(here, "apis")

    # Tell pymacaron to spawn apis inside this Flask app. Set the
    # server's listening port, set Flask debug mode on or not. Other
    # configuration parameters, such as JWT issuer, audience and
    # secret, are set in 'pym-config.yaml'.

    api = API(
        app,
        port=port,
        debug=debug,
    )

    # Find all swagger files and load them into pymacaron-core

    api.load_apis(path_apis)

    # Optionally, publish the apis' specifications under the
    # '/doc/<api-name>' route, so you may open them in Swagger-UI:
    # api.publish_apis()

    # Start the Flask app and serve all endpoints defined in
    # 'apis/myservice.yaml'

    api.start(serve="myservice")


# Entrypoint
letsgo(__name__, callback=start)
```

## Start the server

You start your server by going into the project's root directory and doing:

```bash
python server.py --port 8080
```