# Backend Development Prompt: Budget-Aware Entertainment Assistant

## Role
You are a senior .NET backend developer. Generate production-ready code following enterprise best practices.

## Project Overview
Build a REST API backend for a "Budget-Aware Entertainment Assistant" — an AI-powered app that:
1. Understands users' financial situation from uploaded documents
2. Tracks their entertainment preferences
3. Recommends entertainment options they can afford with reasons like a personal assistant using financial data and preferences.

## Tech Stack (Required)
- **Framework:** ASP.NET Core 8 Web API
- **Language:** C# 12
- **ORM:** Entity Framework Core 8
- **Database:** PostgreSQL (AWS RDS compatible)
- **Authentication:** JWT with AWS Cognito integration
- **File Storage:** AWS S3 for document uploads
- **Real-time:** SignalR for streaming chat responses
- **AI Integration:** Ollama REST API (local LLM)
- **Containerization:** Docker + Docker Compose

I provide you lots of details below including database schema, API endpoints, key services, and configuration but you don't need to implement at once. I would prefer you to read all and understand and come back with questions if any before starting implementation. I want you to break down this project into one-person manageable tasks in bullets point in a text file in this directory under the format of user stories (like in agile method)

## Architecture Requirements
Follow Clean Architecture with these layers:
```
src/
├── BudgetEntertainment.Api/           # Presentation layer
│   ├── Controllers/
│   ├── Hubs/                          # SignalR
│   ├── Middleware/
│   └── Program.cs
├── BudgetEntertainment.Application/   # Business logic
│   ├── Services/
│   ├── DTOs/
│   ├── Interfaces/
│   └── Validators/
├── BudgetEntertainment.Domain/        # Entities
│   ├── Entities/
│   ├── Enums/
│   └── ValueObjects/
├── BudgetEntertainment.Infrastructure/ # External services
│   ├── Data/
│   ├── Repositories/
│   ├── Services/
│   │   ├── OllamaService.cs
│   │   ├── S3StorageService.cs
│   │   ├── DocumentParsingService.cs
│   │   └── CognitoAuthService.cs
│   └── Configuration/
└── BudgetEntertainment.Tests/
```

## Database Schema

### Users Table
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    cognito_sub VARCHAR(255) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    display_name VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Financial Profiles Table
```sql
CREATE TABLE financial_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    monthly_income DECIMAL(12,2),
    fixed_expenses DECIMAL(12,2),
    savings_goal_percent DECIMAL(5,2),
    entertainment_budget DECIMAL(12,2),
    currency VARCHAR(3) DEFAULT 'USD',
    last_analyzed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Expense Categories Table
```sql
CREATE TABLE expense_categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    category VARCHAR(50) NOT NULL,
    monthly_amount DECIMAL(12,2),
    is_fixed BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Financial Documents Table
```sql
CREATE TABLE financial_documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    document_type VARCHAR(50) NOT NULL, -- 'bank_statement', 'pay_stub', 'receipt'
    file_name VARCHAR(255) NOT NULL,
    s3_key VARCHAR(500) NOT NULL,
    file_size_bytes BIGINT,
    mime_type VARCHAR(100),
    processing_status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'processing', 'completed', 'failed'
    extracted_data JSONB,
    uploaded_at TIMESTAMP DEFAULT NOW(),
    processed_at TIMESTAMP
);
```

