# DotNet And C#

## Brainfarts

## Controller responses

Add response field to controller

```C#
private readonly ResponseDto _response = new();
```

```c#
public class ResponseDto
{
    public object? Result { get; set; }
    public bool IsSucces { get; set; } = true;
    public string Message { get; set; } = "";
} 
```



## Entity Framework

### Cheatsheet

| Command                                                      | Description               |
| ------------------------------------------------------------ | ------------------------- |
| `dotnet ef database update --project Mango.Services.CouponAPI` | Update DB with migrations |
| `dotnet ef migrations add InitialCreate --project Mango.Services.CouponAPI` | Add a migration           |
| `dotnet tool update --global dotnet-ef`                      | Update Entity Framework   |
| `dotnet ef migrations remove`                                | Remove last migration     |
|                                                              |                           |



### Coding

#### Add Database Context and Automatically apply migrations

appsettings.cs
```json
  "ConnectionStrings": {
    "DefaultConnection" :"Host=localhost; Database=postgres; Username=mango; Password=mango"
  }
```

program.cs
```c#
// Add services to the container.
builder.Services.AddDbContext<AppDbContext>();

... 

ApplyMigrations();
app.Run();


void ApplyMigrations()
{
    using (var scope = app.Services.CreateScope())
    {
        var _db = scope.ServiceProvider.GetRequiredService<AppDbContext>();

        if (_db.Database.GetPendingMigrations().Count() > 0)
        {
            _db.Database.Migrate();
        }
    }
}
```

AppDbContext.cs
```c#
public class AppDbContext : DbContext
{
    private readonly IConfiguration Configuration;

    public DbSet<Coupon> Coupons { get; set; }

    public AppDbContext(DbContextOptions<AppDbContext> options, IConfiguration configuration) : base(options)
    {
        Configuration = configuration;
    }

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseNpgsql(Configuration.GetConnectionString("DefaultConnection"));
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<Coupon>().HasData(new Coupon
        {
            CouponId = 1,
            CouponCode = "20%OFF",
            DiscountAmmount = 20,
            MinAmount = 40
        });
    }
}
```