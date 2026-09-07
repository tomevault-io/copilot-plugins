## ccg-springbootmongodbmoviecrud

> This is a Spring Boot REST API application demonstrating MongoDB (NoSQL) integration for movie management.

# Claude Code Instructions

## Project Overview

This is a Spring Boot REST API application demonstrating MongoDB (NoSQL) integration for movie management.

## Key Components

### Model Layer
- `Movie.java` - Document entity annotated with `@Document`
- Uses `@Id` from `org.springframework.data.annotation` (NOT JPA)

### Repository Layer
- `MovieRepository.java` - Interface extending `MongoRepository<Movie, String>`
- Provides automatic CRUD implementations
- Custom query methods following Spring Data naming conventions

### Controller Layer
- `MovieController.java` - REST controller with `/movie` base path
- Returns `Map<String, Object>` responses with message and data

## Important Patterns

### MongoDB vs JPA Differences
```java
// MongoDB
@Document(collection = "movies")
public class Movie {
    @Id  // from org.springframework.data.annotation
    private String movieId;
}

// JPA (for comparison)
@Entity
@Table(name = "movies")
public class Movie {
    @Id  // from javax.persistence
    @GeneratedValue
    private Long movieId;
}
```

### Save Method Behavior
- `save()` creates new document if ID doesn't exist
- `save()` updates existing document if ID exists
- No separate `update()` method in MongoRepository

### Query Methods
```java
// Automatic implementation based on method name
List<Movie> findByMovieName(String name);
List<Movie> findByMovieDurationBetween(Double min, Double max);

// Custom MongoDB query
@Query("{ 'movieHero' : ?0 }")
List<Movie> findMoviesByHero(String hero);
```

## Build & Run

```bash
# Build
mvn clean install

# Run (requires MongoDB on localhost:27017)
mvn spring-boot:run

# Run tests (uses embedded MongoDB)
mvn test
```

## Configuration

Application connects to MongoDB using:
- Database: `eduCbaNoSQL`
- Host: `localhost`
- Port: `27017`

## API Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/movie/createMovie` | Create movie |
| GET | `/movie/movieById?movieId={id}` | Get by ID |
| GET | `/movie/readAllMovies` | Get all |
| PUT | `/movie/updateMovie?movieId={id}&movieName={name}` | Update name |
| DELETE | `/movie/deleteMovie?movieId={id}` | Delete |

## When Modifying This Project

1. Always use `@Document` instead of `@Entity` for MongoDB
2. MongoRepository provides `save()`, `findById()`, `findAll()`, `deleteById()`, etc.
3. Custom query methods follow naming convention: `findBy{Field}{Condition}`
4. Test with embedded MongoDB (configured in test/resources)
5. MongoDB creates database/collections automatically on first insert

---
> Source: [KennethJefferson/ccg_SpringBootMongoDBMovieCrud](https://github.com/KennethJefferson/ccg_SpringBootMongoDBMovieCrud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
