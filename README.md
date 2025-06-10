#ASP.NET Core Web API with Identity, JWT Authentication & SQL Server
This project demonstrates how to build a secure ASP.NET Core Web API with user registration, login, and role-based authorization using ASP.NET Core Identity and JWT tokens. It connects to a SQL Server database using Entity Framework Core and seeds default roles (Admin and User). The API protects endpoints with JWT-based authentication and role-based authorization.

Features
User registration with role assignment

User login with JWT token generation

Role-based authorization (Admin, User)

Entity Framework Core with SQL Server database

Seed default roles in the database

Secure endpoints with [Authorize] attribute

Clean architecture and easy customization

Getting Started
Prerequisites
.NET 8 SDK

SQL Server instance (local or remote)

Postman or any REST client for testing APIs

Setup
Clone the repository

bash
Copy
Edit
git clone https://github.com/yourusername/AspNetCore-Jwt-Identity-Auth.git
cd AspNetCore-Jwt-Identity-Auth
Update connection string

In appsettings.json, update the SQL Server connection string:

json
Copy
Edit
"ConnectionStrings": {
  "DefaultConnection": "Server=YOUR_SERVER;Database=YourDbName;Trusted_Connection=True;MultipleActiveResultSets=true"
},
"Jwt": {
  "Key": "YourVerySecretKeyHere",
  "Issuer": "YourIssuer",
  "Audience": "YourAudience"
}
Run migrations and update database

Use the CLI to create and apply migrations:

bash
Copy
Edit
dotnet ef migrations add InitialCreate
dotnet ef database update
Run the application

bash
Copy
Edit
dotnet run
API Endpoints
Register a new user
URL: POST /api/auth/register

Body:

json
Copy
Edit
{
  "username": "testuser",
  "email": "testuser@example.com",
  "password": "YourStrongPassword123!"
}
Response: 200 OK if registration succeeds

Login
URL: POST /api/auth/login

Body:

json
Copy
Edit
{
  "username": "testuser",
  "password": "YourStrongPassword123!"
}
Response:

json
Copy
Edit
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
Using the token
Add the JWT token as a Bearer token in the Authorization header to access protected endpoints:

makefile
Copy
Edit
Authorization: Bearer YOUR_JWT_TOKEN
Code Highlights
Role seeding in OnModelCreating:
csharp
Copy
Edit
builder.Entity<IdentityRole>().HasData(
    new IdentityRole { Id = "1", Name = "Admin", NormalizedName = "ADMIN" },
    new IdentityRole { Id = "2", Name = "User", NormalizedName = "USER" }
);
Register endpoint:
csharp
Copy
Edit
[HttpPost("register")]
public async Task<IActionResult> Register([FromBody] RegisterModel model)
{
    var user = new ApplicationUser { UserName = model.Username, Email = model.Email };
    var result = await _userManager.CreateAsync(user, model.Password);
    if (result.Succeeded)
    {
        await _userManager.AddToRoleAsync(user, "User");
        return Ok("User registered");
    }
    return BadRequest(result.Errors);
}
Login endpoint with JWT generation:
csharp
Copy
Edit
[HttpPost("login")]
public async Task<IActionResult> Login([FromBody] LoginModel model)
{
    var user = await _userManager.FindByNameAsync(model.Username);
    if (user != null && await _userManager.CheckPasswordAsync(user, model.Password))
    {
        var token = GenerateJwtToken(user);
        return Ok(new { token });
    }
    return Unauthorized();
}

private string GenerateJwtToken(ApplicationUser user)
{
    var claims = new[]
    {
        new Claim(JwtRegisteredClaimNames.Sub, user.UserName),
        new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
    };

    var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["Jwt:Key"]));
    var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
    var token = new JwtSecurityToken(
        issuer: _config["Jwt:Issuer"],
        audience: _config["Jwt:Audience"],
        claims: claims,
        expires: DateTime.Now.AddHours(1),
        signingCredentials: creds);

    return new JwtSecurityTokenHandler().WriteToken(token);
}
