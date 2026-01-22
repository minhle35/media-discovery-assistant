# Budget-Aware Entertainment Assistant
## Complete Development Task List & User Stories

**Project Duration:** 13 Weeks (1 Developer)
**Tech Stack:** 
ASP.NET Core 8, Entity Framework Core 8, PostgreSQL, Ollama (Phi-3 Mini), SignalR, AWS (Cognito, S3, RDS)

---

## Table of Contents

1. [Phase 1: Foundation & Infrastructure](#phase-1-foundation--infrastructure-weeks-1-2)
2. [Phase 2: Authentication & User Management](#phase-2-authentication--user-management-weeks-3-4)
3. [Phase 3: AI Core - The Brain](#phase-3-ai-core---the-brain-weeks-5-7)
4. [Phase 4: Document Processing](#phase-4-document-processing-week-8)
5. [Phase 5: Chat & Real-Time Communication](#phase-5-chat--real-time-communication-weeks-9-10)
6. [Phase 6: Recommendations Engine](#phase-6-recommendations-engine-week-11)
7. [Phase 7: Testing & Quality Assurance](#phase-7-testing--quality-assurance-week-12)
8. [Phase 8: AWS Deployment & Production](#phase-8-aws-deployment--production-week-13)

---

## Phase 1: Foundation & Infrastructure (Weeks 1-2)

### Epic 1: Project Setup & Clean Architecture

#### Story 1.1: Solution Structure Setup
**Priority:** Critical | **Estimate:** 1 day

**As a** developer
**I want** to set up the Clean Architecture solution structure
**So that** the codebase is maintainable, testable, and follows best practices

**Tasks:**
- [ ] Create solution with 5 projects:
  - `EntertainmentAssistant.Domain` (entities, interfaces, domain logic)
  - `EntertainmentAssistant.Application` (use cases, DTOs, services interfaces)
  - `EntertainmentAssistant.Infrastructure` (EF Core, external services, repositories)
  - `EntertainmentAssistant.Api` (controllers, middleware, configuration)
  - `EntertainmentAssistant.Tests` (unit and integration tests)
- [ ] Configure project dependencies (Domain → none, Application → Domain, Infrastructure → Application, Api → all)
- [ ] Set up Directory.Build.props for shared settings
- [ ] Enable nullable reference types and C# 12 features
- [ ] Add .editorconfig for consistent code style
- [ ] Create initial README.md with setup instructions

**Acceptance Criteria:**
- [ ] Solution builds successfully with `dotnet build`
- [ ] All projects reference correctly without circular dependencies
- [ ] Nullable reference types enabled across all projects

**Technical Notes:**
- Dependencies: None
- Libraries: None yet

---

#### Story 1.2: Core NuGet Packages Configuration
**Priority:** Critical | **Estimate:** 0.5 day

**As a** developer
**I want** to configure all required NuGet packages
**So that** dependencies are properly managed across the solution

**Tasks:**
- [ ] Add packages to Domain project:
  - None (keep domain pure)
- [ ] Add packages to Application project:
  - `FluentValidation` (validation)
  - `MediatR` (optional, for CQRS pattern)
- [ ] Add packages to Infrastructure project:
  - `Npgsql.EntityFrameworkCore.PostgreSQL`
  - `Microsoft.EntityFrameworkCore.Tools`
  - `AWSSDK.S3`
  - `AWSSDK.CognitoIdentityProvider`
  - `Polly` (resilience)
  - `Serilog.AspNetCore`
- [ ] Add packages to Api project:
  - `Microsoft.AspNetCore.Authentication.JwtBearer`
  - `Microsoft.AspNetCore.SignalR`
  - `Swashbuckle.AspNetCore`
- [ ] Add packages to Tests project:
  - `xUnit`
  - `Moq`
  - `FluentAssertions`
  - `Microsoft.AspNetCore.Mvc.Testing`
  - `Testcontainers.PostgreSql`

**Acceptance Criteria:**
- [ ] All packages install without conflicts
- [ ] Package versions are compatible

---

#### Story 1.3: Docker Environment Setup
**Priority:** Critical | **Estimate:** 1 day

**As a** developer
**I want** to configure Docker environment for local development
**So that** all services run consistently in containers

**Tasks:**
- [ ] Create `Dockerfile` for API with multi-stage build
- [ ] Create `docker-compose.yml` with services:
  - `api` - ASP.NET Core application
  - `db` - PostgreSQL 16
  - `ollama` - Ollama with Phi-3 Mini model
- [ ] Create `docker-compose.override.yml` for development settings
- [ ] Configure environment variables and secrets
- [ ] Set up volume mounts for data persistence
- [ ] Create `init-ollama.sh` script to pull Phi-3 model on first run
- [ ] Add health checks for all services
- [ ] Create `.dockerignore` file

**Acceptance Criteria:**
- [ ] `docker-compose up` starts all services successfully
- [ ] API can connect to PostgreSQL
- [ ] API can communicate with Ollama
- [ ] Data persists across container restarts

**Technical Notes:**
```yaml
# docker-compose.yml structure
services:
  api:
    build: .
    ports: ["5000:8080"]
    depends_on: [db, ollama]
  db:
    image: postgres:16
    volumes: [pgdata:/var/lib/postgresql/data]
  ollama:
    image: ollama/ollama:latest
    volumes: [ollama_models:/root/.ollama]
```

---

### Epic 2: Domain Layer & Database Schema

#### Story 2.1: Core Domain Entities
**Priority:** Critical | **Estimate:** 1.5 days

**As a** developer
**I want** to define all domain entities
**So that** the data model accurately represents the business domain

**Tasks:**
- [ ] Create `User` entity:
  ```csharp
  - Id (Guid)
  - CognitoUserId (string)
  - Email (string)
  - DisplayName (string)
  - CreatedAt, UpdatedAt (DateTime)
  ```
- [ ] Create `FinancialProfile` entity:
  ```csharp
  - Id, UserId (FK)
  - MonthlyIncome (decimal?)
  - MonthlyBudgetLimit (decimal)
  - CurrentPeriodSpent (decimal)
  - PeriodStartDate (DateTime)
  - IncomeSource (enum: Manual, Document)
  - DiscretionaryBudget (decimal?)
  ```
- [ ] Create `EntertainmentPreferences` entity:
  ```csharp
  - Id, UserId (FK)
  - LikedGenres (List<string>) - JSON column
  - DislikedGenres (List<string>) - JSON column
  - PreferredPlatforms (List<string>) - JSON column
  - MaxPricePerItem (decimal)
  - LearnedNotes (List<string>) - JSON column
  ```
- [ ] Create `ActivityRecord` entity:
  ```csharp
  - Id, UserId (FK)
  - Title (string)
  - Type (enum: Movie, Series, Game, Book, Event)
  - Cost (decimal)
  - UserRating (int?)
  - Genre (string)
  - Platform (string)
  - RecordedAt (DateTime)
  ```
- [ ] Create `EntertainmentItem` entity:
  ```csharp
  - Id
  - Title, Description (string)
  - Type (enum)
  - Genres (List<string>) - JSON
  - Platforms (List<string>) - JSON
  - BasePrice (decimal)
  - CurrentPrice (decimal?)
  - ImageUrl (string?)
  - ExternalId (string?) - for API references
  - IsActive (bool)
  ```
- [ ] Create `ChatSession` entity:
  ```csharp
  - Id, UserId (FK)
  - StartedAt, LastMessageAt (DateTime)
  - IsActive (bool)
  - Summary (string?) - AI-generated summary
  ```
- [ ] Create `ChatMessage` entity:
  ```csharp
  - Id, SessionId (FK)
  - Role (enum: User, Assistant, System)
  - Content (string)
  - TokenCount (int)
  - CreatedAt (DateTime)
  - ExtractedData (JSON?) - structured extraction result
  ```
- [ ] Create `Recommendation` entity:
  ```csharp
  - Id, UserId (FK), EntertainmentItemId (FK)
  - RecommendedAt (DateTime)
  - Reason (string) - AI explanation
  - UserFeedback (enum?: Liked, Disliked, Saved, Dismissed)
  - WasActedUpon (bool)
  ```
- [ ] Create `FinancialDocument` entity:
  ```csharp
  - Id, UserId (FK)
  - FileName, S3Key (string)
  - DocumentType (enum: PayStub, BankStatement, Receipt, Other)
  - UploadedAt, ProcessedAt (DateTime?)
  - Status (enum: Pending, Processing, Completed, Failed)
  - ExtractionResult (JSON?)
  - ConfidenceScore (decimal?)
  ```

**Acceptance Criteria:**
- [ ] All entities have proper data annotations
- [ ] Relationships are correctly defined
- [ ] Enums are created for all type fields

---

#### Story 2.2: Entity Framework Configuration
**Priority:** Critical | **Estimate:** 1 day

**As a** developer
**I want** to configure Entity Framework with proper mappings
**So that** the database schema is optimized and relationships work correctly

**Tasks:**
- [ ] Create `AppDbContext` class
- [ ] Create entity configurations (IEntityTypeConfiguration<T>):
  - `UserConfiguration`
  - `FinancialProfileConfiguration`
  - `EntertainmentPreferencesConfiguration`
  - `ActivityRecordConfiguration`
  - `EntertainmentItemConfiguration`
  - `ChatSessionConfiguration`
  - `ChatMessageConfiguration`
  - `RecommendationConfiguration`
  - `FinancialDocumentConfiguration`
- [ ] Configure JSON columns for PostgreSQL
- [ ] Add proper indexes:
  - User.CognitoUserId (unique)
  - User.Email (unique)
  - ActivityRecord.UserId + RecordedAt
  - ChatMessage.SessionId + CreatedAt
  - EntertainmentItem.Type + IsActive
- [ ] Configure cascade delete rules
- [ ] Set up audit fields (CreatedAt, UpdatedAt) with value generators

**Acceptance Criteria:**
- [ ] All configurations compile without errors
- [ ] JSON columns serialize/deserialize correctly
- [ ] Indexes are created in migration

---

#### Story 2.3: Database Migrations
**Priority:** Critical | **Estimate:** 0.5 day

**As a** developer
**I want** to create and test database migrations
**So that** the database schema can be versioned and deployed

**Tasks:**
- [ ] Generate initial migration: `dotnet ef migrations add InitialCreate`
- [ ] Review generated migration for correctness
- [ ] Test migration up: `dotnet ef database update`
- [ ] Test migration down: `dotnet ef database update 0`
- [ ] Create migration for seed data
- [ ] Document migration commands in README

**Acceptance Criteria:**
- [ ] Migration creates all tables correctly
- [ ] Migration can be rolled back
- [ ] Schema matches entity definitions

---

#### Story 2.4: Seed Entertainment Data
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to seed the database with entertainment items
**So that** there is data available for testing and demonstration

**Tasks:**
- [ ] Create seed data for 50+ entertainment items:
  - 15+ Movies (various genres, $3-20 range)
  - 10+ TV Series (streaming platforms)
  - 10+ Video Games ($0-70 range)
  - 10+ Books ($5-25 range)
  - 5+ Board/Card Games ($15-50 range)
- [ ] Include diverse:
  - Price ranges (free to premium)
  - Genres (action, comedy, drama, sci-fi, etc.)
  - Platforms (Netflix, Prime, Steam, etc.)
- [ ] Create `SeedDataMigration` or use `HasData()` in configuration
- [ ] Add realistic descriptions and metadata

**Acceptance Criteria:**
- [ ] Database has 50+ entertainment items after seeding
- [ ] Items cover all entertainment types
- [ ] Price ranges span from free to $70

---

### Epic 3: Configuration & Logging

#### Story 3.1: Application Configuration
**Priority:** High | **Estimate:** 0.5 day

**As a** developer
**I want** to set up proper configuration management
**So that** settings are environment-specific and secure

**Tasks:**
- [ ] Create `appsettings.json` with base configuration
- [ ] Create `appsettings.Development.json` for local settings
- [ ] Create strongly-typed configuration classes:
  - `DatabaseSettings`
  - `OllamaSettings`
  - `AwsSettings`
  - `JwtSettings`
- [ ] Configure options pattern in `Program.cs`
- [ ] Set up user secrets for local development
- [ ] Document all configuration options

**Acceptance Criteria:**
- [ ] Configuration loads correctly per environment
- [ ] Secrets are not committed to source control
- [ ] All settings have sensible defaults

---

#### Story 3.2: Structured Logging Setup
**Priority:** High | **Estimate:** 0.5 day

**As a** developer
**I want** to configure structured logging with Serilog
**So that** logs are searchable and useful for debugging

**Tasks:**
- [ ] Configure Serilog in `Program.cs`
- [ ] Set up console sink for development
- [ ] Configure log levels per namespace
- [ ] Add request/response logging middleware
- [ ] Create correlation ID middleware
- [ ] Set up log enrichment (UserId, SessionId, etc.)

**Acceptance Criteria:**
- [ ] Logs include structured properties
- [ ] Request/response logging works
- [ ] Correlation IDs trace through requests

---

## Phase 2: Authentication & User Management (Weeks 3-4)

### Epic 4: AWS Cognito Integration

#### Story 4.1: AWS Resources Setup
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to set up AWS resources
**So that** the application can use AWS services

**Tasks:**
- [ ] Create or configure AWS account
- [ ] Set up AWS CLI with credentials
- [ ] Create Cognito User Pool:
  - Configure password policy
  - Set up required attributes (email)
  - Create App Client (no secret for SPA compatibility)
- [ ] Create S3 bucket for documents:
  - Configure CORS policy
  - Set up bucket policy for secure access
  - Enable versioning
- [ ] Document all AWS resource ARNs and IDs
- [ ] Create IAM user/role for application with minimal permissions

**Acceptance Criteria:**
- [ ] Cognito User Pool created and accessible
- [ ] S3 bucket created with proper permissions
- [ ] AWS credentials work from local environment

**Technical Notes:**
- Keep AWS resources in a single region (e.g., us-east-1)
- Use resource naming convention: `entertainment-assistant-{env}-{resource}`

---

#### Story 4.2: Cognito Authentication Service
**Priority:** High | **Estimate:** 1.5 days

**As a** developer
**I want** to implement Cognito authentication service
**So that** users can securely authenticate

**Tasks:**
- [ ] Create `ICognitoAuthService` interface:
  ```csharp
  Task<AuthResult> RegisterAsync(RegisterRequest request)
  Task<AuthResult> LoginAsync(LoginRequest request)
  Task<AuthResult> RefreshTokenAsync(string refreshToken)
  Task<bool> ConfirmEmailAsync(string email, string code)
  Task<bool> ForgotPasswordAsync(string email)
  Task<bool> ResetPasswordAsync(ResetPasswordRequest request)
  ```
- [ ] Implement `CognitoAuthService`
- [ ] Create request/response DTOs
- [ ] Add input validation with FluentValidation
- [ ] Implement retry logic with Polly for AWS calls
- [ ] Handle Cognito exceptions and map to application errors

**Acceptance Criteria:**
- [ ] Users can register with email/password
- [ ] Users can login and receive JWT tokens
- [ ] Token refresh works correctly
- [ ] Proper error messages for invalid credentials

---

#### Story 4.3: JWT Authentication Middleware
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to configure JWT validation middleware
**So that** API endpoints are protected

**Tasks:**
- [ ] Configure JWT Bearer authentication in `Program.cs`
- [ ] Set up Cognito JWT validation parameters
- [ ] Create custom claims transformation (extract user info)
- [ ] Implement `ICurrentUserService` to access authenticated user
- [ ] Add `[Authorize]` attribute to protected controllers
- [ ] Configure Swagger to support JWT authentication

**Acceptance Criteria:**
- [ ] Protected endpoints reject requests without valid token
- [ ] User claims are accessible in controllers
- [ ] Swagger UI allows token input

---

#### Story 4.4: Authentication Controller
**Priority:** High | **Estimate:** 1 day

**As a** user
**I want** API endpoints for authentication
**So that** I can register, login, and manage my account

**Tasks:**
- [ ] Create `AuthController` with endpoints:
  - `POST /api/auth/register`
  - `POST /api/auth/login`
  - `POST /api/auth/refresh`
  - `POST /api/auth/confirm-email`
  - `POST /api/auth/forgot-password`
  - `POST /api/auth/reset-password`
  - `GET /api/auth/me` (get current user profile)
- [ ] Create DTOs for all requests/responses
- [ ] Add FluentValidation validators
- [ ] Implement proper HTTP status codes
- [ ] Add rate limiting for auth endpoints

**Acceptance Criteria:**
- [ ] All auth endpoints work end-to-end
- [ ] Validation errors return 400 with details
- [ ] Successful login returns access + refresh tokens

---

### Epic 5: User Profile Management

#### Story 5.1: User Service Implementation
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to implement user profile management
**So that** user data is properly managed

**Tasks:**
- [ ] Create `IUserService` interface:
  ```csharp
  Task<UserDto> GetByIdAsync(Guid id)
  Task<UserDto> GetByCognitoIdAsync(string cognitoId)
  Task<UserDto> CreateAsync(CreateUserRequest request)
  Task<UserDto> UpdateAsync(Guid id, UpdateUserRequest request)
  Task<bool> DeleteAsync(Guid id)
  ```
- [ ] Implement `UserService`
- [ ] Create `IUserRepository` and implementation
- [ ] Handle user creation on first login (from Cognito)
- [ ] Add caching for frequently accessed user data

**Acceptance Criteria:**
- [ ] Users are created in database on first login
- [ ] User profiles can be retrieved and updated
- [ ] Proper error handling for not found cases

---

#### Story 5.2: Financial Profile Service
**Priority:** High | **Estimate:** 1.5 days

**As a** user
**I want** to set up and manage my financial profile
**So that** the assistant knows my budget constraints

**Tasks:**
- [ ] Create `IFinancialProfileService` interface:
  ```csharp
  Task<FinancialProfileDto> GetByUserIdAsync(Guid userId)
  Task<FinancialProfileDto> CreateOrUpdateAsync(Guid userId, FinancialProfileRequest request)
  Task<BudgetSummaryDto> GetBudgetSummaryAsync(Guid userId)
  Task RecordExpenseAsync(Guid userId, decimal amount, string description)
  Task ResetPeriodAsync(Guid userId)
  ```
- [ ] Implement `FinancialProfileService`
- [ ] Add budget calculation logic:
  - Calculate remaining budget
  - Track spending by period
  - Auto-reset on period start
- [ ] Create budget alert thresholds (50%, 75%, 90%)
- [ ] Add validation for reasonable budget amounts

**Acceptance Criteria:**
- [ ] Users can create financial profiles
- [ ] Budget calculations are accurate
- [ ] Spending is tracked correctly
- [ ] Period resets work automatically

---

#### Story 5.3: Entertainment Preferences Service
**Priority:** High | **Estimate:** 1 day

**As a** user
**I want** to manage my entertainment preferences
**So that** recommendations match my interests

**Tasks:**
- [ ] Create `IPreferencesService` interface:
  ```csharp
  Task<PreferencesDto> GetByUserIdAsync(Guid userId)
  Task<PreferencesDto> UpdateAsync(Guid userId, UpdatePreferencesRequest request)
  Task AddLikedGenreAsync(Guid userId, string genre)
  Task AddDislikedGenreAsync(Guid userId, string genre)
  Task UpdatePlatformsAsync(Guid userId, List<string> platforms)
  Task AddLearnedNoteAsync(Guid userId, string note)
  ```
- [ ] Implement `PreferencesService`
- [ ] Validate genres against known list
- [ ] Validate platforms against known list
- [ ] Implement preference merging (don't lose existing data)

**Acceptance Criteria:**
- [ ] Preferences can be created and updated
- [ ] Invalid genres/platforms are rejected
- [ ] Updates merge with existing data

---

#### Story 5.4: Profile Controller
**Priority:** High | **Estimate:** 1 day

**As a** user
**I want** API endpoints to manage my profile
**So that** I can view and update my information

**Tasks:**
- [ ] Create `ProfileController` with endpoints:
  - `GET /api/profile` - get complete profile
  - `PUT /api/profile` - update basic info
  - `GET /api/profile/financial` - get financial profile
  - `PUT /api/profile/financial` - update financial profile
  - `GET /api/profile/preferences` - get preferences
  - `PUT /api/profile/preferences` - update preferences
  - `GET /api/profile/budget-summary` - get budget status
- [ ] Create comprehensive DTOs
- [ ] Add validation for all inputs
- [ ] Return proper error responses

**Acceptance Criteria:**
- [ ] All profile endpoints work correctly
- [ ] Data validation is enforced
- [ ] Only authenticated users can access

---

## Phase 3: AI Core - The Brain (Weeks 5-7)

### Epic 6: Ollama Integration & Model Selection

#### Story 6.1: Model Selection Spike
**Priority:** Critical | **Estimate:** 1 day

**As a** developer
**I want** to evaluate and select the optimal local model
**So that** the application performs well with limited resources

**Tasks:**
- [ ] Set up test environment with Ollama
- [ ] Test candidate models:
  - `phi3:mini` (3.8B parameters)
  - `llama3.2:3b` (3B parameters)
  - `qwen2.5:3b` (3B parameters)
- [ ] Evaluate each model on:
  - JSON extraction accuracy (10 test cases)
  - Conversation quality (5 scenarios)
  - Response latency (average of 20 calls)
  - RAM usage under load
- [ ] Document findings with benchmarks
- [ ] Select final model with justification

**Acceptance Criteria:**
- [ ] All candidate models tested
- [ ] Benchmark results documented
- [ ] Final model selected with clear reasoning
- [ ] Model can run on 8GB RAM machine

**Technical Notes:**
- Test prompts should match real use cases
- Measure cold start vs warm response times

---

#### Story 6.2: Ollama Service Implementation
**Priority:** Critical | **Estimate:** 1.5 days

**As a** developer
**I want** to implement Ollama integration service
**So that** the application can communicate with the local LLM

**Tasks:**
- [ ] Create `IOllamaService` interface:
  ```csharp
  Task<string> GenerateAsync(string prompt, GenerationOptions options)
  IAsyncEnumerable<string> StreamGenerateAsync(string prompt, GenerationOptions options)
  Task<T> GenerateStructuredAsync<T>(string prompt, GenerationOptions options)
  Task<bool> IsHealthyAsync()
  ```
- [ ] Implement `OllamaService` with HttpClient
- [ ] Create `GenerationOptions` class:
  ```csharp
  - Temperature (default: 0.7)
  - TopP (default: 0.9)
  - MaxTokens (default: 1024)
  - NumCtx (context window, default: 2048)
  ```
- [ ] Implement streaming with IAsyncEnumerable
- [ ] Add retry logic with Polly (3 retries, exponential backoff)
- [ ] Implement connection pooling
- [ ] Add request/response logging
- [ ] Handle timeout scenarios (30s default)

**Acceptance Criteria:**
- [ ] Can send prompts and receive responses
- [ ] Streaming works correctly
- [ ] Retries handle transient failures
- [ ] Timeouts are handled gracefully

---

#### Story 6.3: Prompt Template Management
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to manage prompt templates centrally
**So that** prompts are consistent and maintainable

**Tasks:**
- [ ] Create `IPromptTemplateService` interface
- [ ] Implement template storage (embedded resources or files)
- [ ] Create base templates:
  - `system_prompt.txt` - base assistant personality
  - `extraction_prompt.txt` - for data extraction
  - `intent_classification_prompt.txt` - for intent detection
  - `recommendation_prompt.txt` - for generating recommendations
  - `financial_analysis_prompt.txt` - for document analysis
  - `followup_prompt.txt` - for generating follow-up questions
- [ ] Implement template variable substitution
- [ ] Add template validation (check for required variables)
- [ ] Create template versioning for A/B testing

**Acceptance Criteria:**
- [ ] Templates load correctly
- [ ] Variable substitution works
- [ ] Missing variables throw clear errors

---

### Epic 7: Intent Classification System

#### Story 7.1: Intent Classifier Implementation
**Priority:** Critical | **Estimate:** 1.5 days

**As a** developer
**I want** to classify user message intents
**So that** the system routes messages to appropriate handlers

**Tasks:**
- [ ] Define `ConversationIntent` enum:
  ```csharp
  Onboarding,           // New user setup
  PreferenceUpdate,     // Updating likes/dislikes
  Recommendation,       // Asking for suggestions
  BudgetQuery,          // Asking about budget/spending
  TrackExpense,         // Recording a purchase
  ActivityLog,          // Logging watched/played item
  Search,               // Searching for specific item
  Feedback,             // Rating a recommendation
  GeneralChat,          // Casual conversation
  Help,                 // Asking for help
  Unknown               // Cannot determine
  ```
- [ ] Create `IIntentClassifier` interface:
  ```csharp
  Task<IntentClassificationResult> ClassifyAsync(string message, ConversationContext context)
  ```
- [ ] Create `IntentClassificationResult`:
  ```csharp
  - Intent (ConversationIntent)
  - Confidence (decimal 0-1)
  - ExtractedEntities (Dictionary<string, string>)
  ```
- [ ] Implement hybrid classifier:
  - Rule-based for common patterns (fast path)
  - Model-based for ambiguous cases
- [ ] Create rule patterns for each intent:
  - "recommend" / "suggest" / "what should I" → Recommendation
  - "spent" / "bought" / "cost me" → TrackExpense
  - "budget" / "how much left" / "remaining" → BudgetQuery
  - etc.
- [ ] Add confidence threshold (below 0.6 → ask for clarification)

**Acceptance Criteria:**
- [ ] Classifies 90%+ of common messages correctly
- [ ] Returns confidence scores
- [ ] Extracts relevant entities (amounts, titles, etc.)
- [ ] Fast path handles obvious intents in <50ms

---

#### Story 7.2: Intent Handler Registry
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to route intents to appropriate handlers
**So that** each intent type is processed correctly

**Tasks:**
- [ ] Create `IIntentHandler` interface:
  ```csharp
  ConversationIntent HandledIntent { get; }
  Task<HandlerResult> HandleAsync(IntentContext context)
  ```
- [ ] Create handler implementations:
  - `OnboardingHandler`
  - `RecommendationHandler`
  - `BudgetQueryHandler`
  - `TrackExpenseHandler`
  - `ActivityLogHandler`
  - `PreferenceUpdateHandler`
  - `GeneralChatHandler`
- [ ] Create `IntentHandlerRegistry` for handler resolution
- [ ] Implement handler pipeline with pre/post processing
- [ ] Add handler metrics (execution time, success rate)

**Acceptance Criteria:**
- [ ] Each intent maps to a handler
- [ ] Handlers execute correctly
- [ ] Unknown intents have fallback behavior

---

### Epic 8: Extraction Pipeline

#### Story 8.1: Extraction Service Implementation
**Priority:** Critical | **Estimate:** 2 days

**As a** developer
**I want** to extract structured data from user messages
**So that** conversations update the user profile automatically

**Tasks:**
- [ ] Create extraction result models:
  ```csharp
  public class ExtractionResult
  {
      public BudgetUpdate? BudgetUpdate { get; set; }
      public List<PreferenceSignal> PreferenceSignals { get; set; }
      public ActivityRecord? ActivityToLog { get; set; }
      public WatchlistUpdate? WatchlistUpdate { get; set; }
      public decimal Confidence { get; set; }
      public bool RequiresFollowUp { get; set; }
      public string? FollowUpQuestion { get; set; }
  }
  
  public class BudgetUpdate
  {
      public decimal? AmountSpent { get; set; }
      public decimal? NewLimit { get; set; }
      public string? Description { get; set; }
  }
  
  public class PreferenceSignal
  {
      public PreferenceType Type { get; set; } // Like, Dislike
      public PreferenceCategory Category { get; set; } // Genre, Platform, PriceRange
      public string Value { get; set; }
      public decimal Confidence { get; set; }
  }
  ```
- [ ] Create `IExtractionService` interface:
  ```csharp
  Task<ExtractionResult> ExtractFromMessageAsync(string userMessage, string? assistantResponse, ConversationIntent intent)
  Task<ExtractionResult> ExtractFromConversationAsync(List<ChatMessage> messages)
  ```
- [ ] Implement `ExtractionService`:
  - Build extraction prompt with JSON schema
  - Call Ollama with structured output request
  - Parse and validate JSON response
  - Handle malformed responses gracefully
- [ ] Create extraction prompts per intent type
- [ ] Implement confidence scoring
- [ ] Add fallback extraction (regex for amounts, titles)

**Acceptance Criteria:**
- [ ] Extracts spending amounts correctly (e.g., "spent $15" → 15)
- [ ] Extracts preferences (e.g., "love sci-fi" → like:genre:sci-fi)
- [ ] Extracts activity records (e.g., "watched Dune" → title:Dune, type:movie)
- [ ] Confidence scores reflect extraction quality
- [ ] Malformed model output doesn't crash the service

**Technical Notes:**
- Use JSON schema in prompt for reliable structure
- Always validate extracted data before applying

---

#### Story 8.2: Profile Update Service
**Priority:** Critical | **Estimate:** 1 day

**As a** developer
**I want** to apply extractions to user profiles
**So that** the profile stays current with each conversation

**Tasks:**
- [ ] Create `IProfileUpdateService` interface:
  ```csharp
  Task<UpdateResult> ApplyExtractionAsync(Guid userId, ExtractionResult extraction)
  Task<UpdateResult> ApplyBudgetUpdateAsync(Guid userId, BudgetUpdate update)
  Task<UpdateResult> ApplyPreferenceSignalsAsync(Guid userId, List<PreferenceSignal> signals)
  Task<UpdateResult> LogActivityAsync(Guid userId, ActivityRecord activity)
  ```
- [ ] Implement update logic:
  - Validate updates against business rules
  - Apply budget updates (add to spent amount)
  - Merge preference signals (handle conflicts)
  - Log activities with deduplication
- [ ] Create merge strategies:
  - Preferences: Recency wins, or require multiple signals
  - Budget: Always apply (with validation)
  - Activities: Dedupe by title + date
- [ ] Add update audit trail
- [ ] Implement rollback capability for testing

**Acceptance Criteria:**
- [ ] Budget updates reflect in profile
- [ ] Preferences merge correctly
- [ ] Activities are logged without duplicates
- [ ] All updates are auditable

---

#### Story 8.3: Extraction Validation & Grounding
**Priority:** Critical | **Estimate:** 1 day

**As a** developer
**I want** to validate extracted data against real sources
**So that** hallucinated data doesn't corrupt the profile

**Tasks:**
- [ ] Create `IExtractionValidator` interface:
  ```csharp
  Task<ValidationResult> ValidateAsync(ExtractionResult extraction, ValidationContext context)
  ```
- [ ] Implement validation rules:
  - **Budget**: Amount > 0, Amount < MonthlyLimit * 2
  - **Activity titles**: Fuzzy match against entertainment catalog
  - **Genres**: Must be in known genre list
  - **Platforms**: Must be in known platform list
  - **Prices**: Must be reasonable ($0-500 range)
- [ ] Create `ValidationResult` with:
  - IsValid (bool)
  - Corrections (list of auto-corrections made)
  - Rejections (list of rejected extractions with reasons)
- [ ] Implement auto-correction:
  - Fix obvious typos in genre names
  - Normalize platform names (e.g., "prime" → "Amazon Prime")
- [ ] Log validation failures for analysis

**Acceptance Criteria:**
- [ ] Invalid amounts are rejected
- [ ] Unknown genres are flagged
- [ ] Titles are matched against catalog
- [ ] Auto-corrections are logged

---

### Epic 9: Context Assembly & Token Management

#### Story 9.1: Context Assembler Implementation
**Priority:** Critical | **Estimate:** 1.5 days

**As a** developer
**I want** to assemble efficient context for model calls
**So that** the model has relevant information within token limits

**Tasks:**
- [ ] Create `IContextAssembler` interface:
  ```csharp
  Task<AssembledContext> AssembleAsync(Guid userId, ConversationIntent intent, string userMessage)
  ```
- [ ] Create `AssembledContext`:
  ```csharp
  - SystemPrompt (string)
  - UserContext (string) - profile summary
  - ConversationHistory (string) - recent turns
  - GroundingData (string) - intent-specific data
  - TotalTokenEstimate (int)
  ```
- [ ] Implement token budget allocation:
  ```
  Total: 2048 tokens
  - System prompt: 400 tokens (fixed)
  - User profile: 300 tokens (compressed)
  - Conversation history: 400 tokens (last 2-3 turns)
  - Grounding data: 500 tokens (intent-specific)
  - User message: 200 tokens (variable)
  - Response space: 248 tokens (remainder)
  ```
- [ ] Create intent-specific loading strategies:
  - **Recommendation**: Load full preferences + affordable items
  - **BudgetQuery**: Load detailed budget info
  - **TrackExpense**: Load recent activity (for context)
  - **GeneralChat**: Minimal context
- [ ] Implement token estimation (rough: chars/4)
- [ ] Add context truncation with priority (keep most important)

**Acceptance Criteria:**
- [ ] Context stays within 2048 tokens
- [ ] Relevant data is included per intent
- [ ] Token estimation is reasonably accurate
- [ ] Truncation preserves critical information

---

#### Story 9.2: Compact Profile Serialization
**Priority:** Critical | **Estimate:** 1 day

**As a** developer
**I want** to serialize user profiles compactly
**So that** profile context fits in ~300 tokens

**Tasks:**
- [ ] Create `IProfileSerializer` interface:
  ```csharp
  string SerializeCompact(UserProfile profile, int maxTokens = 300)
  string SerializeForIntent(UserProfile profile, ConversationIntent intent)
  ```
- [ ] Implement compact format:
  ```
  Budget: $47 remaining of $100 (spent $53 this month)
  Likes: sci-fi, thriller, comedy | Dislikes: horror, musical
  Platforms: Netflix, Prime | Max price: $15
  Recent: Dune (loved), The Office (good), Elden Ring (playing)
  Notes: prefers weekends, likes short films
  ```
- [ ] Create intent-specific variants:
  - Recommendation: Full preferences, top history
  - BudgetQuery: Detailed budget breakdown
  - TrackExpense: Recent activity focus
- [ ] Implement aggressive summarization:
  - Top 5 genres only
  - Last 5 activities only
  - Top 3 learned notes only
- [ ] Add token counting validation

**Acceptance Criteria:**
- [ ] Full profile serializes in <300 tokens
- [ ] All critical information is preserved
- [ ] Intent-specific variants are appropriately focused

---

#### Story 9.3: Conversation History Management
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to manage conversation history efficiently
**So that** context includes relevant recent messages

**Tasks:**
- [ ] Create `IConversationHistoryService` interface:
  ```csharp
  Task<List<ChatMessage>> GetRecentMessagesAsync(Guid sessionId, int maxMessages = 3)
  Task<string> FormatForContextAsync(Guid sessionId, int maxTokens = 400)
  Task<string> SummarizeSessionAsync(Guid sessionId)
  ```
- [ ] Implement history retrieval:
  - Get last N messages
  - Filter to user + assistant roles only
  - Exclude system messages from context
- [ ] Implement history formatting:
  ```
  User: What should I watch tonight?
  Assistant: Based on your love of sci-fi, I recommend Arrival. It's $4 on Prime.
  User: That sounds good, I'll watch it.
  ```
- [ ] Implement session summarization (for long sessions):
  - Use model to generate 2-3 sentence summary
  - Store summary in ChatSession.Summary
  - Use summary instead of full history for long sessions
- [ ] Add history pruning for old sessions

**Acceptance Criteria:**
- [ ] Recent messages retrieved correctly
- [ ] History formatted cleanly
- [ ] Long sessions get summarized
- [ ] Token budget respected

---

### Epic 10: Follow-Up Question Generation

#### Story 10.1: Data Sufficiency Checker
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to check if user input has sufficient data
**So that** the system asks for missing information

**Tasks:**
- [ ] Create `ISufficiencyChecker` interface:
  ```csharp
  Task<SufficiencyResult> CheckAsync(string message, ConversationIntent intent, UserProfile profile)
  ```
- [ ] Create `SufficiencyResult`:
  ```csharp
  - IsSufficient (bool)
  - MissingFields (List<string>)
  - RequiredConfidence (decimal)
  - ActualConfidence (decimal)
  ```
- [ ] Define required fields per intent:
  ```csharp
  TrackExpense: [amount OR title+platform] // can infer amount from title
  Recommendation: [none required, but better with genre/mood]
  Onboarding: [budget_limit]
  ActivityLog: [title, type OR platform]
  ```
- [ ] Implement checking logic:
  - Extract entities from message
  - Compare against required fields
  - Check profile for defaults (use existing preferences)
- [ ] Handle "good enough" scenarios (can proceed with partial data)

**Acceptance Criteria:**
- [ ] Required fields defined for all intents
- [ ] Checker correctly identifies missing data
- [ ] Profile defaults reduce required fields

---

#### Story 10.2: Follow-Up Question Generator
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to generate natural follow-up questions
**So that** users provide missing information conversationally

**Tasks:**
- [ ] Create `IFollowUpGenerator` interface:
  ```csharp
  Task<string> GenerateAsync(SufficiencyResult sufficiency, ConversationContext context)
  ```
- [ ] Create question templates per missing field:
  ```csharp
  ["amount"] → "How much did that cost?"
  ["title"] → "What was it called?"
  ["title", "type"] → "What did you watch/play? Was it a movie, show, or game?"
  ["genre"] → "Any particular genre you're in the mood for?"
  ["budget"] → "What's your entertainment budget for this month?"
  ```
- [ ] Implement smart question selection:
  - Prioritize most important missing field
  - Combine related questions (title + type)
  - Consider context (don't ask for budget if recently set)
- [ ] Add variety to questions (multiple phrasings)
- [ ] Use model for complex/ambiguous cases

**Acceptance Criteria:**
- [ ] Questions are natural and conversational
- [ ] Single question per response (not overwhelming)
- [ ] Context-aware (uses existing knowledge)

---

#### Story 10.3: Clarification Flow Handler
**Priority:** High | **Estimate:** 0.5 day

**As a** developer
**I want** to handle multi-turn clarification flows
**So that** incomplete information is gathered across messages

**Tasks:**
- [ ] Create `ClarificationState` tracking:
  ```csharp
  - OriginalIntent (ConversationIntent)
  - MissingFields (List<string>)
  - CollectedData (Dictionary<string, object>)
  - AttemptCount (int)
  ```
- [ ] Implement state persistence in session
- [ ] Handle follow-up responses:
  - Match response to asked question
  - Update collected data
  - Re-check sufficiency
  - Proceed if sufficient, ask next question if not
- [ ] Add max attempts limit (3) with graceful fallback
- [ ] Allow user to skip ("never mind", "skip")

**Acceptance Criteria:**
- [ ] Multi-turn clarification works
- [ ] Collected data persists across turns
- [ ] Max attempts prevents infinite loops
- [ ] Users can abort clarification flow

---

### Epic 11: Audit & Observability

#### Story 11.1: AI Decision Audit Trail
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to log all AI decisions
**So that** recommendations and extractions are auditable

**Tasks:**
- [ ] Create `AuditLog` entity:
  ```csharp
  - Id, UserId, SessionId
  - Timestamp (DateTime)
  - ActionType (enum: Extraction, Recommendation, IntentClassification)
  - Input (JSON) - message, context
  - Output (JSON) - model response, structured result
  - ValidationResult (JSON)
  - ProcessingTimeMs (int)
  - ModelUsed (string)
  - TokensUsed (int)
  ```
- [ ] Create `IAuditService` interface:
  ```csharp
  Task LogAsync(AuditEntry entry)
  Task<List<AuditEntry>> GetByUserAsync(Guid userId, AuditFilter filter)
  Task<List<AuditEntry>> GetBySessionAsync(Guid sessionId)
  ```
- [ ] Implement automatic logging in services
- [ ] Add admin endpoint to query audit logs
- [ ] Implement retention policy (90 days default)

**Acceptance Criteria:**
- [ ] All AI decisions are logged
- [ ] Logs are queryable by user/session
- [ ] Retention policy is enforced

---

#### Story 11.2: Metrics & Monitoring
**Priority:** Medium | **Estimate:** 1 day

**As a** developer
**I want** to collect operational metrics
**So that** system health is observable

**Tasks:**
- [ ] Define key metrics:
  - Model response time (p50, p95, p99)
  - Extraction accuracy (based on validation)
  - Intent classification confidence distribution
  - Token usage per request
  - Follow-up question rate
  - Error rate by type
- [ ] Implement metrics collection with Prometheus format
- [ ] Create health check endpoint with detailed status:
  - Database connectivity
  - Ollama availability
  - S3 connectivity
- [ ] Add endpoint for metrics: `GET /metrics`
- [ ] Create basic dashboard queries (Grafana-ready)

**Acceptance Criteria:**
- [ ] Metrics endpoint returns valid Prometheus format
- [ ] Health check shows all dependencies
- [ ] Key metrics are being collected

---

## Phase 4: Document Processing (Week 8)

### Epic 12: File Upload & S3 Storage

#### Story 12.1: S3 Storage Service
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to implement S3 file storage
**So that** user documents are stored securely

**Tasks:**
- [ ] Create `IFileStorageService` interface:
  ```csharp
  Task<UploadResult> UploadAsync(Stream file, string fileName, string contentType, Guid userId)
  Task<Stream> DownloadAsync(string key)
  Task<bool> DeleteAsync(string key)
  Task<string> GetPresignedUrlAsync(string key, TimeSpan expiry)
  ```
- [ ] Implement `S3StorageService`
- [ ] Implement `LocalFileStorageService` for development
- [ ] Configure based on environment (local vs AWS)
- [ ] Add file validation:
  - Max size: 10MB
  - Allowed types: PDF, PNG, JPG
- [ ] Implement secure key generation (user-scoped paths)
- [ ] Add virus scanning placeholder (for future)

**Acceptance Criteria:**
- [ ] Files upload to S3 successfully
- [ ] Files can be downloaded
- [ ] Presigned URLs work for temporary access
- [ ] Local fallback works for development

---

#### Story 12.2: Document Upload Endpoint
**Priority:** High | **Estimate:** 1 day

**As a** user
**I want** to upload financial documents
**So that** the assistant can analyze my finances

**Tasks:**
- [ ] Create `DocumentsController` with endpoints:
  - `POST /api/documents/upload` - multipart upload
  - `GET /api/documents` - list user's documents
  - `GET /api/documents/{id}` - get document details
  - `GET /api/documents/{id}/status` - get processing status
  - `DELETE /api/documents/{id}` - delete document
- [ ] Implement upload handling:
  - Validate file type and size
  - Generate secure S3 key
  - Create FinancialDocument record
  - Queue for processing
- [ ] Add upload progress tracking (for large files)
- [ ] Implement rate limiting (5 uploads/hour)

**Acceptance Criteria:**
- [ ] Documents upload successfully
- [ ] Invalid files are rejected with clear errors
- [ ] Document records track upload status
- [ ] Rate limiting prevents abuse

---

### Epic 13: PDF Processing

#### Story 13.1: PDF Text Extraction
**Priority:** High | **Estimate:** 1.5 days

**As a** developer
**I want** to extract text from PDF documents
**So that** content can be analyzed by the AI

**Tasks:**
- [ ] Research and select PDF library:
  - Option A: PdfPig (pure .NET, good text extraction)
  - Option B: iText7 (powerful, AGPL license)
  - Option C: Docnet (wrapper around PDFium)
- [ ] Create `IPdfExtractor` interface:
  ```csharp
  Task<PdfExtractionResult> ExtractTextAsync(Stream pdfStream)
  ```
- [ ] Implement `PdfExtractionResult`:
  ```csharp
  - FullText (string)
  - Pages (List<PageText>)
  - Tables (List<ExtractedTable>)
  - Metadata (Dictionary<string, string>)
  - IsPasswordProtected (bool)
  ```
- [ ] Handle common PDF issues:
  - Scanned documents (OCR placeholder)
  - Password-protected files
  - Corrupted files
  - Multi-column layouts
- [ ] Add table extraction for financial statements

**Acceptance Criteria:**
- [ ] Text extracted from standard PDFs
- [ ] Tables are identified and structured
- [ ] Errors handled gracefully
- [ ] Performance acceptable (<5s for typical document)

---

#### Story 13.2: Document Processing Service
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to process documents asynchronously
**So that** uploads don't block the user

**Tasks:**
- [ ] Create `IDocumentProcessingService` interface:
  ```csharp
  Task QueueForProcessingAsync(Guid documentId)
  Task ProcessDocumentAsync(Guid documentId)
  Task<ProcessingStatus> GetStatusAsync(Guid documentId)
  ```
- [ ] Implement background processing:
  - Download from S3
  - Extract text
  - Run AI analysis
  - Update document record
- [ ] Use .NET BackgroundService for queue processing
- [ ] Add retry logic for failed processing
- [ ] Implement status updates:
  - Pending → Processing → Completed/Failed
- [ ] Add SignalR notifications for completion

**Acceptance Criteria:**
- [ ] Documents process in background
- [ ] Status updates correctly
- [ ] Failed documents can be retried
- [ ] Users notified on completion

---

### Epic 14: Financial Document Analysis

#### Story 14.1: Financial Extraction Prompts
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** AI prompts for financial document analysis
**So that** key financial data is extracted accurately

**Tasks:**
- [ ] Create prompt templates for document types:
  - **Pay stub**: Extract gross income, net income, pay frequency, employer
  - **Bank statement**: Extract income deposits, recurring expenses, account balance
  - **Receipt**: Extract amount, vendor, category, date
- [ ] Define extraction schemas:
  ```csharp
  public class FinancialDocumentExtraction
  {
      public string DocumentType { get; set; }
      public decimal? MonthlyIncome { get; set; }
      public decimal? NetIncome { get; set; }
      public string? PayFrequency { get; set; }
      public List<RecurringExpense>? RecurringExpenses { get; set; }
      public decimal ConfidenceScore { get; set; }
      public List<string> Warnings { get; set; }
  }
  ```
- [ ] Implement chunking for long documents (>2000 tokens)
- [ ] Add extraction validation rules
- [ ] Create test cases with sample documents

**Acceptance Criteria:**
- [ ] Pay stubs extract income correctly
- [ ] Bank statements identify patterns
- [ ] Confidence scores reflect quality
- [ ] Warnings flag ambiguous data

---

#### Story 14.2: Financial Profile Update from Documents
**Priority:** High | **Estimate:** 1 day

**As a** user
**I want** my financial profile updated from documents
**So that** I don't have to enter information manually

**Tasks:**
- [ ] Create document → profile mapping logic
- [ ] Implement update flow:
  1. Extract data from document
  2. Present to user for confirmation (if confidence < 0.8)
  3. Update profile on confirmation
- [ ] Handle conflicting data:
  - Document says income is $5000, profile says $4500
  - Present both, let user choose
- [ ] Create confirmation endpoint:
  - `POST /api/documents/{id}/confirm`
  - `POST /api/documents/{id}/reject`
- [ ] Calculate suggested entertainment budget:
  - Default: 5-10% of discretionary income
  - Adjustable by user

**Acceptance Criteria:**
- [ ] High-confidence extractions auto-apply (with notification)
- [ ] Low-confidence extractions require confirmation
- [ ] Conflicts are presented clearly
- [ ] Budget suggestions are reasonable

---

## Phase 5: Chat & Real-Time Communication (Weeks 9-10)

### Epic 15: SignalR Infrastructure

#### Story 15.1: SignalR Hub Setup
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to set up SignalR for real-time communication
**So that** chat responses can stream to users

**Tasks:**
- [ ] Configure SignalR in `Program.cs`
- [ ] Create `ChatHub` class:
  ```csharp
  public class ChatHub : Hub
  {
      Task SendMessage(string message)
      Task JoinSession(Guid sessionId)
      Task LeaveSession(Guid sessionId)
  }
  ```
- [ ] Implement authentication in hub
- [ ] Create connection management:
  - Track active connections per user
  - Handle reconnection gracefully
  - Clean up on disconnect
- [ ] Add hub filters for logging/error handling
- [ ] Configure CORS for SignalR

**Acceptance Criteria:**
- [ ] Clients can connect to hub
- [ ] Authentication works in hub context
- [ ] Connections tracked correctly
- [ ] Reconnection works smoothly

---

#### Story 15.2: Streaming Response Implementation
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to stream AI responses to clients
**So that** users see responses as they generate

**Tasks:**
- [ ] Create client methods:
  ```csharp
  Clients.Caller.SendAsync("ReceiveChunk", chunk)
  Clients.Caller.SendAsync("MessageComplete", messageId)
  Clients.Caller.SendAsync("Error", error)
  Clients.Caller.SendAsync("TypingIndicator", isTyping)
  ```
- [ ] Implement streaming pipeline:
  ```csharp
  await foreach (var chunk in _ollama.StreamGenerateAsync(prompt))
  {
      await Clients.Caller.SendAsync("ReceiveChunk", chunk);
  }
  ```
- [ ] Add chunk buffering (send every 50ms, not every token)
- [ ] Handle streaming errors gracefully
- [ ] Implement cancellation support
- [ ] Add message reconstruction on server (for logging)

**Acceptance Criteria:**
- [ ] Responses stream to client in real-time
- [ ] Typing indicator shows during generation
- [ ] Errors are reported to client
- [ ] Full message is logged server-side

---

### Epic 16: Chat Orchestration

#### Story 16.1: Chat Orchestration Service
**Priority:** Critical | **Estimate:** 2 days

**As a** developer
**I want** a central orchestration service for chat
**So that** all chat logic flows through one pipeline

**Tasks:**
- [ ] Create `IChatOrchestrationService` interface:
  ```csharp
  IAsyncEnumerable<ChatChunk> ProcessMessageAsync(Guid userId, Guid sessionId, string message)
  ```
- [ ] Implement orchestration pipeline:
  ```
  1. Validate session ownership
  2. Classify intent
  3. Check data sufficiency
  4. If insufficient → generate follow-up, return
  5. Assemble context
  6. Build prompt
  7. Stream response from Ollama
  8. Extract data from response
  9. Validate extraction
  10. Update profile
  11. Log to audit trail
  12. Save messages to session
  ```
- [ ] Create `ChatChunk` for streaming:
  ```csharp
  - Type (enum: Content, Metadata, Complete, Error)
  - Content (string)
  - Metadata (Dictionary<string, object>)
  ```
- [ ] Add pipeline middleware support (for extensibility)
- [ ] Implement timeout handling (30s max)
- [ ] Add circuit breaker for Ollama failures

**Acceptance Criteria:**
- [ ] Full pipeline executes correctly
- [ ] Streaming works end-to-end
- [ ] Profile updates after messages
- [ ] Errors handled at each step

---

#### Story 16.2: Chat Hub Message Handler
**Priority:** High | **Estimate:** 1 day

**As a** user
**I want** to send messages and receive streaming responses
**So that** I can have conversations with the assistant

**Tasks:**
- [ ] Implement `SendMessage` in ChatHub:
  ```csharp
  public async Task SendMessage(string message)
  {
      var userId = GetUserId();
      var sessionId = GetCurrentSession();
      
      await Clients.Caller.SendAsync("TypingIndicator", true);
      
      await foreach (var chunk in _orchestration.ProcessMessageAsync(userId, sessionId, message))
      {
          if (chunk.Type == ChunkType.Content)
              await Clients.Caller.SendAsync("ReceiveChunk", chunk.Content);
      }
      
      await Clients.Caller.SendAsync("MessageComplete");
  }
  ```
- [ ] Add message validation (not empty, not too long)
- [ ] Implement rate limiting (10 messages/minute)
- [ ] Handle concurrent message prevention
- [ ] Add message queuing if needed

**Acceptance Criteria:**
- [ ] Messages process through full pipeline
- [ ] Responses stream to caller
- [ ] Rate limiting works
- [ ] Concurrent messages handled

---

### Epic 17: Conversation Session Management

#### Story 17.1: Session Service Implementation
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to manage chat sessions
**So that** conversations are organized and persistent

**Tasks:**
- [ ] Create `IChatSessionService` interface:
  ```csharp
  Task<ChatSessionDto> CreateSessionAsync(Guid userId)
  Task<ChatSessionDto> GetSessionAsync(Guid sessionId)
  Task<List<ChatSessionDto>> GetUserSessionsAsync(Guid userId, int limit = 20)
  Task<ChatSessionDto> UpdateSessionAsync(Guid sessionId, UpdateSessionRequest request)
  Task DeleteSessionAsync(Guid sessionId)
  Task<ChatSessionDto> GetOrCreateActiveSessionAsync(Guid userId)
  ```
- [ ] Implement session lifecycle:
  - Create: New session when user starts chat
  - Active: Session with recent activity
  - Inactive: No activity for 30+ minutes
  - Archived: Explicitly closed or old
- [ ] Add session metadata:
  - Title (auto-generated from first message)
  - Summary (AI-generated periodically)
  - Message count
  - Last activity timestamp
- [ ] Implement session cleanup job (archive old sessions)

**Acceptance Criteria:**
- [ ] Sessions created and retrieved correctly
- [ ] Session list shows recent sessions
- [ ] Inactive sessions auto-archive
- [ ] Session metadata updates correctly

---

#### Story 17.2: Message Persistence
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to persist chat messages
**So that** conversation history is available

**Tasks:**
- [ ] Create `IChatMessageService` interface:
  ```csharp
  Task<ChatMessageDto> SaveMessageAsync(Guid sessionId, SaveMessageRequest request)
  Task<List<ChatMessageDto>> GetSessionMessagesAsync(Guid sessionId, int skip = 0, int take = 50)
  Task<int> GetMessageCountAsync(Guid sessionId)
  ```
- [ ] Implement message storage:
  - Save user messages immediately
  - Save assistant messages after completion
  - Store extraction results with messages
- [ ] Add token counting for messages
- [ ] Implement message search within session
- [ ] Add pagination for long sessions

**Acceptance Criteria:**
- [ ] Messages persist correctly
- [ ] Message history retrievable
- [ ] Token counts tracked
- [ ] Pagination works for long sessions

---

#### Story 17.3: Chat Controller
**Priority:** High | **Estimate:** 0.5 day

**As a** user
**I want** REST endpoints for chat management
**So that** I can manage sessions without WebSocket

**Tasks:**
- [ ] Create `ChatController` with endpoints:
  - `GET /api/chat/sessions` - list user's sessions
  - `POST /api/chat/sessions` - create new session
  - `GET /api/chat/sessions/{id}` - get session details
  - `GET /api/chat/sessions/{id}/messages` - get messages (paginated)
  - `DELETE /api/chat/sessions/{id}` - delete session
  - `POST /api/chat/sessions/{id}/summarize` - generate summary
- [ ] Add session DTO with message preview
- [ ] Implement session search by title/content

**Acceptance Criteria:**
- [ ] All endpoints work correctly
- [ ] Pagination works for sessions and messages
- [ ] Only owner can access their sessions

---

## Phase 6: Recommendations Engine (Week 11)

### Epic 18: Entertainment Catalog

#### Story 18.1: Entertainment Service Implementation
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** to implement entertainment catalog service
**So that** items can be searched and filtered

**Tasks:**
- [ ] Create `IEntertainmentService` interface:
  ```csharp
  Task<PagedResult<EntertainmentItemDto>> SearchAsync(EntertainmentSearchRequest request)
  Task<EntertainmentItemDto> GetByIdAsync(Guid id)
  Task<List<EntertainmentItemDto>> GetAffordableAsync(decimal maxPrice, PreferencesFilter preferences)
  Task<List<EntertainmentItemDto>> GetByIdsAsync(List<Guid> ids)
  Task<List<string>> GetGenresAsync()
  Task<List<string>> GetPlatformsAsync()
  ```
- [ ] Implement search with filters:
  - By type (movie, series, game, etc.)
  - By genre (multiple)
  - By platform
  - By price range
  - By text search (title, description)
- [ ] Add sorting options:
  - Price (low to high, high to low)
  - Title (alphabetical)
  - Relevance (for text search)
- [ ] Implement efficient queries with EF Core
- [ ] Add caching for genre/platform lists

**Acceptance Criteria:**
- [ ] Search returns relevant results
- [ ] All filters work correctly
- [ ] Pagination works
- [ ] Performance acceptable (<200ms)

---

#### Story 18.2: Entertainment Controller
**Priority:** High | **Estimate:** 0.5 day

**As a** user
**I want** to browse and search entertainment items
**So that** I can find things I might enjoy

**Tasks:**
- [ ] Create `EntertainmentController` with endpoints:
  - `GET /api/entertainment` - search with filters
  - `GET /api/entertainment/{id}` - get item details
  - `GET /api/entertainment/genres` - list all genres
  - `GET /api/entertainment/platforms` - list all platforms
  - `GET /api/entertainment/types` - list all types
- [ ] Create comprehensive search DTO:
  ```csharp
  public class EntertainmentSearchRequest
  {
      public string? Query { get; set; }
      public List<string>? Types { get; set; }
      public List<string>? Genres { get; set; }
      public List<string>? Platforms { get; set; }
      public decimal? MinPrice { get; set; }
      public decimal? MaxPrice { get; set; }
      public string? SortBy { get; set; }
      public int Page { get; set; } = 1;
      public int PageSize { get; set; } = 20;
  }
  ```

**Acceptance Criteria:**
- [ ] All endpoints return correct data
- [ ] Search filters work as expected
- [ ] Response includes pagination metadata

---

### Epic 19: Recommendation Engine

#### Story 19.1: Recommendation Service Implementation
**Priority:** High | **Estimate:** 1.5 days

**As a** developer
**I want** to implement AI-powered recommendations
**So that** users get personalized suggestions

**Tasks:**
- [ ] Create `IRecommendationService` interface:
  ```csharp
  Task<List<RecommendationDto>> GenerateAsync(Guid userId, RecommendationRequest request)
  Task<RecommendationDto> GetByIdAsync(Guid id)
  Task<List<RecommendationDto>> GetUserHistoryAsync(Guid userId, int limit = 20)
  Task RecordFeedbackAsync(Guid recommendationId, UserFeedback feedback)
  ```
- [ ] Implement recommendation flow:
  1. Load user profile (budget, preferences)
  2. Query affordable items matching preferences
  3. Filter out recently recommended/watched
  4. Build recommendation prompt with candidates
  5. Get AI selection and explanation
  6. Validate selections exist in catalog
  7. Store recommendations
  8. Return with explanations
- [ ] Create recommendation prompt:
  ```
  User budget: $X remaining
  User likes: [genres]
  User dislikes: [genres]
  Recent activity: [titles]
  
  Available options:
  1. Title A - $X - genres - description
  2. Title B - $Y - genres - description
  ...
  
  Select 3-5 best matches and explain why each fits this user.
  ```
- [ ] Add diversity in recommendations (not all same genre)
- [ ] Implement fallback for no matches

**Acceptance Criteria:**
- [ ] Recommendations match user preferences
- [ ] All recommendations fit user budget
- [ ] Explanations are personalized
- [ ] Recently watched items excluded

---

#### Story 19.2: Recommendation Validation
**Priority:** High | **Estimate:** 0.5 day

**As a** developer
**I want** to validate AI recommendations
**So that** only real, affordable items are suggested

**Tasks:**
- [ ] Create `IRecommendationValidator` interface:
  ```csharp
  Task<ValidationResult> ValidateAsync(List<AiRecommendation> recommendations, ValidationContext context)
  ```
- [ ] Implement validation checks:
  - Item exists in catalog
  - Item is active
  - Price fits remaining budget
  - Item not in recent history
  - Genre matches (sanity check)
- [ ] Handle validation failures:
  - Remove invalid items
  - Log for analysis
  - Regenerate if too many removed
- [ ] Add metrics for validation pass rate

**Acceptance Criteria:**
- [ ] Invalid items are filtered out
- [ ] Budget violations caught
- [ ] Validation failures logged

---

#### Story 19.3: Recommendation Feedback
**Priority:** Medium | **Estimate:** 0.5 day

**As a** user
**I want** to give feedback on recommendations
**So that** future suggestions improve

**Tasks:**
- [ ] Create feedback endpoint:
  - `POST /api/recommendations/{id}/feedback`
- [ ] Define feedback types:
  ```csharp
  public enum UserFeedback
  {
      Liked,      // User liked recommendation
      Disliked,   // User didn't like it
      Saved,      // Added to watchlist
      Watched,    // User watched/played/read it
      Dismissed   // Not interested
  }
  ```
- [ ] Update recommendation record with feedback
- [ ] Use feedback to improve future recommendations:
  - Liked → reinforce genre preference
  - Disliked → reduce genre weight
  - Dismissed → avoid similar items temporarily
- [ ] Create feedback summary endpoint

**Acceptance Criteria:**
- [ ] Feedback recorded correctly
- [ ] Feedback influences preference signals
- [ ] Feedback history viewable

---

## Phase 7: Testing & Quality Assurance (Week 12)

### Epic 20: Unit Testing

#### Story 20.1: Service Layer Unit Tests
**Priority:** High | **Estimate:** 2 days

**As a** developer
**I want** comprehensive unit tests for services
**So that** business logic is verified

**Tasks:**
- [ ] Test `ExtractionService`:
  - Extract spending amounts
  - Extract preferences
  - Handle malformed input
  - Handle model errors
- [ ] Test `IntentClassifier`:
  - Classify each intent type
  - Handle ambiguous messages
  - Verify confidence scores
- [ ] Test `ContextAssembler`:
  - Token budget respected
  - Intent-specific loading
  - Truncation works correctly
- [ ] Test `ProfileUpdateService`:
  - Budget updates
  - Preference merging
  - Conflict handling
- [ ] Test `RecommendationService`:
  - Filter by budget
  - Match preferences
  - Exclude recent items
- [ ] Test `FinancialProfileService`:
  - Budget calculations
  - Period reset logic

**Acceptance Criteria:**
- [ ] 80%+ code coverage for services
- [ ] All edge cases tested
- [ ] Mocks used for external dependencies

---

#### Story 20.2: Domain Logic Unit Tests
**Priority:** High | **Estimate:** 1 day

**As a** developer
**I want** unit tests for domain logic
**So that** core business rules are verified

**Tasks:**
- [ ] Test entity validation:
  - Budget limits (positive numbers)
  - Preference lists (valid genres)
  - Activity records (required fields)
- [ ] Test calculations:
  - Remaining budget
  - Budget period logic
  - Token estimation
- [ ] Test serialization:
  - Compact profile format
  - JSON columns
- [ ] Test enums and value objects

**Acceptance Criteria:**
- [ ] All domain logic tested
- [ ] Validation rules verified
- [ ] Calculations accurate

---

### Epic 21: Integration Testing

#### Story 21.1: API Integration Tests
**Priority:** High | **Estimate:** 1.5 days

**As a** developer
**I want** integration tests for API endpoints
**So that** end-to-end flows are verified

**Tasks:**
- [ ] Set up `WebApplicationFactory` for testing
- [ ] Set up Testcontainers for PostgreSQL
- [ ] Create mock for Ollama (predictable responses)
- [ ] Test authentication flow:
  - Register → Login → Access protected endpoint
- [ ] Test profile flow:
  - Create profile → Update preferences → Get summary
- [ ] Test chat flow:
  - Create session → Send message → Get history
- [ ] Test recommendation flow:
  - Set preferences → Get recommendations → Give feedback
- [ ] Test document flow:
  - Upload → Process → Confirm extraction

**Acceptance Criteria:**
- [ ] All major flows tested
- [ ] Tests run against real database
- [ ] Tests are isolated (clean DB per test)

---

#### Story 21.2: SignalR Integration Tests
**Priority:** Medium | **Estimate:** 0.5 day

**As a** developer
**I want** integration tests for SignalR
**So that** real-time communication is verified

**Tasks:**
- [ ] Set up SignalR test client
- [ ] Test connection establishment
- [ ] Test message streaming
- [ ] Test error handling
- [ ] Test reconnection behavior

**Acceptance Criteria:**
- [ ] SignalR connection works in tests
- [ ] Streaming verified
- [ ] Error scenarios handled

---

### Epic 22: Error Handling & Quality

#### Story 22.1: Global Error Handling
**Priority:** High | **Estimate:** 0.5 day

**As a** developer
**I want** consistent error handling
**So that** errors are reported uniformly

**Tasks:**
- [ ] Create `ErrorResponse` model:
  ```csharp
  public class ErrorResponse
  {
      public string Type { get; set; }
      public string Message { get; set; }
      public string? Detail { get; set; }
      public string? TraceId { get; set; }
      public Dictionary<string, string[]>? Errors { get; set; }
  }
  ```
- [ ] Implement `ExceptionHandlingMiddleware`
- [ ] Map exceptions to HTTP status codes:
  - `NotFoundException` → 404
  - `ValidationException` → 400
  - `UnauthorizedException` → 401
  - `ForbiddenException` → 403
  - Generic → 500
- [ ] Add problem details format (RFC 7807)
- [ ] Log exceptions with context

**Acceptance Criteria:**
- [ ] All errors return consistent format
- [ ] Status codes are appropriate
- [ ] Errors logged with trace ID

---

#### Story 22.2: Input Validation
**Priority:** High | **Estimate:** 0.5 day

**As a** developer
**I want** comprehensive input validation
**So that** invalid data is rejected early

**Tasks:**
- [ ] Create FluentValidation validators for all DTOs
- [ ] Configure automatic validation in pipeline
- [ ] Return validation errors in standard format
- [ ] Add common validation rules:
  - Email format
  - Positive numbers for amounts
  - String length limits
  - Required fields
- [ ] Test validation edge cases

**Acceptance Criteria:**
- [ ] All inputs validated
- [ ] Validation errors are clear
- [ ] Invalid requests return 400

---

## Phase 8: AWS Deployment & Production (Week 13)

### Epic 23: AWS Infrastructure

#### Story 23.1: RDS PostgreSQL Setup
**Priority:** High | **Estimate:** 0.5 day

**As a** developer
**I want** to set up RDS PostgreSQL
**So that** production has a managed database

**Tasks:**
- [ ] Create RDS PostgreSQL instance:
  - Instance class: db.t3.micro (for start)
  - Storage: 20GB gp3
  - Multi-AZ: No (for cost, enable later)
  - Backup retention: 7 days
- [ ] Configure security group (VPC only access)
- [ ] Set up parameter group for performance
- [ ] Create application database and user
- [ ] Test connection from local with SSH tunnel
- [ ] Document connection strings

**Acceptance Criteria:**
- [ ] RDS instance running
- [ ] Application can connect
- [ ] Backups configured

---

#### Story 23.2: ECS Fargate Deployment
**Priority:** High | **Estimate:** 1.5 days

**As a** developer
**I want** to deploy to ECS Fargate
**So that** the application runs in a managed container environment

**Tasks:**
- [ ] Create ECR repository for Docker images
- [ ] Create ECS cluster
- [ ] Create task definition:
  - CPU: 256 (0.25 vCPU)
  - Memory: 512 MB
  - Environment variables from Secrets Manager
- [ ] Create ECS service:
  - Desired count: 1
  - Health check configuration
- [ ] Create Application Load Balancer:
  - HTTPS listener with ACM certificate
  - Target group with health check
- [ ] Configure auto-scaling (optional for now)
- [ ] Test deployment

**Acceptance Criteria:**
- [ ] Container runs in ECS
- [ ] ALB routes traffic correctly
- [ ] Health checks pass
- [ ] HTTPS works

**Technical Notes:**
- For Ollama in production, consider:
  - Running on EC2 with GPU
  - Using a cloud AI API as fallback
  - Self-hosting on separate container (CPU inference)

---

#### Story 23.3: Secrets & Configuration
**Priority:** High | **Estimate:** 0.5 day

**As a** developer
**I want** to manage secrets securely
**So that** credentials are not exposed

**Tasks:**
- [ ] Create AWS Secrets Manager secrets:
  - Database connection string
  - Cognito credentials
  - S3 access keys (or use IAM roles)
- [ ] Configure ECS task role for Secrets Manager access
- [ ] Update application to read from Secrets Manager
- [ ] Create environment-specific configurations
- [ ] Document secret rotation process

**Acceptance Criteria:**
- [ ] Secrets stored in Secrets Manager
- [ ] Application reads secrets correctly
- [ ] No secrets in source control

---

### Epic 24: CI/CD Pipeline

#### Story 24.1: GitHub Actions Pipeline
**Priority:** Medium | **Estimate:** 1 day

**As a** developer
**I want** automated CI/CD pipeline
**So that** deployments are consistent and reliable

**Tasks:**
- [ ] Create `.github/workflows/ci.yml`:
  - Trigger: Push to main, PR
  - Steps: Checkout, Setup .NET, Restore, Build, Test
- [ ] Create `.github/workflows/deploy.yml`:
  - Trigger: Push to main (after CI passes)
  - Steps: Build Docker, Push to ECR, Update ECS
- [ ] Configure GitHub secrets:
  - AWS credentials
  - ECR repository URL
- [ ] Add test reporting
- [ ] Add deployment notifications

**Acceptance Criteria:**
- [ ] CI runs on every PR
- [ ] Deployment runs on merge to main
- [ ] Failed tests block deployment

---

### Epic 25: Monitoring & Observability

#### Story 25.1: CloudWatch Setup
**Priority:** Medium | **Estimate:** 0.5 day

**As a** developer
**I want** centralized logging and monitoring
**So that** production issues are detectable

**Tasks:**
- [ ] Configure Serilog CloudWatch sink
- [ ] Create log groups with retention policy
- [ ] Set up CloudWatch alarms:
  - Error rate > 1%
  - Response time p95 > 2s
  - CPU utilization > 80%
  - Memory utilization > 80%
- [ ] Create CloudWatch dashboard:
  - Request count
  - Error count
  - Latency metrics
  - Resource utilization
- [ ] Set up SNS notifications for alarms

**Acceptance Criteria:**
- [ ] Logs visible in CloudWatch
- [ ] Alarms configured and working
- [ ] Dashboard shows key metrics

---

#### Story 25.2: Production Runbook
**Priority:** Medium | **Estimate:** 0.5 day

**As a** developer
**I want** documented operational procedures
**So that** issues can be resolved quickly

**Tasks:**
- [ ] Document common issues and solutions:
  - High latency
  - Database connection issues
  - Ollama failures
  - Memory issues
- [ ] Create deployment rollback procedure
- [ ] Document scaling procedures
- [ ] Create incident response checklist
- [ ] Document backup/restore procedures

**Acceptance Criteria:**
- [ ] Runbook covers common scenarios
- [ ] Procedures are clear and tested
- [ ] Team can follow runbook independently

---

## Summary

### Total Stories: 52
### Estimated Duration: 13 Weeks

| Phase | Weeks | Stories | Focus |
|-------|-------|---------|-------|
| 1. Foundation | 1-2 | 9 | Project setup, entities, Docker |
| 2. Authentication | 3-4 | 8 | AWS Cognito, profiles |
| 3. AI Core | 5-7 | 13 | Ollama, extraction, context |
| 4. Documents | 8 | 5 | S3, PDF, financial analysis |
| 5. Chat | 9-10 | 7 | SignalR, orchestration |
| 6. Recommendations | 11 | 5 | Catalog, engine |
| 7. Testing | 12 | 5 | Unit, integration, quality |
| 8. Deployment | 13 | 5 | AWS, CI/CD, monitoring |

### Critical Path
1. Epic 1 (Setup) → Epic 2 (Entities) → Epic 6 (Ollama) → Epic 8 (Extraction)
2. Epic 8 (Extraction) → Epic 16 (Orchestration) → Epic 19 (Recommendations)

### Key Risks
1. **Model Performance**: Local model may not perform well enough
   - Mitigation: Early spike to test models
2. **Extraction Accuracy**: AI may hallucinate data
   - Mitigation: Validation layer, confidence scoring
3. **AWS Costs**: Resources may exceed budget
   - Mitigation: Start small, monitor usage

### Definition of Done
- [ ] Code complete and builds
- [ ] Unit tests passing
- [ ] Integration tests passing (where applicable)
- [ ] Code reviewed
- [ ] Documentation updated
- [ ] Deployed to environment
