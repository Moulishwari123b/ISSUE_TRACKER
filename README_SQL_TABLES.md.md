# Backend SQL Tables

This project uses Django with a custom `User` model in the `core` app. The main backend tables are:

## `core_user`
Stores application users and their roles.

```sql
CREATE TABLE core_user (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    password VARCHAR(128) NOT NULL,
    last_login DATETIME NULL,
    is_superuser BOOLEAN NOT NULL,
    username VARCHAR(150) NOT NULL UNIQUE,
    first_name VARCHAR(150) NOT NULL,
    last_name VARCHAR(150) NOT NULL,
    email VARCHAR(254) NOT NULL,
    is_staff BOOLEAN NOT NULL,
    is_active BOOLEAN NOT NULL,
    date_joined DATETIME NOT NULL,
    role VARCHAR(20) NOT NULL DEFAULT 'user'
);
```

### Key columns
- `id`: Primary key.
- `username`: Unique account name.
- `role`: User role, one of `user`, `admin`, `developer`.
- `is_staff`, `is_superuser`: Django authorization helpers.

## `core_issue`
Tracks reported issues, approvals, assignments, and lifecycle status.

```sql
CREATE TABLE core_issue (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    priority VARCHAR(10) NOT NULL,
    category VARCHAR(10) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'open',
    validation_status VARCHAR(20) NOT NULL DEFAULT 'pending',
    rejection_reason TEXT NOT NULL,
    created_by_id INTEGER NOT NULL REFERENCES core_user(id) ON DELETE CASCADE,
    assigned_to_id INTEGER NULL REFERENCES core_user(id) ON DELETE SET NULL,
    validated_by_id INTEGER NULL REFERENCES core_user(id) ON DELETE SET NULL,
    attachment VARCHAR(100) NULL,
    solution TEXT NOT NULL,
    deadline DATE NULL,
    created_at DATETIME NOT NULL,
    updated_at DATETIME NOT NULL
);
```

### Key columns
- `title`, `description`: Issue details.
- `priority`: One of `low`, `medium`, `high`.
- `category`: One of `bug`, `feature`, `task`.
- `status`: Workflow state such as `open`, `assigned`, `in_progress`, `resolved`, `approved`, `closed`, `reopened`.
- `validation_status`: Review state such as `pending`, `approved`, `rejected`.
- `created_by_id`: Reporter.
- `assigned_to_id`: Developer assigned to the issue.
- `validated_by_id`: Admin who approved or rejected.
- `attachment`: Optional file path.
- `solution`: Developer-submitted solution text.

## `core_comment`
Stores comments attached to issues.

```sql
CREATE TABLE core_comment (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    issue_id INTEGER NOT NULL REFERENCES core_issue(id) ON DELETE CASCADE,
    user_id INTEGER NOT NULL REFERENCES core_user(id) ON DELETE CASCADE,
    text TEXT NOT NULL,
    attachment VARCHAR(100) NULL,
    created_at DATETIME NOT NULL
);
```

### Key columns
- `issue_id`: Issue reference.
- `user_id`: Comment author.
- `text`: Comment body.
- `attachment`: Optional uploaded file path.

## Relationships
- `core_issue.created_by_id` → `core_user.id` (reporter)
- `core_issue.assigned_to_id` → `core_user.id` (developer)
- `core_issue.validated_by_id` → `core_user.id` (approving admin)
- `core_comment.issue_id` → `core_issue.id`
- `core_comment.user_id` → `core_user.id`

## Notes
- The backend is primarily driven by these three tables in the `core` app.
- Django’s auth system also creates standard tables such as `auth_group`, `auth_permission`, `django_content_type`, `django_migrations`, and `django_session`, but the key application data is stored in the `core_*` tables above.

# MySQL Migration Instructions

## Step 1: CREATE DATABASE
```sql
CREATE DATABASE IF NOT EXISTS project_db
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
USE project_db;
```