### Entertainment Preferences Table
```sql
CREATE TABLE entertainment_preferences (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    preferred_genres TEXT[], -- Array of genres
    disliked_genres TEXT[],
    streaming_subscriptions TEXT[], -- Netflix, Spotify, etc.
    preferred_activities TEXT[], -- movies, games, concerts, dining
    price_sensitivity VARCHAR(20), -- 'budget', 'moderate', 'flexible'
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Entertainment Items Table
```sql
CREATE TABLE entertainment_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    type VARCHAR(50) NOT NULL, -- 'movie', 'book', 'game', 'event', 'restaurant'
    description TEXT,
    genre VARCHAR(100),
    price DECIMAL(10,2),
    price_type VARCHAR(20), -- 'free', 'one_time', 'subscription', 'variable'
    platform VARCHAR(100), -- Where to access (Netflix, Steam, etc.)
    external_url VARCHAR(500),
    image_url VARCHAR(500),
    rating DECIMAL(3,2),
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Chat Sessions Table
```sql
CREATE TABLE chat_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255),
    context_summary TEXT, -- AI-generated summary for long conversations
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Chat Messages Table
```sql
CREATE TABLE chat_messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id UUID REFERENCES chat_sessions(id) ON DELETE CASCADE,
    role VARCHAR(20) NOT NULL, -- 'user', 'assistant', 'system'
    content TEXT NOT NULL,
    extracted_preferences JSONB, -- Any preferences extracted from this message
    extracted_financial_info JSONB, -- Any financial info mentioned
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Recommendations History Table
```sql
CREATE TABLE recommendations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    session_id UUID REFERENCES chat_sessions(id),
    entertainment_id UUID REFERENCES entertainment_items(id),
    reason TEXT,
    estimated_cost DECIMAL(10,2),
    user_response VARCHAR(20), -- 'liked', 'disliked', 'saved', 'purchased', null
    created_at TIMESTAMP DEFAULT NOW()
);
```

## API Endpoints

### Authentication
```
POST   /api/auth/register          - Register via Cognito
POST   /api/auth/login             - Login, return JWT
POST   /api/auth/refresh           - Refresh token
POST   /api/auth/logout            - Logout
GET    /api/auth/me                - Get current user
```

### Financial Profile
```
GET    /api/financial/profile                    - Get user's financial profile
PUT    /api/financial/profile                    - Update financial profile
POST   /api/financial/documents                  - Upload financial document
GET    /api/financial/documents                  - List uploaded documents
DELETE /api/financial/documents/{id}             - Delete document
GET    /api/financial/documents/{id}/analysis    - Get AI analysis of document
POST   /api/financial/analyze                    - Trigger full financial analysis
GET    /api/financial/budget/entertainment       - Get calculated entertainment budget
GET    /api/financial/spending/summary           - Get spending summary
```

### Entertainment Preferences
```
GET    /api/preferences                          - Get entertainment preferences
PUT    /api/preferences                          - Update preferences
GET    /api/preferences/subscriptions            - Get active subscriptions
POST   /api/preferences/subscriptions            - Add subscription
DELETE /api/preferences/subscriptions/{id}       - Remove subscription
```

### Chat & Recommendations
```
POST   /api/chat/sessions                        - Create new chat session
GET    /api/chat/sessions                        - List user's sessions
GET    /api/chat/sessions/{id}                   - Get session with messages
DELETE /api/chat/sessions/{id}                   - Delete session
POST   /api/chat/sessions/{id}/messages          - Send message (streams via SignalR)
```

### Entertainment Catalog
```
GET    /api/entertainment                        - List items (paginated, filtered)
GET    /api/entertainment/{id}                   - Get item details
GET    /api/entertainment/search                 - Search items
GET    /api/entertainment/recommendations        - Get AI recommendations
POST   /api/entertainment/{id}/save              - Save to wishlist
POST   /api/entertainment/{id}/rate              - Rate an item
```

### Health & Admin
```
GET    /api/health                               - Health check
GET    /api/health/ready                         - Readiness check (DB, Ollama, S3)
```

## Key Services to Implement

### 1. OllamaService
```csharp
public interface IOllamaService
{
    Task<string> GenerateAsync(string prompt, CancellationToken ct = default);
    IAsyncEnumerable<string> StreamGenerateAsync(string prompt, CancellationToken ct = default);
    Task<float[]> GetEmbeddingAsync(string text, CancellationToken ct = default);
    Task<T> GenerateStructuredAsync<T>(string prompt, CancellationToken ct = default);
}
```

