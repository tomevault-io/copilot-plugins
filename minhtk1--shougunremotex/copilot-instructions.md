## xshougun

> - Each class/function should have only one reason to change


# Python Project Rules - SOLID Principles & Best Practices

## Core SOLID Principles

### 1. Single Responsibility Principle (SRP)
- Each class/function should have only one reason to change
- Keep functions focused on a single task
- Separate concerns into different modules/classes
- Example: Data validation, business logic, and data persistence should be separate

### 2. Open/Closed Principle (OCP)
- Classes should be open for extension but closed for modification
- Use inheritance and composition over direct modification
- Implement abstract base classes (ABC) for extensible interfaces
- Use dependency injection for flexible behavior

### 3. Liskov Substitution Principle (LSP)
- Derived classes must be substitutable for their base classes
- Maintain behavioral contracts in inheritance hierarchies
- Override methods should not weaken preconditions or strengthen postconditions
- Use proper type hints and abstract methods

### 4. Interface Segregation Principle (ISP)
- Clients should not be forced to depend on interfaces they don't use
- Create specific interfaces rather than large, general ones
- Use Protocol for interface definitions
- Split large interfaces into smaller, focused ones

### 5. Dependency Inversion Principle (DIP)
- Depend on abstractions, not concretions
- High-level modules should not depend on low-level modules
- Use dependency injection and inversion of control
- Abstract interfaces should not depend on details

## Python-Specific Rules

### Code Structure
- Use type hints for all function parameters and return values
- Follow PEP 8 style guidelines
- Use dataclasses for simple data containers
- Implement `__str__` and `__repr__` methods appropriately
- Use context managers (`with` statements) for resource management

### Error Handling
- Use specific exception types, not generic `Exception`
- Create custom exceptions for domain-specific errors
- Use `try/except/finally` blocks appropriately
- Log errors with proper context information
- Never use bare `except:` clauses

### Testing
- Write unit tests for all public methods
- Use pytest as the testing framework
- Aim for high test coverage (>80%)
- Use fixtures for test data setup
- Mock external dependencies in tests

### Performance & Memory
- Use generators for large data processing
- Implement `__slots__` for classes with many instances
- Use `functools.lru_cache` for expensive computations
- Profile code before optimizing
- Use appropriate data structures (collections module)

### Security
- Validate all input data
- Use parameterized queries for database operations
- Sanitize user inputs
- Use environment variables for sensitive configuration
- Implement proper authentication and authorization

### Documentation
- Write docstrings for all classes and functions
- Use Google-style docstrings
- Include type information in docstrings
- Document complex algorithms and business logic
- Keep comments up-to-date with code changes

### Code Comments (Vietnamese)
- **LUÔN LUÔN** comment code bằng tiếng Việt cho các xử lý logic
- Comment giải thích mục đích và cách thức hoạt động của code
- Sử dụng tiếng Việt có dấu và rõ ràng
- Comment cho các thuật toán phức tạp và business logic
- Comment cho các đoạn code không rõ ràng hoặc có thể gây hiểu lầm
- Ví dụ: `# Kiểm tra điều kiện đăng nhập của người dùng`

## File Organization
- One class per file (except for small related classes)
- Use `__init__.py` files for package initialization
- Group related functionality into modules
- Keep main execution code in `if __name__ == "__main__":` blocks
- Use absolute imports for clarity

## Dependencies
- Pin dependency versions in requirements.txt
- Use virtual environments for isolation
- Minimize external dependencies
- Keep dependencies up-to-date
- Document why each dependency is needed
- **BẮT BUỘC: MỖI LẦN sử dụng thư viện mới phải thêm vào requirements.txt**
- **KHÔNG BAO GIỜ** import thư viện mà chưa có trong requirements.txt
- Sử dụng format: `package_name>=version` hoặc `package_name==version`
- Nhóm dependencies theo mục đích sử dụng (Core, Development, Build, Optional)
- **Comment bằng tiếng Việt** cho từng nhóm dependencies và mục đích sử dụng
- Ví dụ comment: `# Thư viện xử lý JSON và cấu hình`

## Feature Development Process
- **MỖI LẦN thêm tính năng mới phải comment bằng tiếng Việt**
- Comment giải thích mục đích và cách thức hoạt động của tính năng
- Sử dụng tiếng Việt có dấu và rõ ràng trong tất cả comment
- Comment cho các thuật toán phức tạp và business logic
- Comment cho các đoạn code không rõ ràng hoặc có thể gây hiểu lầm
- Ví dụ comment tính năng:
```python
# Tính năng: Theo dõi thay đổi file trong thư mục
# Mục đích: Phát hiện khi có file mới được tạo hoặc sửa đổi
# Cách hoạt động: Sử dụng watchdog để monitor file system events
```

## Build Process
- **LUÔN LUÔN activate virtual environment trước khi chạy build**
- Command: `D:\project\H74\virtualenv\xpra\Scripts\Activate.ps1`
- Sau đó mới chạy: `python build.py`
- Đảm bảo tất cả dependencies đã được cài đặt trong virtual environment
- Kiểm tra Python version và dependencies trước khi build

