# **Lesson 7 — Participant Registration, Capacity, and Reliability**

Revision: 2026.10.29.6

# Table of Contents

- [Review of Last Lesson](#review-of-last-lesson)
- [Learning Objectives](#learning-objectives)
- [SRS Requirements and Use Cases](#srs-requirements-and-use-cases)
- [Guided Feature Preview](#guided-feature-preview)
- [Guided Design and Coding](#guided-design-and-coding)
	- [Inspect the inherited solution and establish a baseline](#inspect-the-inherited-solution-and-establish-a-baseline)
	- [Model current Registration and database integrity](#model-current-registration-and-database-integrity)
	- [Create, inspect, and apply the Registration migration](#create-inspect-and-apply-the-registration-migration)
	- [Display live availability on Event Details](#display-live-availability-on-event-details)
	- [Implement the atomic registration decision](#implement-the-atomic-registration-decision)
	- [Add protected registration and cancellation requests](#add-protected-registration-and-cancellation-requests)
	- [Render My Registrations and its navigation](#render-my-registrations-and-its-navigation)
	- [Complete Registration-dependent Organizer rules](#complete-registration-dependent-organizer-rules)
- [Guided Discussion](#guided-discussion)
	- [Registration as a persisted relational association](#registration-as-a-persisted-relational-association)
	- [Derived availability and consistency across views](#derived-availability-and-consistency-across-views)
	- [Authenticated commands, ownership, and temporal eligibility](#authenticated-commands-ownership-and-temporal-eligibility)
	- [Layered validation, relational integrity, and controlled failures](#layered-validation-relational-integrity-and-controlled-failures)
	- [Concurrency and the final-seat invariant](#concurrency-and-the-final-seat-invariant)
	- [Cross-workflow consistency and shared invariants](#cross-workflow-consistency-and-shared-invariants)
	- [Verification strategies and the strength of evidence](#verification-strategies-and-the-strength-of-evidence)
- [Guided Verification and Validation](#guided-verification-and-validation)
	- [Automated](#automated)
		- [Add relational Registration service tests](#add-relational-registration-service-tests)
		- [Extend inherited HTTP and persistence tests](#extend-inherited-http-and-persistence-tests)
		- [Execute and interpret the regression suite](#execute-and-interpret-the-regression-suite)
		- [Observe SQL Server lock coordination with two sessions](#observe-sql-server-lock-coordination-with-two-sessions)
	- [Manual](#manual)
	- [SRS Traceability Matrix Update with Evidence](#srs-traceability-matrix-update-with-evidence)
- [Guided Exercises](#guided-exercises)
	- [Exercise: extend eligibility regression coverage](#exercise-extend-eligibility-regression-coverage)
	- [Exercise: analyze an exact-start boundary](#exercise-analyze-an-exact-start-boundary)
	- [Exercise: challenge a shared capacity invariant](#exercise-challenge-a-shared-capacity-invariant)
	- [Exercise: evaluate relational constraint responsibilities](#exercise-evaluate-relational-constraint-responsibilities)
	- [Exercise: review the migration and repository boundary](#exercise-review-the-migration-and-repository-boundary)
- [Troubleshooting Common Problems](#troubleshooting-common-problems)
- [Checking Your Understanding](#checking-your-understanding)
- [Summary](#summary)
- [Checkpoint](#checkpoint)
- [Next Lesson Preview](#next-lesson-preview)
- [Appendix](#appendix)
	- [Database concurrency control and isolation strategies](#database-concurrency-control-and-isolation-strategies)
		- [Isolation levels and visibility](#isolation-levels-and-visibility)
		- [Pessimistic and optimistic coordination](#pessimistic-and-optimistic-coordination)
		- [Alternative seat-allocation designs](#alternative-seat-allocation-designs)
		- [Lock diagnostics and controlled recovery](#lock-diagnostics-and-controlled-recovery)
	- [Transactions across multiple services and data stores](#transactions-across-multiple-services-and-data-stores)
		- [Coordination and compensation](#coordination-and-compensation)
		- [Delivery, idempotency, and eventual consistency](#delivery-idempotency-and-eventual-consistency)
	- [Registration lifecycle and capacity-model alternatives](#registration-lifecycle-and-capacity-model-alternatives)
		- [Current associations, history, and status models](#current-associations-history-and-status-models)
		- [Derived counts versus materialized counters](#derived-counts-versus-materialized-counters)
		- [Waitlists, holds, and partial reservations](#waitlists-holds-and-partial-reservations)

# Review of Last Lesson

Lesson 6 turned Event lifecycle rules into guarded, server-authoritative state changes. You practiced safe HTTP methods, resource ownership, time-sensitive validation, persistence checks, and the distinction between cancellation and deletion. Its Organizer workflow supports Publish, Cancel, Draft deletion confirmation, and Draft/Published Edit while preserving the public catalog's selected 10/25/50 page size, search, Category filter, and direct Details access for Cancelled Events.

The accepted CEMS solution already includes ASP.NET Core MVC, Razor, SQL Server, EF Core, Identity, `TimeProvider`, a sibling xUnit test project, isolated migrated SQL Server fixtures, and authenticated HTTP regression tests. This lesson adds the persisted Participant/Event relationship needed to enforce registration limits, derive live availability, and finalize previously data-dependent Event rules.

# Learning Objectives

By the end of this lesson, you should be able to:

1. **Design and justify** a persisted association entity using relational identities, foreign keys, uniqueness, and deliberate deletion behavior.
2. **Distinguish** persisted facts from derived application state and evaluate when a displayed value is only a snapshot of shared data.
3. **Apply** authentication, resource ownership, request-method safety, and time-dependent eligibility to protected mutation workflows.
4. **Analyze** complementary application validation, database constraints, and controlled failure outcomes as layers of an integrity strategy.
5. **Explain and prevent** read–check–write races using transactions, resource-level coordination, and authoritative rechecks.
6. **Evaluate** consistency across independently initiated workflows that read and modify the same business invariant.
7. **Design and interpret** automated and manual verification using isolated relational data, controlled time, observable outcomes, and appropriately limited evidence claims.
8. **Maintain** incremental, reviewable changes that preserve earlier application contracts and regression coverage.

# SRS Requirements and Use Cases

The approved CEMS SRS v1.3.3 defines current Registration as a Participant/Event association, with cancellation removing that current association. The following requirements govern registration and capacity management alongside the existing catalog, Identity, and Event lifecycle behavior.

| SRS reference | Required behavior / constraint | Lesson contribution |
| --- | --- | --- |
| `UC-003`; `FR-REG-001`–`FR-REG-005`; `BR-REG-001`–`BR-REG-004` | Authenticated users register once for an eligible Published, not-started Event when a place remains; receive confirmation; never exceed capacity | Add persisted Registration, duplicate uniqueness, guarded POST, transactionally checked capacity, and controlled outcomes |
| `UC-004`; `FR-REG-006`–`FR-REG-008`; `BR-REG-005`–`BR-REG-006` | Participant views only current own Registrations and may cancel an owned Registration before Event start, releasing the place | Add owner-scoped listing, cancellation POST, and count-derived capacity release |
| `FR-SRH-004`; `FR-SRH-008` | Details shows relevant availability, and known Cancelled Details remain directly accessible | Display current and remaining places from persisted rows while retaining Published/Cancelled visibility |
| `FR-EVT-011`; `BR-EVT-008` | Delete an owned Draft only when no current Registrations exist | Add an authoritative registration check to the inherited protected Delete POST |
| `FR-EVT-004`; `FR-EVT-006`; `BR-EVT-009` | Permit eligible owned Event editing while refusing a Published Capacity decrease below current Registration count | Add a count-based floor inside the guarded Published Edit transaction |
| `BR-AUT-001`; `BR-AUT-004`; `NFR-SEC-001`; `NFR-SEC-003` | Enforce server-side authorization for protected operations, reject unauthorized requests, and protect state-changing forms against request forgery | Resolve user identity on the server, scope mutations to authorized actors, and validate antiforgery tokens on POST |
| `NFR-REL-001`–`NFR-REL-003`; `NFR-USA-004` | Durable writes, no partial rejected mutations, dependable capacity correctness, and understandable feedback | Combine database integrity, coordinated writes, expected failure results, and regression checks |
| `AC-004`; `AC-005`; `AC-009`; `AC-010` | Accumulate evidence for registration, data integrity, error handling, and automated critical behavior | Add relational and HTTP regression scenarios plus manual evidence procedures; record observed outcomes before assigning verified status |

**Lesson-level acceptance contract:** In the resulting application, an authenticated Participant can register for an eligible future Published Event once, view and cancel only their own current Registration before its scheduled start, and observe accurate current/remaining capacity. Competing users cannot both acquire the last available place. Draft deletion and Published Capacity Edit use Registration data alongside their inherited ownership and lifecycle guards. 

# Guided Feature Preview

Consider a future Published Event with `Capacity = 2` and no Registrations. A signed-in Participant visits its public Details page and sees **Current registrations: 0** and **Remaining capacity: 2**. Selecting **Register** submits an antiforgery-protected POST. After the redirect, a success message and the updated count appear; **My Registrations** lists the Event. A second registration attempt yields a controlled duplicate response and does not add a row.

A second Participant takes the remaining place. A later eligible request sees **No places are currently available**. If two different users compete for one remaining place, exactly one registration may persist; the unsuccessful request receives ordinary feedback rather than an overbooked Event. Cancelling a current Registration before the Event's scheduled start removes the association and makes that place available again.

An Organizer can still edit their own Published Event, but a proposed Capacity below the number of current Registrations is rejected with a field-level error. Draft deletion is also refused when current Registrations exist. A previously Published and subsequently Cancelled Event remains visible by direct Details URL, but its Register action is unavailable.

> **Screenshot placeholder — live registration availability**
>
> **Illustration needed:** Public Details for an upcoming Published Event after one of two available places has been registered, including its Registration section.
>
> **Expected visible result:** Current registrations 1, remaining capacity 1, and an appropriately rendered Register action or already-registered message.
>
> **Application state:** Development, future Published Event of Capacity 2, one persisted Registration, and authenticated Participant browser.
>
> ![Expected Event Details showing persisted current registrations and remaining capacity](images/lesson-07/live-registration-availability.png)

Each Registration command coordinates its authoritative decision around the Event row:

```text
Authenticated Register POST → transaction → lock Event → recheck current state
    → check duplicate and current count → insert or refuse → commit/rollback → redirect
```

# Guided Design and Coding

## Inspect the inherited solution and establish a baseline

Work from the solution root containing `CampusEvents.slnx`, `src/CampusEvents/`, and `tests/CampusEvents.Tests/`. Continue with the established test project, migrated SQL Server fixture, `FakeTimeProvider`, and authenticated HTTP host. Use the same established Development connection, exact `Organizer`/`Administrator` elevated role names, and authenticated-user Participant baseline. Preserve `EventNotFound()`, `TempData["StatusMessage"]`, `TempData["ErrorMessage"]`, the public 10/25/50 pagination contract, and POST-only Logout.

```bat
git switch -c lesson-07-participant-registration
dotnet restore .\CampusEvents.slnx
dotnet build .\CampusEvents.slnx
```

**Expected checkpoint:** The accepted Lesson 6 application builds and continues to provide public search, filtering, selectable pagination, Identity, Organizer Create/Edit, lifecycle commands, and direct Cancelled Details. Record any actual baseline failures before proceeding.

## Model current Registration and database integrity

A Registration has its own identity and a server-assigned creation timestamp. Its foreign keys connect the existing `ApplicationUser` and `Event` entities. The authenticated `ApplicationUser` represents the Participant actor, using the established account and role model. Create the following entity.

**File: `src/CampusEvents/Models/Registration.cs`**

```csharp
namespace CampusEvents.Models;

public class Registration
{
	public int Id { get; set; }

	public string UserId { get; set; } = string.Empty;

	public ApplicationUser User { get; set; } = null!;

	public int EventId { get; set; }

	public Event Event { get; set; } = null!;

	public DateTime CreatedAt { get; set; }
}
```

Retain the existing user and Event properties. Add only navigation collections; these model one user's Registrations and one Event's Registrations without replacing their existing Organizer or Category relationships.

**File: `src/CampusEvents/Models/ApplicationUser.cs`** — add the collection inside the existing class:

```csharp
public ICollection<Registration> Registrations { get; set; } = new List<Registration>();
```

**File: `src/CampusEvents/Models/Event.cs`** — add the same collection in `Event`:

```csharp
public ICollection<Registration> Registrations { get; set; } = new List<Registration>();
```

**File: `src/CampusEvents/Data/ApplicationDbContext.cs`** — retain all inherited mappings and add the set to the existing context:

```csharp
public DbSet<Registration> Registrations => Set<Registration>();
```

Inside the existing `OnModelCreating` method, after the Event configuration, add:

```csharp
modelBuilder.Entity<Registration>(entity =>
{
	entity.HasOne(registration => registration.User).WithMany(user => user.Registrations)
		.HasForeignKey(registration => registration.UserId).OnDelete(DeleteBehavior.Restrict);

	entity.HasOne(registration => registration.Event).WithMany(eventItem => eventItem.Registrations)
		.HasForeignKey(registration => registration.EventId).OnDelete(DeleteBehavior.Restrict);

	entity.HasIndex(registration => registration.EventId);
	entity.HasIndex(registration => new { registration.UserId, registration.EventId }).IsUnique();
});
```

The EventId index supports Event-scoped counts; the composite unique index makes two current rows for the same User/Event pair invalid even when requests overlap. Restrict foreign keys prevent an unrelated parent deletion from silently cascading to a current Registration. Cancellation removes the current association, as specified in SRS §8.2.

## Create, inspect, and apply the Registration migration

The model and database schema evolve together. Create the migration from the solution root using the established EF Core tool and Development startup configuration:

```bat
dotnet ef migrations add AddParticipantRegistrations -p src\CampusEvents ^
	-s src\CampusEvents -- --environment Development
```

Open the generated `AddParticipantRegistrations` migration and model snapshot. Check that `Up()` creates `Registrations` with `Id`, `UserId`, `EventId`, and `CreatedAt`; foreign keys to `AspNetUsers` and `Events` with restricted deletion; an `EventId` index; and a unique `(UserId, EventId)` index. Check that `Down()` reverses these additions. Confirm that existing Event constraints, Identity tables, and earlier migrations remain intact.

Apply the inspected migration and rebuild:

```bat
dotnet ef database update --project src\CampusEvents --startup-project src\CampusEvents -- --environment Development
dotnet build .\CampusEvents.slnx
```

**Expected observable checkpoint:** SQL Server Object Explorer shows the new table, foreign keys, and unique index. The Registration table initially contains no rows, and the application builds with the earlier Event data intact. All schema changes use migrations.

## Display live availability on Event Details

Capacity is configured on `Event`; current registrations are counted in persisted Registration rows for that Event. Present these values with a view-specific model rather than adding mutable counts to the Event entity.

**File: `src/CampusEvents/ViewModels/EventDetailsViewModel.cs`**

```csharp
using CampusEvents.Models;

namespace CampusEvents.ViewModels;

public class EventDetailsViewModel
{
	public Event EventItem { get; set; } = null!;

	public int CurrentRegistrationCount { get; set; }

	public int RemainingCapacity { get; set; }

	public bool IsCurrentUserRegistered { get; set; }
}
```

Replace **only** the inherited public `Details(int id)` action in `src/CampusEvents/Controllers/EventsController.cs`; retain Index, Manage, lifecycle commands, the existing constructor, and the `EventNotFound()` behavior. Add `using CampusEvents.ViewModels;` when needed. Keep the Published/Cancelled direct-Details predicate and determine whether the authenticated user has a current Registration.

```csharp
[HttpGet]
public async Task<IActionResult> Details(int id)
{
	var eventItem = await _context.Events
		.AsNoTracking()
		.Include(eventItem => eventItem.Category)
		.Include(eventItem => eventItem.Organizer)
		.FirstOrDefaultAsync(eventItem => eventItem.Id == id &&
			(eventItem.Status == EventStatus.Published || eventItem.Status == EventStatus.Cancelled));

	if (eventItem is null) return EventNotFound();

	var currentRegistrationCount = await _context.Registrations.CountAsync(registration =>
				registration.EventId == eventItem.Id);

	var isCurrentUserRegistered = false;

	if (User.Identity?.IsAuthenticated == true)
	{
		var currentUser = await _userManager.GetUserAsync(User);

		if (currentUser is not null)
		{
			isCurrentUserRegistered = await _context.Registrations.AnyAsync(registration =>
						registration.EventId == eventItem.Id && registration.UserId == currentUser.Id);
		}
	}

	var model = new EventDetailsViewModel
	{
		EventItem = eventItem,
		CurrentRegistrationCount = currentRegistrationCount,
		RemainingCapacity = Math.Max(0, eventItem.Capacity - currentRegistrationCount),
		IsCurrentUserRegistered = isCurrentUserRegistered
	};

	return View(model);
}
```

**File: `src/CampusEvents/Views/Events/Details.cshtml`** — replace the previously entity-typed view with the view-model version. Inject the inherited application clock and display current availability and eligibility. Show read-only Registration availability alongside the Event's semantic metadata and status messages.

```cshtml
@using CampusEvents.Models
@inject TimeProvider Clock
@model CampusEvents.ViewModels.EventDetailsViewModel

@{
	ViewData["Title"] = Model.EventItem.Title;
	var now = Clock.GetLocalNow().DateTime;
	var detailsReturnUrl = Url.Action("Details", "Events", new { id = Model.EventItem.Id });
}

<h1>@Model.EventItem.Title</h1>

@if (TempData["StatusMessage"] is string statusMessage)
{
	<p class="status-message" role="status">
		<strong>Success:</strong> @statusMessage
	</p>
}

@if (TempData["ErrorMessage"] is string errorMessage)
{
	<p class="error-message" role="alert">
		@errorMessage
	</p>
}

@if (Model.EventItem.Status == EventStatus.Cancelled)
{
	<p class="error-message" role="status">
		This Event has been cancelled.
	</p>
}

<dl class="event-meta">
	<div>
		<dt>Status</dt>
		<dd>@Model.EventItem.Status</dd>
	</div>
	<div>
		<dt>Category</dt>
		<dd>@Model.EventItem.Category.Name</dd>
	</div>
	<div>
		<dt>Location</dt>
		<dd>@Model.EventItem.Location</dd>
	</div>
	<div>
		<dt>Starts</dt>
		<dd>
			<time datetime="@Model.EventItem.StartDateTime.ToString("yyyy-MM-ddTHH:mm:ss")">
				@Model.EventItem.StartDateTime.ToString("f")
			</time>
		</dd>
	</div>
	<div>
		<dt>Ends</dt>
		<dd>
			<time datetime="@Model.EventItem.EndDateTime.ToString("yyyy-MM-ddTHH:mm:ss")">
				@Model.EventItem.EndDateTime.ToString("f")
			</time>
		</dd>
	</div>
	<div>
		<dt>Organizer</dt>
		<dd>@Model.EventItem.Organizer.DisplayName</dd>
	</div>
	<div>
		<dt>Capacity</dt>
		<dd>@Model.EventItem.Capacity</dd>
	</div>
	<div>
		<dt>Current registrations</dt>
		<dd>@Model.CurrentRegistrationCount</dd>
	</div>
	<div>
		<dt>Remaining capacity</dt>
		<dd>@Model.RemainingCapacity</dd>
	</div>
</dl>

<section aria-labelledby="event-description-heading">
	<h2 id="event-description-heading">Description</h2>
	<p>@Model.EventItem.Description</p>
</section>

<section aria-labelledby="registration-heading">
	<h2 id="registration-heading">Registration</h2>

	@if (Model.EventItem.Status != EventStatus.Published)
	{
		<p>
			Registration is unavailable because this Event
			is not Published.
		</p>
	}
	else if (Model.EventItem.StartDateTime <= now)
	{
		<p>
			Registration is closed because this Event
			has already started.
		</p>
	}
	else if (Model.IsCurrentUserRegistered)
	{
		<p role="status">
			You already have a current Registration
			for this Event.
		</p>

		<a asp-controller="Registrations" asp-action="Mine" class="button-secondary">
			View My Registrations
		</a>
	}
	else if (Model.RemainingCapacity == 0)
	{
		<p role="status">
			No places are currently available.
		</p>
	}
	else if (User.Identity?.IsAuthenticated != true)
	{
		<p>
			Log in before registering for this Event.
		</p>

		<a asp-controller="Account" asp-action="Login" asp-route-returnUrl="@detailsReturnUrl" class="button-primary">
			Login to Register
		</a>
	}
	else
	{
		<p role="status">Places are currently available for registration.</p>
	}
</section>

<p>
	<a asp-controller="Events" asp-action="Index">
		Back to Events
	</a>
</p>
```

**Expected observable checkpoint:** `/Events/Details/{id}` displays configured Capacity and current and remaining counts from persisted Registration rows. An eligible Published Event displays its availability status. A Cancelled Event remains accessible through its direct Details URL and displays its status; a Draft retains the inherited not-found behavior. Check the initial counts against the empty Registration table.

## Implement the atomic registration decision

A check followed by an insert must be coordinated because another user can take the last place in between. Use the accepted SQL Server provider and a narrow service whose responsibility is attempting one Registration. The transaction begins before `UPDLOCK, HOLDLOCK` acquires the target Event row; duplicate and capacity checks then run while that Event lock is held. The service returns a typed outcome for expected refusal, and unexpected database exceptions remain visible to diagnostic handling.

**File: `src/CampusEvents/Services/RegistrationService.cs`**

```csharp
using CampusEvents.Data;
using CampusEvents.Models;
using Microsoft.Data.SqlClient;
using Microsoft.EntityFrameworkCore;

namespace CampusEvents.Services;

public enum RegistrationAttemptStatus
{
	Success,
	EventNotFound,
	Unavailable,
	Duplicate,
	Full
}

public class RegistrationService
{
	private readonly ApplicationDbContext _context;
	private readonly TimeProvider _timeProvider;

	public RegistrationService(ApplicationDbContext context, TimeProvider timeProvider)
	{
		_context = context;
		_timeProvider = timeProvider;
	}

	public async Task<RegistrationAttemptStatus> TryRegisterAsync(string userId, int eventId)
	{
		await using var transaction = await _context.Database.BeginTransactionAsync();

		var eventItem = await _context.Events.FromSqlInterpolated($@"
				SELECT *
				FROM [Events] WITH (UPDLOCK, HOLDLOCK)
				WHERE [Id] = {eventId}").SingleOrDefaultAsync();

		if (eventItem is null)
		{
			await transaction.RollbackAsync();
			return RegistrationAttemptStatus.EventNotFound;
		}

		var now = _timeProvider.GetLocalNow().DateTime;

		if (eventItem.Status != EventStatus.Published || eventItem.StartDateTime <= now)
		{
			await transaction.RollbackAsync();
			return RegistrationAttemptStatus.Unavailable;
		}

		var alreadyRegistered = await _context.Registrations.AnyAsync(registration => registration.UserId == userId &&
					registration.EventId == eventId);

		if (alreadyRegistered)
		{
			await transaction.RollbackAsync();
			return RegistrationAttemptStatus.Duplicate;
		}

		var currentRegistrationCount = await _context.Registrations.CountAsync(registration =>
					registration.EventId == eventId);

		if (currentRegistrationCount >= eventItem.Capacity)
		{
			await transaction.RollbackAsync();
			return RegistrationAttemptStatus.Full;
		}

		var registration = new Registration
		{
			UserId = userId, EventId = eventId, CreatedAt = now
		};

		_context.Registrations.Add(registration);

		try
		{
			await _context.SaveChangesAsync();
			await transaction.CommitAsync();

			return RegistrationAttemptStatus.Success;
		}
		catch (DbUpdateException exception) when (IsUniqueRegistrationViolation(exception))
		{
			await transaction.RollbackAsync();
			_context.ChangeTracker.Clear();

			return RegistrationAttemptStatus.Duplicate;
		}
	}

	private static bool IsUniqueRegistrationViolation(DbUpdateException exception)
	{
		return exception.InnerException is SqlException sqlException && (sqlException.Number == 2601 ||
				sqlException.Number == 2627);
	}
}
```

**File: `src/CampusEvents/Program.cs`** — preserve all existing registrations, Identity services, middleware ordering, seeding, and the existing `TimeProvider` registration. Add `using CampusEvents.Services;` and one scoped dependency registration before `builder.Build()`:

```csharp
builder.Services.AddScoped<RegistrationService>();
```

An attempted Registration can end in `Success`, `Duplicate`, `Unavailable`, `Full`, or `EventNotFound`. The unique-index exception filter handles SQL Server duplicate-key codes 2601 and 2627; other database errors reach normal diagnostics. The accepted transaction coordinates successful registration with capacity updates that use the same Event-row lock discipline.

**Expected observable checkpoint:** The application compiles with the new service and its constructor-injected `TimeProvider`. The scoped service exposes a consistent Registration outcome for the protected controller to translate into feedback.

## Add protected registration and cancellation requests

The controller uses `[Authorize]` because Participant capability is the normal authenticated-user baseline. Each POST supplies an Event or Registration ID; the authenticated principal identifies the actor. Register delegates eligibility and capacity decisions to the transaction-protected service, then maps its typed outcome to an HTTP response and a clear message. Cancel checks the current user’s ownership and the Event start time before deleting a Registration.

**File: `src/CampusEvents/Controllers/RegistrationsController.cs`**

```csharp
using CampusEvents.Data;
using CampusEvents.Models;
using CampusEvents.Services;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace CampusEvents.Controllers;

[Authorize]
public class RegistrationsController : Controller
{
	private readonly ApplicationDbContext _context;
	private readonly UserManager<ApplicationUser> _userManager;
	private readonly RegistrationService _registrationService;
	private readonly TimeProvider _timeProvider;

	public RegistrationsController(ApplicationDbContext context, UserManager<ApplicationUser> userManager,
		RegistrationService registrationService, TimeProvider timeProvider)
	{
		_context = context;
		_userManager = userManager;
		_registrationService = registrationService;
		_timeProvider = timeProvider;
	}

	[HttpPost]
	[ValidateAntiForgeryToken]
	public async Task<IActionResult> Register(int id)
	{
		var currentUser = await _userManager.GetUserAsync(User);

		if (currentUser is null)
		{
			return Challenge();
		}

		var result = await _registrationService.TryRegisterAsync(currentUser.Id, id);

		switch (result)
		{
			case RegistrationAttemptStatus.Success:
				TempData["StatusMessage"] = "Registration confirmed.";
				break;

			case RegistrationAttemptStatus.Duplicate:
				TempData["ErrorMessage"] = "You are already registered for this Event.";
				break;

			case RegistrationAttemptStatus.Full:
				TempData["ErrorMessage"] = "No places are currently available.";
				break;

			case RegistrationAttemptStatus.Unavailable:
				TempData["ErrorMessage"] = "Registration is unavailable for this Event.";
				return RedirectToAction("Index", "Events");

			case RegistrationAttemptStatus.EventNotFound:
				return NotFound();
		}

		return RedirectToAction("Details", "Events", new { id });
	}

	[HttpGet]
	public async Task<IActionResult> Mine()
	{
		var currentUser = await _userManager.GetUserAsync(User);

		if (currentUser is null)
		{
			return Challenge();
		}

		var registrations = await _context.Registrations.AsNoTracking().Include(registration => registration.Event)
					.ThenInclude(eventItem => eventItem.Category).Where(registration =>
					registration.UserId == currentUser.Id).OrderBy(registration => registration.Event.StartDateTime)
				.ToListAsync();

		return View(registrations);
	}

	[HttpPost]
	[ValidateAntiForgeryToken]
	public async Task<IActionResult> Cancel(int id)
	{
		var currentUser = await _userManager.GetUserAsync(User);

		if (currentUser is null)
		{
			return Challenge();
		}

		var registration = await _context.Registrations.Include(registration => registration.Event)
				.FirstOrDefaultAsync(registration => registration.Id == id && registration.UserId == currentUser.Id);

		if (registration is null)
		{
			return NotFound();
		}

		if (registration.Event.StartDateTime <= _timeProvider.GetLocalNow().DateTime)
		{
			TempData["ErrorMessage"] = "A Registration can be cancelled only before the Event begins.";

			return RedirectToAction(nameof(Mine));
		}

		_context.Registrations.Remove(registration);

		await _context.SaveChangesAsync();

		TempData["StatusMessage"] = "Your Registration was cancelled and the place is available again.";

		return RedirectToAction(nameof(Mine));
	}
}
```

Register resolves the signed-in user and passes their ID and the requested Event ID to `RegistrationService`. The service returns `Success`, `Duplicate`, `Full`, `Unavailable`, or `EventNotFound` after its database decision. The controller turns those results into confirmation, refusal, or not-found behavior. For `Unavailable`, the Events catalog provides a reachable page for feedback even when the target Event is a Draft; successful, duplicate, and full responses return to Event Details. Cancel queries both the Registration ID and current User ID, verifies the scheduled start has not been reached, and deletes only that row. A Cancelled Event can still have a cancellable current Registration before its scheduled start.

To show an ineligible-registration refusal on a page open to all Participants, add the following alert region directly after `<h1>Upcoming Campus Events</h1>` in the inherited `src/CampusEvents/Views/Events/Index.cshtml`. Preserve the existing search, Category, and pagination form and its result rendering.

```cshtml
@if (TempData["ErrorMessage"] is string errorMessage)
{
	<p class="error-message" role="alert">@errorMessage</p>
}
```

The catalog now reports rejected direct Register requests against Draft, Cancelled, and already-started Events. Its established Published/upcoming query continues to determine which Events appear in normal browsing.

## Render My Registrations and its navigation

**File: `src/CampusEvents/Views/Registrations/Mine.cshtml`**

```cshtml
@inject TimeProvider Clock
@model IReadOnlyList<CampusEvents.Models.Registration>

@{
	ViewData["Title"] = "My Registrations";
	var now = Clock.GetLocalNow().DateTime;
}

<h1>My Registrations</h1>

@if (TempData["StatusMessage"] is string statusMessage)
{
	<p class="status-message" role="status">
		<strong>Success:</strong> @statusMessage
	</p>
}

@if (TempData["ErrorMessage"] is string errorMessage)
{
	<p class="error-message" role="alert">
		@errorMessage
	</p>
}

@if (Model.Count == 0)
{
	<p role="status">
		You do not currently have any Event Registrations.
	</p>
}
else
{
	<div class="registration-list">
		@foreach (var registration in Model)
		{
			<article class="registration-item">
				<h2>@registration.Event.Title</h2>

				<dl class="event-meta">
					<div>
						<dt>Status</dt>
						<dd>@registration.Event.Status</dd>
					</div>
					<div>
						<dt>Category</dt>
						<dd>@registration.Event.Category.Name</dd>
					</div>
					<div>
						<dt>Location</dt>
						<dd>@registration.Event.Location</dd>
					</div>
					<div>
						<dt>Starts</dt>
						<dd>
							<time datetime="@registration.Event.StartDateTime.ToString("yyyy-MM-ddTHH:mm:ss")">
								@registration.Event.StartDateTime.ToString("f")
							</time>
						</dd>
					</div>
					<div>
						<dt>Registered</dt>
						<dd>
							<time datetime="@registration.CreatedAt.ToString("yyyy-MM-ddTHH:mm:ss")">
								@registration.CreatedAt.ToString("f")
							</time>
						</dd>
					</div>
				</dl>

				<div class="event-actions">
					<a asp-controller="Events" asp-action="Details" asp-route-id="@registration.EventId"
					   class="button-secondary">
						View Event
					</a>

					@if (registration.Event.StartDateTime > now)
					{
						<form asp-controller="Registrations" asp-action="Cancel" asp-route-id="@registration.Id"
							  method="post"
							  asp-antiforgery="true"
							  class="event-action-form">
							<button type="submit" class="button-danger">
								Cancel My Registration
							</button>
						</form>
					}
				</div>
			</article>
		}
	</div>
}
```

The query in `Mine()` is already restricted by `registration.UserId == currentUser.Id`; the View displays only the returned records. Update the Participant Area and shared navigation without removing Identity links, Organizer navigation, Logout POST, or public catalog links.

**File: `src/CampusEvents/Views/Participant/Index.cshtml`**

```cshtml
@{
	ViewData["Title"] = "Participant Area";
}

<h1>Participant Area</h1>

<p>
	You are authenticated and can use Participant
	registration features.
</p>

<div class="event-actions">
	<a asp-controller="Events" asp-action="Index" class="button-primary">
		Browse Events
	</a>

	<a asp-controller="Registrations" asp-action="Mine" class="button-secondary">
		My Registrations
	</a>
</div>
```

**File: `src/CampusEvents/Views/Shared/_Layout.cshtml`** — inside the existing authenticated navigation block, add:

```cshtml
<li>
	<a class="site-nav-link" asp-controller="Registrations" asp-action="Mine">
		My Registrations
	</a>
</li>
```

**File: `src/CampusEvents/wwwroot/css/site.css`** — reuse existing status, button, and Event metadata rules; add only the Registration list layout:

```css
.registration-list {
	display: grid;
	gap: 1rem;
}

.registration-item {
	padding: 1rem;
	border: 1px solid currentColor;
	border-radius: 0.35rem;
}

.registration-item h2 {
	margin-top: 0;
}
```

Connect the Register action now that both the protected `Register` POST and the `Mine` page are available. In `src/CampusEvents/Views/Events/Details.cshtml`, replace the final `else` branch inside the Registration section:

```cshtml
else
{
	<p role="status">Places are currently available for registration.</p>
}
```

with the antiforgery-protected POST form:

```cshtml
else
{
	<form asp-controller="Registrations" asp-action="Register" asp-route-id="@Model.EventItem.Id" method="post"
		  asp-antiforgery="true">
		<button type="submit" class="button-primary">Register</button>
	</form>
}
```

An eligible authenticated Participant can now submit Register directly from Details, receive the redirect and status message, and open My Registrations. Run the application after rebuilding:

```bat
dotnet build .\CampusEvents.slnx
dotnet run --project src\CampusEvents --launch-profile https
```

**Expected observable checkpoint:** After a successful Register POST, Details displays a success message and one more current Registration. My Registrations displays the new row. Cancel uses POST with an antiforgery token, redirects, and decreases the next Details count. A forged foreign Registration ID returns not found without modifying the row.

> **Screenshot placeholder — My Registrations and cancellation**
>
> **Illustration needed:** Participant's My Registrations page for one future Event, showing metadata and the cancellation POST control.
>
> **Expected visible result:** Current Event entry, start information, View Event link, and Cancel My Registration control.
>
> **Application state:** Signed-in Participant with one Registration for a future Published Event.
>
> ![Expected participant registration list with owned Event and cancellation control](images/lesson-07/my-registrations.png)

## Complete Registration-dependent Organizer rules

The inherited Draft deletion confirmation and Edit form remain in place. Add a shared helper that loads a currently owned Event under the accepted SQL Server lock. Its interpolated SQL binds parameters; the explicit transaction comes from the caller.

**File: `src/CampusEvents/Controllers/EventsController.cs`** — add inside the existing controller:

```csharp
private Task<Event?> LoadOwnedEventForUpdateAsync(int id, string organizerId)
{
	return _context.Events.FromSqlInterpolated($@"
			SELECT *
			FROM [Events] WITH (UPDLOCK, HOLDLOCK)
			WHERE [Id] = {id}
			  AND [OrganizerId] = {organizerId}").SingleOrDefaultAsync();
}
```

Replace **only** the inherited `DeleteConfirmed(int id)` POST with the following version, which checks persisted current Registrations before physically deleting an owned Draft:

```csharp
[HttpPost]
[Authorize(Roles = RoleNames.Organizer)]
[ValidateAntiForgeryToken]
public async Task<IActionResult> DeleteConfirmed(int id)
{
	var organizer = await _userManager.GetUserAsync(User);

	if (organizer is null)
	{
		return Challenge();
	}

	await using var transaction = await _context.Database.BeginTransactionAsync();

	var eventItem = await LoadOwnedEventForUpdateAsync(id, organizer.Id);

	if (eventItem is null)
	{
		await transaction.RollbackAsync();
		return EventNotFound();
	}

	if (eventItem.Status != EventStatus.Draft)
	{
		await transaction.RollbackAsync();

		TempData["ErrorMessage"] = "Only a Draft Event can be permanently deleted.";

		return RedirectToAction(nameof(Manage));
	}

	var hasCurrentRegistrations = await _context.Registrations.AnyAsync(registration =>
				registration.EventId == eventItem.Id);

	if (hasCurrentRegistrations)
	{
		await transaction.RollbackAsync();

		TempData["ErrorMessage"] = "This Draft Event cannot be deleted while current Registrations exist.";

		return RedirectToAction(nameof(Manage));
	}

	_context.Events.Remove(eventItem);

	await _context.SaveChangesAsync();
	await transaction.CommitAsync();

	TempData["StatusMessage"] = $"{eventItem.Title} was permanently deleted.";

	return RedirectToAction(nameof(Manage));
}
```

For eligible Published Edit, Capacity must never be reduced below the number of persisted current Registrations. Replace **only** the inherited Edit POST, preserving GET behavior, the `EventInputModel`, Category validation, current owner, and the server-controlled status:

```csharp
[HttpPost]
[Authorize(Roles = RoleNames.Organizer)]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Edit(int id, EventInputModel model)
{
	var organizer = await _userManager.GetUserAsync(User);

	if (organizer is null)
	{
		return Challenge();
	}

	await using var transaction = await _context.Database.BeginTransactionAsync();

	var eventItem = await LoadOwnedEventForUpdateAsync(id, organizer.Id);

	if (eventItem is null)
	{
		await transaction.RollbackAsync();
		return EventNotFound();
	}

	if (eventItem.Status == EventStatus.Cancelled)
	{
		await transaction.RollbackAsync();

		TempData["ErrorMessage"] = "A Cancelled Event can no longer be edited.";

		return RedirectToAction(nameof(Manage));
	}

	await ValidateCategoryAsync(model);

	if (eventItem.Status == EventStatus.Published && model.Capacity is int proposedCapacity)
	{
		var currentRegistrationCount = await _context.Registrations.CountAsync(registration =>
					registration.EventId == eventItem.Id);

		if (proposedCapacity < currentRegistrationCount)
		{
			ModelState.AddModelError(nameof(EventInputModel.Capacity),
				$"Capacity cannot be lower than the current registration count ({currentRegistrationCount}).");
		}
	}

	if (!ModelState.IsValid)
	{
		await transaction.RollbackAsync();

		await PopulateCategoryOptionsAsync(model);

		return View(model);
	}

	eventItem.Title = model.Title.Trim();
	eventItem.Description = model.Description.Trim();
	eventItem.CategoryId = model.CategoryId!.Value;
	eventItem.Location = model.Location.Trim();
	eventItem.StartDateTime = model.StartDateTime!.Value;
	eventItem.EndDateTime = model.EndDateTime!.Value;
	eventItem.Capacity = model.Capacity!.Value;

	await _context.SaveChangesAsync();
	await transaction.CommitAsync();

	TempData["StatusMessage"] = $"{eventItem.Title} was updated.";

	return RedirectToAction(nameof(Manage));
}
```

The Edit POST participates in the same Event-row locking discipline as Register. A loss of the capacity floor becomes a `ModelState` error; Category options are reloaded and no partial edit is persisted. `DeleteConfirmed` similarly checks persisted Registration rows rather than trusting a submitted count. Retain the existing confirmation GET and its read-only behavior.

**Expected observable checkpoint:** A Published Capacity Edit with a proposed value lower than current registrations remains on Edit with a field error and unchanged persisted Capacity. An owned Draft containing an anomalous current Registration cannot be deleted. A valid unregistered Draft can still be deleted. Check the persisted Event and Registration rows with a fresh database context.

# Guided Discussion

## Registration as a persisted relational association

A Participant may attend many Events, and an Event may have many Participants. A bare many-to-many join would capture that relationship, but CEMS needs to identify and timestamp each *current* enrollment, prevent duplicates, and let the Participant cancel a particular enrollment. `Registration` therefore has its own primary key and `CreatedAt` value as well as foreign keys to `ApplicationUser` and `Event`.

```text
ApplicationUser (1) ── (many) Registration (many) ── (1) Event
                                  |
                            UserId + EventId
                            unique for current rows
```

The two foreign keys ensure each Registration refers to existing records. The unique pair index enforces *one current association per User/Event pair*, including under near-simultaneous requests. The separate EventId index supports counts and Event-scoped queries. Each index has a different responsibility: the EventId index improves access to rows, while the unique pair index rejects prohibited duplicate data. Foreign-key `Restrict` rules require the application to resolve current Registrations before deleting their parent records.

The SRS uses a **current-association model**: cancelling removes the Registration row. After cancellation, that user can register for the same eligible Event again, creating a new current association and timestamp. For alternative historical/status models, see [Registration lifecycle and capacity-model alternatives](#registration-lifecycle-and-capacity-model-alternatives).

## Derived availability and consistency across views

The Event stores its configured `Capacity`, and Registration rows represent occupied places. A count query provides `CurrentRegistrationCount`. Remaining capacity is calculated from those two facts:

\[
\text{RemainingCapacity}=\max(0,\text{Capacity}-\text{CurrentRegistrationCount})
\]

With Capacity 20 and 19 current rows, Details shows one remaining place. When an owner cancels an eligible current Registration, the row disappears, and the next count reports 18 current and two remaining places. The count is derived on retrieval rather than maintained in a separate `SeatsRemaining` column that every writer would need to synchronize.

The public Details page represents a **point-in-time view**. If two Participants open the page at the same time, both may see one seat remaining. The first successful Register command can change that shared state before the other Participant submits. The mutating request consequently rechecks the database under the transaction's coordination boundary. This separates *information shown to the user* from *authority to reserve a place*.

A view model presents the Event alongside registration count, remaining places, and the signed-in user's current registration state. Those fields are derived for a particular response; the underlying relational rows remain the source of truth.

## Authenticated commands, ownership, and temporal eligibility

Registration and cancellation are distinct protected commands. They share an authenticated user identity and antiforgery-protected POST contract, but act on different resources and enforce different business conditions.

| Decision | Register | Cancel own Registration |
| --- | --- | --- |
| Request target | Event ID | Registration ID |
| Server-derived actor | Current authenticated user | Current authenticated user |
| Resource scope | Published Event before its start | Registration owned by that user |
| Additional eligibility | No duplicate and a place remains | Scheduled Event start is still in the future |
| Persisted change | Insert current Registration | Delete current Registration |
| User feedback | Confirmation or controlled refusal | Cancellation confirmation or controlled refusal |

A route ID identifies the requested resource; the authenticated principal determines who is acting. Cancellation combines the requested Registration ID with the current User ID in the database query. That pairing ensures a forged ID cannot authorize deleting another Participant's row. The cancellation deadline depends on the Event start time: an owned Registration for a Cancelled Event can still be cancelled before its scheduled start.

A single injected `TimeProvider` supplies application-local time to the service, controller, views, and tests. Strict comparison (`StartDateTime > now`) means a command received at the exact scheduled start is too late. A GET may make the appropriate control visible, but the POST performs its own current-state and identity checks for every submission.

## Layered validation, relational integrity, and controlled failures

Several safeguards act at different points in a request. Their contribution is clearest when matched to the invariant they enforce.

| Layer | Primary responsibility | Complementary safeguard |
| --- | --- | --- |
| Controller authentication and outcome mapping | Resolve the current actor and translate service results into HTTP responses and feedback | Service performs the authoritative registration decision |
| Service eligibility checks | Examine Published/future status, duplicate registration, and available capacity under the Event-row lock | SQL Server constraints and coordinated writers protect persisted integrity |
| Composite unique index | Reject a second current User/Event pair | Application duplicate check returns friendly feedback |
| Event-row transaction and lock | Coordinate capacity decisions among participating writers | Indexes protect individual-row uniqueness and references |
| Foreign-key restriction | Preserve relational references during parent deletes | Authorized Delete workflow checks status and current Registrations |
| Transaction rollback and controlled outcomes | Keep expected refusals from producing partial writes | Fresh database reads establish the resulting persisted state |

For example, a service-level duplicate check usually prevents an unnecessary insert. If another request inserts the pair first, SQL Server's unique index still rejects the duplicate and the service translates the recognized database error into the `Duplicate` result. A database error unrelated to the unique constraint is handled by normal diagnostics rather than presented as a routine duplicate.

In the Organizer Edit form, an invalid Capacity produces a `ModelState` field error with the user's attempted values and the Category options reconstructed for redisplay. The transaction ends without saving partial changes. The controller translates the service result into an appropriate destination and message; service validation and database integrity remain coordinated around the write. The same separation of responsibilities applies to other multi-step forms and commands.

## Concurrency and the final-seat invariant

Consider two different users and an Event with Capacity 20 and 19 Registrations. Each request can observe 19 rows before the other inserts. If both then insert based on that observation, the final count becomes 21. This is a **read–check–write race**: the problem is the uncoordinated interval between reading shared state and writing. Asynchronous I/O changes how callers wait for database work; transactions and coordination determine how concurrent decisions interact.

The chosen design makes the **Event row the coordination resource** for every Register and Published Capacity Edit operation. A short SQL Server transaction acquires that Event row using `UPDLOCK, HOLDLOCK`; while holding it, the Register service checks publication status, scheduled start, duplicate association, current count, and Capacity. Successful insertion and commit complete the decision before the next participating writer proceeds.

```text
A: acquire Event 12 lock ── count 19 ── insert one row ── commit
B: request Event 12 lock ── wait ── acquire lock ── count 20 ── Full; rollback
```

`UPDLOCK` requests update locking appropriate for a writer planning to make a capacity decision. `HOLDLOCK` retains the relevant lock protection through the enclosing transaction. A different Event uses a different row as its coordination point, allowing independent Events to proceed without sharing one application-wide lock. Keep database transactions short, use a consistent lock order, and check eligibility *after* acquiring the protected row.

The four ACID properties help frame this behavior: **atomicity** groups the insert and transaction completion; **consistency** requires the resulting state to satisfy the invariants; **isolation** constrains how competing requests interact; and **durability** preserves committed changes. The concrete capacity guarantee comes from the Event-row protocol, the authoritative recount, and participation by both capacity-increasing and capacity-decreasing writers. SQL Server isolation alternatives and lock trade-offs are developed in [Database concurrency control and isolation strategies](#database-concurrency-control-and-isolation-strategies).

## Cross-workflow consistency and shared invariants

Registration increases the count; Organizer Edit can decrease Capacity. Both affect the same invariant:

\[
\text{CurrentRegistrationCount}\leq\text{Capacity}
\]

Suppose the Event has Capacity 20 and 19 Registrations. Participant A requests the last place while the Organizer requests a reduction to Capacity 19. Both operations use the Event lock, so the outcome depends on who obtains it first:

| First transaction | First decision | Second decision | Final valid state |
| --- | --- | --- | --- |
| Register | Adds the twentieth Registration | Edit to 19 finds count 20 and shows a field error | Count 20; Capacity 20 |
| Edit | Sets Capacity to 19 while count is 19 | Register recounts and returns Full | Count 19; Capacity 19 |

The important engineering principle is to identify **every writer that can affect an invariant**. Reviewing Register alone would overlook the Capacity Edit race. The Draft Delete workflow also operates within an Event transaction and checks whether current Registrations exist; the foreign key reinforces the application guard against deleting a referenced Event. Cancellation deletes an owned current row and therefore releases capacity through the next derived count.

When a workflow crosses databases or services, its coordination strategy must account for multiple commit boundaries. [Transactions across multiple services and data stores](#transactions-across-multiple-services-and-data-stores) compares those extensions.

## Verification strategies and the strength of evidence

Verification begins with a controlled scenario: known initial rows, an application-local clock, an operation, and a query of the resulting committed state. An isolated migrated SQL Server database supports real foreign keys, unique indexes, transactions, and SQL Server locking. Independent EF contexts on the competing requests provide separate units of work; a new context used for assertions reads the committed result.

Match the verification method to the engineering question. Relational service integration examines business outcomes and committed rows under real database constraints. HTTP integration exercises authenticated cookies, antiforgery, redirects, rendered feedback, and persistence through the request pipeline. A controlled two-session demonstration makes Event-row lock coordination observable. Browser checks assess the Registration pages, keyboard access, and understandable feedback.

Use a fresh database context to inspect committed rows after each operation. For competing writers, compare returned outcomes with the final registration count and Event Capacity. The database state and user-facing feedback should describe the same decision.

# Guided Verification and Validation

Verification checks whether the implementation follows its specified contract; validation checks whether the behavior makes sense to the intended user and satisfies the accepted product requirements. Continue the existing regression suite and its disposable migrated SQL Server databases. Avoid the Development database for tests that create, cancel, or race Registrations.

## Automated

**Test-case inventory — new and retained test responsibilities:**

| Test / case | Behavior or boundary | Expected assertion / evidence |
| --- | --- | --- |
| New: `DraftEvent_IsUnavailable` | Registration against a Draft | `Unavailable` and no persisted Registration |
| New: `SameUser_CannotRegisterTwice` | Duplicate user/Event association | One `Success`, one `Duplicate`, exactly one persisted row |
| New: `CompetingRequests_ForLastSeat_OnlyOnePersists` | Two users and Capacity 1 | One `Success`, one `Full`, and count never above Capacity |
| New HTTP: `Register_ValidPost_PersistsAndConfirms` | Authorized Register form and successful response | Details redirect and confirmation; one current row, count 1, remaining Capacity minus 1 |
| New HTTP: `Register_DraftPost_ExplainsUnavailability` | Direct authenticated POST targeting a private Draft | Catalog redirect, specific alert, and no inserted Registration; Draft Details remains protected |
| New HTTP: `Register_RequiresAuthenticationAndAntiforgery` | Anonymous POST and authenticated POST without token | Login redirect or HTTP 400; no persisted Registration |
| New HTTP: `OwnCancellation_DoesNotDeleteAnotherUsersRegistration` | Forged Registration ID followed by own cancellation | Foreign 404 and unchanged row; owned row removed and capacity released |
| New HTTP: `PublishedCapacity_BelowCurrentCount_IsRejected` | Owned Published Edit lowers Capacity below count | Field error, unchanged persisted Capacity and other Event fields |
| New HTTP: `RegisterAndCapacityEdit_OrderedWritesPreserveInvariant` (2 parameterized cases) | Register then Edit, and Edit then Register | First operation succeeds, second receives specific refusal text; count and Capacity agree |
| New HTTP: `RegisterAndCapacityEdit_CompetingPostsPreserveInvariant` | Two independent HTTP sessions submit Register and Edit simultaneously | Committed count and Capacity select one valid ordering; both HTTP outcomes match that ordering |
| New HTTP: `DraftDelete_WithRegistration_IsRefused` | Owned Draft with a current association | Controlled refusal and both records retained |
| Retained: public catalog/Details, including page sizes 10/25/50 | Search/filter/pagination, Published/Cancelled Details | Existing browsing contracts retained; update Details view-model assertions |
| Retained: Identity and Organizer HTTP lifecycle tests | Authentication, access control, Publish, Cancel, Edit, and Draft Delete | Previously accepted authorizations and state transitions remain valid |
| Independent exercise: `CancelledEvent_IsUnavailable` | Register against Cancelled state | `Unavailable` and no insertion, as checked by the student's added test |

The three service `[Fact]` methods exercise the relational Registration decision. Add the HTTP cases to `OrganizerEventHttpTests` using its existing host and browser helpers. Record discovered test methods, parameterized cases, and executed results from the test runner.

### Add relational Registration service tests

**File: `tests/CampusEvents.Tests/RegistrationServiceTests.cs`** — use the existing `IsolatedSqlServerDatabase` fixture and `FakeTimeProvider` package. Each test operates against its own disposable migrated database and uses a fixed UTC−04 application-local wall clock.

```csharp
using CampusEvents.Models;
using CampusEvents.Services;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Time.Testing;
using Xunit;

namespace CampusEvents.Tests;

public class RegistrationServiceTests
{
	private static FakeTimeProvider FixedClock()
	{
		var clock = new FakeTimeProvider(new DateTimeOffset(2026, 4, 15, 16, 0, 0, TimeSpan.Zero));
		clock.SetLocalTimeZone(TimeZoneInfo.CreateCustomTimeZone("RegistrationUtcMinus4", TimeSpan.FromHours(-4),
			"CEMS UTC-04 test zone", "CEMS UTC-04 test zone"));
		return clock;
	}

	private static async Task<(int eventId, string userA, string userB)> ScenarioAsync(
		IsolatedSqlServerDatabase database, EventStatus status, int capacity, FakeTimeProvider clock)
	{
		await using var db = database.CreateContext();
		var organizer = new ApplicationUser { Id = Guid.NewGuid().ToString("N"), UserName = "organizer@test.local",
			Email = "organizer@test.local", DisplayName = "Test Organizer" };
		var a = new ApplicationUser { Id = Guid.NewGuid().ToString("N"), UserName = "a@test.local",
			Email = "a@test.local", DisplayName = "Test A" };
		var b = new ApplicationUser { Id = Guid.NewGuid().ToString("N"), UserName = "b@test.local",
			Email = "b@test.local", DisplayName = "Test B" };
		db.Users.AddRange(organizer, a, b);
		var category = new Category { Name = "Concurrency Test Category" };
		db.Categories.Add(category);
		await db.SaveChangesAsync();
		var start = clock.GetLocalNow().DateTime.AddDays(1);
		var item = new Event { Title = "Capacity Test Event", Description = "Current Registration test",
			Location = "Library Room", CategoryId = category.Id, OrganizerId = organizer.Id,
			Status = status, StartDateTime = start, EndDateTime = start.AddHours(2), Capacity = capacity,
			CreatedAt = clock.GetLocalNow().DateTime };
		db.Events.Add(item);
		await db.SaveChangesAsync();
		return (item.Id, a.Id, b.Id);
	}

	[Fact]
	public async Task DraftEvent_IsUnavailable()
	{
		await using var database = await IsolatedSqlServerDatabase.CreateAsync();
		var clock = FixedClock();
		var (eventId, userA, _) = await ScenarioAsync(database, EventStatus.Draft, 2, clock);
		await using var db = database.CreateContext();
		var result = await new RegistrationService(db, clock).TryRegisterAsync(userA, eventId);
		Assert.Equal(RegistrationAttemptStatus.Unavailable, result);
		Assert.Equal(0, await db.Registrations.CountAsync());
	}

	[Fact]
	public async Task SameUser_CannotRegisterTwice()
	{
		await using var database = await IsolatedSqlServerDatabase.CreateAsync();
		var clock = FixedClock();
		var (eventId, userA, _) = await ScenarioAsync(database, EventStatus.Published, 2, clock);
		await using (var first = database.CreateContext())
			Assert.Equal(RegistrationAttemptStatus.Success,
				await new RegistrationService(first, clock).TryRegisterAsync(userA, eventId));
		await using (var second = database.CreateContext())
			Assert.Equal(RegistrationAttemptStatus.Duplicate,
				await new RegistrationService(second, clock).TryRegisterAsync(userA, eventId));
		await using var check = database.CreateContext();
		Assert.Equal(1, await check.Registrations.CountAsync(r => r.UserId == userA && r.EventId == eventId));
	}

	[Fact]
	public async Task CompetingRequests_ForLastSeat_OnlyOnePersists()
	{
		await using var database = await IsolatedSqlServerDatabase.CreateAsync();
		var clock = FixedClock();
		var (eventId, userA, userB) = await ScenarioAsync(database, EventStatus.Published, 1, clock);
		var gate = new TaskCompletionSource<bool>(TaskCreationOptions.RunContinuationsAsynchronously);
		async Task<RegistrationAttemptStatus> AttemptAsync(string userId)
		{
			await gate.Task;
			await using var db = database.CreateContext();
			return await new RegistrationService(db, clock).TryRegisterAsync(userId, eventId);
		}
		var first = AttemptAsync(userA);
		var second = AttemptAsync(userB);
		gate.SetResult(true);
		var outcomes = await Task.WhenAll(first, second);
		Assert.Equal(1, outcomes.Count(x => x == RegistrationAttemptStatus.Success));
		Assert.Equal(1, outcomes.Count(x => x == RegistrationAttemptStatus.Full));
		await using var verify = database.CreateContext();
		var count = await verify.Registrations.CountAsync(r => r.EventId == eventId);
		var capacity = await verify.Events.Where(e => e.Id == eventId).Select(e => e.Capacity).SingleAsync();
		Assert.Equal(1, count);
		Assert.True(count <= capacity);
	}
}
```

The two different users in the final-seat test share an Event but use **independent EF contexts**. They pass a simultaneous start gate and the test checks both service outcomes and committed SQL Server state. Use the inherited migrated fixture, disposable database lifecycle, and fixed clock throughout.

### Extend inherited HTTP and persistence tests

The inherited `tests/CampusEvents.Tests/EventsControllerTests.cs` contains direct-controller catalog checks and `Details_DraftOrMissing_ReturnsEventNotFoundWith404`. Preserve those cases and add a positive Details regression in the same class. The existing helper `CreateController(verify, clock)` supplies the controller and its anonymous request context. Add `using System.Security.Claims;` alongside the existing imports; `CampusEvents.ViewModels` is already imported.

**File: `tests/CampusEvents.Tests/EventsControllerTests.cs`** — add this method inside `EventsControllerTests`:

```csharp
[Fact]
public async Task Details_PublishedEvent_ProvidesCurrentAvailabilityModel()
{
	var clock = FixedClock();
	var now = clock.GetLocalNow().DateTime;
	await using var database = await IsolatedSqlServerDatabase.CreateAsync();
	int publishedId;

	await using (var arrange = database.CreateContext())
	{
		var category = new Category { Name = "Details Capacity" };
		var organizer = new ApplicationUser { Id = "details-capacity-owner", DisplayName = "Events Team" };
		var participant = new ApplicationUser { Id = "details-capacity-participant", DisplayName = "Participant" };
		arrange.Categories.Add(category);
		arrange.Users.AddRange(organizer, participant);
		await arrange.SaveChangesAsync();

		var eventItem = MakeEvent("Capacity Workshop", EventStatus.Published, now.AddDays(5),
			category.Id, organizer.Id);
		eventItem.Capacity = 2;
		arrange.Events.Add(eventItem);
		await arrange.SaveChangesAsync();
		publishedId = eventItem.Id;

		arrange.Registrations.Add(new Registration
		{
			UserId = participant.Id,
			EventId = publishedId,
			CreatedAt = now.AddHours(-1)
		});
		await arrange.SaveChangesAsync();
	}

	await using var verify = database.CreateContext();
	var controller = CreateController(verify, clock);
	controller.ControllerContext.HttpContext.User = new ClaimsPrincipal(new ClaimsIdentity());
	var view = Assert.IsType<ViewResult>(await controller.Details(publishedId));
	var model = Assert.IsType<EventDetailsViewModel>(view.Model);

	Assert.Equal(publishedId, model.EventItem.Id);
	Assert.Equal(EventStatus.Published, model.EventItem.Status);
	Assert.Equal(1, model.CurrentRegistrationCount);
	Assert.Equal(1, model.RemainingCapacity);
	Assert.False(model.IsCurrentUserRegistered);
}
```

The direct Details regression checks the returned view model, persisted count, remaining capacity, and anonymous registration state. The inherited Draft/missing test continues to verify the not-found behavior.

The existing `tests/CampusEvents.Tests/OrganizerEventHttpTests.cs` provides `FixedClock()`, `CreateHost(...)`, `AccountsAsync(...)`, `EventAsync(...)`, `Browser(...)`, `SignInAsync(...)`, `FormTokenAsync(...)`, `SendFormAsync(...)`, `CommandAsync(...)`, and `ReadEventAsync(...)`. Add the following methods **inside that class**, with its existing imports for Identity, EF Core, HTTP, `Regex`, and `WebUtility`. Use a fresh fixture for each case. The HTTP host exercises routing, cookie authentication, authorization, model binding, and antiforgery. The feedback assertion reads the status or alert region rendered by the redirected page.

```csharp
private static async Task AssertFeedbackAsync(
	HttpResponseMessage response, string cssClass, string role, string expectedMessage)
{
	Assert.Equal(HttpStatusCode.OK, response.StatusCode);
	var html = await response.Content.ReadAsStringAsync();
	var pattern = $@"<p class=""{cssClass}"" role=""{role}"">(?<message>[\s\S]*?)</p>";
	var match = Regex.Match(html, pattern);
	Assert.True(match.Success, $"Expected a {role} message in the {cssClass} element.");
	Assert.Contains(expectedMessage, WebUtility.HtmlDecode(match.Groups["message"].Value));
}

[Fact]
public async Task Register_ValidPost_PersistsAndConfirms()
{
	await using var database = await IsolatedSqlServerDatabase.CreateAsync();
	using var host = CreateHost(database, FixedClock());
	var (ownerId, _) = await AccountsAsync(host);
	var categoryId = await CategoryAsync(database);
	var eventId = await EventAsync(database, categoryId, ownerId, "Successful Registration", EventStatus.Published);
	using var browser = Browser(host);
	await SignInAsync(browser, ParticipantEmail);
	var token = await FormTokenAsync(browser, $"/Events/Details/{eventId}");
	var fields = new Dictionary<string, string> { ["__RequestVerificationToken"] = token };
	var post = await SendFormAsync(browser, $"/Registrations/Register/{eventId}", fields);
	Assert.Equal($"/Events/Details/{eventId}", Location(post).AbsolutePath);
	var details = await browser.GetAsync($"/Events/Details/{eventId}");
	Assert.Equal(HttpStatusCode.OK, details.StatusCode);
	var html = await details.Content.ReadAsStringAsync();
	await AssertFeedbackAsync(details, "status-message", "status", "Registration confirmed.");
	Assert.Matches(@"Current registrations</dt>\s*<dd>1</dd>", html);
	Assert.Matches(@"Remaining capacity</dt>\s*<dd>29</dd>", html);
	Assert.Contains("You already have a current Registration", html);
	await using var check = database.CreateContext();
	var participantId = await check.Users.Where(user => user.Email == ParticipantEmail)
		.Select(user => user.Id).SingleAsync();
	var registration = await check.Registrations
		.SingleAsync(row => row.UserId == participantId && row.EventId == eventId);
	Assert.Equal(FixedClock().GetLocalNow().DateTime, registration.CreatedAt);
	Assert.Equal(1, await check.Registrations.CountAsync(row => row.EventId == eventId));
}

[Fact]
public async Task Register_DraftPost_ExplainsUnavailability()
{
	await using var database = await IsolatedSqlServerDatabase.CreateAsync();
	using var host = CreateHost(database, FixedClock());
	var (ownerId, _) = await AccountsAsync(host);
	var categoryId = await CategoryAsync(database);
	var publishedId = await EventAsync(database, categoryId, ownerId, "Token Source", EventStatus.Published);
	var draftId = await EventAsync(database, categoryId, ownerId, "Private Draft", EventStatus.Draft);
	using var browser = Browser(host);
	await SignInAsync(browser, ParticipantEmail);
	var token = await FormTokenAsync(browser, $"/Events/Details/{publishedId}");
	var fields = new Dictionary<string, string> { ["__RequestVerificationToken"] = token };
	var post = await SendFormAsync(browser, $"/Registrations/Register/{draftId}", fields);
	Assert.Equal("/Events", Location(post).AbsolutePath);
	var catalog = await browser.GetAsync(Location(post).PathAndQuery);
	await AssertFeedbackAsync(catalog, "error-message", "alert", "Registration is unavailable for this Event.");
	var draftDetails = await browser.GetAsync($"/Events/Details/{draftId}");
	Assert.Equal(HttpStatusCode.NotFound, draftDetails.StatusCode);
	await using var check = database.CreateContext();
	Assert.Equal(0, await check.Registrations.CountAsync(row => row.EventId == draftId));
}

[Fact]
public async Task Register_RequiresAuthenticationAndAntiforgery()
{
	await using var database = await IsolatedSqlServerDatabase.CreateAsync();
	using var host = CreateHost(database, FixedClock());
	var (ownerId, _) = await AccountsAsync(host);
	var categoryId = await CategoryAsync(database);
	var eventId = await EventAsync(database, categoryId, ownerId, "Protected Registration", EventStatus.Published);
	using var anonymous = Browser(host);
	var anonymousPost = await anonymous.PostAsync($"/Registrations/Register/{eventId}",
		new FormUrlEncodedContent(new Dictionary<string, string>()));
	Assert.Equal("/Account/Login", Location(anonymousPost).AbsolutePath);
	using var participant = Browser(host);
	await SignInAsync(participant, ParticipantEmail);
	var noToken = await participant.PostAsync($"/Registrations/Register/{eventId}",
		new FormUrlEncodedContent(new Dictionary<string, string>()));
	Assert.Equal(HttpStatusCode.BadRequest, noToken.StatusCode);
	await using var check = database.CreateContext();
	Assert.Equal(0, await check.Registrations.CountAsync(r => r.EventId == eventId));
}

[Fact]
public async Task OwnCancellation_DoesNotDeleteAnotherUsersRegistration()
{
	await using var database = await IsolatedSqlServerDatabase.CreateAsync();
	using var host = CreateHost(database, FixedClock());
	var (ownerId, otherId) = await AccountsAsync(host);
	var categoryId = await CategoryAsync(database);
	var eventId = await EventAsync(database, categoryId, ownerId, "Cancellation Scope", EventStatus.Published);
	int ownId;
	int foreignId;
	await using (var setup = database.CreateContext())
	{
		var participantId = await setup.Users.Where(u => u.Email == ParticipantEmail).Select(u => u.Id).SingleAsync();
		var own = new Registration { UserId = participantId, EventId = eventId,
			CreatedAt = FixedClock().GetLocalNow().DateTime };
		var foreign = new Registration { UserId = otherId, EventId = eventId,
			CreatedAt = FixedClock().GetLocalNow().DateTime };
		setup.Registrations.AddRange(own, foreign);
		await setup.SaveChangesAsync();
		ownId = own.Id;
		foreignId = foreign.Id;
	}
	using var browser = Browser(host);
	await SignInAsync(browser, ParticipantEmail);
	var token = await FormTokenAsync(browser, "/Registrations/Mine");
	var fields = new Dictionary<string, string> { ["__RequestVerificationToken"] = token };
	var foreignPost = await SendFormAsync(browser, $"/Registrations/Cancel/{foreignId}", fields);
	Assert.Equal(HttpStatusCode.NotFound, foreignPost.StatusCode);
	await using (var check = database.CreateContext())
		Assert.Equal(2, await check.Registrations.CountAsync(r => r.EventId == eventId));
	var ownPost = await SendFormAsync(browser, $"/Registrations/Cancel/{ownId}", fields);
	Assert.Equal("/Registrations/Mine", Location(ownPost).AbsolutePath);
	var mine = await browser.GetAsync("/Registrations/Mine");
	await AssertFeedbackAsync(mine, "status-message", "status",
		"Your Registration was cancelled and the place is available again.");
	await using var verify = database.CreateContext();
	Assert.Equal(1, await verify.Registrations.CountAsync(r => r.EventId == eventId));
	Assert.True(await verify.Registrations.AnyAsync(r => r.Id == foreignId));
}

[Fact]
public async Task PublishedCapacity_BelowCurrentCount_IsRejected()
{
	await using var database = await IsolatedSqlServerDatabase.CreateAsync();
	using var host = CreateHost(database, FixedClock());
	var (ownerId, otherId) = await AccountsAsync(host);
	var categoryId = await CategoryAsync(database);
	var eventId = await EventAsync(database, categoryId, ownerId, "Capacity Floor", EventStatus.Published);
	await using (var setup = database.CreateContext())
	{
		var participantId = await setup.Users.Where(u => u.Email == ParticipantEmail).Select(u => u.Id).SingleAsync();
		setup.Registrations.AddRange(new Registration
			{
				UserId = participantId, EventId = eventId, CreatedAt = FixedClock().GetLocalNow().DateTime
			},
			new Registration { UserId = otherId, EventId = eventId, CreatedAt = FixedClock().GetLocalNow().DateTime });
		await setup.SaveChangesAsync();
	}
	var before = EventFields(await ReadEventAsync(database, eventId));
	using var browser = Browser(host);
	await SignInAsync(browser, OwnerEmail);
	var fields = ValidFields(categoryId, await FormTokenAsync(browser, $"/Events/Edit/{eventId}"));
	fields["Capacity"] = "1";
	var response = await SendFormAsync(browser, $"/Events/Edit/{eventId}", fields);
	Assert.Equal(HttpStatusCode.OK, response.StatusCode);
	Assert.Contains("Capacity cannot be lower", await response.Content.ReadAsStringAsync());
	Assert.Equal(before, EventFields(await ReadEventAsync(database, eventId)));
}

[Fact]
public async Task DraftDelete_WithRegistration_IsRefused()
{
	await using var database = await IsolatedSqlServerDatabase.CreateAsync();
	using var host = CreateHost(database, FixedClock());
	var (ownerId, _) = await AccountsAsync(host);
	var categoryId = await CategoryAsync(database);
	var eventId = await EventAsync(database, categoryId, ownerId, "Protected Draft", EventStatus.Draft);
	await using (var setup = database.CreateContext())
	{
		var participantId = await setup.Users.Where(u => u.Email == ParticipantEmail).Select(u => u.Id).SingleAsync();
		setup.Registrations.Add(new Registration { UserId = participantId, EventId = eventId,
			CreatedAt = FixedClock().GetLocalNow().DateTime });
		await setup.SaveChangesAsync();
	}
	using var browser = Browser(host);
	await SignInAsync(browser, OwnerEmail);
	var response = await CommandAsync(browser, $"/Events/DeleteConfirmed/{eventId}");
	Assert.Equal("/Events/Manage", Location(response).AbsolutePath);
	var manage = await browser.GetAsync("/Events/Manage");
	await AssertFeedbackAsync(manage, "error-message", "alert",
		"This Draft Event cannot be deleted while current Registrations exist.");
	await using var check = database.CreateContext();
	Assert.True(await check.Events.AnyAsync(e => e.Id == eventId));
	Assert.True(await check.Registrations.AnyAsync(r => r.EventId == eventId));
}

private static async Task<int> PrepareCapacityCompetitionAsync(
	IsolatedSqlServerDatabase database, int categoryId, string ownerId, string existingUserId)
{
	var eventId = await EventAsync(database, categoryId, ownerId, "Capacity Coordination", EventStatus.Published);
	await using var db = database.CreateContext();
	var eventItem = await db.Events.SingleAsync(item => item.Id == eventId);
	eventItem.Capacity = 2;
	db.Registrations.Add(new Registration
	{
		UserId = existingUserId, EventId = eventId, CreatedAt = FixedClock().GetLocalNow().DateTime
	});
	await db.SaveChangesAsync();
	return eventId;
}

[Theory]
[InlineData(true)]
[InlineData(false)]
public async Task RegisterAndCapacityEdit_OrderedWritesPreserveInvariant(bool registerFirst)
{
	await using var database = await IsolatedSqlServerDatabase.CreateAsync();
	using var host = CreateHost(database, FixedClock());
	var (ownerId, existingUserId) = await AccountsAsync(host);
	var categoryId = await CategoryAsync(database);
	var eventId = await PrepareCapacityCompetitionAsync(database, categoryId, ownerId, existingUserId);
	using var organizer = Browser(host);
	using var participant = Browser(host);
	await SignInAsync(organizer, OwnerEmail);
	await SignInAsync(participant, ParticipantEmail);
	var edit = ValidFields(categoryId, await FormTokenAsync(organizer, $"/Events/Edit/{eventId}"));
	edit["Capacity"] = "1";
	var token = await FormTokenAsync(participant, $"/Events/Details/{eventId}");
	var register = new Dictionary<string, string> { ["__RequestVerificationToken"] = token };
	if (registerFirst)
	{
		var registrationResponse = await SendFormAsync(participant, $"/Registrations/Register/{eventId}", register);
		Assert.Equal($"/Events/Details/{eventId}", Location(registrationResponse).AbsolutePath);
		var registrationDetails = await participant.GetAsync($"/Events/Details/{eventId}");
		await AssertFeedbackAsync(registrationDetails, "status-message", "status", "Registration confirmed.");
		var editResponse = await SendFormAsync(organizer, $"/Events/Edit/{eventId}", edit);
		Assert.Equal(HttpStatusCode.OK, editResponse.StatusCode);
		Assert.Contains("Capacity cannot be lower", await editResponse.Content.ReadAsStringAsync());
	}
	else
	{
		var editResponse = await SendFormAsync(organizer, $"/Events/Edit/{eventId}", edit);
		Assert.Equal("/Events/Manage", Location(editResponse).AbsolutePath);
		var registrationResponse = await SendFormAsync(participant, $"/Registrations/Register/{eventId}", register);
		Assert.Equal($"/Events/Details/{eventId}", Location(registrationResponse).AbsolutePath);
		var registrationDetails = await participant.GetAsync($"/Events/Details/{eventId}");
		await AssertFeedbackAsync(registrationDetails, "error-message", "alert", "No places are currently available.");
	}
	await using var check = database.CreateContext();
	var count = await check.Registrations.CountAsync(row => row.EventId == eventId);
	var capacity = await check.Events.Where(item => item.Id == eventId).Select(item => item.Capacity).SingleAsync();
	Assert.Equal(registerFirst ? 2 : 1, count);
	Assert.Equal(registerFirst ? 2 : 1, capacity);
}

[Fact]
public async Task RegisterAndCapacityEdit_CompetingPostsPreserveInvariant()
{
	await using var database = await IsolatedSqlServerDatabase.CreateAsync();
	using var host = CreateHost(database, FixedClock());
	var (ownerId, existingUserId) = await AccountsAsync(host);
	var categoryId = await CategoryAsync(database);
	var eventId = await PrepareCapacityCompetitionAsync(database, categoryId, ownerId, existingUserId);
	using var organizer = Browser(host);
	using var participant = Browser(host);
	await SignInAsync(organizer, OwnerEmail);
	await SignInAsync(participant, ParticipantEmail);
	var edit = ValidFields(categoryId, await FormTokenAsync(organizer, $"/Events/Edit/{eventId}"));
	edit["Capacity"] = "1";
	var token = await FormTokenAsync(participant, $"/Events/Details/{eventId}");
	var register = new Dictionary<string, string> { ["__RequestVerificationToken"] = token };
	var gate = new TaskCompletionSource<bool>(TaskCreationOptions.RunContinuationsAsynchronously);
	async Task<HttpResponseMessage> SubmitAsync(HttpClient browser, string route, Dictionary<string, string> fields)
	{
		await gate.Task;
		return await SendFormAsync(browser, route, fields);
	}
	var registrationTask = SubmitAsync(participant, $"/Registrations/Register/{eventId}", register);
	var editTask = SubmitAsync(organizer, $"/Events/Edit/{eventId}", edit);
	gate.SetResult(true);
	var responses = await Task.WhenAll(registrationTask, editTask);
	Assert.Equal($"/Events/Details/{eventId}", Location(responses[0]).AbsolutePath);
	await using var check = database.CreateContext();
	var count = await check.Registrations.CountAsync(row => row.EventId == eventId);
	var capacity = await check.Events.Where(item => item.Id == eventId).Select(item => item.Capacity).SingleAsync();
	Assert.True(count <= capacity);
	var registrationDetails = await participant.GetAsync($"/Events/Details/{eventId}");
	if (count == 2 && capacity == 2)
	{
		Assert.Equal(HttpStatusCode.OK, responses[1].StatusCode);
		Assert.Contains("Capacity cannot be lower", await responses[1].Content.ReadAsStringAsync());
		await AssertFeedbackAsync(registrationDetails, "status-message", "status", "Registration confirmed.");
	}
	else
	{
		Assert.Equal(1, count);
		Assert.Equal(1, capacity);
		Assert.Equal("/Events/Manage", Location(responses[1]).AbsolutePath);
		await AssertFeedbackAsync(registrationDetails, "error-message", "alert", "No places are currently available.");
	}
}
```

The Draft-registration regression submits a direct authenticated POST and confirms that the catalog displays its refusal. The Draft deletion regression seeds a current Registration directly against a Draft Event to exercise the deletion guard. Continue running inherited Identity HTTP tests for authentication and role transitions.

Run the positive Details regression alongside the inherited Draft/missing 404 and public Index checks. Preserve the catalog’s 10/25/50 page sizes, search filters, and `EventNotFound()` behavior.

### Execute and interpret the regression suite

Use the authorized SQL Server LocalDB instance on Windows, or an explicitly configured disposable SQL Server test endpoint via `CEMS_TEST_SQLSERVER` (must connect to `master`, not an application database). The existing fixture creates GUID-named databases, applies real migrations, and cleans up only the databases it owns.

```bat
dotnet restore .\CampusEvents.slnx
dotnet build .\CampusEvents.slnx --no-restore
dotnet test .\tests\CampusEvents.Tests\CampusEvents.Tests.csproj --no-build --logger "console;verbosity=normal"
```

Record discovered methods, executed `[Theory]` cases, skips, failures, SQL Server endpoint type, and fixed-clock assumptions from the runner. Enter the observed outcomes in the working traceability record. Investigate migration errors, invalid antiforgery tokens, unexpected redirects, and persisted-state differences separately.

### Observe SQL Server lock coordination with two sessions

Use an **authorized disposable SQL Server test database** containing an eligible future Published Event, at least one available place, and an authenticated Participant who has not registered for it. Session A is a SQL query connection; session B is the application's Register request, using a different database connection.

**Session A — acquire and hold the Event lock.** Identify the Event ID in the disposable database, substitute it below, and execute these statements in one SQL query connection. Confirm that the `SELECT` returns the expected Event row. Keep this connection open with its transaction active.

```sql
-- Session A: replace 17 with the verified Event ID in the disposable database.
DECLARE @EventId int = 17;
BEGIN TRANSACTION;
SELECT [Id], [Capacity] FROM [Events] WITH (UPDLOCK, HOLDLOCK) WHERE [Id] = @EventId;
```

**Session B — observe the competing request.** In a separately authenticated browser session or HTTP client, submit a valid antiforgery-protected `POST /Registrations/Register/{id}` for the same Event. Observe that the pending request waits for session A's lock. Keep session A's transaction open while recording the wait.

**Session A — release the lock.** Execute this statement in **the same query connection** that acquired the lock, after observing session B's pending request:

```sql
COMMIT TRANSACTION;
```

**Session B — confirm the result.** Observe the redirect and registration confirmation for an eligible Event with available capacity, then query committed Registrations and Event Capacity using a fresh connection. Record the Event ID, blocker and waiter sessions, lock release, HTTP result, and resulting rows. If the experiment is interrupted, roll back session A's still-active transaction in its original connection before ending the session.

## Manual

Use an authorized Development dataset with known Event IDs and at least two separately authenticated users. For each scenario record the Event status, start time in application-local time, initial Registration rows, HTTP result or visible message, final persisted rows, and the actual observation date. Do not perform destructive or contention scenarios on production data.

| Scenario | Prerequisites, inputs, and steps | Expected result and evidence |
| --- | --- | --- |
| Registration — valid enrollment and availability | Published Event starting in the future, Capacity 2, zero Registrations; sign in as Participant A, open Details, submit Register | Details redirect with confirmation; one persisted row, current 1, remaining 1; record Details and database state |
| Registration — duplicate enrollment | Same signed-in Participant A, same Event; submit another authorized Register POST | Controlled duplicate feedback and one current User/Event row; record HTTP response and unique-index query |
| Registration — full Event | Register Participant B for last place; attempt as a third distinct signed-in user | No new Registration; full-capacity message and persisted count 2; record Event and Registration rows |
| Registration — ineligible Event | Use distinct Draft, Cancelled, and started Event IDs; attempt authenticated Register POST directly with a valid token | Events catalog displays the unavailable alert; zero new rows, including equality at start time; Draft Details remains protected |
| Security — anonymous or tokenless POST | Send Register POST without authentication; then signed-in POST without antiforgery token | Login redirect or HTTP 400 respectively; no persisted row |
| Ownership — My Registrations scope | Prepare two users with separate Registrations; sign in as each and open Mine | Each sees only their own rows; browser captures and SQL scope check |
| Cancellation — own row before start | Sign in as owner of a current Registration; submit Cancel | Redirect with success; owned row removed; next Details count decreases; capture before/after |
| Cancellation — foreign Registration ID | As Participant A, POST Cancel using Participant B's Registration ID with a valid token | Not found; Participant B's row remains; record HTTP and SQL state |
| Cancellation — started Event | Use owned Registration whose Event start equals or precedes controlled now; POST Cancel | Controlled refusal; row remains |
| Cancellation — Cancelled Event before start | Sign in as current Registration owner for a Cancelled Event with future scheduled start; POST Cancel | Allowed cancellation and removed row, even though no new registration may be created |
| Organizer — Draft deletion guard | Insert a current test Registration for owned Draft directly into disposable data; submit DeleteConfirmed | Controlled refusal; both Event and Registration survive |
| Organizer — Published Capacity floor | Own Published Event with 2 Registrations; submit Edit with Capacity 1 | Field-level error and unchanged persisted Event; confirm previously valid fields are retained |
| Organizer — valid Capacity update | Same owned Event, submit Edit with Capacity 2 or higher | Accepted edit; persisted owner/status unchanged; count remains within Capacity |
| Concurrency — competing final seat | Future Published Capacity 1 Event, two independently authenticated participants, separate requests released together | Exactly one new Registration persists and the other request receives Full; record both outcomes and final rows |
| Concurrency — Register versus Edit | With Capacity 2 and one current Registration, simultaneously submit a Register POST and an Edit to Capacity 1 | Final count and Capacity both equal 1 or both equal 2; inspect the corresponding feedback and SQL rows |
| Regression — public and lifecycle | Browse with search, Category, page sizes 10/25/50; open known Cancelled Details; verify Organizer lifecycle and POST Logout | Previously accepted routes, state-specific access, pagination/filter retention, and feedback remain intact |
| Interface — semantic and keyboard | Navigate Details, login link, Register, My Registrations, and Cancel using keyboard and accessible names | Focus visible, descriptive buttons, native POST forms, status/alert text; record browser observations |

> **Screenshot placeholder — registration confirmation and capacity change**
>
> **Illustration needed:** Details immediately after an authenticated successful Register POST and redirected message.
>
> **Expected visible result:** Registration confirmation, current registration count increased, remaining count decreased, and View My Registrations available for the signed-in user.
>
> **Application state:** Development, future Published Capacity 2 Event, authenticated successful registrant, successful database insert.
>
> ![Expected confirmation on Event Details after a committed registration](images/lesson-07/registration-confirmation.png)

## SRS Traceability Matrix Update with Evidence

Update the working traceability record with each executed test or manual scenario, including its environment and observed result. The starting statuses below identify implementation work awaiting execution records.

| Requirement / Reference | Implementation / Artifact | Verification / Evidence | Status | Notes |
| --- | --- | --- | --- | --- |
| `UC-003`; `FR-REG-001`–`FR-REG-005`; `BR-REG-001`–`BR-REG-004` | `Registration`, `RegistrationService`, protected Register POST, SQL Server indexes/transaction | Relational outcomes, successful and rejected HTTP POSTs, SQL rows; Not executed | IMPLEMENTED—UNVERIFIED | Capacity and final-seat claims require observed relational evidence |
| `UC-004`; `FR-REG-006`–`FR-REG-008`; `BR-REG-005`–`BR-REG-006` | Mine, owner-scoped Cancel, derived counts | HTTP ownership test, cancellation browser/SQL checks; Not executed | IMPLEMENTED—UNVERIFIED | Verify exact-start and Cancelled Event scenarios |
| `FR-SRH-004`; `FR-SRH-008` | Details view model and persisted counts; retained Cancelled visibility | Direct and HTTP Details tests, manual UI; Not executed | IMPLEMENTED—UNVERIFIED | Keep public Catalog predicates and 10/25/50 paging unchanged |
| `FR-EVT-011`; `BR-EVT-008` | Transactional Draft Delete POST and Registration existence check | HTTP anomalous Draft test; Not executed | IMPLEMENTED—UNVERIFIED | Retain ownership and Draft status guard |
| `FR-EVT-004`; `FR-EVT-006`; `BR-EVT-009` | Owned Edit transaction and current Registration count floor | Published Capacity floor, ordered and competing Register/Edit HTTP tests; Not executed | IMPLEMENTED—UNVERIFIED | Revalidate previously inherited Edit behavior |
| `NFR-REL-001`–`NFR-REL-003`; `AC-005`; `AC-009` | SQL Server persistence, constraints, coordinated writers, controlled rejections | Relational and contention procedures; Not executed | PARTIAL | Check successful, rejected, and competing writes against the persisted invariant |
| `AC-004` | Participant Register, Mine, Cancel | End-to-end browser and database scenarios; Not executed | IMPLEMENTED—UNVERIFIED | Lesson-slice registration flow needs real evidence |
| `AC-010` | Existing cumulative test suite plus added registration tests | Full suite and independent discovery output; Not executed | PARTIAL | Record cumulative critical behavior and authorization cases from the actual runner |
| `FR-ACC-006` | Participant/Organizer authorization retained | Prior tests plus new role/ownership scenarios; Not executed for new slice | PARTIAL | Current scope covers Participant and Organizer operations; full acceptance includes Administrator scope |

**Evidence context:** Date: Pending | Environment / dataset: isolated migrated SQL Server test database | Procedure / command or scenario: Pending | Observed outcome: Not executed | Commit / repository reference: Pending

For verified rows, attach the execution result and observed database or HTTP outcome. Then inspect the working tree and record a reviewable commit containing the intended application, migration, test, and supporting lesson artifacts:

```bat
git status
git diff --check
git diff --stat
git add src\CampusEvents tests\CampusEvents.Tests
git diff --cached --stat
git commit -m "Add participant registration and coordinated capacity enforcement"
git push -u origin lesson-07-participant-registration
```

Stage the intended application, migration, and test changes. Keep credentials, disposable databases, and unrelated files outside the commit. Use the established pull-request review workflow with the execution record and migration summary.

# Guided Exercises

## Exercise: extend eligibility regression coverage

**Purpose and expected outcome:** Independently design a negative relational regression test for a lifecycle exclusion (Objectives 3, 4, and 7).

**Prerequisites and SRS ties:** Completed registration service tests and the fixed-clock SQL Server fixture; `FR-REG-005`, `BR-REG-004`.

**Task:** Add `CancelledEvent_IsUnavailable` as a new `[Fact]` in `RegistrationServiceTests.cs`. Seed a future Cancelled Event and a user without a current Registration; call the real service.

**Hints or checkpoints:** Follow `DraftEvent_IsUnavailable`; change only the Event state. Assert both returned outcome and fresh persisted row count.

**Deliverable:** One new test method and a recorded command/result.

**Completion and verification:** Run the focused class in an isolated SQL Server database and record actual discovery/execution status; expect `Unavailable` and no inserted row.

## Exercise: analyze an exact-start boundary

**Purpose and expected outcome:** Practice using a controlled clock to justify strict temporal inequalities (Objectives 3 and 7).

**Prerequisites and SRS ties:** An isolated relational test fixture, registered `TimeProvider`, and SRS `FR-REG-005`, `FR-REG-007`.

**Task:** Create two focused cases for start time exactly equal to the test clock: a Register attempt and a Cancel attempt on an owned Registration. For each, predict and then test the persisted state.

**Hints or checkpoints:** Compare `StartDateTime <= now` in both command paths; keep the clock fixed and query the database with a new context afterward.

**Deliverable:** Test code, before/after database state, and a brief explanation of the strict boundary.

**Completion and verification:** Registration is unavailable and cancellation is refused at the exact start; neither path changes current Registration rows.

## Exercise: challenge a shared capacity invariant

**Purpose and expected outcome:** Explain why correct individual requests can still produce an incorrect combined state without coordination (Objectives 5 and 6).

**Prerequisites and SRS ties:** The completed Register and Published Edit flows; `BR-REG-003`, `BR-EVT-009`, `NFR-REL-003`.

**Task:** Draw an interleaving for a Published Event with Capacity 20 and 19 Registrations: one user attempts registration while the owner proposes Capacity 19. Annotate the points at which both requests lock the Event, recheck authoritative state, and commit or refuse. Compare with the unsafe uncoordinated sequence.

**Hints or checkpoints:** Show each possible lock acquisition order. Count only persisted current Registrations, not posted or cached counts.

**Deliverable:** Two request timelines and an invariant statement.

**Completion and verification:** In both protected orderings, the final persisted count is no greater than Capacity; the unprotected timeline identifies the possible violation.

**Challenge (optional):** Design an additional relational test that deliberately interleaves a capacity reduction and a registration, with controlled synchronization and without relying on arbitrary delays.

## Exercise: evaluate relational constraint responsibilities

**Purpose and expected outcome:** Differentiate duplicate protection, referential integrity, and aggregate capacity enforcement (Objectives 1, 4, and 7).

**Prerequisites and SRS ties:** New Registration migration and applied SQL Server schema; `FR-REG-003`, `FR-REG-004`, `BR-EVT-008`.

**Task:** Create a table of the invariant, its enforcing layer, a deliberate invalid scenario, and an assertion that proves its rejection. Include duplicate pair, missing foreign Event, over-capacity registration, and deletion of an Event with a current Registration.

**Hints or checkpoints:** Use the composite unique index for pair uniqueness, foreign keys for referential integrity, and the coordinated count-and-insert transaction for aggregate capacity. Use disposable fixtures for destructive experiments.

**Deliverable:** Completed comparison table and at least one new independently written negative assertion.

**Completion and verification:** Each scenario identifies the correct authority and specifies observable HTTP or persistence evidence; no production data is modified.

## Exercise: review the migration and repository boundary

**Purpose and expected outcome:** Preserve architectural continuity and make changes auditable (Objective 8).

**Prerequisites and SRS ties:** Completed lesson code and tests; SRS §3.1–3.2 and `AC-010` evidence practices.

**Task:** Inspect the new migration, changed controller actions, test additions, `git diff --check`, and staged files. Identify any accidental changes to Identity, catalog pagination, or Event ownership.

**Hints or checkpoints:** Compare the inherited Lesson 6 baseline and current repository diff. Confirm the test fixture and clock are reused and that no secrets are staged.

**Deliverable:** Concise change summary, reviewed diff, and commit reference when the verification and review are complete.

**Completion and verification:** The staged diff contains the intended application/test changes and migration; the commit records the reviewed state, and the traceability record contains separately observed test outcomes.

# Troubleshooting Common Problems

| Symptom | Likely cause / diagnosis | Correction / verification |
| --- | --- | --- |
| `Registration` navigation fails to compile | Collections or `DbSet<Registration>` omitted, or namespace mismatch | Add relationships to existing `ApplicationUser` and `Event`; inspect compilation and model mapping |
| Migration lacks the unique index | Composite `(UserId, EventId)` configuration missing or migration generated from stale model | Inspect model and migration; regenerate before applying to disposable data |
| EF reports missing `Registrations` table | Migration not applied to the configured database | Inspect `DefaultConnection` and migration history; run the intended Development update |
| Duplicate registrations persist | Composite unique `(UserId, EventId)` index missing | Verify the unique index in SQL Server and run duplicate-pair regression |
| Two users take the last seat | Count and insert occur outside one coordinating transaction | Inspect Event-row `UPDLOCK, HOLDLOCK` and locked rechecks in every participating writer |
| Lock hint appears ineffective | Transaction ends before count/insert, or requests coordinate on different resources | Confirm one transaction spans load, eligibility, count, insert, and commit |
| Legitimate Edit unexpectedly allows Capacity below count | Edit path uses client count or omits Event-row coordination | Requery current Registrations while holding Event lock; verify failed form and unchanged row |
| `DbUpdateException` becomes a misleading duplicate message | Catch block converts all database errors into `Duplicate` | Filter only known SQL Server duplicate-key codes and surface other failures appropriately |
| Test fails with a LocalDB connection error | Missing Windows LocalDB or unauthorized override | Use the inherited isolated fixture and authorized SQL Server endpoint; verify `CEMS_TEST_SQLSERVER` targets `master` |
| `EnsureCreated()` causes schema conflicts | New ad hoc test setup bypasses migrations | Reuse `IsolatedSqlServerDatabase.CreateAsync()` and actual EF migrations |
| Cancellation removes another user's row | Query matches only Registration ID | Require `Id` **and** authenticated `UserId`; test forged ID and persisted foreign row |
| Cancel action disappears when Event is Cancelled | UI erroneously requires Published for cancellation | Use ownership and start-time eligibility for existing Registrations; test cancelled-before-start scenario |
| Details shows wrong status or wrong model | Inherited Details predicate changed or old Razor model declaration remains | Keep Published/Cancelled Details and update `@model` to `EventDetailsViewModel` |
| Direct Draft Register POST redirects to 404 without feedback | Unavailable result redirects to private Draft Details | Redirect unavailable requests to Events Index and render the `ErrorMessage` alert |
| Future-status or start-time test differs by machine | Runtime uses `DateTime.Now` instead of injected `TimeProvider` | Use `GetLocalNow().DateTime` consistently and fixed test time-zone policy |
| Existing catalog tests break after Lesson 7 | New controller code accidentally changes Index pagination or search | Preserve Index unchanged and re-run 10/25/50 paging and filter regression |
| Anonymous request unexpectedly creates a Registration | Missing `[Authorize]`, or POST lacks antiforgery | Require authenticated controller and `[ValidateAntiForgeryToken]`; inspect HTTP and SQL evidence |
| Current count does not decrease after cancellation | Row not removed, or a tracked/cached count is reused | Inspect fresh SQL Server count after SaveChanges; avoid mutable `SeatsRemaining` |
| Test case count differs from number of methods | Parameterized theories are discovered as multiple cases | Inspect actual discovered/executed case output; distinguish methods from cases |

# Checking Your Understanding

<details><summary>Why is a current Registration represented as an entity rather than a browser-only flag?</summary>

It associates a persisted user and Event with its own identity and timestamp. Server-side workflows must query and constrain the association across requests; browser state cannot enforce relational integrity, duplicate protection, or durable cancellation.

</details>

<details><summary>What distinct guarantees come from a unique pair index and an Event-row transaction?</summary>

The unique index enforces one current `(UserId, EventId)` row. The Event-row transaction coordinates aggregate capacity decisions across different users. Both safeguards are required for the Registration invariant.

</details>

<details><summary>Why can an availability display say one place remains even when a Register POST returns Full?</summary>

Details reflects a read snapshot. Another user can commit the final Registration between that read and the later POST. The POST rechecks current authoritative state under the shared coordination boundary.

</details>

<details><summary>Why does asynchronous execution alone not make count-and-insert atomic?</summary>

Asynchronous I/O changes waiting behavior; separate requests can still interleave their reads and writes. Atomicity and isolation require an appropriate database transaction and coordination design.

</details>

<details><summary>Which rule allows a Participant to cancel a Registration for a Cancelled Event?</summary>

The server checks that the Registration belongs to the authenticated user and that the Event has not reached its scheduled start. The SRS cancellation rule does not require current Published status.

</details>

<details><summary>Why must the Published Capacity Edit use coordination compatible with Register?</summary>

A count-increasing registration and a capacity-decreasing edit affect the same invariant. If each makes its decision from a stale independent snapshot, both could succeed and leave current Registrations greater than Capacity.

</details>

<details><summary>What can a simultaneous-start test prove, and what additional evidence shows actual lock contention?</summary>

The start-gate test checks outcomes and final persistence when requests are released together. A controlled transaction holding the Event lock while another request waits additionally demonstrates the lock and waiting behavior.

</details>

<details><summary>Why should rejected writes be checked with a fresh database context?</summary>

A fresh context reads committed persisted data rather than returning values from an existing change tracker. It can establish whether a rejected request left the authoritative database row unchanged.

</details>

# Summary

A reliable relational workflow starts by identifying which facts must be persisted, which values should be derived, and which constraints the database can enforce directly. Explicit associations, foreign keys, and uniqueness make individual records meaningful and protect their relationships. Response-specific models can present derived information without creating a second authoritative state that other writers must synchronize.

Protected mutations combine trustworthy identity, resource ownership, current-state eligibility, appropriate HTTP methods, and deliberate failure feedback. Application validation improves usability, while database constraints and transaction boundaries remain necessary because competing requests can invalidate a pre-check. A read–check–write operation that depends on a shared count must coordinate the entire decision, not merely its final write.

Consistency also extends across different workflows that change the same invariant. Evaluating all participating writers reveals race conditions that cannot be seen by reviewing one controller action in isolation. Reproducible relational tests, fixed clocks, independent contexts, HTTP-level checks, and manual observations provide complementary forms of evidence; each claim should be limited to what was actually exercised. Incremental migrations, regression checks, and reviewed commits keep these changes understandable and transferable to other systems.

# Checkpoint

**Inherited baseline:** CEMS has ASP.NET Core MVC/Razor, SQL Server/EF Core, Identity with authenticated-user Participant capabilities and elevated Organizer/Administrator roles, the relational Event/Category model, public search/Category filtering with selectable 10/25/50 pagination, direct Published/Cancelled Details, Organizer Create/Edit and lifecycle commands, `TimeProvider`, transient feedback, and an existing xUnit/HTTP SQL Server regression fixture.

**New in this lesson:**

- `Registration` entity, bidirectional navigations, EF mapping, foreign keys, unique User/Event index, and `AddParticipantRegistrations` migration.
- `EventDetailsViewModel` and live persisted registration count, remaining capacity, and signed-in registration state.
- Scoped `RegistrationService` with typed attempt outcomes and SQL Server Event-row transaction/locking for final-seat enforcement.
- Protected Register and Cancel POST actions, current-user `Mine` query, corresponding Razor page, navigation, and focused styling.
- Registration-aware Draft deletion and Published Capacity Edit floor with compatible Event-row locking.
- New relational service tests and HTTP authorization/ownership/capacity regression cases added to the inherited test suite.

**Resulting baseline for next lesson:** The lesson's implementation provides current Participant/Event Registration persistence, guarded enrollment and cancellation, live capacity, and Registration-dependent Event safeguards alongside the earlier public and Organizer workflows. Lesson 8 can build Organizer registration oversight and Administrator operations on the persisted Registration relationships, while preserving the accepted test infrastructure and application constraints.

# Next Lesson Preview

Lesson 8 extends the same relationships and protected request patterns to Organizer oversight of registrations for owned Events and Administrator workflows for cross-organizer Event management, Organizer-role assignment, and Category administration. You will reuse the current Registration counts, relational integrity rules, authorization boundaries, and automated verification foundation while distinguishing ordinary ownership-scoped authority from explicit Administrator authority.

# Appendix

## Database concurrency control and isolation strategies

The final-seat workflow coordinates writers through a short SQL Server transaction around a particular Event row. This is a form of **pessimistic concurrency control**: a writer acquires protection before making a decision from shared state. Other designs can detect conflicts after speculative work rather than blocking at the start. Neither family is categorically superior; the correct choice depends on the invariant, contention, database capabilities, and acceptable retry behavior.

### Isolation levels and visibility

A database's isolation level controls what a transaction may observe while other transactions operate. Distinguish the isolation setting from an application's chosen coordination protocol: selecting a level does not automatically identify the right resource to protect or ensure all writers follow the same discipline.

| SQL Server isolation approach | Illustrative behavior | Engineering consideration |
| --- | --- | --- |
| Read Uncommitted | May see changes not yet committed | Rarely appropriate for decisions that require reliable current state |
| Read Committed | Does not read uncommitted data; locking or row-version behavior depends on database settings | An ordinary count and later insert can still race without extra coordination |
| Repeatable Read | Protects repeatedly read rows against some concurrent changes | New matching rows (phantoms) can remain a concern |
| Serializable | Protects the transaction against relevant phantom insertions for its predicate/range | Stronger blocking and contention effects; proper indexed predicates matter |
| Snapshot | Reads a consistent versioned view and detects certain write conflicts | A transaction-wide snapshot does not by itself resolve every aggregate write-skew invariant |

CEMS explicitly acquires and holds a lock on the Event record. This makes the Event a coordination point for participating capacity decisions. The safety argument is about the shared protocol and lock duration, not an assumption that every query automatically receives serializable isolation.

### Pessimistic and optimistic coordination

Consider an Event with Capacity 1. Under pessimistic locking, the first writer holds the Event lock, writes its Registration, and commits before the next writer recounts. Under optimistic coordination, both writers may read the initial version. One must then detect the competing change and retry the *entire* capacity decision against fresh state. If an optimistic design only checks a Registration row's version, two different rows can still be inserted while the Event aggregate count changes.

With pessimistic locking, a competing request may wait before it can inspect or modify the protected resource. Benefits include a straightforward single-resource critical section and immediately authoritative rechecks. Costs include blocked requests, deadlocks when different workflows acquire locks in incompatible orders, and reduced throughput for heavily contended Events. Keep transactions narrow and maintain consistent lock ordering.

Optimistic concurrency commonly reads a version token (for example, a row version), computes a proposed change, and updates only if the version still matches. A conflict can be rejected or retried after re-reading state. This can reduce waiting when contention is rare, but a retry must re-evaluate the *entire* count/capacity rule. Merely adding a version column to one entity does not automatically protect arbitrary aggregate invariants involving rows in another table.

### Alternative seat-allocation designs

A reservation counter on the Event row could be changed with a single conditional update, such as incrementing only while the count is below capacity. That approach moves coordination into one row but introduces a denormalized value that must remain synchronized with cancellations, administrative changes, and any abnormal write path. A unique User/Event index remains necessary for duplicate protection. A database-specific stored procedure or explicit serialization policy could also centralize the decision; it changes the enforcement boundary and maintenance/testing trade-offs. The current association-counting design avoids a secondary counter while requiring participating writers to coordinate around the Event row.

### Lock diagnostics and controlled recovery

When requests appear slow, distinguish lock waits from query execution time, connection-pool exhaustion, and test-host scheduling. Inspect transaction duration, lock ownership, blocker and waiter sessions, and the exact SQL being executed in an authorized test environment. A deadlock involves cycles of waiting resources; SQL Server may select a victim whose transaction must be retried or handled as an expected failure according to the operation's policy. For a lock timeout, inspect the transaction outcome and unchanged or committed database state. Diagnostics should preserve the database state, release deliberately held test locks, and avoid indiscriminate broad exception handling.

## Transactions across multiple services and data stores

The CEMS Registration transaction modifies one SQL Server database, so the database can commit or roll back the coordinated unit. A different design might require a seat reservation in one database, a payment in another service, and a confirmation message through an external provider. No single local database transaction necessarily encloses all three effects.

### Coordination and compensation

Imagine registration followed by an external payment. The seat insert commits in SQL Server, but the payment request fails. A two-phase-commit design would need participating transactional resources and coordinated prepare/commit behavior. A saga design can instead record the seat reservation and, on payment refusal, issue a compensating seat release; recovery must also handle failure of that release operation. The local capacity guarantee still applies to each insertion or release within its own database boundary.

**Two-phase commit** seeks coordinated commit across participating transactional resources through a prepare and commit protocol. It can support strong cross-resource agreement but introduces coordinator, availability, configuration, and operational costs. **Sagas** instead compose local transactions with explicit compensating actions when later steps fail. A compensation is an explicit business operation that addresses effects already committed or observed by another service.

### Delivery, idempotency, and eventual consistency

External messages can be delayed, duplicated, or delivered after a process crash. An **idempotent** operation safely accepts repeated delivery of the same request without duplicating its effect. A transactional outbox can persist an outgoing message intent in the same local transaction as the registration change, while a background publisher delivers it later. Consumers still need a duplication strategy; consumers must also handle repeated delivery safely.

**Eventual consistency** permits different components to briefly disagree, provided the system has a defined convergence and recovery process. This is suitable for some notification and reporting projections but not a reason to abandon the seat-capacity invariant inside the authoritative registration database. Local atomicity and distributed eventual consistency can coexist at different system boundaries.

## Registration lifecycle and capacity-model alternatives

CEMS stores current Registrations: cancellation removes a row, and a count of remaining associations reflects occupied places. This keeps the required behavior compact and makes the unique pair index straightforward. Other product requirements could motivate a more elaborate lifecycle.

### Current associations, history, and status models

Under the current-association model, a cancelled row is deleted. A re-registration for the same Event inserts a new row with a new creation timestamp; the count includes exactly the rows representing active places. A historical model would preserve the old row and represent the new attempt separately, so both uniqueness and capacity queries must incorporate an active-state condition.

A historical registration model might preserve `Active`, `Cancelled`, and `Waitlisted` states or record cancellation timestamps in an audit log. That design could answer questions such as who previously held a place, but count queries must explicitly include only active associations. A unique `(UserId, EventId)` constraint would then need reassessment: prohibiting any historical duplicate may accidentally prevent a user from registering again after cancellation, while allowing all duplicates would fail to protect current capacity. A filtered unique index or separate current-association table can express a more precise rule, depending on database capabilities.

### Derived counts versus materialized counters

Derived counts reduce synchronization responsibilities but require database reads to determine current occupancy. Materialized counts may make heavily used displays faster, but every register, cancel, import, administrator action, and repair must update them consistently. Caching a count for display is less demanding than treating the cache as authoritative for new seat allocation. An application can use read-optimized cached views while continuing to make mutations against authoritative persisted state.

### Waitlists, holds, and partial reservations

A waitlist introduces distinct states for **requesting a place** and **holding a place**. Temporary holds add expirations and recovery: the system must decide when a hold consumes capacity, how expiration is coordinated, and whether payment or confirmation finalizes it. These features expand the state machine, concurrency protocol, and acceptance criteria substantially. They illustrate how changes in domain requirements lead to different persistence and consistency models, rather than a single universal Registration schema.
