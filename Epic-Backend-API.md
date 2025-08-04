

# Requirements and Assumptions

This application is built based on the following assumptions:

1. We are having a database with the following tables:
      - `countries` - (This table stores the country details. i.e. country code ISO 3166, country name)
      - `users` (This table stores the user details i.e. user_id, username, country_code)
      - `games` (This table stores the available game details. i.e. game_id, game_name, description, country_code, max_players)
      - `game_modes` (available game modes for each game. one game can have multiple game modes)
      - `plays` (This table stores the user plays details. i.e. user_id, game_id, game_mode_id, country_code, timestamp)
      - `popularity_counts` (This table stores the popularity count details. i.e. country_code, game_mode_id, game_id, count)


    # ER Diagram
    ![img.png](img.png)    

    # DML Script
    schema in src/main/resources/data.sql

     
2. Need to build a RESTful API with the following two endpoints to support the following use cases:
    - **Use case 1**: API endpoint for the user to fetch the user’s `region(country)` and `game mode`. 
        example: user `user1` is from country `US` and is playing the game `Game1` in game mode `Mode1 for Game1`.
                 API endpoint should return country - `US` and game mode - `Mode1 for Game1`.   
                    
    - **Use case 2**: API endpoint for the user to fetch the current most popular game modes for the region(country) in which the player is located.
        example: user `user1` is from country `US` and he wants to know the most popular game modes for the game `Game1`.
                 API endpoint should return the most popular game modes for the region(country) `US` and game `Game1`.

3. Design a RESTful web service to support this functionality, which will easily scale to millions of concurrent users.


# Epic Backend API Design and Implementation

1. **Use case 1** Fetch the user’s region(country) and game mode.
   - Swagger UI: http://localhost:8089/swagger-ui/index.html
   - API endpoint: http://localhost:8089/api/v1/playingmode/00000001-0001-0002-0003-000000000001/000003e9-03e9-03ea-03eb-0000000003e9
   - Implemented the logic to fetch the data asynchronously.

2. **Use case 2** Fetch the current most popular game modes for the region(country) in which the player is located and the game he is playing.
   - Swagger UI: http://localhost:8089/swagger-ui/index.html
   - API endpoint: http://localhost:8089/api/v1/popularity/gamemode/US/000003e9-03e9-03ea-03eb-0000000003e9
   - **Redis is available**: 
       - Implemented the logic to fetch the data from Redis.
       - Periodically reads counters from Redis.
       - Aggregates and merges values into the SQL database.
       - Clears flushed entries from Redis after successful sync.
       - Runs automatically at a fixed rate (e.g., every 1 minute).
     **Redis is not available**:
       - Implemented the logic to fetch the data from SQL database.
      

This application is built using Java, Spring Boot, JPA, Redis, and H2 Database.

3. **How this application is designed to handle millions of concurrent users**

### Scalability Highlights of This Design
- **RESTful APIs:** - Designed for stateless, concurrent access.
- **Redis for Aggregation:** Offloads real-time counters from SQL DB, enabling ultra-fast updates/reads.
- **Scheduled Tasks:** Periodic aggregation allows efficient batch updates instead of constant DB writes.
- **Separation of Concerns:** Layered architecture (Controller/Service/Repo/Domain/Integration) makes scaling and refactoring straightforward.

### Final Steps for Production Readiness
- Swap H2 with an AWS RDS engine.
- Move to ElastiCache Redis.
- Review and optimize configuration for Java process memory, connection pools, etc.

### Final Thoughts
We can choose AWS EBS or ECS or EKS for deployment.
- **EBS** is fast and simple for a Java/Spring Boot app.
- **ECS** is a straightforward move if you’re ready for Docker and AWS-native containers.
- **EKS** is the most powerful—ideal if you want Kubernetes standardization and flexibility.


---

## Architecture & Layers

## Layered Architecture

### 1. API Layer (Controller)
- **Exposes** REST endpoints for retrieving game, mode, and user data.
- **Handles** validation, status codes, error mapping.

### 2. Service Layer
- **Implements** business logic, aggregation, and orchestration.
- **Manages** Redis interactions and data sync.
- **Handles** exception/business rules.

### 3. Repository Layer
- **Spring Data JPA** abstraction for CRUD & queries.
- **Entities:** Games, GameModes, Countries, Users, Plays, PopularityCount.

### 4. Domain/Model Layer
- Core data classes:
    - `User`, `Game`, `GameMode`, `Country`, `Plays`, `PopularityCount`
    - Relationships, persistence, and mapping via annotations.

### 5. Integration Layer (Redis)
- **Handles** in-memory aggregation via Spring Data Redis.
- **Periodically flushes** aggregated counters to SQL.

### 6. Scheduled Tasks
- **Reads**, merges, and **clears** popularity counters from Redis to SQL at fixed intervals (default: 1 min).

### Scheduled Tasks

- **Purpose:** Ensures synchronization between Redis and persistent storage.
- **Responsibilities:**
  - Periodically reads counters from Redis.
  - Aggregates and merges values into the SQL database.
  - Clears flushed entries from Redis after successful sync.
  - Runs automatically at a fixed rate (e.g., every 1 minute).

---

## Tech Stack

- **Java 17**
- **Spring Boot 3.5.x**
- **Spring Data JPA**
- **Spring Data Redis**
- **Spring Web**
- **Spring Boot Actuator**
- **Lombok**
- **H2 Database** (for development)
- **Redis** (required for aggregation and caching)

---

## Getting Started

### Prerequisites

- Java 17+
- Maven 3.6+
- Redis server running on `localhost:6379`

### Build and Run


1. **Start Redis:**  
   Ensure the Redis server is up (default port `6379`).

2. **Build the project:**
   ```sh
   mvn clean package
   ```

3**Run the application:**
   ```sh
   mvn spring-boot:run
   ```
   The API will be available at:  
   `http://localhost:8089/`

4. **Access H2 Database Console:**  
   - URL: `http://localhost:8089/h2-console`
   - JDBC URL: `jdbc:h2:mem:gamemodedb`
   - Username: `sa`, Password: `password`

5. **Access Swagger UI:**  
   - URL: `http://localhost:8089/swagger-ui/index.html`



---

## Configuration

Main settings are in `application-dev.yml`:

- **Port:** 8089
- **H2 Console Path:** `/h2-console`
- **Logs:** `logs/application.log`
- **Redis:** `localhost:6379`
- **Database:** In-memory by default (H2)
- **Aggregates:** Flushed every minute from Redis to SQL

---

## How the System Works

1. User plays a **game mode** in a **game** from a certain **country**.
2. **Play event** is recorded; corresponding popularity count for (country, game, mode) is incremented in Redis.
3. **Background service** (every minute) reads all aggregate counts from Redis, updates the persistent SQL database (`popularity_counts`), and deletes processed Redis entries.
4. **API clients** can retrieve fresh popularity statistics for reporting.

---

## Useful Maven Commands

- **Run Tests:**  
  `mvn test`
- **Build JAR:**  
  `mvn clean package`
- **Start Application:**  
  `mvn spring-boot:run`

---