### 2. DocumentParsingService
```csharp
public interface IDocumentParsingService
{
    Task<FinancialDocumentAnalysis> AnalyzeBankStatementAsync(Stream document, string fileName);
    Task<FinancialDocumentAnalysis> AnalyzePayStubAsync(Stream document, string fileName);
    Task<FinancialDocumentAnalysis> AnalyzeReceiptAsync(Stream document, string fileName);
    Task<ExtractedFinancialData> ExtractFinancialDataAsync(string documentText);
}
```

### 3. FinancialAnalysisService
```csharp
public interface IFinancialAnalysisService
{
    Task<FinancialSummary> AnalyzeUserFinancesAsync(Guid userId);
    Task<decimal> CalculateEntertainmentBudgetAsync(Guid userId);
    Task<SpendingInsights> GetSpendingInsightsAsync(Guid userId);
    Task<List<BudgetAlert>> GetBudgetAlertsAsync(Guid userId);
}
```

### 4. RecommendationService
```csharp
public interface IRecommendationService
{
    Task<List<EntertainmentRecommendation>> GetPersonalizedRecommendationsAsync(
        Guid userId, 
        decimal? maxPrice = null,
        string? mood = null,
        int limit = 10);
    Task<string> ExplainRecommendationAsync(Guid userId, Guid entertainmentId);
}
```

### 5. ChatOrchestrationService
```csharp
public interface IChatOrchestrationService
{
    IAsyncEnumerable<string> ProcessMessageAsync(
        Guid sessionId, 
        Guid userId, 
        string userMessage,
        CancellationToken ct = default);
    Task<ConversationContext> BuildContextAsync(Guid sessionId, Guid userId);
    Task<ExtractedData> ExtractDataFromConversationAsync(string userMessage, string assistantResponse);
}
```

## System Prompt for Chat Assistant
```csharp
public static class SystemPrompts
{
    public static string GetMainAssistantPrompt(UserContext context) => $"""
        You are a friendly Budget-Aware Entertainment Assistant. Your role is to help users 
        discover entertainment options (movies, books, games, events, dining) that match 
        both their interests AND their budget.

        ## USER'S FINANCIAL CONTEXT
        - Monthly Entertainment Budget: {context.EntertainmentBudget:C}
        - Budget Used This Month: {context.BudgetUsedThisMonth:C}
        - Remaining Budget: {context.RemainingBudget:C}
        - Active Subscriptions: {string.Join(", ", context.Subscriptions)}
        - Price Sensitivity: {context.PriceSensitivity}

        ## USER'S ENTERTAINMENT PREFERENCES
        - Favorite Genres: {string.Join(", ", context.PreferredGenres)}
        - Dislikes: {string.Join(", ", context.DislikedGenres)}
        - Preferred Activities: {string.Join(", ", context.PreferredActivities)}
        - Recent Likes: {string.Join(", ", context.RecentLikes)}

        ## YOUR BEHAVIOR GUIDELINES
        1. Always consider the user's budget when making recommendations
        2. Mention prices and explain why something fits their budget
        3. Suggest free alternatives when budget is tight
        4. Be conversational and friendly, not salesy
        5. If you learn new preferences, acknowledge them naturally
        6. Ask clarifying questions when needed, but don't interrogate
        7. Celebrate when users find something they love
        8. Be honest if something is outside their budget

        ## WHEN RECOMMENDING
        - Always include approximate cost
        - Explain WHY this matches their taste
        - Mention if it's available on their existing subscriptions (free for them!)
        - Offer alternatives at different price points

        ## TOOLS AVAILABLE
        You can search the entertainment database and get user's financial summary.
        When you need to search, output:
        {{"action": "search", "query": "...", "type": "movie|book|game|event", "max_price": number}}
        
        When you need financial context, output:
        {{"action": "get_budget_status"}}

        ## CONVERSATION STYLE
        - Warm and encouraging
        - Use emojis sparingly but naturally
        - Keep responses concise unless user wants details
        - Remember context from earlier in the conversation
        """;
}
```

