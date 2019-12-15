
# mongol driver for openresty

example
---------------------------

```lua
local mongo = require "resty.mongol"
conn = mongo:new()
conn:set_timeout(1000)
ok, err = conn:connect()
if not ok then
	ngx.say("connect failed: "..err)
end

local db = conn:new_db_handle ( "test" )
col = db:get_col("test")

r = col:find_one({name="dog"})
ngx.say(r["name"])
```
