# RyuuStream Database Setup Guide

This guide contains all the necessary SQL queries to set up and maintain your anime streaming platform's database.

## Table Creation Queries

### Users Table
```sql
CREATE TABLE IF NOT EXISTS users (
  id SERIAL PRIMARY KEY,
  username TEXT NOT NULL UNIQUE,
  password TEXT NOT NULL,
  is_admin BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### Series Table
```sql
CREATE TABLE IF NOT EXISTS series (
  id SERIAL PRIMARY KEY,
  title TEXT NOT NULL,
  description TEXT NOT NULL,
  cover_image TEXT NOT NULL,
  total_episodes INTEGER NOT NULL
);
```

### Episodes Table
```sql
CREATE TABLE IF NOT EXISTS episodes (
  id SERIAL PRIMARY KEY,
  series_id INTEGER NOT NULL REFERENCES series(id) ON DELETE CASCADE,
  episode_number INTEGER NOT NULL,
  title TEXT NOT NULL,
  embed_url TEXT NOT NULL
);
```

### Notifications Table
```sql
CREATE TABLE IF NOT EXISTS notifications (
  id SERIAL PRIMARY KEY,
  title TEXT NOT NULL,
  message TEXT NOT NULL,
  series_id INTEGER REFERENCES series(id) ON DELETE SET NULL,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  is_read BOOLEAN NOT NULL DEFAULT false
);
```

## Admin User Creation
To create an admin user, first register through the application, then run:
```sql
UPDATE users
SET is_admin = true
WHERE username = 'your_username';
```

## Sample Data Queries

### Add Sample Series
```sql
INSERT INTO series (title, description, cover_image, total_episodes)
VALUES (
  'Sample Anime',
  'An exciting adventure series',
  'https://example.com/cover.jpg',
  12
);
```

### Add Sample Episodes
```sql
INSERT INTO episodes (series_id, episode_number, title, embed_url)
VALUES (
  1, -- series_id (replace with actual series id)
  1, -- episode_number
  'Episode 1: The Beginning',
  'https://example.com/embed/ep1'
);
```

### Add Sample Notification
```sql
INSERT INTO notifications (title, message, series_id)
VALUES (
  'New Series Added!',
  'Check out our latest addition to the library',
  1 -- series_id (replace with actual series id)
);
```

## Useful Queries

### Get All Series with Episode Count
```sql
SELECT 
  s.*,
  COUNT(e.id) as current_episode_count
FROM series s
LEFT JOIN episodes e ON s.id = e.series_id
GROUP BY s.id;
```

### Get Latest Episodes
```sql
SELECT 
  e.*,
  s.title as series_title
FROM episodes e
JOIN series s ON e.series_id = s.id
ORDER BY e.id DESC
LIMIT 10;
```

### Get Unread Notifications
```sql
SELECT *
FROM notifications
WHERE is_read = false
ORDER BY created_at DESC;
```

## Maintenance Queries

### Delete Old Read Notifications
```sql
DELETE FROM notifications
WHERE is_read = true
AND created_at < NOW() - INTERVAL '30 days';
```

### Update Series Episode Count
```sql
UPDATE series s
SET total_episodes = (
  SELECT COUNT(*)
  FROM episodes e
  WHERE e.series_id = s.id
);
```

## Notes
- All tables use SERIAL for auto-incrementing IDs
- Foreign keys are set up with ON DELETE CASCADE where appropriate
- Timestamps are automatically set using NOW()
- The users table password field should store hashed passwords only
- series_id in notifications can be NULL (ON DELETE SET NULL)

## Important
- Never run DELETE queries without a WHERE clause
- Always backup data before running maintenance queries
- Test queries in a development environment first
- Use prepared statements in your application code
- Keep the NOTIFICATION_API_KEY secure
