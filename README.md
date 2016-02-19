# Introduction

Thank you for participating in Okta's OpenID Connect (OIDC) Beta.

In this document, we will show you how to get started using OpenID
Connect with Okta. 

Specifically demonstrated are the following two use-cases:

1.  Using OpenID Connect to authenticate Okta users to a backend
    web server (If your familiar with SAML, this is roughly
    equivalent to "SP-initiated SAML").
2.  Using OpenID Connect with a JavaScript Single Page Application.

If you have any questions, comments, or suggestions for this
document please contact Joël Franusic <joel.franusic@okta.com>

# How to run this demo

This repository comes with code that you can run yourself to see how
OIDC works. You can run this code locally on your machine, or you
can deploy the code to Heroku.

## Prerequisites

All examples in this guide assume that you have an Okta org, API
token, and Okta OAuth Client ID. 

Running the code samples locally will require the use of Python and
the [pip](https://en.wikipedia.org/wiki/Pip_%28package_manager%29) package manager.

Here is how to get those things if you do not have them already:

### Okta org

If you do not have an Okta org, you can [sign up for a free Developer
Edition Okta org](https://www.okta.com/developer/signup/).

### Okta API token

If you do not have an API token, follow our guide for
[getting a token](http://developer.okta.com/docs/api/getting_started/getting_a_token.html).

### Okta OAuth Client ID


At the moment, the only way to register an OAuth client with Okta
is via Okta's `/oauth2/` API endpoint.

The recommended method of doing this is via [Postman](http://developer.okta.com/docs/api/getting_started/api_test_client.html), using this
Postman Collection for Okta's [Client Registration API endpoint](https://beta.getpostman.com/collections/2bece1641e75a7d4a222).

Use the "Create OAuth Client" template in Postman, replacing data in the
sections as appropriate for your situation. The most important
value to change in the example JSON payload is the array value for the
`redirect_uris` key. 

It is important that the `redirect_uris` array contains the URL
for the site that will be making requests against Okta. If you're
using the instructions below, this URL will look like
"`https://abc123de4.ngrok.io`" or
"`https://example.herokuapp.com`".

Don't worry if you don't yet know the URL that you'll be using
here, you can always update the value of `redirect_uris` using an
HTTP PUT.

It isn't required, but we suggest that
you also change the values for the `client_name`, `client_uri`, `logo_uri`, and `jwks_uri` keys.

Here is an example HTTP request to create an Okta OAuth client via
the `/oauth2/` API endpoint:

    POST /oauth2/v1/clients HTTP/1.1
    Host: example.okta.com
    Accept: application/json
    Content-Type: application/json
    Authorization: SSWS 01A2bcDefGhI34JKlmnOp5qRstUVWXy6ZABCdefgHi
    
    {
        "client_name": "Example Okta OAuth Client",
        "client_uri": "https://example.com",
        "logo_uri": "https://static.example.com/logo.png",
        "redirect_uris": [
             "https://example.com/oauth/callback1",
             "https://example.com/oauth/callback2"
        ],
        "response_types": [
             "code",
             "token",
             "id_token"
        ],
        "grant_types": [
             "authorization_code",
             "implicit"
        ],
        "token_endpoint_auth_method": "private_key_jwt",
        "jwks_uri": "https://static.example.com/certs/public.jwks"
    }

### Python

While the code samples in this guide are written in Python, you do
not need Python to use OpenID Connect with Okta.

To run the code samples in this project you will need a working
copy of Python 2.7+ and the pip package manager. See this guide on
"[Properly Installing Python](http://docs.python-guide.org/en/latest/starting/installation/)"  for instructions for setting up
Python on your system.

## Setup

Once you have all the prerequisites, the next step will be do get
this example code running. You can either run this code from your
local machine, or by running the code from Heroku.

## Make a local copy of this repository

Before you can make use of the code in this guide, you will need a
local copy of the code to work from. You can either download a copy
of this repository using the "Download ZIP" button or using the `git
   clone` command.

Using `git clone` is the suggested method for making a local copy of
this repository, because getting updates to this repository will be
as easy as running `git pull origin master`.

## Running on your local machine

With a local copy of this repository on your machine, the next step
will be to set up the project.

You can do this on Mac OS X and Linux by running these commands from the shell:

    $ virtualenv venv
    $ source venv/bin/activate
    $ pip install -r requirements.txt

If you are using Homebrew on OS X, you *might* need to follow the
[Homebrew specific installation instructions](http://cryptography.readthedocs.org/en/latest/installation/#building-cryptography-on-os-x) to install the Python `cryptography` library:

    $ env CRYPTOGRAPHY_OSX_NO_LINK_FLAGS=1 LDFLAGS="$(brew --prefix openssl)/lib/libssl.a $(brew --prefix openssl)/lib/libcrypto.a" CFLAGS="-I$(brew --prefix openssl)/include" pip install cryptography

On OS X or Linux, **replace the example values in the commands below
with your data** and then run the modified commands in your shell to
configure the application:

    $ export OKTA_API_TOKEN=00A0B12CDefGHijkLmN3OPQRsTu4VWxyzABCdEf56G
    $ export OKTA_BASE_URL=https://example.okta.com
    $ export OKTA_CLIENT_ID=aBcDEfG0HiJkL1mn2oP3

Use this command to run the application locally on your system:

    $ python app.py

### Make the example available via HTTPS using ngrok

As a last step, you will need to make your local copy of the
example code available via HTTPS. You need to do this because the
[OpenID Connect specification requires that you do so](http://openid.net/specs/openid-connect-core-1_0.html#ImplicitAuthRequest). 

The easiest way to do this is using the excellent tool "[ngrok](https://ngrok.com/)".

To get started with ngrok, visit the
["Download" page for ngrok](https://ngrok.com/download), download ngrok, then start it on your
system.

Assuming that your example code is listening on
`http://localhost:5000`, start ngrok with the following command:

    $ ngrok http 5000

When ngrok starts, you will see a page that give you information
on the ngrok. Look for the line that starts with **Forwarding** and
then copy the URL that starts with "https", it will look something
like this: `https://ab123cd4.ngrok.io` - this is the URL that you
will use in the following steps.

## Running on Heroku

Assuming that you've already installed the
[Heroku Toolbelt](https://toolbelt.heroku.com/), here are the commands you'd use to deploy this
application to Heroku:

    $ heroku create
    $ git push heroku master

Then, configure the application using these commands below. 
**Make sure to replace the values below with your data!**

    $ heroku config:set OKTA_API_TOKEN=00A0B12CDefGHijkLmN3OPQRsTu4VWxyzABCdEf56G
    $ heroku config:set OKTA_BASE_URL=https://example.okta.com
    $ heroku config:set OKTA_CLIENT_ID=aBcDEfG0HiJkL1mn2oP3

Finally:

    $ heroku open

## Whitelist URL in Okta

The last thing that you will need to do is add the URL for your
example application to the appropriate Okta whitelists. This is
done in two places: 

1.  The OAuth client configuration in your Okta org
2.  The CORS settings in your Okta org

If you're using ngrok or Heroku to host your example application,
then your URL will look like this "`https://abc123de4.ngrok.io`" or
 "`https://example.herokuapp.com`".

### Update the OAuth Client `redirect_uris` array

If you didn't do it when you created your OAuth Client ID (See section 2.1.3), you
will need to go back to that section and follow the instructions
to add your URL to the `redirect_uris` whitelist.

### Update CORS configuration on the Okta web page

You will also need to enable the URL for CORS. See 
[Okta's guide to Enabling CORS](http://developer.okta.com/docs/api/getting_started/enabling_cors.html) for details on how to do this.

## Open the URL for the example application in your browser

If you're using ngrok or Heroku to host your example application,
then your URL will look like this "`https://abc123de4.ngrok.io`" or
 "`https://example.herokuapp.com`".

# How it works

The core of using Open ID Connect with your application is the
`id_token`, which is a JSON Web Token (JWT).

Below is an example of what a JWT looks like:

    eyJhbGciOiJSUzI1NiJ9.eyJ2ZXIiOjEsImlzcyI6Imh0dHBzOi8vZXhhbXBsZS5va3RhLmNvbSIsIn
    N1YiI6IjAwdTBhYmNkZWZHSElKS0xNTk9QIiwibG9naW4iOiJ1c2VybmFtZUBleGFtcGxlLmNvbSIsI
    mF1ZCI6IkFiY0RFMGZHSEkxamsyTE0zNG5vIiwiaWF0IjoxNDQ5Njk1NjAwLCJleHAiOjE0NDk2OTky
    MDAsImFtciI6WyJwd2QiXSwiYXV0aF90aW1lIjoxNDQ5Njk1NjAwfQ.btq43W2-SOsc7BA_SyMPEKcu
    2xUYoyLuY948k6tWzZAsy__MndK9pX3WjYYMwkGqfthLjMWXMuYem2-uWcdwfDCDpWoxK4Es3N8dnsQ
    NeS_U0_FfVZfkj_OMGw28RPDLRErNAuyXFj2DegXUh74PEZcDaKSz5-17znEpXgzbT14

**Note:** The line breaks have been added for readability.

A JWT is, essentially, a base64 encoded JSON object. Here is what
the JWT above looks like after it has been decoded and validated:

    {
      "ver": 1,
      "iss": "https://example.okta.com",
      "sub": "00u0abcdefGHIJKLMNOP",
      "login": "username@example.com",
      "aud": "AbcDE0fGHI1jk2LM34no",
      "iat": 1449695600,
      "exp": 1449699200,
      "amr": [
        "pwd"
      ],
      "auth_time": 1449695600
    }

# Getting an id\_token from Okta

The easiest way to get an `id_token` from Okta is to use the Okta
Sign-In Widget. Here is how to configure the Okta Sign-In Widget
to give you an `id_token`:

    function setupOktaSignIn(baseUrl, clientId) {
        var oktaSignIn = new OktaSignIn({
            baseUrl: baseUrl,
            clientId: clientId,
            authParams: {
                responseType: 'id_token',
                responseMode: 'okta_post_message',
                scope: ['openid']
            }
        });
        return oktaSignIn;
    };

Note: Other valid types for `authParams.scope` are: `openid`,
`email`, `profile`, `address`, `phone`. 

# Use cases

The OpenID Connect specification makes provisions for many different
use cases. For this beta, we are support two use cases:

1.  Server-side web application
    Authenticating against a web application that runs on a server.
2.  Single Page Application
    Authenticating a client-side JavaScript application that runs in
    a web browser.

## Server-side web application

This use case demonstrates how to have a server-side web
application authenticate users via OpenID Connect. If you are
familiar with SAML, this is the same use case as "SP initiated
SAML".

Authenticating Okta users against your server-side web application
consists of these core steps:

1.  Okta authenticates a user.
2.  Upon a successful authentication, Okta issues the user an OIDC
    `id_token` and direct the users browser to deliver the
    `id_token` to your web application.
3.  Your server-side web application will validate the `id_token`
    and, if the token is valid, will create a session for the user
    so that the user is "logged in" to your web application.

Step 2 is covered below in the "Getting an OIDC `id_token` from
Okta" section and Step 3 is covered in the "Validating an OIDC
`id_token` from Okta" section.

### Getting an OIDC `id_token` from Okta

Currently, there are three ways to get an `id_token` from Okta,
sorted in order if "ease of implementation":

1.  Having users click on a special link that will redirect them
    through Okta.
2.  Authenticating users via the Okta Sign-In Widget.
3.  Authenticating users via [/authn](http://developer.okta.com/docs/api/resources/authn.html) and [/oauth2](http://developer.okta.com/docs/api/resources/oidc.html) Okta API endpoints.

Which method you select depends on how customized you want the
user's login experience to be. 

If you don't care about a customized login experience, the easiest
way to get an `id_token` from Okta is to have users click on a
special link that will redirect them through Okta to your
application.

The Okta Sign-In Widget handles all possible user states and is
moderately customizable. It is a good choice if you don't have
extremely detailed design requirements.

Using the Okta API endpoints directly gives you the most
flexibility in terms of customization at the expense of requiring
you to support all of the possible flows that your users will go
through.

Details on each of these methods are below:

### Getting an `id_token` via a special Okta link

If you don't mind your users seeing an Okta branded login page,
having your users login to your application using the OpenID
Connect "Code Flow".

The basics of implementing the Code Flow are below. For more
information in the Code Flow, we suggest reading the "OpenID Connect Basic Client
Implementer's Guide", which contains a good [guide to implementing the
OIDC Code Flow](https://openid.net/specs/openid-connect-basic-1_0.html#CodeFlow).

See our [OIDC documentation for details on the request parameters](http://developer.okta.com/docs/api/resources/oidc.html#request-parameters)
for more details on how Okta uses the OIDC request parameters.

Below is an example of what this link might look like:

    https://example.okta.com/oauth2/v1/authorize?redirect_uri=https%3A%2F%2Fexample.com%2Fsse%2Foidc&response_type=id_token&client_id=a0bcdEfGhIJkLmNOPQr1&scope=openid&response_mode=form_post

### Getting an `id_token` via the Okta Sign-In Widget

The easiest way customize the login experience that your users
see is to use the [Okta Sign-In Widget](http://developer.okta.com/docs/guides/okta_sign-in_widget.html).

To use the Okta Sign-In Widget with your application, follow the
[guide for setting up the Okta Sign-In Widget](http://developer.okta.com/docs/guides/okta_sign-in_widget.html) but make the
following two changes to your configuration of the Okta Sign-In Widget:

1.  Configure the Sign-In Widget to request an OIDC `id_token`:
    
        var oktaSignIn = new OktaSignIn({
            baseUrl: baseUrl,
            clientId: clientId,
            authParams: {
                responseType: 'id_token',
                responseMode: 'okta_post_message',
                scope: ['openid']
            }
        });
2.  Add a "SUCCESS" handler to the widget which will extract the
    `id_token` and pass it on to your application backend service.
    
    Here is how this is done in the example application in this project:
    
        oktaSignIn.renderEl(
          { el: '#okta-sign-in-widget' },
         function (res) {
            console.log(res);
            var id_token = res.id_token || res.idToken;
            if (res.status === 'SUCCESS') {
              $.post("/sso/oidc", {"id_token": id_token}, function(data) {
                window.location.href="/secret";
              });
            }
          },
         function (err) { console.log('Unexpected error authenticating user: %o', err); }
        );

### Getting an `id_token` via Okta API endpoints

Lastly, if you need to make customizations to the login
experience beyond what the Sign-In Widget allows, you can do that
by making API requests directly to the Okta API.

At a high level, what you will need to do is write some code on
your application backend that will do the following:

-   Accepts a **username** and **password**
-   Uses the **username** and **password** to make a request to Okta's
    `/authn` API endpoint and extracts the `sessionToken` from the
    results of a successful request.
-   Redirects the user to an Okta's `/oauth2/v1/authorize` API
    endpoint using the `sessionToken` in the request parameters.

Here is how this is done in the example application:

    @app.route("/login", methods=['POST'])
    def login_with_password():
        payload = {
            'username': request.form['username'],
            'password': request.form['password'],
            }
    
        authn_url = "{}/api/v1/authn".format(okta['base_url'])
        r = requests.post(authn_url, headers=headers, data=json.dumps(payload))
        result = r.json()
    
        redirect_uri = url_for(
            'sso_oidc',
            _external=True,
            _scheme='https')
        redirect_url = create_authorize_url(
            base_url=okta['base_url'],
            sessionToken=result['sessionToken'],
            client_id=okta['client_id'],
            scope='openid',
            response_type='id_token',
            response_mode='form_post',
            redirect_uri=redirect_uri,
            )
        return redirect(redirect_url)

And here is the `create_authorize_url` function that is used to
construct the request to `/oauth2/v1/authorize` with the proper
request parameters:

    def create_authorize_url(**kwargs):
        base_url = kwargs['base_url']
        del(kwargs['base_url'])
        redirect_url = "{}/oauth2/v1/authorize?{}".format(
            base_url,
            urllib.urlencode(kwargs),
        )
        return redirect_url

### Validating an OIDC `id_token` from Okta

An OIDC `id_token` is a JWT and validating a JWT is easy. Below is a
demonstration of how to validate a JWT in Python using the [pyjwt](https://github.com/jpadilla/pyjwt#pyjwt)
Python library.

(See [JWT.io](http://jwt.io/#libraries-io) for a list of JWT libraries in your favorite language.)

The [pyjwt](https://github.com/jpadilla/pyjwt#pyjwt) library handles a lot of ancillary JWT validation by
default. In particular, it validates the `audience` attribute,
which means that it will return an error unless the value
`audience` attribute matches what we pass into this method.

    def parse_jwt(id_token):
        rv = jwt.decode(
            id_token,
            public_key,
            algorithms='RS256',
            audience=okta['client_id'])
        return rv

Note that we are setting two specific parameters to `parse_jwt`:

1.  Force the JWT signing algorithm to `RS256`
    
    This line forces the JWT signing algorithm to `RS256`:
    
        algorithms='RS256',
    
    We do this because it is a best practice for handling JWTs and
    is done to avoid [critical vulnerabilities in JSON Web Token libraries](https://www.chosenplaintext.ca/2015/03/31/jwt-algorithm-confusion.html).

2.  The OAuth Client ID
    
    This line sets the audience to the value of the Okta OAuth
    Client ID:
    
        audience=okta['client_id'])
    
    Okta uses the OAuth Client ID as the audience in the
    `id_token` JWTs that it issues. We pass this value to `pyjwt` so
    that our JWTs are properly validated.

Where does the `public_key` come from? It is fetched from the
[Okta JSON Web Key endpoint](https://example.okta.com/oauth2/v1/keys) - which can be discovered via the
[.well-known/openid-configuration](https://example.okta.com/.well-known/openid-configuration) URL.

The code below demonstrations how to fetch a public key to validate
a JWT from Okta, using the endpoint URI defined as part of the JWK
standard.

    def fetch_jwt_public_key(base_url=None):
        if base_url is None:
            raise Exception('base_url required')
        jwks_url = "{}/oauth2/v1/keys".format(base_url)
        r = requests.get(jwks_url)
        jwks = r.json()
        x5c = jwks['keys'][0]['x5c'][0]
        pem_data = base64.b64decode(str(x5c))
        cert = x509.load_der_x509_certificate(pem_data, default_backend())
        return cert.public_key()

**Note:** The code above pulls from the URI directly. It should really be
discovering that URI from the `.well-known/openid-configuration` URL endpoint that is
used for discovery.

## Single Page App

This use case demonstrates how to have a Single Page application
authenticate users via OpenID Connect.

The code in this example is contained in two static files:
`templates/spa.html` for the HTML and `static/single-page.js` for
the application JavaScript.

The JavaScript used to demonstrate this use case is covered below:

We start with the code used to initialize the Okta Sign-In Widget
in the `spa.html` file, Note that the `{{okta.base_url}}` and
`{{okta.client_id}}` strings are place holders for the [Jinja2](http://jinja.pocoo.org/)
templating engine that Flask uses to render the `spa.html`
template.

    var oktaSignIn = setupOktaSignIn('{{okta.base_url}}', '{{okta.client_id}}');
    
    $(document).ready(function () {
        // defined in 'single-page.js'
        renderOktaWidget();
    });

The rest of the code used in this demonstration is contained in the
`single-page.js` file. 

This demonstration application is a very simplistic and
*unrealistic* implementation of a Single Page Application. Instead
of using a framework [Angular](https://angularjs.org/), [Ember](http://emberjs.com/), or [React](https://facebook.github.io/react/), this examples uses
[jQuery](https://jquery.com/) to update the page.

(Using jQuery is easier to understand, but you *should not* use jQuery
to write a production quality Single Page Application.)

The `single-page.js` file defines three functions:

-   `renderOktaWidget()`
         This handles rendering of the Okta widget.
-   `renderLogin()`
    What gets called when a user logs in with a `status` of
    "`SUCCESS`".
-   `renderLogout()`
         What gets called when a user clicks a "Logout" button or link.

We will cover each function below.

### `renderOktaWidget()`

Below is the `renderOktaWidget()` function which calls the
`renderEl` ("render El"ement) method of
`oktaSignIn`. `renderEl` takes three arguments:

1.  `widget-location-object`
    A JavaScript object which contains the `id` of the HTML element
    that should be turned into the Okta Sign-In Widget.
2.  `widget-success-function` 
           A function that is called on successful authentications.
3.  `widget-error-function`
           A function that is called when error conditions are encountered.

Here is what the `renderEl` function looks like at a high level:

    function renderOktaWidget() {
        oktaSignIn.renderEl(
            { el: '#okta-sign-in-widget' },
            function (res) {
                if (res.status === 'SUCCESS') {
                    console.log(res);
                    var id_token = res.id_token || res.idToken;
                    $.ajax({
                        type: "GET",
                        dataType: 'json',
                        url: "/users/me",
                        beforeSend: function(xhr) {
                            xhr.setRequestHeader("Authorization", "Bearer " + id_token);
                        },
                        success: function(data){
                            renderLogin(data.user_id);
                        }
                    });
                }
            },
            function (err) { console.log('Unexpected error authenticating user: %o', err); }
        );
    }

Let's cover each of those sections in detail:

Below we pass `renderEl` "`#okta-sign-in-widget`", which is the
HTML `id` for the `<div>` tag that we want to contain the Okta
Sign-In Widget.

    { el: '#okta-sign-in-widget' }

Next, we pass `renderEl` a function that makes an [Ajax](https://en.wikipedia.org/wiki/Ajax_(programming)) request to
`/users/me`. This call passes the `id_token` in the
`Authorization` header to validate the request. If everything
works as expected, then we call the `renderLogin()` function with
the user's Okta ID as a parameter.

    function (res) {
        if (res.status === 'SUCCESS') {
            console.log(res);
            var id_token = res.id_token || res.idToken;
            $.ajax({
                type: "GET",
                dataType: 'json',
                url: "/users/me",
                beforeSend: function(xhr) {
                    xhr.setRequestHeader("Authorization", "Bearer " + id_token);
                },
                success: function(data){
                    renderLogin(data.user_id);
                }
            });
        }
    }

Lastly, we pass `renderEl` an error handling function. In this
example, we pass in a very simple error handling function that
just calls `console.log()` with the error message. This is only
useful while developing your custom logic for the Okta Sign-In
Widget and you will want to do something different in a production
deployment.

    function (err) { console.log('Unexpected error authenticating user: %o', err); }

### `renderLogin()`

Below is an overview of what the `renderLogin()` function
does:

    function renderLogin(user_id) {
        $('#navbar > ul').empty().append('<li><a id="logout" href="/logout">Log out</a></li>');
        $('#logout').click(function(event) {
            event.preventDefault();
            renderLogout();
        });
        $('#logged-out-message').hide();
        $('#logged-in-message').show();
    
        $('#okta-sign-in-widget').hide();
        $('#okta-user-id').empty().append(user_id);
        $('#logged-in-user-id').show();
    }

Here is what each of the sections above do:

First, we add a "Log out" item
to the navbar, then register a `click()` event for when the user
clicks on "Log out":

    $('#navbar > ul').empty().append('<li><a id="logout" href="/logout">Log out</a></li>');
    $('#logout').click(function(event) {
        event.preventDefault();
        renderLogout();
    });

Next, we hide the "logged out" message and display the "logged in" message:

    $('#logged-out-message').hide();
    $('#logged-in-message').show();

Lastly, in the `<<display-user-id>>` section, we hide the Okta Sign-In
Widget append the user's Okta ID into page, then show the part of
the page with the user's Okta ID:

    $('#okta-sign-in-widget').hide();
    $('#okta-user-id').empty().append(user_id);
    $('#logged-in-user-id').show();

**Note:** The `#okta-sign-in-widget` element can only be
instantiated once per page, so for a Single Page Application, it
is critical that you hide the element instead of removing it.

Convert your code to show and hide the `#okta-sign-in-widget`
element if your browser's JavaScript console shows an error that says:
"Backbone.history has already been started" 

### `renderLogout()`

The `renderLogout()` function is essentially the opposite of the
`renderLogin()`, it clears out the navigation bar with `empty`,
hides the "logged in" message and shows the "logged out" message,
hides the users Okta ID and shows the Okta Sign-In Widget. (This code
also clears out the password field in the sign-in widget).

    function renderLogout() {
        $('#navbar > ul').empty();
        $('#logged-in-message').hide();
        $('#logged-out-message').show();
        $('#logged-in-user-id').hide();
        $('#okta-sign-in .okta-form-input-field input[type="password"]').val('');
        $('#okta-sign-in-widget').show();
    }

# Learn more

Want to learn more about Open ID Connect and OAuth?

Here is what we suggest that you read to learn more:

-   Aaron Parecki's "[OAuth 2 Simplified](https://aaronparecki.com/articles/2012/07/29/1/oauth2-simplified)" post.
    
    Start here if you don't know anything about OAuth 2.
-   Karl McGuinness' "[Demystifying OAuth](http://developer.okta.com/blog/2015/12/07/oauth/)" video and slides.
    
    This is a great high level guide that covers the basics of OAuth.
-   [OpenID Connect Implicit Client Implementer's Guide](http://openid.net/specs/openid-connect-implicit-1_0.html)
    
    An official guide for implementing the "implicit" flow. Language
    agnostic and very useful for learning the details on how things
    work.