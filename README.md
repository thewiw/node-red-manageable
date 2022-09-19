# node-red-manageable
Manage your NodeRed containers using environment variables and custom settings files

This project takes standard and official NodeRed containers (https://hub.docker.com/r/nodered/node-red) and add features to make easier the container configuration.
Standard settings.js file is just slightly modified so you keep the existing behaviour by default.

It can be use like standard NodeRed image. Just add environment variables to get the new features. E.g.:
```
docker run -it -p 1880:1880 -v myNodeREDdata:/data -e NODE_RED_CREDENTIAL_SECRET="foo" --name mynodered thewiwcorp/node-red
docker run -it -p 1880:1880 -v myNodeREDdata:/data -v myusers.txt:/data/auth/editor_users.txt -e NODE_RED_SET_EDITOR_USERS_BY_FILE=true --name mynodered thewiwcorp/node-red-manageable
```

## Current features
- Editor and API: Users, password and permissions can be set in a file. This file is read in a dynamic way, so you can change it anytime.
- Credential secret: Can be set by an environment variable. If not provided, randomly generated by NodeRed
- HTTP node and HTTP static resource basic authentication set by an environment variable. If not provided, authentication is disabled

## Detailed features
### Editor and API users
User are defined with the same syntax than the one expected by NodeRed in a file. Default user may be set too.
Each time a user try to connect, the file is read so you can change the values without restarting NodeRed.

This feature is enabled when you set `NODE_RED_SET_EDITOR_USERS_BY_FILE=true`

Expected file format is:
```
[
  {
    "username": "admin",
    "password": "$2b$08XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "permissions": ["*"]
  },
  {
    "username": "writer",
    "password": "$2b$08XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "permissions": ["read","flows.write","nodes.read"]
  },
  {
    "username": "reader",
    "password": "$2b$08XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "permissions": ["read"]
  }
]
```

All passwords must be encrypted with bcrypt first.

Default file location: `/data/auth/editor_users.txt'

You can change the file location by setting `NODE_RED_EDITOR_USERS_FILE`

You can change the behaviour by providing your own js file and setting the variable `NODE_RED_EDITOR_USERS_CUSTOM_JS`. This js file MUST export `users`, `authenticate` and `default` functions. See https://nodered.org/docs/user-guide/runtime/securing-node-red#custom-user-authentication.

### Credential secret
Credential secret may be stored in the `NODE_RED_CREDENTIAL_SECRET` environment variable.

You can change the behaviour by providing your own js file and setting the variable `NODE_RED_CREDENTIAL_SECRET_CUSTOM_JS`. This js file MUST contain a getCredentialSecret function than returns a string. String value is only read at NodeRed startup.

### HTTP node and static authentication
One user for HTTP nodes and access to static resources can be defined by setting `NODE_RED_HTTP_NODE_USER` and `NODE_RED_HTTP_STATIC_USER` variables.

Expected format is `user:encryptedpassword`. Passwords must be encrypted with bcrypt.

You can change the behaviour by providing your own js files and setting the variables `NODE_RED_HTTP_NODE_AUTH_CUSTOM_JS` and `NODE_RED_HTTP_STATIC_AUTH_CUSTOM_JS`. This js files MUST contain getHttpNodeUser and getHttpStaticUser functions. Both of them MUST return a JS object `{"user": "myhttpuser", "pass": "mypassword"`.
JS objects are only read at NodeRed startup.
}

## Password encryption
All password must be encrypted with bcrypt and a salt of 8.
See https://nodered.org/docs/user-guide/runtime/securing-node-red#generating-the-password-hash.

### Build and run
## Docker compose
```
services:
  node-red:
    image: node-red-manageable:3.0.2-16
    build:
        context: .
        args:
          - NODE_RED_VERSION=3.0.2
          - NODE_VERSION=16
    environment:
      - NODE_RED_SET_EDITOR_USERS_BY_FILE=true
      - NODE_RED_CREDENTIAL_SECRET=mystaticsecret
      - NODE_RED_HTTP_NODE_USER=uiuser:$$2b$$08$$mybcryppassword1
      - NODE_RED_HTTP_STATIC_USER=resourcesuser:$$2b$$08$$mybcryppassword2
    cap_drop:
      - ALL
    cap_add:
      - CAP_NET_BIND_SERVICE
    deploy:
      replicas: 1
    expose:
      - "1880"
    ports:
      - "1880:1880"
    volumes:
      - node-red.data:/data
volumes:
    node-red.data:
```
In `docker-compose.yml` file, pay attention that all $ must be typed twice.

### Build the image
Either:
```
docker build --pull --build-arg NODE_VERSION=16 --build-arg NODE_RED_VERSION=3.0.2 --tag yournodered:3.0.2-16 ./
```
Or:
```
docker compose build
```
### Run a container
Either:
```
docker run -it -p 1880:1880 -v myNodeREDdata:/data -v myusers.txt:/data/auth/editor_users.txt -e NODE_RED_SET_EDITOR_USERS_BY_FILE=true --name mynodered thewiwcorp/node-red-manageable
```
Or:
```
docker compose up
```

## Project
### Roadmap
- Management of a default user for editor and user authorization file
- Command to update editor and user authorization file 
- Advanced middleware for HTTP node, providing user management

### Maintenance
If you want to improve this project, feel free to contact us or to send a pull request.

### Change log
- 0.0.1: Initial features: editor and api authorization file, credential secret, http node and http static users
- 0.0.2: Missing file and README fix
- 0.0.1: Initial features: editor and api authorization file, credential secret, http node and http static users