## Configuration (appsettings.json structure)
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Database=budget_entertainment;Username=postgres;Password=xxx"
  },
  "AWS": {
    "Region": "ap-southeast-2",
    "S3": {
      "BucketName": "budget-entertainment-documents",
      "PresignedUrlExpiration": 3600
    },
    "Cognito": {
      "UserPoolId": "ap-southeast-2_xxxxx",
      "ClientId": "xxxxxxxxx",
      "Authority": "https://cognito-idp.ap-southeast-2.amazonaws.com/ap-southeast-2_xxxxx"
    }
  },
  "Ollama": {
    "BaseUrl": "http://localhost:11434",
    "Model": "llama3.2:3b",
    "EmbeddingModel": "nomic-embed-text",
    "TimeoutSeconds": 120
  },
  "Features": {
    "EnableDocumentParsing": true,
    "EnableStreamingChat": true,
    "MaxDocumentSizeMb": 10
  }
}
```

## Docker Compose for Local Development
```yaml
version: '3.8'
services:
  api:
    build: 
      context: .
      dockerfile: src/BudgetEntertainment.Api/Dockerfile
    ports:
      - "5000:5000"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Host=db;Database=budget_entertainment;Username=postgres;Password=postgres
      - Ollama__BaseUrl=http://ollama:11434
    depends_on:
      - db
      - ollama

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: budget_entertainment
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  ollama:
    image: ollama/ollama:latest
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    deploy:
      resources:
        limits:
          memory: 4G

volumes:
  pgdata:
  ollama_data:
```

## Deliverables Expected

Please generate:

1. **Complete Entity classes** with EF Core configurations
2. **Repository interfaces and implementations** with common patterns
3. **Service layer** with all business logic
4. **API Controllers** with proper routing, validation, and error handling
5. **SignalR Hub** for streaming chat
6. **DTOs** for all API requests/responses
7. **Middleware** for error handling and logging
8. **Extension methods** for dependency injection setup
9. **Docker configurations**
10. **Unit test examples** for key services

## Code Quality Requirements

- Use nullable reference types
- Use records for DTOs
- Use FluentValidation for input validation
- Use MediatR for CQRS (optional but preferred)
- Include XML documentation on public methods
- Follow Microsoft naming conventions
- Use async/await properly throughout
- Include proper exception handling
- Use ILogger for logging

---- LLM Answers Below This Line ----

I've read through the entire requirements document. This is a comprehensive Budget-Aware Entertainment Assistant API with Clean Architecture, PostgreSQL, AWS integration (Cognito + S3), Ollama for AI, and SignalR for real-time chat.

Before I create the task breakdown, I have a few clarifying questions:

**Questions:**

1. **Document Parsing Format**: The requirements mention parsing bank statements, pay stubs, and receipts. What formats should be supported? (PDF, images/OCR, CSV exports from banks?)
Answer: initiallly i only want to support PDF files

2. **Entertainment Data Source**: How should the entertainment_items table be populated? Options:
   - Manual seeding with sample data
   - Admin endpoints to manage items  
   - External API integration (TMDB for movies, IGDB for games, etc.)
Answer: the data in this table should include seeding data such as boardgames, card games, movies, books
There should be some api endpoint to scrape data from amazon for books, boardgames for pricing, and product-related reviews can be found on Youtube or google search in top 5 resulted pages. but for testing the main features - conversational chat, we should create static data for these external requests 

3. **AWS Setup**: Is there an existing AWS environment (Cognito user pool, S3 bucket), or should I include infrastructure setup guidance?
Answer: there is no existing environment. everything starts from scratch. Guidance is very appreciated

4. **MVP Priority**: Should any features be prioritized for an initial release? (e.g., basic auth + chat before document parsing)
Answer: Yes. They are document parsing using AI supported functions together with exisitng libraries to parse pdf files. there should be common libraries. ask before using and downloading any libraries

5. **Testing Scope**: Unit tests only, or also integration tests with test containers?
I've read through the entire requirements document. This is a comprehensive Budget-Aware Entertainment Assistant API with Clean Architecture, PostgreSQL, AWS integration (Cognito + S3), Ollama for AI, and SignalR for real-time chat.
Answer: yes, unit tests only.