 Movie Night Application Documentation

## Purpose and Intent
Movie Night is an ASP.NET Core MVC web application that lets users search the RapidAPI-powered OMDb alternative service for movie information. The goal is to deliver a playful, movie-themed experience that highlights the essentials of a title—name, year, lead actor, and genre—while demonstrating how to wire up a typed HttpClient, server-side view models, and themed Razor views in a minimal codebase.

## Architectural Overview
The solution follows the standard MVC layout used by ASP.NET Core:

- **Program bootstrap** (`Program.cs`): Configures MVC services, registers a named `HttpClient` for the RapidAPI host, and wires configuration binding for API credentials.
- **Controllers** (`Controllers/MoviesController.cs`): Accepts search queries, coordinates service calls, and passes `MovieSearchViewModel` data to the Razor view. Action methods are lean and rely on the service layer for external communication.
- **Services** (`Services/MovieApiService.cs`, `Services/RapidApiOptions.cs`): Encapsulate all HTTP requests to the movie database, including search and detail lookups. `RapidApiOptions` binds the API key and host configuration, while `MovieApiService` shapes responses into presentation-friendly models.
- **Models** (`Models/SearchResponse.cs`, `Models/MovieDetailResponse.cs`): DTOs that deserialize the RapidAPI/OMDb JSON payloads for search results and detailed movie metadata (actors, genre, plot, etc.).
- **View models** (`ViewModels/MovieSearchViewModel.cs`, `ViewModels/MovieResultViewModel.cs`): UI-facing types that combine the search term, result collection, and any error messaging needed by the Razor view.
- **Views** (`Views/Movies/Index.cshtml` plus layout and shared imports): Razor markup that renders the search box, result cards, and themed layout chrome. The accompanying stylesheet (`wwwroot/css/site.css`) provides neon-inspired gradients, card hover states, and responsive layout rules.

### Data Flow
1. Users submit a query from the Movies page.
2. `MoviesController` calls `MovieApiService.SearchAsync`, which issues a search request to RapidAPI. For each hit, it enriches the data with a detail call to retrieve lead actor, genre, and plot fields.
3. The aggregated `MovieResultViewModel` instances are returned to the view to render cards with title, year, cast lead, genre, and plot summary. Errors from the API or empty results are surfaced to the user via view model flags and friendly messaging.

## API Integration Details
- **Endpoint**: `https://movie-database-alternative.p.rapidapi.com` (OMDb alternative).
- **Authentication**: Uses RapidAPI headers (`x-rapidapi-key` and `x-rapidapi-host`) set on the configured `HttpClient`.
- **Configuration**: The API key is read from `appsettings.json` (`RapidApi:Key`) or the `RapidApi__Key` environment variable. `RapidApiOptions` binds these values at startup so the service can inject them when building requests.
- **Error handling**: The service checks HTTP status codes, handles `Response == "False"` payloads from the API, and records user-friendly errors in the view model instead of throwing, keeping controller logic simple.

## Functional Highlights
- **Search experience**: Users can type any movie title and receive a set of stylized cards showing title, year, lead actor, genre, and a short plot line.
- **Movie-night aesthetic**: The Movies view blends neon gradients, marquee-inspired headers, and subtle animations to make the utilitarian search feel thematic and fun.
- **Responsive layout**: Cards and hero sections adapt to different viewport sizes, ensuring readability on laptops and tablets.
- **Extensible service layer**: The `MovieApiService` centralizes HTTP behavior, making it easy to adjust query parameters, add caching, or extend to additional endpoints without touching controllers or views.

## Solution Structure Reference
- `backend.sln` — Visual Studio solution binding the MVC project for easy IDE launch.
- `backend.csproj` — Project configuration targeting .NET 8.0 with Razor support and static file serving.
- `Program.cs` — Service registration, middleware pipeline (static files, routing), and endpoint mapping to controllers.
- `Controllers/MoviesController.cs` — Routes `/Movies` requests and orchestrates search/display logic.
- `Services/MovieApiService.cs` — Performs search and detail lookups, shaping results for the UI.
- `Models/*.cs` — DTOs for API payloads.
- `ViewModels/*.cs` — View-facing models that carry search terms, results, and errors.
- `Views/Movies/Index.cshtml` & `wwwroot/css/site.css` — Razor view and styling that render the movie-themed experience.

## Nice-to-Have Improvements and Future Development
- **Client-side enhancements**: Add lightweight interactivity (e.g., live search suggestions, loading skeletons) via unobtrusive JavaScript.
- **Caching and rate limiting**: Introduce response caching for recent searches and graceful handling of RapidAPI quota limits.
- **Accessibility upgrades**: Improve keyboard navigation, focus management, and ARIA labeling for screen-reader support.
- **Resilient error states**: Show offline/maintenance banners and retry options when the API is unreachable.
- **Search refinements**: Add filters (year, genre, actor) and pagination controls, plus sorting by relevance or release date.
- **Detail pages**: Create a dedicated movie detail view with richer metadata, similar titles, and trailer links.
- **Observability**: Add logging scopes and metrics for API latency, error rates, and user search behavior to guide optimizations.
- **Deployment readiness**: Provide containerization (Dockerfile), CI workflows, and environment-specific configurations for staging/production.