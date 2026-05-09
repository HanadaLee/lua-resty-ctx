# lua-resty-ctx

A module for sharing `ngx.ctx` between subrequests in OpenResty.

Extracted from [Kong](https://github.com/Kong/kong), which itself adapted it from [3scale/apicast](https://github.com/3scale/apicast). The original work was done by Alex Zhang (openresty/lua-nginx-module#1057).

## Background

In OpenResty, `ngx.ctx` provides a per-request Lua table for storing arbitrary data. However, when issuing subrequests via `ngx.location.capture` or internal redirects, each subrequest gets its own isolated `ngx.ctx` — data from the parent request is not accessible. This module solves that problem by stashing the `ngx.ctx` reference into an nginx variable (`ctx_ref`), allowing a subrequest to restore ("apply") it.

## API

### stash_ref

Stashes the current request's `ngx.ctx` reference into `ngx.var.ctx_ref`.

```lua
local ctx = require "resty.ctx"
ctx.stash_ref()
```

Call this in the parent request (e.g., in an `access` phase handler) **before** issuing subrequests. This function is idempotent — calling it multiple times is safe.

### apply_ref

Restores the stashed `ngx.ctx`, replacing the current request's context with the parent's.

```lua
local ctx = require "resty.ctx"
ctx.apply_ref()
```

Call this at the start of a subrequest handler. If no `ctx_ref` was stashed, or no parent request exists, a warning is logged and execution continues.

## Usage

```lua
-- parent request (e.g., access phase)
local ctx = require "resty.ctx"

ngx.ctx.user_id = 123
ngx.ctx.session  = { role = "admin" }

ctx.stash_ref()

local res = ngx.location.capture("/sub")

-- subrequest handler
local ctx = require "resty.ctx"
ctx.apply_ref()

-- ngx.ctx now contains user_id and session from the parent
ngx.say(ngx.ctx.user_id)  --> 123
```

## Compatibility

Supports both `http` and `stream` subsystems. Tested with OpenResty 1.19.x and later.

## License

Apache License, Version 2.0. See [LICENSE](LICENSE).
