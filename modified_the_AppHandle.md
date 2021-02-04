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