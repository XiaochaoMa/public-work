#### Change in "app.hpp"

```
void handle(Request& req, std::shared_ptr<bmcweb::AsyncResp> aResp)
//change the incoming parameter
{
        router.handle(req, aResp);
}
```

#### Change in "query_param.hpp"

Here calling app.handle()

```
void excuteQueryParamAll(crow::App& app, const crow::Request& req,
                         std::shared_ptr<bmcweb::AsyncResp> aResp)
{    
    ....
    if (membersT[0].find("@odata.id") != membersT[0].end())
    {
          std::string url = membersT[0]["@odata.id"];
          .....
          app.handle(*req_m, aResp);
          //The incoming parameter is changed to shared_ptr.
    }
}
```

#### Change the Router.handle() in "routing.hpp" 

```
    void handle(Request& req, std::shared_ptr<bmcweb::AsyncResp> aResp) 
    //change the paramter from "Response& res" to shared_ptr
    {
    ......
            crow::connections::systemBus->async_method_call(
            [&req, aResp{std::move(aResp)}, &rules, ruleIndex, found](
                const boost::system::error_code ec,
                std::map<std::string, std::variant<bool, std::string,
                                                   std::vector<std::string>>>
             userInfo) {
                    if (ec)
                    {
                    ...
                    }
                    ......
                    req.userRole = userRole;
                    rules[ruleIndex]->handle(req, aResp->res, found.second);  
                    //change the paramter from "res" to "aResp->res"
              },
              "xyz.openbmc_project.User.Manager", "/xyz/openbmc_project/user",
              "xyz.openbmc_project.User.Manager", "GetUserInfo",
              req.session->username);
    }
```



After the above modification, it is found that the first res.end() was executed much earlier than the start of the second app.handle().
Eventually it still led to coredump.

Follows is the logs：

[1]the first res.end() executed

[2]the second app.handle() start

```
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [ERROR "http_response.hpp":116] HANDLECOUNT = 1
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "http_response.hpp":140] calling completion handler-------------------------> [1]
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "http_response.hpp":143] completion handler was valid
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [INFO "http_connection.hpp":414] Response: 0x1fccc50 /redfish/v1/Systems 200 keepalive=1
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "timer_queue.hpp":53] timer add inside: 0x1f30710 7
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "http_connection.hpp":755] 0x1fccc50 timer added: 0x1f30710 7
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "http_connection.hpp":659] 0x1fccc50 doWrite
Jan 01 00:02:01 fp5280g2 bmcweb[459]: here All excuteQueryParam
Jan 01 00:02:01 fp5280g2 bmcweb[459]: do route. xiaochao---------------------> [2]
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "query_param.hpp":183] aResp->res.bhandle= 1
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "routing.hpp":1288] Matched rule '/redfish/v1/Systems/system/' 2 / 4
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "http_connection.hpp":667] 0x1fccc50 async_write 620 bytes
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "http_connection.hpp":701] 0x1fccc50 timer cancelled: 0x1f30710 7
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "http_connection.hpp":685] 0x1fccc50 Clearing response
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "http_response.hpp":102] 0x1fceea0 Clearing response containers
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "http_connection.hpp":498] 0x1fccc50 doReadHeaders
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [ERROR "http_connection.hpp":506] 0x1fccc50 async_read_header 0 Bytes
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [ERROR "http_connection.hpp":512] 0x1fccc50 Error while reading: end of stream
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "http_connection.hpp":530] 0x1fccc50 from read(1)
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "http_connection.hpp":272] 0x1fccc50 Connection closed, total 1
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "routing.hpp":1306] aResp->res.bhandle= 0
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "routing.hpp":1330] userName = root userRole = priv-admin
Jan 01 00:02:01 fp5280g2 bmcweb[459]: (1970-01-01 00:02:01) [DEBUG "routing.hpp":1408] aResp->res.bhandle= 0
Jan 01 00:02:02 fp5280g2 systemd[1]: bmcweb.service: Main process exited, code=dumped, status=11/SEGV
Jan 01 00:02:02 fp5280g2 systemd[1]: bmcweb.service: Failed with result 'core-dump'.

```

#### About the counter:
I added an'int' variable to the'res' constructor. When creating a response, it counts as +1; when calling app.handle, it also counts as +1. Finally check the counter in res.end.
The above log shows: before the counter increases, the first res.end has started to execute.
