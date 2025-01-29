# QuranAPI-ASP

# **📌 Beginner's Guide: Fetching Data from an External API in ASP.NET Core MVC**
We will **display Surah names from the Quran API** in an **ASP.NET Core MVC application**.

---

## **Step 1: Create a New ASP.NET Core MVC Project**
1. **Open Visual Studio**.
2. Click **Create a new project**.
3. Choose **ASP.NET Core Web App (Model-View-Controller)**.
4. Click **Next**, enter **Project Name** (`QuranMVC`), and click **Create**.
5. Select **.NET 6/7/8** and click **Create**.

---

## **Step 2: Install Required Package**
We need **HttpClient** to call an API. If not already available, install it:

**Open the NuGet Package Manager Console (Tools → NuGet Package Manager → Console) and run:**
```powershell
dotnet add package Microsoft.Extensions.Http
```

---

## **Step 3: Create a Model to Store Quran Data**
A **model** represents the data from the API.

1. Inside the **Models** folder, create a file called `Surah.cs`.
2. Add the following code:

```csharp
public class QuranResponse
{
    public List<Surah> Data { get; set; }
}

public class Surah
{
    public int Number { get; set; }
    public string Name { get; set; }
    public string EnglishName { get; set; }
    public string EnglishNameTranslation { get; set; }
    public string RevelationType { get; set; }
}
```

✅ **Explanation:**  
- The API response contains a **list of Surahs**, so we use `QuranResponse` to hold the data.  
- Each **Surah** has properties like `Name`, `EnglishName`, `Translation`, etc.  

---

## **Step 4: Create a Service to Fetch Data**
A **service** is used to separate API calls from the controller.

1. Inside the **Services** folder, create a file called `QuranService.cs`.
2. Add the following code:

```csharp
using System.Net.Http;
using System.Text.Json;
using System.Threading.Tasks;
using System.Collections.Generic;

public class QuranService
{
    private readonly HttpClient _httpClient;

    public QuranService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<List<Surah>> GetAllSurahsAsync()
    {
        var response = await _httpClient.GetAsync("https://api.alquran.cloud/v1/surah");

        if (response.IsSuccessStatusCode)
        {
            var jsonData = await response.Content.ReadAsStringAsync();
            var result = JsonSerializer.Deserialize<QuranResponse>(jsonData, new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
            return result?.Data ?? new List<Surah>();
        }

        return new List<Surah>();
    }
}
```

✅ **Explanation:**  
- This class uses `HttpClient` to fetch **Quran Surah data** from the API.  
- It **deserializes** (converts JSON to C# objects).  
- If the API call is successful, it returns **a list of Surahs**.

---

## **Step 5: Register the Service in `Program.cs`**
Now, we need to **register** the service so we can use it in our controller.

1. Open `Program.cs`.
2. Modify it as follows:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();
builder.Services.AddHttpClient<QuranService>(); // Register Quran Service

var app = builder.Build();

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Quran}/{action=Index}/{id?}"
);

app.Run();
```

✅ **Explanation:**  
- `builder.Services.AddHttpClient<QuranService>();` → Registers **QuranService** for API calls.  
- Sets the default **route** to `QuranController`.

---

## **Step 6: Create a Controller**
A **controller** handles user requests and gets data from the **QuranService**.

1. Inside the **Controllers** folder, create a file called `QuranController.cs`.
2. Add this code:

```csharp
using Microsoft.AspNetCore.Mvc;
using System.Threading.Tasks;
using System.Collections.Generic;

public class QuranController : Controller
{
    private readonly QuranService _quranService;

    public QuranController(QuranService quranService)
    {
        _quranService = quranService;
    }

    public async Task<IActionResult> Index()
    {
        var surahs = await _quranService.GetAllSurahsAsync();
        return View(surahs);
    }
}
```

✅ **Explanation:**  
- `QuranController` calls the **QuranService** to fetch Surah data.  
- The **Index() method** fetches the Surahs and sends them to the **View**.  

---

## **Step 7: Create a View to Display the Surahs**
The **view** is what the user sees on the screen.

1. Inside the **Views** → **Quran** folder, create a file called `Index.cshtml`.
2. Add this code:

```html
@model List<Surah>

@{
    ViewData["Title"] = "Quran Surah List";
}

<h2>List of Surahs</h2>

<table class="table table-striped">
    <thead>
        <tr>
            <th>#</th>
            <th>Arabic Name</th>
            <th>English Name</th>
            <th>Translation</th>
            <th>Revelation Type</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var surah in Model)
        {
            <tr>
                <td>@surah.Number</td>
                <td>@surah.Name</td>
                <td>@surah.EnglishName</td>
                <td>@surah.EnglishNameTranslation</td>
                <td>@surah.RevelationType</td>
            </tr>
        }
    </tbody>
</table>
```

✅ **Explanation:**  
- **`@model List<Surah>`** → Passes the list of Surahs from the controller to the view.  
- Displays Surah details in a **table format**.  

---

## **Step 8: Run the Application**
1. **Press `Ctrl + F5`** or **Click Run** in Visual Studio.
2. Open your browser and go to:
   ```
   https://localhost:5001/Quran/Index
   ```
3. 🎉 You should see a **table listing all Surahs** from the API!

---

# **🎯 Summary**
✅ **Created an ASP.NET Core MVC project**  
✅ **Used HttpClient to fetch Quran Surah data**  
✅ **Created a QuranService for API calls**  
✅ **Built a Controller (QuranController.cs)**  
✅ **Created an Index.cshtml view to display data**  

---

# **🚀 What's Next?**
Here are some **advanced features** you can add:
1. **Surah Details Page** (`/Quran/Details/{id}`)
2. **Audio Playback** for each Surah.
3. **Search Feature** to find Surahs quickly.
4. **Pagination** for better performance.

