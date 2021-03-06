# Tarantool Cartridge WebUI GraphQL IDE module

GraphQL IDE module is used to add GraphQL IDE functionality into Tarantool Cartridge WebUI or pure Tarantool (without Tarantool Cartridge). Module it self and Cartridge role are hot-reloadable.

Based on:

- [Tarantool 1.10.x or 2.x.x](https://www.tarantool.io/en/download/)
- [Tarantool Cartridge 2.3.0+](https://github.com/tarantool/cartridge) (optional)
- [Tarantool Frontend Core 7.8.0](https://github.com/tarantool/frontend-core)
- [Tarantool Http 1.1.0](https://github.com/tarantool/http/tree/1.1.0)
- [GraphiQL 1.4.2](https://github.com/graphql/graphiql)
- [GraphiQL Explorer 0.6.3](https://github.com/OneGraph/graphiql-explorer)

GraphQL IDE interface:

![GraphQL IDE](https://github.com/no1seman/graphqlide/blob/master/resources/graphqlide.jpg "GraphQL IDE")

## Install pre-built rock

Simply run from the root of Tarantool Cartridge App root the following:

```sh
    cd <tarantool-cartridge-application-dir>
    tarantoolctl rocks install https://github.com/no1seman/graphqlide/releases/download/0.0.10/graphqlide-0.0.10-1.all.rock
```

## Lua API

Note: This module is still under heavy development. API may be changed in further versions without notice.
### Init

`init()` - used to initialize graphqlide module for non-Cartridge Tarantool Applications.

### Stop

`stop()` - used to deinitialize graphqlide module for non-Cartridge Tarantool Applications.

### Set endpoint

`set_endpoint(endpoint)` - used to set GraphQL Web UI endpoint in runtime.

where:

* `endpoint` (`table`) - mandatory, endpoint of GraphQL API description with the following parameters:
  * `name` (`string`) - mandatory, schema display name;
  * `path` (`string`) - mandatory, URI-path to graphql endpoint;
  * `default` (`boolean`) - optional, flag to indicate that this endpoint is default, false - if not provided.


Example:

```lua
    graphqlide.set_endpoint({ name = 'Spaces', path = '/admin/graphql', default = true })
    graphqlide.set_endpoint({ name = 'Admin', path = '/admin/api' })
```

### Get endpoints

`get_endpoints()` - method is used to get endpoint.

Returns `endpoints` (`table`) with the following structure:

```lua
    {
        ["<endpoint_name>"] = {default = true/false, path = "<endpoint_path>"},
        ...
    }
```

### Remove endpoints

`remove_endpoint(name)` - method is used to remove endpoint,

where:

* `name` (`string`) -  mandatory, schema display name.

Example:

```lua
    graphqlide.set_endpoint({ name = 'Spaces', path = '/admin/graphql', default = true })

    ...

    graphqlide.remove_endpoint('Spaces')
```

### Front init

`front_init(httpd, opts)` - method to init frontend core. Used for non-Cartridge Tarantool Applications.

where:

* `httpd` (`table`) - instance of a Tarantool HTTP server (only 1.x versions is supported).

* `enforce_root_redirect` (`boolean`) - optional key which controls redirection to frontend core app from '/' path, default true;
* `prefix` (`string`) - optional, adds path prefix to frontend core app;
* `force_init` (`boolean`) - optional, flag to force frontend module initialization. By default front_init() checks whether frontend core module initialized or not, but if force_init == true front_init() will skip checks and init frontend core module anyways.

### Version

`VERSION` is a constant to determine which version of GraphQL IDE is installed.

Example:

```lua
    local graphqlide = require('graphqlide')
    local log = require('log')
    log.info('GraphQL IDE version: %s', graphqlide.VERSION)
```

## Add GraphQL IDE to your Tarantool Cartridge App

There are 3 ways to add GraphQL IDE to Tarantool Cartridge application:

1. Add GraphQL IDE initialization code to Tarantool Cartridge application init.lua:

```lua
    ...

    local ok, err = cartridge.cfg({
        roles = {
            ...
        },
    })

    assert(ok, tostring(err))

    -- Init GraphQL IDE
    local endpoint = '/admin/graphql'
    require('graphqlide').init()
    graphqlide.set_endpoint({ name = 'Spaces', path = '/admin/graphql', default = true })
    ...
```

**Note:** graphqlide.init() must be called after cartridge.cfg()

2. Add GraphQL IDE initialization code into desired Tarantool Cartridge role init() function:

```lua
    local graphqlide = require('graphqlide')
    ...

    local function init(opts) -- luacheck: no unused args
        if opts.is_master then
            ...
        end
        ...

        -- Init GraphQL IDE
        graphqlide.init()
        -- Set GraphQL endpoint
        graphqlide.set_endpoint({ name = 'Spaces', path = '/admin/graphql', default = true })
        return true
    end

    local function stop()
        -- Deinit GraphQL IDE
        graphqlide.stop()
        ...
    end

    ...

    return {
        role_name = 'app.roles.custom',
        init = init,
        stop = stop,
        dependencies = {
            'cartridge.roles.graphqlide',
        }
    }
```

3. Add GraphQL IDE Tarantool Cartridge role as dependency of desired Tarantool Cartridge role:

```lua
    ...
    local function init(opts) -- luacheck: no unused args
        if opts.is_master then
            ...
        end
        ...

        -- Init GraphQL IDE
        graphqlide.init()
        graphqlide.set_endpoint({ name = 'Spaces', path = '/admin/graphql', default = true })
        return true
    end

    local function stop()
        -- Deinit GraphQL IDE
        graphqlide.stop()
        ...
    end
    ...
    return {
        role_name = 'app.roles.custom',
        init = init,
        stop = stop,
        dependencies = {
            'cartridge.roles.graphqlide',
        }
    }
```

In this case may need to set endpoint of GraphQL API. The best way to it simply call `set_endpoint()` method from `init()` of Tarantool Cartridge role:

```lua
    local graphqlide = require('graphqlide')
    ...
    local function init(opts)
        if opts.is_master then
        ...
        end
        ...
        graphqlide.set_endpoint('/admin/graphql')
        ...
    end
```

## Use GraphQL IDE with pure Tarantool (without Tarantool Cartridge)

This module can be used with pure Tarantool (without Tarantool Cartridge).

Caution: GraphQL IDE module is compatible only with 1.x branch of http module [http](https://github.com/tarantool/http)

Example:

```lua
    local http = require('http.server')
    local graphqlide = require('graphqlide')

    local HOST = '0.0.0.0'
    local PORT = 8081
    local ENDPOINT = '/graphqlide'

    local httpd = http.new(HOST, PORT,{ log_requests = false })

    httpd:start()
    -- Init frontend-core module if it was not initialized before
    graphqlide.front_init(httpd)

    -- Init graphqlide module if it was not initialized before
    graphqlide.init()

    -- set default "Default" GraphQL endpoint 
    graphqlide.set_endpoint({ name = 'Default', path = ENDPOINT, default = true })

    box.cfg({work_dir = './tmp'})
```

## Build from sources

### Prerequisites

To build rock you will need the following to be installed:

- [nodejs](https://nodejs.org/)
- [Tarantool 1.10.x or 2.x.x](https://www.tarantool.io/en/download/) 

### Clone repo and install nodejs modules

```sh
    git clone git@github.com:no1seman/graphqlide.git graphqlide
    cd graphqlide
    npm i
```
### Build rock

```sh
    tarantoolctl rocks make
    tarantoolctl rocks pack graphqlide <desired_version>
```

Also you can use `npm run build-rock` to build the rock.

After build completion you will get:

- packed graphqlide rock: `graphqlide/graphqlide-0.0.10-1.all.rock`
- graphqlide rock installed to: graphqlide/.rocks/tarantool

### Install built rock

Simply run from the root of Tarantool Cartridge App root the following:

```sh
    cd <Tarantool Cartridge application dir>
    tarantoolctl rocks install <path_to_rock_file>/graphqlide-0.0.10-1.all.rock
```

## Development

For debug & development purposes VSCode may be used.
Use F5 to run app or Shift-Crtl-B to run production build task.

Useful commands:

- `npm run build - to build production module`
- `npm run start - run application without need to integrate it into Tarantool Cartridge App. Useful for development purposes`
- `npm run build-rock - builds production module and bundles it into the rock`
