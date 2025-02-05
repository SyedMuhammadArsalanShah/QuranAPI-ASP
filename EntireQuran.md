# QuranAPI-ASP

# **📌 Beginner's Guide: Fetching Data from an External API in ASP.NET Core MVC**
We will **display Surah names from the Quran API** in an **ASP.NET Core MVC application** and add functionality to **read a particular Surah** by clicking a "Read" button.

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

public class SurahDetailsResponse
{
    public SurahDetails Data { get; set; }
}

public class SurahDetails
{
    public int Number { get; set; }
    public string Name { get; set; }
    public string EnglishName { get; set; }
    public string EnglishNameTranslation { get; set; }
    public List<Ayah> Ayahs { get; set; }
}

public class Ayah
{
    public int Number { get; set; }
    public string Text { get; set; }
}
```

✅ **Explanation:**  
- `QuranResponse` holds the list of Surahs from the API.  
- `SurahDetailsResponse` holds detailed information about a specific Surah.  
- Each **Surah** and **Ayah** has relevant properties.

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

    public async Task<SurahDetails> GetSurahDetailsAsync(int surahNumber)
    {
        var response = await _httpClient.GetAsync($"https://api.alquran.cloud/v1/surah/{surahNumber}");

        if (response.IsSuccessStatusCode)
        {
            var jsonData = await response.Content.ReadAsStringAsync();
            var result = JsonSerializer.Deserialize<SurahDetailsResponse>(jsonData, new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
            return result?.Data;
        }

        return null;
    }
}
```

✅ **Explanation:**  
- `GetAllSurahsAsync()` fetches the list of Surahs.  
- `GetSurahDetailsAsync(int surahNumber)` fetches details of a specific Surah.

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
- Registers **QuranService** for API calls.  
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

    public async Task<IActionResult> Read(int id)
    {
        var surahDetails = await _quranService.GetSurahDetailsAsync(id);
        if (surahDetails == null)
        {
            return NotFound();
        }
        return View(surahDetails);
    }
}
```

✅ **Explanation:**  
- `Index()` fetches all Surahs and displays them.  
- `Read(int id)` fetches and displays details of a specific Surah.

---

## **Step 7: Create Views to Display the Surahs**
The **views** are what the user sees on the screen.

### **1. Create the Index View**

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
            <th>Action</th>
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
                <td>
                    <a class="btn btn-primary" href="@Url.Action("Read", "Quran", new { id = surah.Number })">Read</a>
                </td>
            </tr>
        }
    </tbody>
</table>
```

### **2. Create the Read View**

1. Inside the **Views** → **Quran** folder, create a file called `Read.cshtml`.
2. Add this code:

```html
@model SurahDetails

@{
    ViewData["Title"] = "Read Surah - " + Model.EnglishName;
}

<h2>@Model.Name - @Model.EnglishName (@Model.EnglishNameTranslation)</h2>

<ul class="list-group">
    @foreach (var ayah in Model.Ayahs)
    {
        <li class="list-group-item">
            <strong>@ayah.Number:</strong> @ayah.Text
        </li>
    }
</ul>
```

✅ **Explanation:**  
- `Index.cshtml` lists all Surahs with a **Read** button.  
- `Read.cshtml` displays Ayahs of the selected Surah.

---

## **Step 8: Run the Application**
1. **Press `Ctrl + F5`** or **Click Run** in Visual Studio.
2. Open your browser and go to:
   ```
   https://localhost:5001/Quran/Index
   ```
3. 🎉 You should see a **table listing all Surahs** with a **Read** button.
4. Click **Read** to view the details of a specific Surah.

---

# **🌟 Summary**
✅ **Created an ASP.NET Core MVC project**  
✅ **Used HttpClient to fetch Quran Surah data**  
✅ **Created a QuranService for API calls**  
✅ **Built a Controller (QuranController.cs)**  
✅ **Created Views to display Surahs and read details**  

---

# **🚀 What's Next?**
Here are some **advanced features** you can add:
1. **Surah Audio Playback** for each Surah.
2. **Search Feature** to find Surahs quickly.
3. **Pagination** for better performance.
4. **Bookmarking** favorite Surahs.
5. **Display Translations** in multiple languages.

---

Happy Coding! 🚀📑

