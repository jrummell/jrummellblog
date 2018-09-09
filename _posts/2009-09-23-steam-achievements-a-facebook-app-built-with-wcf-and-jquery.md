---
permalink: /steam-achievements-a-facebook-app-built-with-wcf-and-jquery
title: Steam Achievements - A Facebook app built with WCF and jQuery
date: 2009-09-23 17:05:12
published: true
tags: facebook-steam-achievements
---

I’ve been wanting to create a Facebook app for a while now, but I didn’t have time or a need to fill. I took some time off over the last week and I was able to come up with both. I’m avid PC gamer and most of my games are on Valve’s Steam platform. Steam has achievements for some of its biggest newer games like Left 4 Dead and Team Fortress 2. They are much like the Xbox 360 and PS3 achievements. But unlike the Xbox and PS3, there wasn’t a Facebook app that notifies your friends with your latest achievements. There was an app that worked great for a while back, but its now broken (and still using the steamachievements url suffix!).

I’ve created an app at http://www.facebook.com/apps/application.php?id=211407042025.

I first tried an FBML Canvas app built with ASP.NET MVC and the MS Facebook SDK, but I couldn’t find a complete working example. Then I tried FBML with ASP.NET WebForms (traditional ASP.NET) but testing was super annoying since it had to be run in the context of Facebook and I couldn’t get FBJS Ajax to work. Then finally, I tried an IFrame Canvas app built with ASP.NET WebForms. I could now test easier and use jQuery instead of FBJS! For more information, see this wiki topic on IFrame vs. FBML Canvas apps.

Now that I was in familiar territory, I threw a wrench in the mix by using WCF web services instead of asmx. Not that WCF is better/worse than asmx, I had just never used them before. I had some help from Rick Strahl’s blog post called jQuery AJAX calls to a WCF REST Service. This is was no easy task for me, so I’ll list all of the details here in case anyone else is struggling. First, here is the service interface:

    [ServiceContract]
    public interface IAchievementService
    {
      [OperationContract]
      [WebInvoke(Method = "POST", RequestFormat = WebMessageFormat.Json, ResponseFormat = WebMessageFormat.Json, BodyStyle = WebMessageBodyStyle.WrappedRequest)]
      List<Achievement> GetAchievements(string steamUserId, int gameId);

      [OperationContract]
      [WebInvoke(Method = "POST", RequestFormat = WebMessageFormat.Json, ResponseFormat = WebMessageFormat.Json, BodyStyle = WebMessageBodyStyle.WrappedRequest)]
      List<Game> GetGames();

      [OperationContract]
      [WebInvoke(Method = "POST", RequestFormat = WebMessageFormat.Json, ResponseFormat = WebMessageFormat.Json, BodyStyle = WebMessageBodyStyle.WrappedRequest)]
      bool UpdateAchievements(string steamUserId);

      [OperationContract]
      [WebInvoke(Method = "POST", RequestFormat = WebMessageFormat.Json, ResponseFormat = WebMessageFormat.Json, BodyStyle = WebMessageBodyStyle.WrappedRequest)]
      bool UpdateSteamUserId(long facebookUserId, string steamUserId);

      [OperationContract]
      [WebInvoke(Method = "POST", RequestFormat = WebMessageFormat.Json, ResponseFormat = WebMessageFormat.Json, BodyStyle = WebMessageBodyStyle.WrappedRequest)]
      bool PublishLatestAchievements(long facebookUserId, string steamUserId);
    }

Note the WCF attributes: ServiceContract and OperationContract. These kind of correspond to WebService and WebMethod in an asmx web service. I say kind of because a WCF service doesn’t have to be a web service, but for this example it is. The WebInvoke attribute tells ASP.NET how the client will interact with the service. The method is POST, and both the request and the response will be json. The WrappedRequest BodyStyle is what allows you to send a json JavaScript object to a service method that gets de-serialized into parameters. If you were to omit this, you’d have to create a class for your parameters and change the method signature to accept one instance of that class. Another thing to note is that I’m returning List<T> instead of IEnumerable<T>. This is because there seems to be a serialization bug with WCF and IEnumerable<T>.

