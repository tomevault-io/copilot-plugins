## fastapi-blog

> This is a basic learning project for FastAPI, created to help developers understand the fundamentals of building web applications with FastAPI. The project is inspired by [Corey Schafer](https://www.youtube.com/@coreyms)'s FastAPI tutorials.

# FastAPI-Blog Project - Agent Documentation

## Project Overview

This is a basic learning project for FastAPI, created to help developers understand the fundamentals of building web applications with FastAPI. The project is inspired by [Corey Schafer](https://www.youtube.com/@coreyms)'s FastAPI tutorials.

### Project Purpose
- Learning FastAPI framework basics
- Understanding web API development with Python
- Practicing modern Python web development patterns

## Technology Stack

- **Framework**: FastAPI (>=0.128.0)
- **Python Version**: 3.14+
- **Package Manager**: uv (modern Python package manager)
- **Dependencies**: FastAPI with standard extras (includes uvicorn, pydantic, etc.)

## Project Structure

```
fastapi-blog/
├── main.py           # Main application file with FastAPI app and routes
├── pyproject.toml    # Project metadata and dependencies
├── README.md         # Basic project description
├── snippets.txt      # Code snippets and examples
├── .python-version   # Python version specification
├── .gitignore       # Git ignore patterns
└── uv.lock          # Dependency lock file
```

## Setup Instructions

### Prerequisites
- Python 3.14 or higher
- uv package manager (recommended) or pip

### Installation Steps

1. **Clone the repository**:
   ```bash
   git clone https://github.com/HengJayWang/fastapi-blog.git
   cd fastapi-blog
   ```

2. **Install dependencies using uv**:
   ```bash
   uv sync
   ```

   Or with pip:
   ```bash
   pip install -e .
   ```

3. **Run the application**:
   ```bash
   fastapi dev main.py
   ```
   
   Or using uvicorn directly:
   ```bash
   uvicorn main:app --reload
   ```

4. **Access the application**:
   - Main page: http://localhost:8000
   - API documentation: http://localhost:8000/docs
   - Alternative API docs: http://localhost:8000/redoc

## API Endpoints

### HTML Endpoints

- **GET /** - Home page (HTML response)
  - Returns an HTML page with the first post's title
  - Not included in OpenAPI schema

- **GET /posts** - Posts listing page (HTML response)
  - Returns an HTML page with the first post's title
  - Not included in OpenAPI schema

### API Endpoints

- **GET /api/posts** - Get all blog posts
  - Returns: JSON object with list of posts
  - Response format:
    ```json
    {
      "posts": [
        {
          "id": 1,
          "author": "Corey Schafer",
          "title": "FastAPI is Awesome",
          "content": "This framework is really easy to use and super fast.",
          "date_posted": "April 20, 2025"
        }
      ]
    }
    ```

## Data Structure

### Post Object
Each blog post contains the following fields:
- `id` (int): Unique identifier for the post
- `author` (str): Name of the post author
- `title` (str): Title of the blog post
- `content` (str): Main content of the post
- `date_posted` (str): Publication date

## Code Conventions

### Python Style
- Type hints are used for function parameters and return types
- Modern Python features (3.14+) are utilized
- List type hints use the modern syntax: `list[dict]` instead of `List[Dict]`

### FastAPI Patterns
- Routes are defined using decorator pattern (`@app.get()`)
- Response classes are explicitly specified when returning HTML
- API routes are prefixed with `/api/` for clarity
- HTML routes exclude themselves from schema using `include_in_schema=False`

## Development Workflow

### Running the Development Server
```bash
fastapi dev main.py
```

This will start the server with auto-reload enabled, allowing you to see changes immediately.

### Testing the API
You can test the API using:
- **Interactive docs**: Navigate to http://localhost:8000/docs
- **curl**: 
  ```bash
  curl http://localhost:8000/api/posts
  ```
- **Browser**: Visit http://localhost:8000/api/posts

## Current Features

1. **Static Data Storage**: Posts are stored in-memory as a Python list
2. **Basic Routes**: Home page and posts listing
3. **RESTful API**: JSON API endpoint for retrieving posts
4. **HTML Responses**: Basic HTML rendering for web pages

## Future Enhancement Opportunities

Potential areas for expansion:
- Database integration (PostgreSQL, SQLite)
- CRUD operations (Create, Update, Delete posts)
- User authentication and authorization
- Pydantic models for data validation
- Template engine integration (Jinja2)
- File upload functionality
- Pagination for posts
- Search and filtering capabilities
- Unit and integration tests

## Learning Resources

This project follows the tutorial series by Corey Schafer. Key concepts covered:
- FastAPI application setup
- Route definitions
- Response models
- HTML and JSON responses
- API documentation (automatic via FastAPI)

## Notes for AI Agents

When working on this project:

1. **Maintain Simplicity**: This is a learning project - keep changes simple and educational
2. **Type Hints**: Always use proper type hints for new functions
3. **FastAPI Best Practices**: Follow FastAPI conventions and patterns
4. **Documentation**: Keep the auto-generated API docs clean and useful
5. **Dependencies**: Use uv for package management when possible
6. **Python Version**: Ensure compatibility with Python 3.14+

### Common Tasks

- **Adding a new endpoint**: Define a new function with appropriate FastAPI decorator
- **Modifying post structure**: Update the `posts` list and ensure type hints match
- **Adding dependencies**: Update `pyproject.toml` and run `uv sync`
- **Testing changes**: Use the `/docs` endpoint to test API changes interactively

## Project Status

Current Version: 0.1.0

This is an active learning project in its initial stages. The basic structure is in place, with room for many enhancements as the learning progresses.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HengJayWang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HengJayWang)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
