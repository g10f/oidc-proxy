### **OIDC Proxy**
----

  - Supports role based uri controls
  - Websocket connection upgrading
  - Token claim matching for additional ACL controls
  - Stateless offline refresh tokens with optional predefined session limits
  - TLS and mutual TLS support
  - JSON field bases access logs
  - Custom Sign-in and access forbidden pages

----

oidc-proxy is a proxy service which integrates with the OpenID Connect Identity Provider authentication service. The service supports both access tokens in browser cookie or bearer tokens.

```shell
$ bin/oidc-proxy help
NAME:
   keycloak-proxy - is a proxy using the keycloak service for auth and authorization

USAGE:
   keycloak-proxy [global options] command [command options] [arguments...]

VERSION:
   v1.0.4

AUTHOR(S):
   Rohith <gambol99@gmail.com>

COMMANDS:
GLOBAL OPTIONS:
--config                                                the path to the configuration file for the keycloak proxy
--listen "127.0.0.1:3000"                               the interface the service should be listening on
--client-secret                                         the client secret used to authenticate to the oauth server
--client-id                                             the client id used to authenticate to the oauth serves
--discovery-url                                         the discovery url to retrieve the openid configuration
--upstream-url "http://127.0.0.1:8081"                  the url for the upstream endpoint you wish to proxy to
--revocation-url "/oauth2/revoke"                       the url for the revocation endpoint to revoke refresh token, not all providers support the revocation_endpoint
--upstream-keepalives                                   enables or disables the keepalive connections for upstream endpoint (defaults true)
--encryption-key                                        the encryption key used to encrpytion the session state
--secure-cookie                                         enforces the cookie to be secure, default to true
--store-url                                             the store url to use for storing the refresh tokens, i.e. redis://127.0.0.1:6379, file:///etc/tokens.file
--no-redirects                                          do not have back redirects when no authentication is present, simple reply with 401 code
--redirection-url                                       the redirection url, namely the site url, note: /oauth will be added to it
--hostname [--hostname option --hostname option]        a list of hostnames the service will respond to, defaults to all
--tls-cert                                              the path to a certificate file used for TLS
--tls-private-key                                       the path to the private key for TLS support
--tls-ca-certificate                                    the path to the ca certificate used for mutual TLS
--skip-upstream-tls-verify                              whether to skip the verification of any upstream TLS (defaults to true)
--scope [--scope option --scope option]                 a variable list of scopes requested when authenticating the user
--claim [--claim option --claim option]                 a series of key pair values which must match the claims in the token present e.g. aud=myapp, iss=http://example.com etcd
--resource [--resource option --resource option]        a list of resources 'uri=/admin|methods=GET|roles=role1,role2'
--signin-page                                           a custom template displayed for signin
--forbidden-page                                        a custom template used for access forbidden
--tag [--tag option --tag option]                       a keypair tag which is passed to the templates when render, i.e. title='My Page',site='my name' etc
--cors-origins [--cors-origins option --cors-origins option]      a set of origins to add to the CORS access control (Access-Control-Allow-Origin)
--cors-methods [--cors-methods option --cors-methods option]      the method permitted in the access control (Access-Control-Allow-Methods)
--cors-headers [--cors-headers option --cors-headers option]      a set of headers to add to the CORS access control (Access-Control-Allow-Headers)
--cors-exposes-headers [--cors-exposes-headers option --cors-exposes-headers option]   set the expose cors headers access control (Access-Control-Expose-Headers)
--cors-max-age "0"                                      the max age applied to cors headers (Access-Control-Max-Age)
--cors-credentials                                      the credentials access control header (Access-Control-Allow-Credentials)
--enable-security-filter                                enables the security filter handler
--skip-token-verification                               testing purposes ONLY, the option allows you to bypass the token verification, expiration and roles are still enforced
--proxy-protocol                                        switches on proxy protocol support on the listen (not supported yet)
--offline-session                                       enables the offline session of tokens via offline access (defaults false)
--json-logging                                          switch on json logging rather than text (defaults true)
--log-requests                                          switch on logging of all incoming requests (defaults true)
--verbose                                               switch on debug / verbose logging
--help, -h                                              show help
--version, -v                                           print the version

```

#### **Configuration**

The configuration can come from a yaml/json file and or the command line options (note, command options have a higher priority and will override any options referenced in a config file)