## Code Quality
- Use linting tools (flake8, black, mypy)
- Run static analysis tools regularly
- Keep functions small (<20 lines when possible)
- Use meaningful variable and function names
- Avoid deep nesting (>3 levels)

## Git Practices
- Write clear, descriptive commit messages
- Use feature branches for development
- Keep commits atomic and focused
- Use pull requests for code review
- Tag releases appropriately

## License Compliance
- **TUÂN THỦ GNU General Public License v2.0** khi copy code từ xpra-client
- Mọi code được copy từ xpra-client phải tuân thủ GPL v2.0 license
- Phải ghi rõ nguồn gốc code và license trong header comment
- Không được sử dụng code từ xpra-client trong các phần proprietary
- Đảm bảo toàn bộ project tuân thủ GPL v2.0 nếu có sử dụng code từ xpra-client
- Ví dụ header comment:
```python
# Code copied from xpra-client under GNU General Public License v2.0
# Original source: xpra-client/path/to/file.py
# License: GNU General Public License v2.0
# This file is part of ShougunRemoteX-Python project
```

## Examples of Good Practices

### Requirements.txt Organization
```txt
# Thư viện cốt lõi - Cập nhật cho Python 3.11.9
pywin32>=306
pythonnet>=3.0.3
psutil>=5.9.6
pydantic>=2.5.2
loguru>=0.7.2
pyyaml>=6.0.1

# Thư viện phát triển và testing
pytest>=7.4.3
pytest-cov>=4.1.0
black>=23.11.0
flake8>=6.1.0
mypy>=1.7.1

# Thư viện build và packaging
pyinstaller>=6.0.0
auto-py-to-exe>=2.40.0

# Thư viện tùy chọn cho tính năng nâng cao
asyncio-mqtt>=0.16.1
redis>=5.0.1
```

### Build Process Example
```powershell
# Bước 1: Activate virtual environment
D:\project\H74\virtualenv\xpra\Scripts\Activate.ps1

# Bước 2: Kiểm tra Python version
python --version

# Bước 3: Cài đặt dependencies (nếu cần)
pip install -r requirements.txt

# Bước 4: Chạy build
python build.py
```

### Dependency Injection
```python
from abc import ABC, abstractmethod
from typing import Protocol

class DatabaseProtocol(Protocol):
    def save(self, data: dict) -> None: ...
    def get(self, id: str) -> dict: ...

class UserService:
    def __init__(self, db: DatabaseProtocol):
        self.db = db
    
    def create_user(self, user_data: dict) -> None:
        # Xử lý logic nghiệp vụ tạo người dùng mới
        self.db.save(user_data)
```

### Error Handling
```python
class UserNotFoundError(Exception):
    """Raised when user is not found in database"""
    pass

def get_user(user_id: str) -> dict:
    try:
        # Lấy thông tin người dùng từ database
        return database.get(user_id)
    except DatabaseError as e:
        # Ghi log lỗi và ném exception tùy chỉnh
        logger.error(f"Database error for user {user_id}: {e}")
        raise UserNotFoundError(f"User {user_id} not found") from e
```

### Type Hints
```python
from typing import List, Optional, Union
from dataclasses import dataclass

@dataclass
class User:
    id: str
    name: str
    email: str
    is_active: bool = True

def process_users(users: List[User]) -> Optional[User]:
    # Tìm người dùng đầu tiên có trạng thái hoạt động
    return next((user for user in users if user.is_active), None)
```

## Anti-Patterns to Avoid
- God classes (classes that do too much)
- Deep inheritance hierarchies (>3 levels)
- Tight coupling between modules
- Global state and mutable global variables
- Long parameter lists (>5 parameters)
- Copy-paste code duplication
- Silent failures and ignored exceptions
- Hard-coded configuration values
- Circular imports
- Premature optimization

## Code Review Checklist
- [ ] Follows SOLID principles
- [ ] Has proper type hints
- [ ] Includes appropriate error handling
- [ ] Has unit tests
- [ ] Follows PEP 8 style
- [ ] Has clear docstrings
- [ ] **Comments bằng tiếng Việt cho các xử lý logic**
- [ ] **BẮT BUỘC: Thư viện mới đã được thêm vào requirements.txt**
- [ ] **KHÔNG BAO GIỜ import thư viện chưa có trong requirements.txt**
- [ ] **Comment bằng tiếng Việt cho tính năng mới**
- [ ] **Virtual environment đã được activate trước khi build**
- [ ] **License compliance: Code từ xpra-client có header comment GPL v2.0**
- [ ] No code duplication
- [ ] Proper separation of concerns
- [ ] Uses dependency injection where appropriate
- [ ] Handles edge cases

---
> Source: [minhtk1/ShougunRemoteX](https://github.com/minhtk1/ShougunRemoteX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
