# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Foodlovers is a .NET 9.0 Minimal API backend system for a travel and food booking platform. It uses MySQL for data persistence, session-based authentication, and Swagger for API documentation.

## Building and Running

**Build the project:**
```bash
dotnet build
```

**Run the application:**
```bash
dotnet run
```

The API will be available at `http://localhost:5240` (or the port shown in console output).

**Access Swagger UI:**
Navigate to `http://localhost:5240/swagger` in your browser.

**Reset database to seed data:**
Send a DELETE request to `/db` endpoint. This is useful for resetting the test environment.

## Project Architecture

### File Structure

The codebase uses a flat, modular architecture where each resource has its own class file:

- **Program.cs** - Entry point. Configures services (sessions, Swagger, database), and maps all REST endpoints.
- **Config.cs** - Simple configuration wrapper holding the MySQL connection string.
- **Login.cs** - Session-based authentication for users and admins.
- **Users.cs** - CRUD operations for user management.
- **Bookings.cs** - Handles all booking operations (create, update, delete, view).
- **Searchings.cs** - Search and filter functionality for packages, hotels, and facilities.

### Database Connection

The MySQL connection string is hardcoded in Program.cs:
```csharp
Config config = new("server=127.0.0.1;uid=foodlovers;pwd=foodlovers;database=foodlovers");
```

All database operations use `MySqlHelper` from the `MySql.Data` package.

### Authentication Pattern

Sessions store authentication state:
- **User login**: `ctx.Session.SetInt32("user_id", id)` via POST `/login`
- **Admin login**: `ctx.Session.SetInt32("admin_id", id)` via POST `/login/admin`
- **Check login**: `Login.Get(ctx)` or `Login.GetAdmin(ctx)` checks for session keys
- **Logout**: `ctx.Session.Clear()` via DELETE `/login`

Endpoints validate session state before allowing operations. For example:
- Admin-only endpoints check for `admin_id` in session
- User-owned resources (bookings) verify ownership against `user_id` in session

### Data Transfer Objects (DTOs)

Each resource class defines DTOs as `record` types at the top of the file. These control request/response shapes for different endpoints. For example, in Bookings.cs:
- `GetAllData` - Response for GET `/bookings` (admin view)
- `Post_Args` - Request body for POST `/bookings` (create booking)
- `Receipt` - Response from database view for total cost calculation

### Database Schema Key Points

**Core tables:**
- `users`, `admins` - Authentication
- `countries`, `destinations` - Geographic data
- `trip_packages`, `stops` - Travel packages and their destinations
- `hotels`, `rooms`, `room_types` - Accommodation
- `facilities`, `accommodation_facilities` - Hotel amenities
- `bookings`, `booking_stops`, `booked_rooms` - Booking data with relationships

**Important view:**
- `receipt` - Automatically calculates total booking cost (package price + room costs)

### Endpoint Organization

Endpoints are mapped in Program.cs by resource:

**Login/Auth:**
- GET `/login` - Check login status
- POST `/login`, POST `/login/admin` - Login endpoints
- DELETE `/login` - Logout

**Users (CRUD):**
- GET `/users` (admin only), GET `/users/{id}`
- POST `/users`, PUT `/users/{id}` (admin), DELETE `/users/{id}`

**Bookings:**
- POST `/bookings` - Create (user must be logged in)
- GET `/bookings` - View all (admin only)
- GET `/bookings/user` - User's own bookings
- GET `/bookings/{id}/totalcost` - Calculate cost via receipt view
- PUT `/bookings/{id}`, DELETE `/bookings/{id}` - Modify (owner only)

**Search/Filtering:**
- GET `/packages` - Search packages with filters (country, city, maxPrice, search)
- GET `/searchings/SuggestedCountry` - Recommendations based on country
- GET `/searchings/customizedPackage` - Build custom package
- GET `/hotels` - Filter hotels (country, dates, facilities, stars, distance)
- GET `/admin/hotels`, `/admin/trips`, `/admin/facilities` - Admin views

**Database Reset:**
- DELETE `/db` - Resets database to seed data

### Common Patterns

**Query parameters for filtering:**
Many GET endpoints accept optional query parameters for filtering (e.g., `/packages?country=Italy&maxPrice=1000`).

**Nested data structures:**
Bookings contain stops, stops contain rooms. This is handled via nested DTOs (e.g., `Post_Args` contains `List<StopSelection>`, which contains `List<RoomSelection>`).

**Ownership validation:**
Booking endpoints verify that the logged-in user owns the resource before allowing modifications.

## Development Guidelines

**Branch naming:**
Use `feature/` prefix for feature branches.

**Variable naming:**
No single-character variable names. Use descriptive names (e.g., `user` not `u`).

**Pull requests:**
- Only create PRs when the feature is complete
- Only merge into `main` branch

**Code understanding:**
Do not implement code you don't understand (especially AI-generated code).
