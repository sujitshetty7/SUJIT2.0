# Sujit 2.0 - Personal Operating System Architecture Document

This document outlines the architectural and technical considerations for Sujit 2.0 based on the provided specifications. It addresses the 10 critical developmental requirements before major implementation.

## 1. Specification Analysis

Sujit 2.0 is designed not just as a productivity app, but as a "Personal Operating System" (Second Brain, Chief of Staff, Mentor, Strategist, etc.). The optimization function is "Maximum Wealth + Business Growth" balanced across six life domains (Health, Business, Wealth, Personal Growth, Spirituality, Relationships).

Key functional highlights:

- Top-2 dynamic prioritization engine.
- Multi-tier goal hierarchy (Vision to Next Action).
- AI interaction supporting three personas (Strategic Mentor, Honest Coach, Chief of Staff).
- Decision-making engine (Priority Scoring).
- Dynamic, hour-by-hour scheduling and task management.
- Anti-procrastination, Daily/Weekly/Monthly reviews, and Bottleneck analysis.
- Separate engines for Business (CRM-lite) and Wealth (Net Worth/Cash Flow).
- Gamification (XP, Levels, Life Score).
- Multimodal AI support (Voice, Natural Language Control).
- Integration ecosystem (Google Sheets, Calendar, ChatGPT, Gemini).

## 2. Technical Limitations

- **Background Processing in iOS:** iOS limits background execution time. The app cannot constantly monitor and run complex AI background tasks unless initiated by a user action, push notification, or background fetch.
- **On-Device vs. Cloud AI:** While running lightweight classification models on-device (via CoreML) is possible, complex reasoning, large-scale memory retrieval (RAG), and natural language processing will require cloud infrastructure, leading to latency and requiring an active internet connection.
- **Voice Recognition Accuracy:** While Apple's Speech framework is good, recognizing complex business context perfectly in real-time ("Rahul... proposal... ₹80,000") will require robust NLP post-processing (e.g., passing transcripts to an LLM for structured extraction).
- **Latency of LLM Chains:** Chaining multiple AI tasks (intent detection -> context retrieval -> generation) can result in slow response times, disrupting the snappy UX required for a "What should I do now?" request.

## 3. Impossible/Unsupported Integrations