```YAML
# is the url for retrieve the openid configuration - normally the <server>/.well-known/openid-configuration
discovery-url: https://sso.example.org/.well-known/openid-configuration
# the client id for the 'client' application
client-id: <CLIENT_ID>
# the secret associated to the 'client' application
client-secret: <CLIENT_SECRET>
# the interface definition you wish the proxy to listen, all interfaces is specified as ':<port>'
listen: 127.0.0.1:3000
# whether to request offline access and use a refresh token
enable-refresh-tokens: true
# log all incoming requests
log-requests: true
# log in json format
log-json-format: true
# debug output
verbose: true
# do not redirec the request, simple 307 it
no-redirects: false
# the location of a certificate you wish the proxy to use for TLS support
tls-cert:
# the location of a private key for TLS
tls-private-key:
# the public key for the ca, used for mutual TLS
tls-ca-certificate:
# the redirection url, essentially the site url, note: /oauth/callback is added at the end
redirection-url: http://127.0.0.3000
# the encryption key used to encode the session state
encryption-key: vGcLt8ZUdPX5fXhtLZaPHZkGWHZrT6T8xKHWf5RPfqAocuiQ6nUbNHyc3oF2toO2tr
# the upstream endpoint which we should proxy request
upstream: http://127.0.0.1:80
# upstream-keepalives specified wheather you want keepalive on the upstream endpoint
upstream-keepalives: true
# additional scopes to add to add to the default (openid+email+profile)
scopes:
  - picture
  - offline_access
# enables a more extra security features
enable-security-filter: true
# a map of claims that MUST exist in the token presented and the value is it MUST match
# So for example, you could match the audience or the issuer or some custom attribute
claims:
  aud: <CLIENT_ID>
  iss:  https://sso.example.org
# a collection of resource i.e. urls that you wish to protect
resources:
  - url: /admin/test
    # the methods on this url that should be protected, if missing, we assuming all
    methods:
      - GET
    # a list of roles the user must have in order to accces urls under the above
    roles:
      - AdminTest
  - url: /admin/white_listed
    # permits a url prefix through, bypassing the admission controls
    white-listed: true
  - url: /admin
    methods:
      - GET
    roles:
      - Admin
```

#### **Example Usage**

Assuming you have some web service you wish protected by Keycloak;

a) Create the *client* under the Keycloak GUI or CLI; the client protocol is *'openid-connect'*, access-type:  *confidential*.
b) Add a Valid Redirect URIs of *http://127.0.0.1:3000/oauth/callback*.
c) Grab the client id and client secret.
d) Create the various roles under the client or existing clients for authorization purposes.

##### **- The default config**

```YAML
discovery-url:  https://sso.example.org/.well-known/openid-configuration
client-id: <CLIENT_ID>
client-secret: <CLIENT_SECRET>
listen: 127.0.0.1:3000
redirection-url: http://127.0.0.3000
refresh_session: false
encryption_key: AgXa7xRcoClDEU0ZDSH4X0XhL5Qy2Z2j
upstream: http://127.0.0.1:80

resources:
  - url: /admin
    methods:
      - GET
    roles:
      - Admin
  - url: /backend
    roles:
      - User
```

Note, anything defined in the configuration file can also be configured as command line options, so the above would be reflected as;

```shell
bin/keycloak-proxy \
    --discovery-url= https://sso.example.org/.well-known/openid-configuration \
    --client-id=<CLIENT_ID> \
    --client-secret=<SECRET> \
    --listen=127.0.0.1:3000 \
    --redirection-url=http://127.0.0.3000 \
    --refresh-sessions=true \
    --encryption-key=AgXa7xRcoClDEU0ZDSH4X0XhL5Qy2Z2j \
    --upstream=http://127.0.0.1:80 \
    --resource="uri=/admin|methods=GET|roles=test1,test2" \
    --resource="uri=/backend|roles=test1"
```

#### **- Google OAuth**
---
Although the role extensions do require a Keycloak IDP or at the very least a IDP that produces a token which contains roles, there's nothing stopping you from using it against any OpenID providers, such as Google. Go to the Google Developers Console and create a new application *(via "Enable and Manage APIs -> Credentials)*. Once you've created the application, take the client id, secret and make sure you've added the callback url to the application scope *(using the default this would be http://127.0.0.1:3000/oauth/callback)*

``` shell
bin/keycloak-proxy \
    --discovery-url=https://accounts.google.com/.well-known/openid-configuration \
    --client-id=<CLIENT_ID> \
    --secret=<CLIENT_SECRET> \
    --resource="uri=/" \   
    --verbose=true
```

Open a browser an go to http://127.0.0.1:3000 and you should be redirected to Google for authenticate and back the application when done and you should see something like the below.

```shell
DEBU[0002] resource access permitted: /                  access=permitted bearer=false expires=57m51.32029042s resource=/ username=gambol99@gmail.com
2016-02-06 13:59:01.680300 I | http: proxy error: dial tcp 127.0.0.1:8081: getsockopt: connection refused
DEBU[0002] resource access permitted: /favicon.ico       access=permitted bearer=false expires=57m51.144004098s resource=/ username=gambol99@gmail.com
2016-02-06 13:59:01.856716 I | http: proxy error: dial tcp 127.0.0.1:8081: getsockopt: connection refused
```
---
#### **- Upstream Headers**

On protected resources the upstream endpoint will receive a number of headers added by the proxy;

