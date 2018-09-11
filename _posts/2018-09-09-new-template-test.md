---
title: New template test
layout: post
date: 2018-09-09 22:00:00 +0000
firstTitle: This is a test
firstParagraph: Let's see if it is working
secondTitle: This is a new template
secondParagraph: Based on our first post
category:
- New category
thirdTitle: ''
thirdParagraph: ''
firstImage: ''
secondImage: ''
thirdImage: ''

---
## Indentity in Asp.Net Core 2

In this tutorial you will learn how to use Asp.Net Core 2 Identity to implement user authentication and authorization.

ASP.NET Core Identity is the new membership system that comes with .NET core. Unlike the previous version it uses claims based authentication which is one of the biggest changes from the previous versions of ASP.NET Identity.

A good thing about ASP.NET Core is it is open source and if needed, we can investigate the source code to understand how it works. ASP.NET Core Identity source code is available in this github repo – [https://github.com/aspnet/Identity](https://github.com/aspnet/Identity "https://github.com/aspnet/Identity")

ASP.NET Core Identity has implemented some APIs (SignInManager, UserManager ,RoleManager, etc.) which simplifies the interactions with the identity objects. When working in an ASP.NET Core project dependency injection will provide the objects for these classes so that we can use those.

Let’s create new webapi ASP.NET Core project. Create folder UserAuthentication, and inside that folder run command dotnet new webapi.  
Create folder Models in root of your project and create a C# class ApplicationUser derived from IdentityUser from namespace Microsoft.AspNetCore.Identity.

![](/uploads/Capture.PNG)