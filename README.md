# guardian

Authentication gateway middle-man for simplifying OAuth requests with a plugin based architecture to allow quick iteration and implementation of new authentication schemes outside of normal or existing flows.

## Install

```bash
$ npm install guardian
```

## Starting

```bash
$ node index.js -c <configuration>
```

### Configuration

Configuration files can be found in the `config` directory, when no configuration file is declared `default.js` is loaded, when declaring which file to use omit the `js` extension like so:

```bash
$ node index.js -c production
```

Basic Options:

- `host`
  *You should set this to be the public ip or domain name as it is utilized to generate the callback uri.*
  
  Default: `localhost:3000`
- `protocol`
  *Host protocol*
  
  Default: `http`
- `port`
  *Port on which guardian runs*
  
  Default: `3000`
- `pid.dir`
  *Directory where the `.guardian.pid` file will be output, in production environments this is usually `/home/<user>/`, with trailing slash.*

  Default: `./`
- `redis.host`
- `redis.port`
- `redis.pass`

## Routes

Each endpoint functions as a step.

### Storage

    POST /store

Stores information given, returns hash to be used later on. `10` second life on the hashed information by default.

#### Parameters

**OAuth 2**
> Details specific to OAuth2

- `client_id`
- `client_secret`
- `grant_type`
  *Highly dependant on flow state, and which flow you are accessing. Common values:*
  - `authorization_code`
  - `client_credentials`
  - `password`
  - `refresh_token`
- `base_url`
- `access_name` *access token name, default `access_token`*
- `authorize_method`
- `state`
- `scope`

**OAuth 1**
> Details specific to OAuth 1.0a

- `consumer_key`
- `consumer_secret`
- `signature_method`
- `oauth_token`

**Authentication** *required*
> General information regarding authentication flow to load plugin.

- `auth_type` *a-z chars accepted only*
  
  Default: `oauth`
- `auth_flow` *a-z_ chars accepted only*
  > This would be a specific flow, a niche if you may. Echo, Owner Resources, etc.. Optional.
- `auth_version` *numeric chars only*
  > What version of `auth_type` are we dealing with? Can be optional.
- `auth_leg` *numeric chars only*
  > What leg of `auth_type` is this?
  
These are combined to create the plugin file name which is composed like so:

```
type.lower + (flow? '_' + flow : '') + (version? '_' + version : '') + (leg? '_' + leg : '')
```
  
**General**

- `request_url`
- `access_url`
- `authorize_url`
- `callback` *for access_token & access_secret response*

### Hash Check

    GET /hash-check
    
Allows you to preview / verify your stored information in-case of error or malformed response.

Once again, stored information by default lasts only `10` seconds.

#### Parameters

- `hash`

### Start

    ALL /start

Begins guardian transactions and authentication steps. These steps are passed with a `302` request and should be followed.

#### Parameters

- `hash`

**OAuth 1.0a**
> Used in the OAuth 1.0a Signature Process for 1-Legged requests. [Example](https://github.com/Mashape/guardian/blob/master/tests/factual.js#L46).

- `url` *Calling URL, query parameters from here are parsed so you don't need to place them in parameters... I don't think.*
- `method` *Calling Method*
- `body` *Calling Payload or Body*
- `parameters` *Calling Parameters for Request Signatures or etc...*

## Tests

Each test in the test folder is based on an API or feature of guardian rather than TDD or BDD based tests, we simply verify whether the authentication succeeds and we get a response from the API about the API information rather than Authentication information.

Each API based test will require something of the likes:

```
$ node tests/api.js -k {Your Consumer/Client Key/Id} -s {Your Consumer/Client Secret} -h {host, ie: localhost or domain}
```

You will recieve a response with the headers sent, and the returned response from the API, guardian must be running locally for these tests to work and on the port `3000`. Unless you alter these files~

## About

Originally called gatekeeper, but someone else had that and no other names seemed appropriate for what this does so we went with guardian which is another form of a gatekeeper.
