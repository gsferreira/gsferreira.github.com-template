---
layout: post
title: Disable your html cache
category: ASP.NET MVC
---


Do you know that usually when you use the back button of your browser, your page won't be requested to the server again? By default this is the behaviour that you can expect.

But, you can control it!

<!--excerpt-->

Go to your web.config and place the following code into the <system.web> element.

    <caching>
      <outputCache enableOutputCache="false"></outputCache>
    </caching>

In your web.config place the following code into the <system.webserver> element to enable the caching for resource files, as images or stylesheets.
    
    <caching>
      <profiles>
        <add extension=".gif" policy="CacheUntilChange" kernelCachePolicy="CacheUntilChange" duration="0.00:30:00" location="Any" />
        <add extension=".png" policy="CacheUntilChange" kernelCachePolicy="CacheUntilChange" duration="0.00:01:00" location="Any" />
        <add extension=".js" policy="CacheUntilChange" kernelCachePolicy="CacheUntilChange" duration="0.00:01:00" location="Any" />
        <add extension=".css" policy="CacheUntilChange" kernelCachePolicy="CacheUntilChange" duration="0.00:01:00" location="Any" />
        <add extension=".jpg" policy="CacheUntilChange" kernelCachePolicy="CacheUntilChange" duration="0.00:01:00" location="Any" />
        <add extension=".jpeg" policy="CacheUntilChange" kernelCachePolicy="CacheUntilChange" duration="0.00:01:00" location="Any" />
        <add extension=".txt" policy="CacheUntilChange" kernelCachePolicy="CacheUntilChange" duration="0.00:01:00" location="Any" />
      </profiles>
    </caching>


Last but not least, create a Base Controller and copy the code below to your base controller.

        protected override void OnActionExecuting(ActionExecutingContext filterContext)
        {
                // Code disables caching by browser.
                Response.Cache.SetCacheability(HttpCacheability.NoCache);
                Response.Cache.SetExpires(DateTime.UtcNow.AddHours(-1));
                Response.Cache.SetNoStore();
        }


Hope this helps.