```GO
# add the header to the upstream endpoint
cx.Request.Header.Add("X-Auth-UserId", id.id)
cx.Request.Header.Add("X-Auth-Subject", id.preferredName)
cx.Request.Header.Add("X-Auth-Username", id.name)
cx.Request.Header.Add("X-Auth-Email", id.email)
cx.Request.Header.Add("X-Auth-ExpiresIn", id.expiresAt.String())
cx.Request.Header.Add("X-Auth-Token", id.token.Encode())
cx.Request.Header.Add("X-Auth-Roles", strings.Join(id.roles, ","))

# plus the default
cx.Request.Header.Add("X-Forwarded-For", <CLIENT_IP>)
cx.Request.Header.Add("X-Forwarded-Proto", <CLIENT_PROTO>)
```

#### **- Encryption Key**

In order to remain stateless and not have to rely on a central cache to persist the 'refresh_tokens', the refresh token is encrypted and added as a cookie using *crypto/aes*.
Naturally the key must be the same if your running behind a load balancer etc. The key length should either 16 or 32 bytes depending or whether you want AES-128 or AES-256.

#### **- Claim Matching**

The proxy supports adding a variable list of claim matches against the presented tokens for additional access control. So for example you can match the 'iss' or 'aud' to the token or custom attributes;
note each of the matches are regex's. Examples,  --claim 'aud=sso.*' --claim iss=https://.*'

#### **- Custom Pages**

By default the proxy will immediately redirect you for authentication and hand back 403 for access denied. Most users will probably want to present the user with a more friendly
sign-in and access denied page. You can pass the command line options (or via config file) paths to the files i.e. --signin-pag=PATH. The sign-in page will have a 'redirect'
passed into the scope hold the oauth redirection url. If you wish pass additional variables into the templates, perhaps title, sitename etc, you can use the --tag key=pair i.e.
--tag title="This is my site"; the variable would be accessible from {{ .title }}

```HTML
<html>
<body>
<a href="{{ .redirect }}">Sign-in</a>
</body>
</html>
```

#### **- White-listed URL's**

Depending on how the application urls are laid out, you might want protect the root / url but have exceptions on a list of paths, i.e. /health etc. Although you should probably
fix this by fixing up the paths, you can add excepts to the protected resources. (Note: it's an array, so the order is important)

```YAML
  resources:
  - url: /some_white_listed_url
    white-listed: true
  - url: /
    methods:
      - GET
    roles:
      - <ROLE_NAME1>
      - <ROLE_NAME2>
```

Or on the command line

```shell
  --resource "uri=/some_white_listed_url,white-listed=true"
  --resource "uri=/"  # requires authentication on the rest
  --resource "uri=/admin|roles=admin,superuser|methods=POST,DELETE
```

#### **- Mutual TLS**

The proxy support enforcing mutual TLS for the clients by simply adding the --tls-ca-certificate command line option or config file option. All clients connecting must present a ceritificate
which was signed by the CA being used.

#### **- Tokens && Stores**

Refresh tokens are either be stored as an encrypted cookie or placed (encrypted) in a shared / local store. At present, redis and boltdb are the only two methods supported. To enable a local boltdb store. --store-url boltdb:///PATH or relative path boltdb://PATH. For redis the option is redis://HOST:PORT. In both cases the refresh token is encrypted before placing into the store

#### **- Refresh & Offline Tokens**

Assuming access response responds with a refresh token and the --enable-refresh-token is true, the proxy will automatically refresh the access token for you. The tokens themselves are kept either as an encrypted (--encryption-key=KEY) cookie (cookie name: kc-state). Alternatively you can place the refresh token (still requires encryption key) in a local boltdb file or shared redis. Naturally the encryption key has to be the same on all instances and boltdb is for single instance only developments.

#### **- Cross Origin Resource Sharing (CORS)**

You are permitted to add CORS following headers into the /oauth uri namespace

 * Access-Control-Allow-Origin
 * Access-Control-Allow-Methods
 * Access-Control-Allow-Headers
 * Access-Control-Expose-Headers
 * Access-Control-Allow-Credentials
 * Access-Control-Max-Age

Either from the config file:

```YAML
cors:
  origins:
  - '*'
  methods:
  - GET
  - POST
```

or via the command line arguments

```shell
--cors-origins [--cors-origins option]                  a set of origins to add to the CORS access control (Access-Control-Allow-Origin)
--cors-methods [--cors-methods option]                  the method permitted in the access control (Access-Control-Allow-Methods)
--cors-headers [--cors-headers option]                  a set of headers to add to the CORS access control (Access-Control-Allow-Headers)
--cors-exposes-headers [--cors-exposes-headers option]  set the expose cors headers access control (Access-Control-Expose-Headers)
```

#### **- Endpoints**

* **/oauth/authorize** is authentication endpoint which will generate the openid redirect to the provider
* **/oauth/callback** is provider openid callback endpoint
* **/oauth/expired** is a helper endpoint to check if a access token has expired, 200 for ok and, 401 for no token and 401 for expired
* **/oauth/health** is the health checking endpoint for the proxy
* **/oauth/login** provides a relay endpoint to login via grant_type=password i.e. POST /oauth/login?username=USERNAME&password=PASSWORD
* **/oauth/logout** provides a convenient endpoint to log the user out, it will always attempt to perform a back channel logout of offline tokens
* **/oauth/token** is a helper endpoint which will display the current access token for you
