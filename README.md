# clearpass-api-python

This directory contains sample Python code demonstrating how to interact with
the ClearPass REST API.

A test driver (cpapi.py) is included that allows command-line usage of the
API.


# Quick Start

NOTE: Python 2.7 is recommended. Python 3 has not been tested with these
instructions.

Create a Python virtual environment:

	$ curl -O https://pypi.python.org/packages/source/v/virtualenv/virtualenv-13.1.2.tar.gz
	$ tar -zxf virtualenv-13.1.2.tar.gz
	$ python virtualenv-13.1.2/virtualenv.py cpapi

Activate the venv:

	$ source cpapi/bin/activate

Install the dependencies:

	$ pip install docopt==0.6.2
	$ pip install requests==2.8.0

Run the script:

	$ ./cpapi.py --help


# OAuth2 Authentication

To use the REST API, first create an API client using the ClearPass Admin UI:

* Navigate to Guest > Administration > API Services > API Clients
* Create a new API Client
* Select the appropriate operator profile
* Select the `client_credentials` grant type
* Make a note of the client ID and client secret

To obtain an access token, use a command such as the following:

	$ ./cpapi.py --host clearpass.example.com -z POST /oauth \
	    grant_type=client_credentials \
	    client_id=Client1 \
	    client_secret=TcErfNRQ4e1g4wWg/YotZH5lktAVDgIJYDKshW4A2ysA

This should produce a result such as:

	{
	    "access_token": "9d368230c61fbe6e505e6da3e55447a401b47bf2",
	    "expires_in": 28800,
	    "scope": null,
	    "token_type": "Bearer"
	}

To verify the access token, perform an API request using it:

	$ ./cpapi.py --host clearpass.example.com GET /oauth/me \
	    --access-token 9d368230c61fbe6e505e6da3e55447a401b47bf2

HTTP 403 Forbidden will be returned if the token is invalid or expired.

This is the basic `client_credentials` authorization grant type.  Other grant
types are possible with OAuth2; for details, refer to RFC 6749.

For convenience, you can save the access token in the environment
variable `access_token`:

    $ export access_token=9d368230c61fbe6e505e6da3e55447a401b47bf2
    $ ./cpapi.py --host clearpass.example.com GET /oauth/me


# API Usage

Start with this Python import statement:

	from clearpass import api

You can then create an api.Client object to make API calls:

	access_token = '9d368230c61fbe6e505e6da3e55447a401b47bf2'
	client = api.Client(host='clearpass.example.com', access_token=access_token)

The client provides methods for GET, POST, PATCH, PUT, and DELETE requests:

	# GET method:
	result = client.get('/oauth/me')
	# result['info'] and result['name'] will now be set

	# POST method:
	user = {'username': 'demo@example.com', 'password': '123456'}
	result = client.post('/guest', user)
	# result will contain guest account properties

Errors will generate an exception.  This may be an exception from the
underlying requests library (e.g. `requests.ConnectionError` or
`requests.ConnectTimeout`), or an `api.Error` object containing details of the
error.

	# PATCH generating a validation error:
	try:
		result = client.patch('/guest/' + result['id'], {'username': ''})
	except api.Error, e:
		print 'Result: ', e.code, e.message
		print 'Details: ', e.details

APIs are not wrapped or bundled; refer to the ClearPass API Explorer for
details on what API to call and what parameters are expected.
