---
permalink: /mocking-controller-user
title: Mocking Controller.User 
layout: post
date: 2009-09-17 21:26:42
published: true
tags: testing
---


I’m currently working on my first [ASP.NET MVC](http://www.asp.net/mvc/) project. Naturally, I’m writing a good number of unit tests. I ran into a problem tonight with mocking [Controller.User](http://msdn.microsoft.com/en-us/library/system.web.mvc.controller.user.aspx). Thankfully, someone at [Stack Overflow](http://stackoverflow.com/) had already asked a [question about this](http://stackoverflow.com/questions/1314370/how-to-setup-iprincipal-for-a-mockup). I took [Bruno Reis’ answer](http://stackoverflow.com/questions/1314370/how-to-setup-iprincipal-for-a-mockup/1314472#1314472):

``` csharp
    var principal = new Moq.Mock<IPrincipal>();
    // ... mock IPrincipal as you wish
    
    var httpContext = new Moq.Mock<HttpContextBase>();
    httpContext.Setup(x => x.User).Returns(principal.Object);
    // ... mock other httpContext's properties, methods, as needed
    
    var reqContext = new RequestContext(httpContext.Object, new RouteData());
    
    // now create the controller:
    var controller = new MyController();
    controller.ControllerContext =
        new ControllerContext(reqContext, controller);
```

and I rewrote it using NUnit.Mocks and wrapped it into an implementation of HttpContextBase:

``` csharp
    /// <summary>
    /// A mock <see cref="HttpContextBase"/> that implements <see cref="HttpContextBase.User"/>.
    /// </summary>
    public class MockHttpContext : HttpContextBase
    {
        private IPrincipal _user;
    
        public MockHttpContext()
        {
            DynamicMock identity = new DynamicMock(typeof (IIdentity));
            identity.ExpectAndReturn("get_Name", "testUser");
    
            DynamicMock user = new DynamicMock(typeof (IPrincipal));
            user.ExpectAndReturn("get_Identity", identity.MockInstance);
    
            _user = (IPrincipal) user.MockInstance;
        }
    
        public override IPrincipal User
        {
            get { return _user; }
            set { _user = value; }
        }
    }
```

Now mocking Controller.User is as easy as this:

``` csharp
    // create an instance of RequestContext using MockHttpContext.
    RequestContext requestContext =
        new RequestContext(new MockHttpContext(), new RouteData());
    
    // initialize the controller's ControllerContext
    _controller.ControllerContext = new ControllerContext(requestContext, _controller);
```