- **Direct Private ChatGPT / Gemini Consumer History Access:** There is no official API to access a user's private conversational history from the ChatGPT or Gemini consumer apps. The solution is an import/sync system where the user exports their data, or interacting exclusively through the Sujit 2.0 app which uses official API keys (OpenAI API / Gemini API).
- **WhatsApp Background Scraping:** WhatsApp does not provide an official API for reading personal account messages in the background on iOS. Integrations must use official WhatsApp Business API (if applicable to the user's business) or rely on iOS Share Sheet/manual input.
- **Fully Automated Cross-App Actions on iOS:** The app cannot seamlessly perform background actions in other apps (e.g., silently sending emails via Gmail app, reading arbitrary non-HealthKit health data) without user interaction or utilizing cloud-to-cloud server-side integrations (OAuth -> Gmail API).

## 4. Security Risks

- **Highly Sensitive Data Centralization:** Sujit 2.0 acts as a single point of failure for all personal, financial, and business data. A database breach would be catastrophic.
- **LLM Data Privacy:** Sending highly sensitive personal and financial data to third-party LLM providers (OpenAI, Google) poses privacy risks. Care must be taken to anonymize data or use Enterprise/API tiers where data is not used for training.
- **Token Management:** Handling OAuth tokens for Google (Drive, Calendar, Gmail) requires secure storage. If the backend is compromised, attackers gain access to the user's entire digital life.
- **Prompt Injection:** If external data (e.g., reading an email or a website) is fed into the LLM, a malicious payload could manipulate the AI's recommendations.

## 5. Missing Requirements

- **Sync/Offline Capabilities:** The specification doesn't detail how the app should function without an internet connection. (e.g., Can I view my schedule or capture tasks offline?)
- **Conflict Resolution:** If data is modified in Google Sheets and on the iOS app simultaneously, what is the source of truth?
- **Data Retention & Pruning:** How long is raw behavioral memory stored? How is the vector database pruned to avoid context bloat?
- **User Onboarding Phase:** How does the app learn the initial "Current State" and "Vision" without presenting a massive 100-question form?
- **Cost Management:** LLM API calls and vector database hosting scale linearly with usage. The specification lacks a monetization/cost-control strategy for heavy AI usage.

## 6. Architecture Proposal

 Clean Architecture Pattern


- **Client (iOS App):** Swift/SwiftUI. Handles UI, local state management, Voice/Speech-to-text, HealthKit integration, and secure local keychain.
- **API Gateway / Backend (Node.js/Go or Python/FastAPI):** Handles authentication, routing, rate limiting, and business logic orchestration.
- **AI Orchestration Layer (LangChain / LlamaIndex):** Manages interactions with LLMs. Implements the "AI Provider" abstraction.
- **Memory Engine (Vector DB):** Pinecone or Qdrant for semantic search over goals, history, and behavioral patterns (RAG).
- **Relational DB (PostgreSQL):** Stores structured user data (Profiles, Tasks, Goals, Transactions).
- **Integration Layer:** Microservices or serverless functions handling OAuth and webhooks (Google APIs, external CRMs).
- **Background Workers (Redis + Celery/Bull):** Handles delayed tasks, daily reviews aggregation, and syncing data with Google Sheets.

## 7. Database Schema Proposal

 Relational (PostgreSQL)

- **User:** ID, Email, PasswordHash, LifeScore, Level, XP.
- **LifeDomain:** ID, UserID, Name (1-6), CurrentPriorityScore.
- **Goal:** ID, UserID, DomainID, Tier (Vision, 5yr, 1yr, Q, M, W), Title, TargetValue, CurrentValue, Deadline, ParentGoalID.
- **Task:** ID, UserID, GoalID, Title, EstTime, PriorityScore, Status, CompletedAt.
- **Business_Entity (Companies/Leads):** ID, UserID, Type, Name, Value, Status, LastContact.
- **Financial_Transaction:** ID, UserID, Type (Income, Expense, Asset, Liability), Amount, Date, Category.
- **ScheduleBlock:** ID, UserID, TaskID, StartTime, EndTime, IsDeepWork.
- **Review:** ID, UserID, Type (Daily, Weekly), Score, Date, JSON_Data.
- **Integration:** ID, UserID, Provider, AccessToken, RefreshToken.

 Vector / Unstructured Data (Pinecone / Document DB)

- **Memory/Insight:** Text chunk, Embedding, UserID, Type (Observation, Mistake, Strategy), Timestamp, ConfidenceScore.

## 8. AI Architecture

 Multi-Agent Context Engine

1. **Intent Router:** A lightweight model (or regex/classification) decides what the user wants (e.g., "Log a task" vs. "What should I do?").
2. **Context Retrieval (RAG):** If strategic advice is needed, query PostgreSQL for current metrics (Net Worth, Pipeline) and Vector DB for behavioral memory and goals.
3. **Persona Prompter:** Wraps the retrieved context into a prompt customized for the "Strategic CEO" or "Chief of Staff" persona.
4. **Execution/Action Layer:** If the AI determines an action (e.g., "Create a task"), it outputs structured JSON, which the API gateway parses and inserts into the database.
5. **Provider Abstraction:** An interface `IAIProvider` with implementations for `OpenAIProvider`, `GeminiProvider`.

## 9. MVP Proposal

The MVP should focus on the core "Right Actions" loop, stripping away complex integrations and automation.

 In Scope for MVP:

- iOS Client (Home, Tasks, Chat/Voice input, Basic Analytics).
- Authentication & Profile.
- Six Life Domains and hierarchical Goals (manual entry).
- Task Manager with manual Priority Scoring (AI suggests top 2 domains).
- Daily "Top 3" and End-of-Day Review.
- AI Chat (Chief of Staff persona) powered by OpenAI API.
- Memory: Basic text-based semantic search for past decisions.
- Basic tracking: Simple manual input for Business (revenue/pipeline) and Wealth (cash/net worth).
- Google Sheets Integration: One-way sync (exporting tasks/goals to a sheet).

## 10. What Should NOT Be Built Yet

- **Complex Automations:** Automatically sending emails, auto-updating CRMs, or generating WhatsApp messages. Too high risk for MVP.
- **Multi-Model Intelligence Routing:** Swapping between OpenAI, Gemini, and Local models dynamically based on task type. Stick to one provider (OpenAI) for MVP.
- **Deep Behavioral Memory (Automated Psychological Profiling):** Having the AI automatically detect complex psychological procrastination patterns. This requires large amounts of historical data that the MVP won't have.
- **Full-scale Gamification (Levels, XP trees):** Beyond basic completion tracking, complex XP balancing will distract from core utility.
- **Integration with everything:** (Calendar, Gmail, Drive, Third-party CRMs). Stick to manual input and basic Google Sheets export first.