The implementation of IAchievementService is very straight forward. It is simply hands all of the heavy lifting to a manager class that uses LINQ to SQL to communicate with the database. The next part is the WCF configuration in web.config.

    <system.serviceModel>
      <services>
        <service behaviorConfiguration="SteamAchievements.Services.AchievementServiceBehavior" name="SteamAchievements.Services.AchievementService">
          <endpoint address="" binding="webHttpBinding" contract="SteamAchievements.Services.IAchievementService" behaviorConfiguration="SteamAchievements.Services.AchievementServiceBehavior" />
          <endpoint address="mex" binding="mexHttpBinding" contract="IMetadataExchange" />
        </service>
      </services>
      <behaviors>
        <serviceBehaviors>
          <behavior name="SteamAchievements.Services.AchievementServiceBehavior">
            <serviceMetadata httpGetEnabled="True" />
            <serviceDebug includeExceptionDetailInFaults="True" />
          </behavior>
        </serviceBehaviors>
        <endpointBehaviors>
          <behavior name="SteamAchievements.Services.AchievementServiceBehavior">
            <webHttp />
          </behavior>
        </endpointBehaviors>
      </behaviors>
      <serviceHostingEnvironment aspNetCompatibilityEnabled="false">
        <baseAddressPrefixFilters>
          <add prefix="http://www.rummell.info/" />
        </baseAddressPrefixFilters>
      </serviceHostingEnvironment>
    </system.serviceModel>

This is fairly standard except for a few things. I’m using the webHttp endpoint behavior to enable AJAX and I also had to add a baseAddressPrefixFilter to get it working on my web host.

Finally, here is the JavaScript that calls the WCF service methods:

    var parameters = { "steamUserId": steamUserId, "gameId": gameId };
    callAjax("GetAchievements", parameters, ondone);

    function callAjax(method, query, ondone)
    {
        var onerror = function(m)
        {
            $("#log").text(m.Message).show();
        };

        $.ajax({
            url: _serviceBase + method,
            data: JSON.stringify(query),
            type: "POST",
            processData: true,
            contentType: "application/json",
            timeout: 10000,
            dataType: "json",
            success: ondone,
            error: function(xhr)
            {
                if (!onerror)
                {
                    return;
                }

                if (xhr.responseText)
                {
                    try
                    {
                        var err = JSON.parse(xhr.responseText);
                        if (err)
                        {
                            onerror(err);
                        }
                        else
                        {
                            onerror({ Message: "Unknown server error." });
                        }
                    }
                    catch (e)
                    {
                        onerror({ Message: e.toString() });
                    }
                }
                return;
            }
        });
    }

This is basically the same snippet from Rick Strahl (see link above) that I’ve modified slightly to fit my application.

One final note on using the Facebook API from the context of a web service. The PublishLatestAchievements service method uses the API to publish achievements to the user’s profile. Since its outside the scope of a page, you can’t use Master.Api. To get around this, you’ll need to create an instance of CanvasSession:

    string appKey = WebConfigurationManager.AppSettings["APIKey"];
    string appSecret = WebConfigurationManager.AppSettings["Secret"];
    List<Enums.ExtendedPermissions> permissions =
        new List<Enums.ExtendedPermissions> {Enums.ExtendedPermissions.publish_stream};
    CanvasSession session = new IFrameCanvasSession(appKey, appSecret, permissions, false);
    Api api = new Api(session);

Then you can publish to the stream:

    api.Stream.Publish(description, attachment, links, null, facebookUserId);

Most samples don’t include the uid parameter (facebookUserId), but its required here since HttpSession is unavailable.

If you’d like to see more, take a look at the [Google Code project](http://code.google.com/p/facebooksteamachievements/).