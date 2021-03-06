---
title: PyMacaron API definition
---

API definition with swagger
===========================

PyMacaron uses the OpenAPI format to define microservice APIs, formerly known
as the swagger format.

PyMacaron relies on
[pymacaron-core](https://github.com/pymacaron/pymacaron-core) to generate the
Flask app running your microservice API.

You will need to know how to write an OpenAPI specification before getting
started. See for example [the OpenAPI
specification](https://swagger.io/specification/) and these
[examples](https://github.com/OAI/OpenAPI-Specification/tree/master/examples/v3.0).

## Custom OpenAPI attributes

[pymacaron-core](https://github.com/pymacaron/pymacaron-core) extends the OpenAPI standard with the following custom attributes when implementing the OpenAPI specification (ie on the server):

* 'x-parent: path.to.parent.class': let the model associated to an OpenAPI schema object inherit from a custom class. 
* 'x-bind-server: path.to.endpoint.method': in an endpoint definition, tell which python method implements that endpoint.
* 'x-decorate-server: pymacaron.auth.requires_auth': path to a decorator to wrap around this endpoint that will check for a valid JWT token.

And when using the specification to generate an API client:

* 'x-bind-client: name_of_client_method': name of the method calling this endpoint as a client.
* 'x-decorate-request: pymacaron.auth.add_auth': path to a decorator that adds the authentication header around the client method.


## Binding an API endpoint to a Python method

For every API endpoint in your microservice, you need to tell PyMacaron which
python method to execute. You do that with the 'x-bind-server' attribute. Below
an example of a fictional login endpoint:

```yaml
/login:
  post:
    summary: Login a user.
    produces:
      - application/json
    x-bind-server: myserver.handlers.do_login
    parameters:
      - in: body
        name: body
        description: User login credentials.
        required: true
        schema:
          $ref: "#/definitions/Credentials"
    responses:
      200:
        description: API version
        schema:
          $ref: '#/definitions/TokenAndProfile'
      default:
        description: Error
        schema:
          $ref: '#/definitions/Error'
```

Your login method could look like:

```python

    from pymacaron import get_model
    from pymacaron.exceptions import PyMacaronException

    def do_login(credentials):
        if not authenticate_user(credentials):
            raise PyMacaronException("Login failed")

        # Get the class representing bravado-core Welcome objects
        return get_model('LoginResponse')(
            token=generate_user_token(),
            profile=get_user_profile(),
        )

```

## Authentication

To require JWT authentication on some of your API endpoints, you need to set the
'x-decorate-request' and 'x-decorate-server' attributes for those endpoints, as
described [here](jwt.html).

## Builtin endpoints

All PyMacaron services implement a few default endpoints defined in this [OpenAPI specification](https://github.com/pymacaron/pymacaron/blob/master/pymacaron/ping.yaml). Those are:

* '/ping': Returns a json dictionary showing the status of the server. Convenient as a health-check endpoint. Does not require authentication.

* '/version': Returns the version of the running server when deployed in a Docker container. Does not require authentication.

* '/auth/version': Same as '/version' but requiring a valid JWT token in the HTTP Authorization header.


## Mandatory Error object

PyMacaron requires that you define an Error object in your microservice API. It should
look as follows:

```yaml
  Error:
    type: object
    description: An api error
    properties:
      status:
        type: integer
        format: int32
        description: HTTP error code.
      error:
        type: string
        description: A unique identifier for this error.
      error_description:
        type: string
        description: A humanly readable error message in the user''s selected language.
    required:
      - status
      - error
      - error_description
    example:
      status: 500
      error: SERVER_ERROR
      error_description: Expected data to send in reply but got none
```
