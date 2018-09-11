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

[?](https://digitalcontrol.me/blog/indentity-asp-net-core-2/#)

| --- | --- |
| 1234567891011 | using Microsoft.AspNetCore.Identity; namespace UserAuthentication.Models{    public class ApplicationUser : IdentityUser    {        public string FirstName { get; set; }        public string LastName { get; set; }     }} |

In the same folder Models create one more C# class for our DbContext. Call it ApplicationDbContext a derive it from generic class IdentityDbContext.

[?](https://digitalcontrol.me/blog/indentity-asp-net-core-2/#)

| --- | --- |
| 12345678910111213141516171819 | using Microsoft.AspNetCore.Identity.EntityFrameworkCore;using Microsoft.EntityFrameworkCore;using UserAuthentication.Models; namespace UserAuthentication.Persistence{    public class ApplicationDbContext : IdentityDbContext<ApplicationUser>    {        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)        : base(options)        {        }         protected override void OnModelCreating(ModelBuilder builder)        {            base.OnModelCreating(builder);        }    }} |

And we need to register it in our Startup.cs. Open Startup.cs and in the ConfigureServices method add following code

[?](https://digitalcontrol.me/blog/indentity-asp-net-core-2/#)

| --- | --- |
| 123456 | services.AddDbContext<ApplicationDbContext>(options =>                                                                options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));             services.AddIdentity<ApplicationUser, IdentityRole>()                                                                .AddEntityFrameworkStores<ApplicationDbContext>()                                                                .AddDefaultTokenProviders(); |

Add the following line of code inside method Configure

[?](https://digitalcontrol.me/blog/indentity-asp-net-core-2/#)

| --- | --- |
| 1 | app.UseAuthentication(); |

Now we have to set our connection string in appsettings.json

[?](https://digitalcontrol.me/blog/indentity-asp-net-core-2/#)

| --- | --- |
| 123 | "ConnectionStrings": {    "DefaultConnection": "Server=localhost;Database=UserAuthentication;Trusted_Connection=True;”} |

Before creating database you should install a few Entity Frameword packages.

* **dotnet add package Microsoft.EntityFrameworkCore.SqlServer**
* **dotnet add package Microsoft.EntityFrameworkCore.Tools**
* **dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design**

And if you are using Visual Studio Code you need to add this code in your .csproj file

[?](https://digitalcontrol.me/blog/indentity-asp-net-core-2/#)

| --- | --- |
| 123 | <ItemGroup>    <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.0" /></ItemGroup> |

Now you can create your database just with running commands **dotnet ef migrations add Initial** and **dotnet ef database update**

When you open your SQL Server you can see that your database UserAuthentication has more than just one table in your database. Those tables are created by ASP.NET Core Indentity and in AspNetUsers table there are two new columns: FirstName and SecondName.

For authentication we will use JWT (JSON Web Token). JSON Web Token is a JSON-based open standard (RFC 7519) for creating access tokens that assert some number of claims. For example, a server could generate a token that has the claim “logged in as admin” and provide that to a client. The client could then use that token to prove that it is logged in as admin. The tokens are signed by the server’s key, so the client and server are both able to verify that the token is legitimate.

So put the following code inside your Stratup.cs ConfigureServices method

[?](https://digitalcontrol.me/blog/indentity-asp-net-core-2/#)

| --- | --- |
| 12345678910111213141516 | services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)            .AddCookie()            .AddJwtBearer(jwtBearerOptions =>            {                jwtBearerOptions.TokenValidationParameters = new TokenValidationParameters()                {                    ValidateActor = false,                    ValidateAudience = false,                    ValidateLifetime = true,                    ValidateIssuerSigningKey = true,                    ValidIssuer = Configuration\["Token:Issuer"\],                    ValidAudience = Configuration\["Token:Audience"\],                    IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes                                                       (Configuration\["Token:Key"\]))                };            }); |

One more thing before we create controller, we should create api resources (view models) for login and register. So create new folder in the root of application and create two C# classes inside:

LoginApiResource

[?](https://digitalcontrol.me/blog/indentity-asp-net-core-2/#)

| --- | --- |
| 123456789101112131415 | using System.ComponentModel.DataAnnotations; namespace UserAuthentication.ApiResources{    public class LoginApiResource    {        \[Required\]        \[EmailAddress\]        public string Email { get; set; }         \[Required\]        \[DataType(DataType.Password)\]        public string Password { get; set; }    }} |

And RegisterApiResource

[?](https://digitalcontrol.me/blog/indentity-asp-net-core-2/#)

| --- | --- |
| 1234567891011121314151617181920212223242526 | using System.ComponentModel.DataAnnotations; namespace UserAuthentication.ApiResources{    public class RegisterApiResource    {        \[Required\]        public string FirstName { get; set; }         \[Required\]        public string LastName { get; set; }         \[Required\]        \[EmailAddress\]        public string Email { get; set; }         \[Required\]        \[StringLength(100, ErrorMessage = "The {0} must be at least {2} and at max {1} characters long.", MinimumLength = 6)\]        \[DataType(DataType.Password)\]        public string Password { get; set; }         \[DataType(DataType.Password)\]        \[Compare("Password", ErrorMessage = "The password and confirmation password do not match.")\]        public string ConfirmPassword { get; set; }    }} |

Now it’s time to create controller for accounts.  
Inside Controllers folder create new C# class,name it AccountsController and create methods for Login, Logout, Register and GetToken method.

[?](https://digitalcontrol.me/blog/indentity-asp-net-core-2/#)

| --- | --- |
| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138 | using System;using System.IdentityModel.Tokens.Jwt;using System.Security.Claims;using System.Text;using System.Threading.Tasks;using Microsoft.AspNetCore.Authentication.JwtBearer;using Microsoft.AspNetCore.Authorization;using Microsoft.AspNetCore.Identity;using Microsoft.AspNetCore.Mvc;using Microsoft.Extensions.Configuration;using Microsoft.IdentityModel.Tokens;using UserAuthentication.ApiResources;using UserAuthentication.Models; namespace UserAuthentication.Controllers{    \[Authorize(AuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)\]    \[Route("api/account")\]    \[Produces("application/json")\]    public class AccountController : Controller    {        private readonly UserManager<ApplicationUser> _userManager;        private readonly SignInManager<ApplicationUser> _signInManager;        private readonly IConfiguration _configuration;         public AccountController(            UserManager<ApplicationUser> userManager,            SignInManager<ApplicationUser> signInManager,            IConfiguration configuration            )        {            _userManager = userManager;            _signInManager = signInManager;            this._configuration = configuration;        }         \[HttpPost("login")\]        \[AllowAnonymous\]        public async Task<IActionResult> Login(\[FromBody\] LoginApiResource model)        {            if (ModelState.IsValid)            {                var result = await _signInManager.PasswordSignInAsync(model.Email, model.Password, isPersistent: true, lockoutOnFailure: false);                if (result.Succeeded)                {                    return await GetToken(model);                }                else                {                    ModelState.AddModelError(string.Empty, "Invalid login attempt.");                    return BadRequest(model);                }            }             return BadRequest(ModelState);        }         \[HttpPost("Logout")\]        public async Task<IActionResult> Logout()        {            await _signInManager.SignOutAsync();            return Ok("Logged out");        }         \[HttpPost("register")\]        \[AllowAnonymous\]        public async Task<IActionResult> Register(\[FromBody\] RegisterApiResource model)        {            if (ModelState.IsValid)            {                var user = new ApplicationUser { UserName = model.Email, Email = model.Email, FirstName = model.FirstName, LastName = model.LastName };                var result = await _userManager.CreateAsync(user, model.Password);                if (result.Succeeded)                {                    await _signInManager.SignInAsync(user, isPersistent: false);                    return Ok(user);                }                AddErrors(result);            }             return BadRequest(ModelState);        }         private void AddErrors(IdentityResult result)        {            foreach (var error in result.Errors)            {                ModelState.AddModelError(string.Empty, error.Description);            }        }         \[HttpGet\]        public IActionResult GetSecuredData()        {            return Ok("Secured data "  + User.FindFirst(ClaimTypes.NameIdentifier).Value);        }         private async Task<IActionResult> GetToken(LoginApiResource model)        {            var user = await _userManager.FindByEmailAsync(model.Email);                if (user != null)                {                     var result = await _signInManager.CheckPasswordSignInAsync                                    (user, model.Password, lockoutOnFailure: false);                     if (!result.Succeeded)                    {                        return Unauthorized();                    }                     var claims = new\[\]                    {                        new Claim(JwtRegisteredClaimNames.Sub, model.Email),                        new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())                    };                     var token = new JwtSecurityToken                    (                        issuer: _configuration\["Token:Issuer"\],                        audience: _configuration\["Token:Audience"\],                        claims: claims,                        expires: DateTime.UtcNow.AddDays(60),                        notBefore: DateTime.UtcNow,                        signingCredentials: new SigningCredentials(new SymmetricSecurityKey                                    (Encoding.UTF8.GetBytes(_configuration\["Token:Key"\])),                                SecurityAlgorithms.HmacSha256)                    );                     return Ok(new JwtSecurityTokenHandler().WriteToken(token) );                }                else                {                    return BadRequest();                }        }    }    } |

And we need to save somewhere the key for coding and decoding tokens. It’s the best approach to save it in appsettings.json like this

[?](https://digitalcontrol.me/blog/indentity-asp-net-core-2/#)

| --- | --- |
| 12345 | "Token": {    "Issuer": "issuer",    "Audience": "audience",    "Key": "2746EF6C-9858-4C8D-935E-20CC6EBB80A2"  } |

You can test now your app using Postman. Method: POST, url: “/api/account/register”

![](https://digitalcontrol.me/wp-content/uploads/2017/12/identity1-1024x262.png =1024x262)

And this is a response

![](https://digitalcontrol.me/wp-content/uploads/2017/12/identity2-1024x373.png =1024x373)

Status is 200 Ok.

If your register form isn’t valid you will get response status 400 Bad Request and message with your errors.

Now when you are registered, you can login. For that you need only your email and password. Method: POST, url: “/api/account/login”

![](https://digitalcontrol.me/wp-content/uploads/2017/12/identity3-1024x189.png =1024x189)

And the response is our JWT.

![](https://digitalcontrol.me/wp-content/uploads/2017/12/identity4-1024x150.png =1024x150)

In our controller there is one method called GetSecuredData() for testing our tokens. You can get that data only if you are logged in. Let’s try that in Postman. After you log in, copy token that you get from server and paste it in header of request. Key is Authorization and Value is “Bearer (your token here)” with one space between word Bearer and your token. Like this

![](https://digitalcontrol.me/wp-content/uploads/2017/12/identity7-1024x245.png =1024x245)

Don’t forget to set method GET and change url to “/api/account”

![](https://digitalcontrol.me/wp-content/uploads/2017/12/identity6-1024x130.png =1024x130)