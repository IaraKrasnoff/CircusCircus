# User Model Structure Documentation

This document describes the enhanced user model structure for the CircusCircus forum application.

## Overview

The user model has been significantly enhanced to support modern forum features including user profiles, settings, direct messaging, and public/private posts functionality.

## Database Models

### User Model (`forum/models.py`)

The `User` model has been enhanced with the following fields:

#### Core User Fields
- `id` (Integer, Primary Key)
- `username` (Text, Unique, Required)
- `password_hash` (Text, Required) 
- `email` (Text, Unique, Required)
- `admin` (Boolean, Default: False)

#### Profile Fields
- `first_name` (Text, Optional)
- `last_name` (Text, Optional)
- `bio` (Text, Optional, Max 500 chars)
- `profile_picture_url` (Text, Optional)

#### User Settings
- `show_email_publicly` (Boolean, Default: False)
- `receive_notifications` (Boolean, Default: True)
- `theme_preference` (Text, Default: 'light') - Options: 'light', 'dark'

#### Account Management
- `is_active` (Boolean, Default: True)
- `created_at` (DateTime, Auto-set)
- `last_seen` (DateTime, Auto-updated)
- `email_verified` (Boolean, Default: False)

#### Relationships
- `posts` - One-to-many with Post model
- `comments` - One-to-many with Comment model
- `sent_messages` - One-to-many with DirectMessage (as sender)
- `received_messages` - One-to-many with DirectMessage (as recipient)

### DirectMessage Model (`forum/models.py`)

New model for user-to-user messaging:

- `id` (Integer, Primary Key)
- `sender_id` (Integer, Foreign Key to User)
- `recipient_id` (Integer, Foreign Key to User)
- `subject` (Text, Required, Max 200 chars)
- `content` (Text, Required, 10-5000 chars)
- `timestamp` (DateTime, Auto-set)
- `is_read` (Boolean, Default: False)

### Enhanced Post Model

The Post model has been updated to support public/private posts:

- Added `is_public` (Boolean, Default: True)
- Updated constructor to accept `is_public` parameter

## User Management Modules

### `forum/user.py` - Core User Utilities

Enhanced with additional validation functions:

- `valid_email(email)` - Email format validation
- `valid_name(name)` - First/last name validation 
- `valid_bio(bio)` - Bio length validation
- `valid_profile_picture_url(url)` - Profile picture URL validation
- `create_user()` - Enhanced user creation
- `send_direct_message()` - Direct messaging functionality
- `get_user_conversations()` - Conversation management
- `mark_messages_as_read()` - Message status management

### `forum/user_settings.py` - User Settings Management

New module providing:

#### UserSettingsManager Class
- `update_profile()` - Update profile information
- `update_settings()` - Update privacy/preference settings
- `update_email()` - Email change with validation
- `change_password()` - Secure password changes
- `deactivate_account()` - Account deactivation
- `reactivate_account()` - Account reactivation (admin)

#### UserStatsManager Class
- `get_user_stats()` - Comprehensive user statistics
- `get_recent_activity()` - Recent posts and comments

### `forum/messaging.py` - Direct Messaging System

New module providing:

#### MessageManager Class
- `send_message()` - Send direct messages
- `get_inbox()` - Retrieve received messages
- `get_sent_messages()` - Retrieve sent messages  
- `get_conversation_with()` - Get conversation history
- `mark_conversation_as_read()` - Mark messages as read
- `delete_message()` - Message deletion
- `get_recent_conversations()` - Recent conversation partners

## User Model Methods

### Profile Methods
- `get_full_name()` - Return formatted full name or username
- `update_last_seen()` - Update last activity timestamp
- `get_post_count()` - Get number of posts by user
- `get_comment_count()` - Get number of comments by user

### Messaging Methods
- `can_send_message_to(user)` - Check messaging permissions
- `get_unread_message_count()` - Get count of unread messages

## Features Implemented

### ✅ User Profiles
- Extended user information (names, bio, profile picture)
- User statistics and activity tracking
- Profile management and updates

### ✅ User Settings
- Privacy settings (email visibility)
- Notification preferences  
- Theme preferences (light/dark)
- Account management (activation/deactivation)

### ✅ Direct Messaging
- User-to-user private messaging
- Message threading and conversations
- Read/unread status tracking
- Message search and management

### ✅ Public/Private Posts
- Posts can be marked as public or private
- Private posts visible only to logged-in users
- Public posts visible to all visitors

### ✅ Enhanced Validation
- Email format validation
- Strong password requirements
- Profile field validation
- Message content validation

## Security Features

- Password hashing using Werkzeug
- Input validation and sanitization
- SQL injection prevention through SQLAlchemy ORM
- User permission checks for messaging and profile access
- Account status management (active/inactive)

## Database Migration Notes

When deploying these changes, the following database migrations may be needed:

```sql
-- Add new columns to User table
ALTER TABLE user ADD COLUMN first_name TEXT;
ALTER TABLE user ADD COLUMN last_name TEXT;
ALTER TABLE user ADD COLUMN bio TEXT;
ALTER TABLE user ADD COLUMN profile_picture_url TEXT;
ALTER TABLE user ADD COLUMN show_email_publicly BOOLEAN DEFAULT FALSE;
ALTER TABLE user ADD COLUMN receive_notifications BOOLEAN DEFAULT TRUE;
ALTER TABLE user ADD COLUMN theme_preference TEXT DEFAULT 'light';
ALTER TABLE user ADD COLUMN is_active BOOLEAN DEFAULT TRUE;
ALTER TABLE user ADD COLUMN created_at DATETIME;
ALTER TABLE user ADD COLUMN last_seen DATETIME;
ALTER TABLE user ADD COLUMN email_verified BOOLEAN DEFAULT FALSE;

-- Add is_public column to Post table
ALTER TABLE post ADD COLUMN is_public BOOLEAN DEFAULT TRUE;

-- Create DirectMessage table
CREATE TABLE direct_message (
    id INTEGER PRIMARY KEY,
    sender_id INTEGER NOT NULL,
    recipient_id INTEGER NOT NULL,
    subject TEXT NOT NULL,
    content TEXT NOT NULL,
    timestamp DATETIME NOT NULL,
    is_read BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (sender_id) REFERENCES user (id),
    FOREIGN KEY (recipient_id) REFERENCES user (id)
);
```

## Usage Examples

### Creating a User with Profile Information
```python
from forum.user import create_user
from forum.models import db

user = create_user(
    email="user@example.com",
    username="johndoe",
    password="secure123",
    first_name="John",
    last_name="Doe",
    bio="Software developer and forum enthusiast"
)
db.session.add(user)
db.session.commit()
```

### Managing User Settings
```python
from forum.user_settings import get_user_settings_manager

settings_manager = get_user_settings_manager(user_id)
success, message = settings_manager.update_settings(
    show_email_publicly=True,
    theme_preference='dark'
)
```

### Sending Direct Messages
```python
from forum.messaging import get_message_manager

msg_manager = get_message_manager(sender_id)
success, result = msg_manager.send_message(
    recipient_username="johndoe",
    subject="Welcome!",
    content="Welcome to the CircusCircus forum!"
)
```

## Testing

The user model structure has been designed with testability in mind. Key areas to test:

1. User registration and validation
2. Profile updates and settings changes
3. Direct messaging functionality
4. Public/private post visibility
5. User authentication and authorization
6. Database relationship integrity

## Future Enhancements

Potential future enhancements to consider:

- Email verification system
- Password reset functionality
- Two-factor authentication
- User role management beyond admin/user
- Message attachments
- User blocking/reporting system
- Social features (friends, followers)
- User badges/achievements
