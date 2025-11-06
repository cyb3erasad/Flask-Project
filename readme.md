# Flask Market Application - Complete Documentation

## Table of Contents

1. [Introduction](#introduction)
2. [System Architecture](#system-architecture)
3. [Application Structure](#application-structure)
4. [Database Design](#database-design)
5. [API Endpoints](#api-endpoints)
6. [Authentication System](#authentication-system)
7. [Configuration](#configuration)
8. [Deployment](#deployment)
9. [Troubleshooting](#troubleshooting)

---

## Introduction

### Purpose

This Flask Market Application is a web-based marketplace system designed as a learning project to understand Flask framework fundamentals. It demonstrates core web development concepts including user authentication, database integration, and marketplace functionality.

### Scope

The application provides:
- User registration and authentication system
- A marketplace interface for buying and selling items
- SQLite database integration for data persistence
- Basic but functional UI for all operations

### Target Audience

- Flask beginners learning web development
- Backend developers exploring Python web frameworks
- Students working on web application projects

---

## System Architecture

### Technology Stack

**Backend:**
- Flask (Web Framework)
- SQLAlchemy (ORM)
- Flask-Login (Authentication)
- Flask-WTF (Forms)
- Flask-Bcrypt (Password Hashing)

**Database:**
- SQLite (Development database)

**Frontend:**
- HTML5
- CSS3
- Jinja2 (Templating)
- Bootstrap (Optional, for UI components)

### Application Flow

```
User Request → Flask Router → View Function → Database Query/Update
                    ↓
            Template Rendering
                    ↓
            HTML Response
```

---

## Application Structure

### Directory Layout

```
market/
|__ __init__.py             # Initialize whole application 
│
├── run.py                  # Application entry point and configuration
├── models.py               # Database models (User, Item)
├── forms.py                # WTForms for input validation
├── routes.py               # URL routes and view functions (optional)
│
├── templates/              # Jinja2 HTML templates
│   ├── base.html          # Base template with common elements
│   ├── index.html          # Landing/index page
│   ├── register.html      # User registration page
│   ├── login.html         # User login page
│   └── market.html        # Marketplace page
│
|               
│
│
├── instance/              # Instance-specific files
│   └── market.db          # SQLite database file
│
├── requirements.txt       # Python dependencies
├── config.py             # Configuration settings (optional)
└── README.md             # Project overview
```

### File Descriptions

**app.py**
- Main application file
- Initializes Flask app and extensions
- Contains route definitions
- Configures database connection
- Handles application startup

**models.py**
- Defines database models using SQLAlchemy
- Contains User and Item models
- Defines relationships between tables
- Includes model methods for business logic

**forms.py**
- Contains WTForm classes for user input
- Defines registration, login, and market forms
- Includes validation rules
- Handles CSRF protection

---

## Database Design

### Schema Overview

The application uses SQLite with two primary tables:

#### Users Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | Integer | Primary Key, Auto-increment | Unique user identifier |
| username | String(30) | Unique, Not Null | User's display name |
| email | String(120) | Unique, Not Null | User's email address |
| password_hash | String(60) | Not Null | Bcrypt hashed password |
| budget | Integer | Default: 1000 | User's available funds |

#### Items Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | Integer | Primary Key, Auto-increment | Unique item identifier |
| name | String(50) | Unique, Not Null | Item name |
| price | Integer | Not Null | Item price in currency units |
| barcode | String(12) | Unique, Not Null | Item barcode/SKU |
| description | String(1024) | Not Null | Item description |
| owner_id | Integer | Foreign Key (users.id) | Current owner of the item |

### Relationships

- **One-to-Many:** User to Items
  - One user can own multiple items
  - Each item belongs to one user
  - Relationship field: `owner_id` in Items table

---

## API Endpoints

### Endpoint Summary

| Endpoint | Method | Authentication | Description |
|----------|--------|----------------|-------------|
| `/` | GET | No | Home page |
| `/home` | GET | No | Home page (alternative route) |
| `/register` | GET, POST | No | User registration |
| `/login` | GET, POST | No | User login |
| `/logout` | GET | Yes | User logout |
| `/market` | GET, POST | Yes | Marketplace page |

### Request/Response Examples

#### Registration Request (POST)

```http
POST /register HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=johndoe&email=john@example.com&password=securepass123&confirm_password=securepass123
```

**Success Response:**
- Status: 302 (Redirect to /market)
- Flash message: "Account created successfully!"

**Error Response:**
- Status: 200 (Form re-rendered with errors)
- Flash message: Error details

#### Login Request (POST)

```http
POST /login HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=johndoe&password=securepass123
```

**Success Response:**
- Status: 302 (Redirect to /market)
- Session cookie set
- Flash message: "Login successful!"

**Error Response:**
- Status: 200 (Form re-rendered)
- Flash message: "Login failed. Check username and password."

#### Purchase Item (POST)

```http
POST /market HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Cookie: session=...

purchase_item=1
```

**Success Response:**
- Status: 302 (Redirect to /market)
- Database updated
- Flash message: "Purchased {item_name}!"

---

## Authentication System

### Password Security

- **Hashing Algorithm:** Bcrypt
- **Salt Rounds:** Default (automatically handled by Flask-Bcrypt)
- **Storage:** Passwords never stored in plain text

### Session Management

- **Session Type:** Flask server-side sessions
- **Storage:** Encrypted session cookie
- **Expiration:** Browser session (default)
- **Security:** CSRF protection enabled with Flask-WTF

### Login Flow

1. User submits credentials
2. System queries database for username
3. Password hash comparison using bcrypt
4. Session created on successful authentication
5. User redirected to authenticated area

### Authorization

- **Protected Routes:** Use `@login_required` decorator
- **User Context:** Available via `current_user` proxy
- **Access Control:** Owner-based permissions for item operations

---

## Configuration

### Environment Variables

For production deployment, use environment variables for sensitive data:

```python
import os

app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY') or 'dev-secret-key'
app.config['SQLALCHEMY_DATABASE_URI'] = os.environ.get('DATABASE_URL') or 'sqlite:///market.db'
```

### Configuration Options

```python
class Config:
    SECRET_KEY = 'your-secret-key'
    SQLALCHEMY_DATABASE_URI = 'sqlite:///market.db'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    WTF_CSRF_ENABLED = True
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'
```

---

## Deployment

### Development Server

```bash
python app.py
```

Access at: http://127.0.0.1:5000

### Production Considerations

**Do NOT use the development server in production.**

Recommended production setup:
- **WSGI Server:** Gunicorn or uWSGI
- **Web Server:** Nginx (reverse proxy)
- **Database:** PostgreSQL or MySQL
- **Environment:** Use environment variables for configuration
- **HTTPS:** Always use SSL/TLS in production

### Production Deployment Steps

1. Set environment variables
2. Use production database
3. Configure WSGI server
4. Set up reverse proxy
5. Enable HTTPS
6. Implement logging
7. Set up monitoring

---

## Troubleshooting

### Common Issues

**Issue:** Database not found error

**Solution:** Initialize the database using:
```python
from app import app, db
with app.app_context():
    db.create_all()
```

**Issue:** Import errors

**Solution:** Ensure all dependencies are installed:
```bash
pip install -r requirements.txt
```

**Issue:** Session errors or login not working

**Solution:** Check SECRET_KEY is set and consistent

**Issue:** Forms not validating

**Solution:** Ensure CSRF token is included in forms

### Debugging Tips

- Enable Flask debug mode for development: `app.run(debug=True)`
- Check Flask logs for error messages
- Use Flask shell for database inspection
- Verify database schema matches models

### Database Commands

**View all users:**
```python
python
>>> from app import app, db, User
>>> with app.app_context():
>>>     users = User.query.all()
>>>     for user in users:
>>>         print(user.username, user.email)
```

**Reset database:**
```python
python
>>> from app import app, db
>>> with app.app_context():
>>>     db.drop_all()
>>>     db.create_all()
```

---

## Appendix

### Dependencies (requirements.txt)

```
Flask==2.3.0
Flask-SQLAlchemy==3.0.0
Flask-Bcrypt==1.0.1
Flask-Login==0.6.2
Flask-WTF==1.1.1
WTForms==3.0.1
```

### Best Practices Applied

- Password hashing for security
- CSRF protection on forms
- SQL injection prevention via ORM
- Session-based authentication
- Proper error handling
- Flash messages for user feedback

### Further Reading

- [Flask Documentation](https://flask.palletsprojects.com/)
- [SQLAlchemy Documentation](https://docs.sqlalchemy.org/)
- [Flask-Login Documentation](https://flask-login.readthedocs.io/)
- [WTForms Documentation](https://wtforms.readthedocs.io/)

---

**Document Version:** 1.0  
**Last Updated:** November 2025  
**Maintained By:** Asad Nadeem