# Hotel Guest Check-in / Check-out Monitoring System

ASP.NET Core MVC laboratory exercise — server-rendered, in-memory data, no
database, no JavaScript required for core functionality.

## 1. Scope

Reception staff can:

- Register a staff account and log in with cookie authentication.
- Check in a new guest (Guest Number is auto-generated as `G-0001`, `G-0002`, ...).
- View all guests in a monitoring table (checked-in **and** historical checked-out guests).
- View full details of a guest.
- Edit guest information while the guest is still checked in.
- Check a guest out, which records the current date/time and flips status to `Checked Out`.
- Search guests by Guest Number, Name, or Room Number.

Out of scope (per the handout): no database, no JavaScript-driven UI logic
(client-side jQuery validation is the only script, provided by the standard
ASP.NET Core scaffolding, and is optional).

## 2. Design

| Concern | Approach |
|---|---|
| Architecture | ASP.NET Core MVC (.NET 8), 3 controllers: `HomeController`, `AccountController`, `GuestController` |
| Data access | Repository Pattern — `IUserRepository` / `UserRepository`, `IGuestRepository` / `GuestRepository`, backed by `static List<T>` in-memory storage, registered as singletons via DI |
| Model binding | Strongly-typed DTOs bound from Razor forms (`RegisterDto`, `LoginDto`, `GuestCreateDto`, `GuestEditDto`) |
| Validation | Data Annotations (`Required`, `EmailAddress`, `Phone`, `Compare`, `Range`, `Display`, `DataType`, `StringLength`) + `ModelState.IsValid` checks in every POST action |
| Authentication | Cookie Authentication (`Microsoft.AspNetCore.Authentication.Cookies`), `[Authorize]` on `GuestController`, `[AllowAnonymous]` on register/login |
| Password storage | SHA-256 hash (sufficient for this in-memory lab; a production app should use ASP.NET Core Identity's salted hasher) |
| UI | Razor views + Bootstrap 5 (via CDN) + Tag Helpers (`asp-controller`, `asp-action`, `asp-for`, `asp-validation-for`, etc.) |
| Search | `GuestRepository.Search(term)` filters by Guest Number, First/Last/Full Name, or Room Number (case-insensitive substring match) |

### Entities

- **User**: `Id, FirstName, LastName, Email, Username, PasswordHash`
- **Guest**: `Id, GuestNumber, FirstName, LastName, ContactNumber, RoomNumber, NumberOfGuests, CheckInDateTime, ExpectedCheckOutDate, ActualCheckOutDateTime, Status, Notes`

### Project layout

```
HotelGuestMonitoring/
├── Controllers/        HomeController, AccountController, GuestController
├── DTOs/                RegisterDto, LoginDto, GuestCreateDto, GuestEditDto
├── Models/              User, Guest (+ GuestStatus enum)
├── Repositories/        IUserRepository/UserRepository, IGuestRepository/GuestRepository
├── Views/
│   ├── Account/         Login.cshtml, Register.cshtml
│   ├── Guest/            Index.cshtml, Create.cshtml, Edit.cshtml, Details.cshtml, CheckOut.cshtml
│   ├── Home/             Index.cshtml
│   └── Shared/           _Layout.cshtml, _ValidationScriptsPartial.cshtml, Error.cshtml
├── wwwroot/css/site.css
└── Program.cs
```

## 3. Implement / Run

Requirements: .NET 8 SDK.

```bash
cd HotelGuestMonitoring
dotnet restore
dotnet run
```

Then open the URL shown in the console (defaults to launching at
`/Account/Login`). Register a staff account, log in, and use **Guest
Monitoring** / **Check-in Guest** in the nav bar.

> Note: data is stored in-memory only (`static List<T>`), so it resets every
> time the application restarts. This is intentional per the lab requirements
> (no database).

## 4. End-to-End User Story

1. Register a staff account (`/Account/Register`).
2. Log in (`/Account/Login`).
3. Land on **Guest Monitoring** (`/Guest`).
4. Check in a guest (`/Guest/Create`).
5. See the guest appear in the monitoring table with status **Checked In**.
6. Edit the guest if needed (`/Guest/Edit/{id}`) — only allowed while checked in.
7. Check the guest out (`/Guest/CheckOut/{id}`) — records `ActualCheckOutDateTime`, flips status to **Checked Out**.
8. Monitoring list now shows both check-in and check-out times for that guest, and the record remains visible for historical monitoring.

## 5. Attached Handout

The original PDF problem statement is included at the repository root:
`9__Hotel_Guest_Checkin_Checkout_Detailed_Requirements.pdf`.