## Step 2: CREATE TABLE queries
```sql
CREATE TABLE core_user (
  id INT AUTO_INCREMENT PRIMARY KEY,
  password VARCHAR(128) NOT NULL,
  last_login DATETIME NULL,
  is_superuser TINYINT(1) NOT NULL DEFAULT 0,
  username VARCHAR(150) NOT NULL UNIQUE,
  first_name VARCHAR(150) NOT NULL DEFAULT '',
  last_name VARCHAR(150) NOT NULL DEFAULT '',
  email VARCHAR(254) NOT NULL DEFAULT '',
  is_staff TINYINT(1) NOT NULL DEFAULT 0,
  is_active TINYINT(1) NOT NULL DEFAULT 1,
  date_joined DATETIME NOT NULL,
  role VARCHAR(20) NOT NULL DEFAULT 'user'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE core_issue (
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  description TEXT NOT NULL,
  priority VARCHAR(10) NOT NULL,
  category VARCHAR(10) NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'open',
  validation_status VARCHAR(20) NOT NULL DEFAULT 'pending',
  rejection_reason TEXT NOT NULL,
  created_by_id INT NOT NULL,
  assigned_to_id INT NULL,
  validated_by_id INT NULL,
  attachment VARCHAR(100) NULL,
  solution TEXT NOT NULL,
  deadline DATE NULL,
  created_at DATETIME NOT NULL,
  updated_at DATETIME NOT NULL,
  KEY idx_core_issue_created_by_id (created_by_id),
  KEY idx_core_issue_assigned_to_id (assigned_to_id),
  KEY idx_core_issue_validated_by_id (validated_by_id),
  CONSTRAINT fk_core_issue_created_by FOREIGN KEY (created_by_id)
    REFERENCES core_user(id) ON DELETE CASCADE,
  CONSTRAINT fk_core_issue_assigned_to FOREIGN KEY (assigned_to_id)
    REFERENCES core_user(id) ON DELETE SET NULL,
  CONSTRAINT fk_core_issue_validated_by FOREIGN KEY (validated_by_id)
    REFERENCES core_user(id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE core_comment (
  id INT AUTO_INCREMENT PRIMARY KEY,
  issue_id INT NOT NULL,
  user_id INT NOT NULL,
  text TEXT NOT NULL,
  attachment VARCHAR(100) NULL,
  created_at DATETIME NOT NULL,
  KEY idx_core_comment_issue_id (issue_id),
  KEY idx_core_comment_user_id (user_id),
  CONSTRAINT fk_core_comment_issue FOREIGN KEY (issue_id)
    REFERENCES core_issue(id) ON DELETE CASCADE,
  CONSTRAINT fk_core_comment_user FOREIGN KEY (user_id)
    REFERENCES core_user(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## Step 3: Django config
In `issue_tracker/settings.py`, replace the `DATABASES` block with:
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'project_db',
        'USER': 'your_mysql_username',
        'PASSWORD': 'your_mysql_password',
        'HOST': '127.0.0.1',
        'PORT': '3306',
        'OPTIONS': {
            'charset': 'utf8mb4',
            'use_unicode': True,
            'init_command': "SET sql_mode='STRICT_TRANS_TABLES', innodb_strict_mode=1",
        },
    }
}
```

## Step 4: Required pip package
Recommended package:
```bash
pip install mysqlclient
```

Alternative if you prefer PyMySQL:
```bash
pip install pymysql
```
```

If using `pymysql`, also add at the top of `settings.py`:
```python
import pymysql
pymysql.install_as_MySQLdb()
```

## Step 5: Migration commands
```bash
python manage.py makemigrations
python manage.py migrate
```

### Notes
- The SQL above is MySQL-compatible and uses `AUTO_INCREMENT`, `TINYINT(1)`, InnoDB, and `utf8mb4`.
- Foreign keys include `ON DELETE CASCADE` for required relationships and `ON DELETE SET NULL` for optional developer/validator references.
- Ensure the MySQL user has privileges to create `project_db` and create tables.
