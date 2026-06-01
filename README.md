# Twick

Twick is a search-first Twitter-like application. The first goal is not to build every social feature immediately. The first goal is to build a strong tweet search engine, then grow the project step by step into a fuller social app.

The project is inspired by Twitter/X search behavior: users should be able to search tweets by keywords, hashtags, users, phrases, and even misspelled words.

## Table of Contents

- [Project Goal](#project-goal)
- [Search-First Strategy](#search-first-strategy)
- [Core Features](#core-features)
- [Technology Stack](#technology-stack)
- [Database Strategy](#database-strategy)
- [High-Level Architecture](#high-level-architecture)
- [Search Flow](#search-flow)
- [Tweet Indexing Flow](#tweet-indexing-flow)
- [Requirements](#requirements)
- [Data Model Plan](#data-model-plan)
- [API Design Plan](#api-design-plan)
- [Search Behavior Plan](#search-behavior-plan)
- [Step-by-Step Build Plan](#step-by-step-build-plan)
- [Testing Plan](#testing-plan)
- [Future Improvements](#future-improvements)
- [Project Status](#project-status)

## Project Goal

Twick will become a Twitter-like application with strong search functionality.

The first version will focus on:

- storing searchable tweet data
- indexing tweets into Elasticsearch
- searching tweets by text
- returning ranked search results
- supporting typo-tolerant search
- supporting hashtag and username search

Later versions can add:

- user registration and login
- user profiles
- posting tweets
- following users
- timeline feed
- likes
- replies
- reposts
- notifications
- frontend application

## Search-First Strategy

Most social apps become large quickly. Twick will avoid that by starting with one important feature: search.

The build order is:

1. Build the backend search API.
2. Connect Spring Boot with Elasticsearch.
3. Create and index tweet data.
4. Search tweets with ranking and fuzzy matching.
5. Add PostgreSQL as the source-of-truth database.
6. Add full Twitter-like social features.
7. Add frontend and production-style improvements.

This keeps the project easier to understand and easier to finish.

## Core Features

### Phase 1: Search MVP

- Search tweets by keyword.
- Return ranked tweet results.
- Support pagination.
- Seed sample tweet data.
- Use Elasticsearch for full-text search.
- Expose search through a Spring Boot REST API.

### Phase 2: Tweet Management

- Create tweets.
- Update tweets.
- Delete tweets.
- Index new tweets into Elasticsearch.
- Update Elasticsearch when a tweet changes.
- Remove tweets from Elasticsearch when deleted.

### Phase 3: Better Search

- Fuzzy search for spelling mistakes.
- Hashtag search, such as `#security`.
- Username search.
- Exact phrase search.
- Prefix/autocomplete search.
- Ranking based on relevance.
- Optional highlighting of matched terms.

### Phase 4: Social App Features

- User registration.
- User login.
- User profiles.
- Follow/unfollow users.
- Home timeline.
- Like tweets.
- Reply to tweets.
- Repost tweets.

### Phase 5: Frontend and Production Polish

- Web frontend.
- API documentation.
- Caching.
- Logging.
- Metrics.
- Docker Compose for full local development.
- Integration tests.
- Deployment preparation.

## Technology Stack

### Backend

- Java
- Spring Boot
- Spring Web
- Spring Data JPA
- Spring Data Elasticsearch
- Bean Validation

### Databases

- PostgreSQL for main application data.
- Elasticsearch for tweet search.

### Local Development

- Docker Compose for PostgreSQL and Elasticsearch.
- Maven for building and running the backend.

### Future Frontend

The frontend can be added later. A likely frontend stack is:

- React
- TypeScript
- Vite

The frontend is not part of the first backend search milestone.

## Database Strategy

Twick should use both PostgreSQL and Elasticsearch.

### PostgreSQL

PostgreSQL is the source of truth.

It stores real application data:

- users
- tweets
- follows
- likes
- replies
- reposts
- profile data
- timestamps

PostgreSQL is a good fit because a Twitter-like app has many relationships:

- one user creates many tweets
- one user follows many users
- many users can like one tweet
- one tweet can have many replies

This relational structure fits SQL well.

### Elasticsearch

Elasticsearch is the search index.

It stores searchable copies of tweet data:

- tweet id
- tweet text
- username
- user id
- hashtags
- created time
- ranking-related fields later

Elasticsearch is used because it is designed for:

- full-text search
- fuzzy search
- ranking
- tokenization
- phrase search
- prefix search
- fast search over large text data

### Final Data Rule

PostgreSQL is the source of truth. Elasticsearch is a derived search index.

If PostgreSQL and Elasticsearch disagree, PostgreSQL should be treated as correct.

```text
PostgreSQL = real application data
Elasticsearch = searchable copy for fast search
```

## High-Level Architecture

```text
User / Frontend
      |
      v
Spring Boot REST API
      |
      |-- PostgreSQL
      |      stores users, tweets, follows, likes, replies
      |
      |-- Elasticsearch
             stores searchable tweet documents
```

For the first milestone, the app can start with Spring Boot and Elasticsearch only. PostgreSQL should be added when tweet creation and user features become real application data.

## Search Flow

When a user searches for tweets:

```text
1. User searches: "quantum computing"
2. Frontend calls: GET /api/search?query=quantum%20computing
3. Spring Boot receives the request.
4. Search service sends the query to Elasticsearch.
5. Elasticsearch finds matching tweet documents.
6. Elasticsearch ranks the results.
7. Spring Boot returns JSON results to the frontend.
```

Example response shape:

```json
{
  "query": "quantum computing",
  "page": 0,
  "size": 10,
  "total": 3,
  "results": [
    {
      "id": "tweet-1002",
      "text": "Quantum computing enhances cybersecurity. #Security",
      "userId": "user-2",
      "username": "cyber_daily",
      "hashtags": ["security"],
      "createdAt": "2026-06-01T10:15:30Z",
      "score": 8.42
    }
  ]
}
```

## Tweet Indexing Flow

When a tweet is created:

```text
1. User creates a tweet.
2. Spring Boot validates the request.
3. Spring Boot saves the tweet in PostgreSQL.
4. Spring Boot creates a searchable tweet document.
5. Spring Boot indexes that document in Elasticsearch.
6. The tweet becomes searchable.
```

For the first MVP, tweets may be seeded directly into Elasticsearch. Later, PostgreSQL should become the main source of truth.

## Requirements

### Functional Requirements

#### Search

- The system must allow users to search tweets by keyword.
- The system must return ranked search results.
- The system must support pagination.
- The system must support fuzzy search for small spelling mistakes.
- The system must support hashtag search.
- The system must support username search.
- The system should support exact phrase search.
- The system should support autocomplete in a later phase.

#### Tweets

- The system must allow creating tweets.
- The system must allow updating tweets.
- The system must allow deleting tweets.
- Created tweets must become searchable.
- Updated tweets must update the search index.
- Deleted tweets must be removed from search results.

#### Users

- The system should allow user registration in a later phase.
- The system should allow login in a later phase.
- The system should allow profile pages in a later phase.
- The system should support usernames for search.

#### Social Features

- The system should support following users in a later phase.
- The system should support a timeline feed in a later phase.
- The system should support likes in a later phase.
- The system should support replies in a later phase.
- The system should support reposts in a later phase.

### Non-Functional Requirements

#### Performance

- Search responses should be fast for normal query sizes.
- Search results should be paginated to avoid returning too much data.
- Elasticsearch should handle full-text search instead of PostgreSQL for search-heavy queries.

#### Reliability

- PostgreSQL should store the durable copy of application data.
- Elasticsearch can be rebuilt from PostgreSQL if needed.
- Failed indexing operations should be logged.

#### Scalability

- Search logic should be separated from controller logic.
- Tweet storage should be separated from tweet indexing.
- The architecture should allow background indexing later.

#### Security

- Public search can exist before authentication.
- Tweet creation should require authentication in a later phase.
- User passwords must never be stored as plain text.
- Input validation should be applied to all write APIs.

#### Maintainability

- Code should be organized by feature.
- API request and response models should be clear.
- Search behavior should be tested with repeatable sample data.

## Data Model Plan

### Initial Search Model

#### Tweet Search Document

This is the document stored in Elasticsearch.

```text
TweetSearchDocument
- id
- text
- userId
- username
- hashtags
- createdAt
- likeCount
- replyCount
- repostCount
```

The first MVP can start without `likeCount`, `replyCount`, and `repostCount`. These fields become useful later for ranking.

#### Search Result

```text
SearchResult
- id
- text
- userId
- username
- hashtags
- createdAt
- score
```

### Later PostgreSQL Models

#### User

```text
User
- id
- username
- displayName
- email
- passwordHash
- createdAt
```

#### Tweet

```text
Tweet
- id
- authorId
- text
- createdAt
- updatedAt
- deletedAt
```

#### Follow

```text
Follow
- followerId
- followingId
- createdAt
```

#### Like

```text
Like
- userId
- tweetId
- createdAt
```

#### Reply

```text
Reply
- id
- tweetId
- authorId
- text
- createdAt
```

#### Repost

```text
Repost
- userId
- tweetId
- createdAt
```

## API Design Plan

The exact API can evolve, but this is the planned direction.

### Search APIs

#### Search Tweets

```text
GET /api/search?query={query}&page={page}&size={size}
```

Example:

```text
GET /api/search?query=quantum%20computing&page=0&size=10
```

Purpose:

- Search indexed tweets.
- Return ranked results.
- Support pagination.

#### Autocomplete Search

```text
GET /api/search/suggest?prefix={prefix}
```

This should be added after normal search works.

### Tweet APIs

#### Create Tweet

```text
POST /api/tweets
```

Example request:

```json
{
  "text": "Quantum computing enhances cybersecurity. #Security"
}
```

Purpose:

- Save a tweet.
- Index it for search.

#### Update Tweet

```text
PUT /api/tweets/{id}
```

Purpose:

- Update the tweet in PostgreSQL.
- Update the Elasticsearch document.

#### Delete Tweet

```text
DELETE /api/tweets/{id}
```

Purpose:

- Delete or soft-delete the tweet in PostgreSQL.
- Remove it from Elasticsearch.

#### Seed Tweets

```text
POST /api/tweets/seed
```

Purpose:

- Add sample tweets for development and testing.
- Useful before user accounts exist.

### Future User APIs

```text
POST /api/auth/register
POST /api/auth/login
GET  /api/users/{username}
PUT  /api/users/me/profile
```

### Future Social APIs

```text
POST   /api/users/{id}/follow
DELETE /api/users/{id}/follow
GET    /api/timeline
POST   /api/tweets/{id}/like
DELETE /api/tweets/{id}/like
POST   /api/tweets/{id}/replies
POST   /api/tweets/{id}/repost
```

## Search Behavior Plan

### Keyword Search

User searches normal words:

```text
quantum computing
```

Expected behavior:

- Return tweets containing both or one of the terms.
- Rank stronger matches higher.
- Tweets with matches in important fields should rank higher.

### Hashtag Search

User searches:

```text
#Security
```

Expected behavior:

- Match tweets containing the hashtag.
- Treat hashtags case-insensitively.

### Username Search

User searches:

```text
cyber_daily
```

Expected behavior:

- Match tweets by username.
- Later, also return user/profile results if global search is added.

### Fuzzy Search

User searches:

```text
quantom computing
```

Expected behavior:

- Still return tweets about `quantum computing`.
- Small spelling mistakes should not produce empty results.

### Phrase Search

User searches:

```text
"quantum computing"
```

Expected behavior:

- Prefer tweets containing the exact phrase.
- Rank exact phrase matches higher than loose keyword matches.

### Autocomplete

User types:

```text
quant
```

Expected behavior:

- Suggest possible queries like `quantum` or `quantum computing`.
- This is a later feature, not required in the first MVP.

### Ranking

Initial ranking should use Elasticsearch relevance scoring.

Later ranking can consider:

- text match strength
- phrase match
- hashtag match
- recency
- likes
- replies
- reposts

## Step-by-Step Build Plan

### Phase 0: Project Documentation

- [x] Write the project README.
- [ ] Define the first implementation milestone.
- [ ] Confirm the backend package structure.
- [ ] Confirm local development tools.

Done when:

- The project has a clear roadmap.
- The first milestone is easy to understand.

### Phase 1: Spring Boot Setup

- [ ] Convert the basic Maven project into a Spring Boot project.
- [ ] Add Spring Web.
- [ ] Add validation support.
- [ ] Add a simple health endpoint.
- [ ] Run the backend locally.

Done when:

- The app starts successfully.
- A test endpoint returns a response.

### Phase 2: Elasticsearch Setup

- [ ] Add Docker Compose for Elasticsearch.
- [ ] Configure Spring Boot to connect to Elasticsearch.
- [ ] Create a tweet search document model.
- [ ] Create the Elasticsearch index mapping.

Done when:

- Elasticsearch runs locally.
- Spring Boot can connect to Elasticsearch.
- The tweet index exists.

### Phase 3: Seed Tweet Search Data

- [ ] Create sample tweet data.
- [ ] Add an endpoint or startup loader to index sample tweets.
- [ ] Include sample tweets related to `quantum computing`.
- [ ] Verify documents exist in Elasticsearch.

Done when:

- Sample tweets are indexed.
- Elasticsearch contains searchable tweet documents.

### Phase 4: Search API MVP

- [ ] Create `GET /api/search`.
- [ ] Accept `query`, `page`, and `size`.
- [ ] Search tweet text in Elasticsearch.
- [ ] Return JSON results.
- [ ] Include total count and pagination data.

Done when:

- Searching `quantum computing` returns relevant tweets.
- Empty or invalid queries are handled cleanly.

### Phase 5: Better Search

- [ ] Add fuzzy matching.
- [ ] Add hashtag search.
- [ ] Add username search.
- [ ] Add phrase boosting.
- [ ] Add result highlighting if useful.

Done when:

- `quantom computing` still finds `quantum computing`.
- `#security` returns hashtagged tweets.
- Exact phrase matches rank higher.

### Phase 6: PostgreSQL Source of Truth

- [ ] Add PostgreSQL to Docker Compose.
- [ ] Add Spring Data JPA.
- [ ] Create PostgreSQL tweet table.
- [ ] Save tweets in PostgreSQL.
- [ ] Index saved tweets into Elasticsearch.

Done when:

- Tweet data is stored in PostgreSQL.
- Search still uses Elasticsearch.
- Elasticsearch can be rebuilt from PostgreSQL data.

### Phase 7: Tweet Write APIs

- [ ] Create tweet API.
- [ ] Update tweet API.
- [ ] Delete tweet API.
- [ ] Keep PostgreSQL and Elasticsearch synchronized.

Done when:

- Created tweets appear in search.
- Updated tweets show updated search text.
- Deleted tweets disappear from search.

### Phase 8: Tests

- [ ] Add unit tests for service logic.
- [ ] Add controller tests for API validation.
- [ ] Add integration tests for Elasticsearch search behavior.
- [ ] Add PostgreSQL integration tests after PostgreSQL is added.

Done when:

- Search behavior is repeatable.
- Main API flows are covered by tests.

### Phase 9: Social Features

- [ ] Add user registration.
- [ ] Add login.
- [ ] Add profiles.
- [ ] Add follows.
- [ ] Add timeline.
- [ ] Add likes.
- [ ] Add replies.
- [ ] Add reposts.

Done when:

- Twick behaves like a basic Twitter-like app.
- Search remains the strongest feature.

### Phase 10: Frontend

- [ ] Create frontend project.
- [ ] Build search page.
- [ ] Build tweet creation page.
- [ ] Build profile page later.
- [ ] Connect frontend to backend APIs.

Done when:

- Users can search tweets from a browser.
- Search results are readable and paginated.

## Testing Plan

### Manual Tests

Use repeatable sample searches:

```text
quantum computing
quantom computing
#security
cyber_daily
"quantum computing"
renewable energy
```

Expected checks:

- Relevant tweets appear.
- Typo queries still return useful results.
- Hashtag queries work.
- Pagination works.
- Deleted tweets do not appear.

### Unit Tests

Test:

- request validation
- hashtag extraction
- search query building
- tweet indexing logic
- tweet update/delete behavior

### Integration Tests

Test:

- Spring Boot can connect to Elasticsearch.
- Tweets can be indexed.
- Search returns ranked results.
- Fuzzy search works.
- PostgreSQL and Elasticsearch stay synchronized after PostgreSQL is added.

### API Tests

Test:

- successful search
- empty query
- invalid page or size
- create tweet
- update tweet
- delete tweet

## Future Improvements

- Cache popular search queries.
- Add search analytics.
- Add trending hashtags.
- Add advanced ranking with engagement and recency.
- Add background indexing.
- Add retry handling for failed indexing.
- Add OpenAPI/Swagger documentation.
- Add frontend search suggestions.
- Add deployment configuration.
- Add monitoring with metrics and logs.

## Project Status

Current status:

- Planning and documentation stage.
- Existing project is a basic Maven Java starter.
- Search-system design document is available in the repository.
- Implementation has not started yet.

Next recommended step:

1. Convert the Maven project to Spring Boot.
2. Add Elasticsearch with Docker Compose.
3. Build the first search endpoint.

