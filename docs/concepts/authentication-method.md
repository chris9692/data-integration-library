# Authentication Methods

## Credentials

DIL supports multiple authentication methods that uses the following credentials.  

- The basic username and password authentication
- The bearer token authentication
    - The OAuth2 authentication
- SSH private key and public key authentication

### Username and Password Credentials

The basic authentication credentials are username and password, or user key and secret key, 
or other forms of pairs of one system controlled name and one user-controlled secret. 

In all such cases, `source.conn.username` holds the username or user key, etc., and `source.conn.password` holds
the password or secret key, etc.

Basic credentials are usually encrypted as a BASE64 string to hide those potential special characters in username or 
password. BASE64 is more of an encoding method than an encryption method because it is reversible. We call it 
encryption to avoid confusion with URL encoding. 

When username and password are BASE64 encrypted, they are concatenated first with normally a column (":") in the middle, 
such as "username:password", then the concatenated string is BASE64 encrypted. 

### Bearer Token Credentials

Bearer tokens are authorizations to a particular resource, such as a Rest API end point. The bearer of the token 
is thus granted access to the resource. Access token is also a bearer token, and they are equivalent in terms of 
configuration. 

Oauth tokens are broader terms of tokens used in OAuth2.0 authentication. It covers authorization code, refresh 
token, and bearer/access token. 

DIL promotes a multistage process to handle OAuth2.0 authentication. See [how to config an authentication job](../how-to/authentication-job.md). 
In the multistage process, the authentication job produces a bearer/access token. The subsequent data 
movement job only need to use that bearer/access token directly. Because of that, Oauth tokens work 
the same way as Bearer/Access tokens, except:
- Bearer/Access tokens are passed in requests with "bearer" tag, such as "bearer xxx", where xxx is the Bearer/Access token
- Oauth tokens are passed in requests with the "oauth" tag, such as "oauth xxx", where xxx is the Oauth token

Bear Token, Access Token, and Oauth token are typically used in HTTP and S3 connections. 

### SSH Private Key and Public Key Credentials

Private Key and Public Key credentials work on SSH based connections. SFTP is an SSH based connection. 

The client of a data system, such an SFTP server, keeps a private key of its own, and also uploads its public key to
the data system. When making a connection, the client will supply the private key in the request so that the data system can
authenticate it. 

## HTTP Protocol Authentication

When making HTTP/HTTPS connections, the client can supply basic credentials or Bearer tokens. 
S3 connections are created on top of HTTP/HTTPS connections, therefore, the following works for S3 as well. 

Http credentials are supply to servers in the "Authorization" header. There are 2 ways to supply the "Authorization" header. 
- the direct way through `ms.http.request.headers` property, this is acceptable but not recommended
- the indirect way through `ms.authentication`property, this is the recommended way

### Direct Configuration

In the direct configuration, an "Authorization" header can be coded explicitly. For example, 
`ms.http.request.headers={"Authorization": "xxx"}`, where "xxx" is the credential. The credential can be:
- a BASE64 encrypted username and password combination
- a Bearer token in the form of "bearer yyy", where yyy is the Bearer/Access token
- an Oauth token in the form of "oauth yyy", where yyy is the Oauth token

In the direct configuration, the value of the "Authorization" header cannot have variables. 

### Indirect Configuration 

In the indirect way, an "Authorization" header is made in DIL through the `ms.authentication` property, and
the `ms.http.request.headers` cannot have an "Authorization" header explicitly. 

The [ms.authentication](../parameters/ms.authentication.md) property is a JsonObject, and it can have
the following attributes:
- "method", the authentication method can be "basic", "bearer", "oauth", or "custom"
  - the "basic" method will prefix the basic credential with a "basic" key word, separated by a space between
    the key word and the credential
  - the "bearer" method will prefix the Bearer token credential with a "bearer" key word, separated by a space between
    the key word and the Bearer token
  - the "oauth" method will prefix the Oauth token credential with an "oauth" key word, separated by a space between
    the key word and the Oauth token
  - the "custom" method doesn't prefix anything, it requires the "token" attribute be completely formed
- "encryption", the credential encryption method can be "base64" or "none". This encryption is on-network encryption. 
  It means the credential is encrypted in the HTTP request. 
- "header", the HTTP header name. It is normally "Authorization" as described in the [direct configuration](#direct-configuration) section. 
  The header can vary between APIs.
- "token", the actual credential, and it can be the following
  - the Bearer/Access/Oauth token when the "method" is "bearer" or "oauth"
  - the username and password combination when the "method" is "basic", in this case `source.conn.username` 
    and `source.conn.password` are no longer needed. When the combination is BASE64 encrypted already, the "encryption"
    attribute can be "none".
  - Any pre-formed credential when the "method" is "custom"
  
The following are examples of indirect configuration:

- `ms.authentication={"method": "basic", "encryption": "base64", "header": "Authorization"}`
- `ms.authentication={"method": "bearer", "encryption": "none", "header": "Authorization", "token": "{{access_token}}"}`
- `ms.authentication={"method": "Bearer", "encryption": "none", "header": "Authorization", "token":"xxx"}`

## Static Credential Encryption 


basic|bearer|oauth|custom

DIL supports various types of authentication methods.



User name + password
User key + secret key
Private key + public key
Tokens (
username/password based
In general if your source requires the following combo to authenticate:

username/password (e.g, JDBC)

client id/client secret (e.g, SFTP)

client key/token

...

You will need to specify the credential pair in the following two parameters:

"source.conn.password" : "ENC(username/client id/etc.)",
"source.conn.username" : "ENC(password/token/secret/etc.)"


SFTP source
For SFTP use cases you can either authenticate using a privateKey or username/password basis.

# Following parameters are required for any SFTP use case
"source.conn.username" : "<username>"
"source.conn.host" : "<Sftp Host>"
"source.conn.port" : "<Connection port if any, default is 22>"


# Include the following if the connection key based
"source.conn.private.key" : "<privateKey path>"


# Include the following if the connection is based username/password
"encrypt.key.loc": "<Path to key file to encrypt the password>"
"source.conn.password": "ENC(password)"


JDBC source
Only JDBC using username password authentication is supported.

OAuth2
DIL supports OAuth indirectly through access token support. The token can be "access", "refresh", "bearer", or custom.

The basic process is:

Define a job to obtain the OAuth2 token
Use the acquired token in subsequent jobs
Basic/Bearer
In many HTTP sources, a simple token based authentication is being used. DIL currently supports two of the most common token types.

If your authorization header is in the form of:

"Authentication Basic/Bearer username:password"
Then use the following configuration:

"ms.authentication" : "{"method": "bearer/basic", "encryption": "base64", "header": "Authorization"}",
"source.conn.password" : "ENC(username/client id/etc.)",
"source.conn.username" : "ENC(password/token/secret/etc.)"
If your token is constructed in other ways, don't use ms.authentication, instead, explicitly include the entire authorization header in ms.http.request.headers:
"ms.http.request.headers" : "{"Content-Type": "application/json", "Authorization\": "ENC(YOUR TOKEN HERE)"}"
