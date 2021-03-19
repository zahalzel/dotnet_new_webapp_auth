```dotnet new webapp --auth singleorg```

*Remote differences for AAD Application in Azure Portal:*
+ "ID Tokens (used for implicit and hybrid flows)" checked


*AppSettings.json changes:*
```
+*/
+  "AzureAd": {
+    "Instance": "https://login.microsoftonline.com/",
+    "Domain": "qualified.domain.name",
+    "TenantId": "22222222-2222-2222-2222-222222222222",
+    "ClientId": "11111111-1111-1111-11111111111111111",
+    "CallbackPath": "/signin-oidc"
+  },
```

*Changes to Startup.cs*
```
+using Microsoft.AspNetCore.Authentication;
+using Microsoft.Identity.Web;
+using Microsoft.Identity.Web.UI;
+using Microsoft.AspNetCore.Authentication.OpenIdConnect;
+using Microsoft.AspNetCore.Authorization;
+using Microsoft.AspNetCore.Mvc.Authorization;

         public void ConfigureServices(IServiceCollection services)
         {
+            services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
+                .AddMicrosoftIdentityWebApp(Configuration.GetSection("AzureAd"));
+
+            services.AddAuthorization(options =>
+            {
+                // By default, all incoming requests will be authorized according to the default policy
+                options.FallbackPolicy = options.DefaultPolicy;
+            });
+            services.AddRazorPages()
+                .AddMvcOptions(options => {})
+                .AddMicrosoftIdentityUI();
         }
 
+            app.UseAuthentication();
 
             app.UseEndpoints(endpoints =>
             {
                 endpoints.MapRazorPages();
+                endpoints.MapControllers();
             });
         }
     }
```

======================================<p>

```dotnet new webapp --auth singleorg --calls-graph```

*Remote differences for AAD Application in Azure Portal:*
+   Client Secret added

*Additional AppSettings.json additions:*
```
+    "ClientSecret": "secret-from-app-registration",
+    "ClientCertificates" : [
+    ],
   },
+  "DownstreamApi": {
+    /*
+     'Scopes' contains space separated scopes of the Web API you want to call. This can be:
+      - a scope for a V2 application (for instance api:b3682cc7-8b30-4bd2-aaba-080c6bf0fd31/access_as_user)
+      - a scope corresponding to a V1 application (for instance <App ID URI>/.default, where  <App ID URI> is the
+        App ID URI of a legacy v1 Web application
+      Applications are registered in the https:portal.azure.com portal.
+    */
+    "BaseUrl": "https://graph.microsoft.com/v1.0",
+    "Scopes": "user.read"
+  },
```

*Changes to Startup.cs*
```
+using Microsoft.Graph;
 
         public void ConfigureServices(IServiceCollection services)
         {
+            var initialScopes = Configuration.GetValue<string>("DownstreamApi:Scopes")?.Split(' ');
+
             services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
+                .AddMicrosoftIdentityWebApp(Configuration.GetSection("AzureAd"))
+                    .EnableTokenAcquisitionToCallDownstreamApi(initialScopes)
+                        .AddMicrosoftGraph(Configuration.GetSection("DownstreamApi"))
+                        .AddInMemoryTokenCaches();
 
```

======================================<p>

```dotnet new webapp --auth singleorg --called-api-url "webapi"```

*Remote differences for AAD Application in Azure Portal:*
+   Client Secret added


*AppSettings.json additions:*
```
+    "ClientSecret": "secret-from-app-registration",
+    "ClientCertificates" : [
+    ],
+  "DownstreamApi": {
+    /*
+     'Scopes' contains space separated scopes of the Web API you want to call. This can be:
+      - a scope for a V2 application (for instance api:b3682cc7-8b30-4bd2-aaba-080c6bf0fd31/access_as_user)
+      - a scope corresponding to a V1 application (for instance <App ID URI>/.default, where  <App ID URI> is the
+        App ID URI of a legacy v1 Web application
+      Applications are registered in the https:portal.azure.com portal.
+    */
+    "BaseUrl": "webapi",
+    "Scopes": "user.read"
+  },
```

*Changes to Startup.cs*

```
         public void ConfigureServices(IServiceCollection services)
         {
+            var initialScopes = Configuration.GetValue<string>("DownstreamApi:Scopes")?.Split(' ');
+
             services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
+                .AddMicrosoftIdentityWebApp(Configuration.GetSection("AzureAd"))
+                    .EnableTokenAcquisitionToCallDownstreamApi(initialScopes)
+                        .AddDownstreamWebApi("DownstreamApi", Configuration.GetSection("DownstreamApi"))
+                        .AddInMemoryTokenCaches();
 
```