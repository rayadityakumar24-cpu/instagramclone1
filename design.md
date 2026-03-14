# Design Document: Instagram Clone

## Overview

The Instagram Clone is a full-featured social media platform that enables users to share photos and videos, build social connections through following, and engage with content through likes, comments, and direct messaging. The system is designed as a modern web application with a client-server architecture, emphasizing scalability, real-time interactions, and media processing capabilities.

### Key Design Goals

- **Scalability**: Support millions of users with efficient data storage and retrieval
- **Performance**: Fast media uploads, feed generation, and real-time notifications
- **Security**: Secure authentication, data encryption, and privacy controls
- **Reliability**: Consistent data storage and fault-tolerant media processing
- **User Experience**: Responsive interface with smooth interactions and quick load times

### Technology Stack

- **Backend**: RESTful API with WebSocket support for real-time features
- **Database**: Relational database (PostgreSQL) for structured data, NoSQL (Redis) for caching
- **Media Storage**: Object storage service (S3-compatible) for photos and videos
- **Authentication**: JWT-based authentication with bcrypt password hashing
- **Media Processing**: Asynchronous job queue for image/video processing
- **Real-time**: WebSocket connections for notifications and messaging

## Architecture

### System Architecture

The system follows a layered architecture with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Layer                          │
│  (Web Application - React/Vue with responsive design)        │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      API Gateway Layer                       │
│     (Load Balancer, Rate Limiting, Request Routing)         │
└─────────────────────────────────────────────────────────────┘
                            │
                ┌───────────┴───────────┐
                ▼                       ▼
┌──────────────────────────┐  ┌──────────────────────────┐
│   Application Services   │  │   WebSocket Service      │
│  - Authentication        │  │  - Real-time messaging   │
│  - User Management       │  │  - Live notifications    │
│  - Post Management       │  │  - Story updates         │
│  - Feed Generation       │  │                          │
│  - Social Interactions   │  │                          │
└──────────────────────────┘  └──────────────────────────┘
                │                       │
    ┌───────────┼───────────┬──────────┘
    ▼           ▼           ▼
┌─────────┐ ┌─────────┐ ┌──────────────┐
│PostgreSQL│ │  Redis  │ │Media Storage │
│ Database │ │  Cache  │ │   (S3-like)  │
└─────────┘ └─────────┘ └──────────────┘
                            │
                            ▼
                  ┌──────────────────┐
                  │ Media Processing │
                  │   Job Queue      │
                  └──────────────────┘
```

### Component Responsibilities

**Authentication Service**
- User registration and login
- Password hashing and validation
- JWT token generation and verification
- Session management
- Password reset functionality

**User Management Service**
- Profile CRUD operations
- Follow/unfollow relationships
- Privacy settings management
- Account deletion and data export

**Post Management Service**
- Post creation and deletion
- Media upload coordination
- Caption and metadata management
- Post retrieval and filtering

**Feed Generation Service**
- Personalized feed assembly
- Chronological ordering
- Pagination and infinite scroll
- Explore page curation

**Social Interaction Service**
- Like/unlike operations
- Comment creation and retrieval
- Notification triggering

**Story Service**
- Story creation and display
- 24-hour expiration management
- View tracking

**Messaging Service**
- Direct message delivery
- Thread management
- Read receipts and delivery status

**Media Processing Service**
- Image resizing and compression
- Thumbnail generation
- Video processing
- Format validation

**Notification Service**
- Notification creation and delivery
- Push notification integration
- Notification grouping and aggregation

**Search Service**
- Username search with partial matching
- Result ranking and filtering

## Components and Interfaces

### Authentication Service

**Interface: AuthenticationAPI**

```typescript
interface AuthenticationAPI {
  // User registration
  register(email: string, password: string, username: string): Promise<AuthResult>
  
  // User login
  login(email: string, password: string): Promise<AuthResult>
  
  // Token validation
  validateToken(token: string): Promise<User>
  
  // Password reset
  requestPasswordReset(email: string): Promise<void>
  resetPassword(token: string, newPassword: string): Promise<void>
  
  // Rate limiting check
  checkRateLimit(userId: string): Promise<boolean>
}

interface AuthResult {
  success: boolean
  token?: string
  user?: User
  error?: string
}
```

**Implementation Details**
- Password hashing: bcrypt with salt rounds = 12
- JWT tokens: 24-hour expiration, signed with secret key
- Email validation: RFC 5322 compliant regex
- Password strength: minimum 8 characters, must include uppercase, lowercase, and number
- Rate limiting: Token bucket algorithm, 100 requests per minute per user

### User Management Service

**Interface: UserManagementAPI**

```typescript
interface UserManagementAPI {
  // Profile operations
  getProfile(userId: string): Promise<Profile>
  updateProfile(userId: string, updates: ProfileUpdate): Promise<Profile>
  uploadProfilePicture(userId: string, image: File): Promise<string>
  
  // Follow operations
  followUser(followerId: string, followeeId: string): Promise<void>
  unfollowUser(followerId: string, followeeId: string): Promise<void>
  getFollowers(userId: string): Promise<User[]>
  getFollowing(userId: string): Promise<User[]>
  isFollowing(followerId: string, followeeId: string): Promise<boolean>
  
  // Privacy and data
  setPrivacy(userId: string, isPrivate: boolean): Promise<void>
  exportUserData(userId: string): Promise<UserDataExport>
  deleteAccount(userId: string): Promise<void>
}

interface Profile {
  userId: string
  username: string
  email: string
  bio: string
  profilePictureUrl: string
  postCount: number
  followerCount: number
  followingCount: number
  isPrivate: boolean
  createdAt: Date
}
```

### Post Management Service

**Interface: PostManagementAPI**

```typescript
interface PostManagementAPI {
  // Post operations
  createPost(userId: string, media: File[], caption: string): Promise<Post>
  getPost(postId: string): Promise<Post>
  deletePost(postId: string, userId: string): Promise<void>
  getUserPosts(userId: string, limit: number, offset: number): Promise<Post[]>
  
  // Like operations
  likePost(postId: string, userId: string): Promise<void>
  unlikePost(postId: string, userId: string): Promise<void>
  getPostLikes(postId: string): Promise<number>
  hasUserLiked(postId: string, userId: string): Promise<boolean>
  
  // Comment operations
  addComment(postId: string, userId: string, text: string): Promise<Comment>
  getComments(postId: string): Promise<Comment[]>
  likeComment(commentId: string, userId: string): Promise<void>
  unlikeComment(commentId: string, userId: string): Promise<void>
}

interface Post {
  postId: string
  userId: string
  username: string
  mediaUrls: string[]
  caption: string
  likeCount: number
  commentCount: number
  createdAt: Date
  isLikedByCurrentUser: boolean
}

interface Comment {
  commentId: string
  postId: string
  userId: string
  username: string
  text: string
  likeCount: number
  createdAt: Date
}
```

### Feed Generation Service

**Interface: FeedGenerationAPI**

```typescript
interface FeedGenerationAPI {
  // Feed retrieval
  getPersonalizedFeed(userId: string, limit: number, offset: number): Promise<Post[]>
  getExploreFeed(userId: string, limit: number, offset: number): Promise<Post[]>
  
  // Feed invalidation
  invalidateUserFeed(userId: string): Promise<void>
}
```

**Implementation Details**
- Feed caching: Redis cache with 5-minute TTL for personalized feeds
- Explore page: Aggregates posts with high engagement (likes + comments) from last 24 hours
- Pagination: Cursor-based pagination for consistent results during scrolling
- Feed assembly: Fetches posts from followed users, orders by timestamp descending

### Story Service

**Interface: StoryAPI**

```typescript
interface StoryAPI {
  // Story operations
  createStory(userId: string, media: File): Promise<Story>
  getActiveStories(userId: string): Promise<Story[]>
  getStoryViewers(storyId: string): Promise<User[]>
  markStoryViewed(storyId: string, viewerId: string): Promise<void>
  
  // Background job
  deleteExpiredStories(): Promise<void>
}

interface Story {
  storyId: string
  userId: string
  username: string
  mediaUrl: string
  mediaType: 'photo' | 'video'
  viewCount: number
  createdAt: Date
  expiresAt: Date
}
```

**Implementation Details**
- Expiration: Cron job runs every hour to delete stories older than 24 hours
- View tracking: Stores viewer IDs in separate table for creator visibility
- Story indicators: Cached list of users with active stories, updated on creation/expiration

### Messaging Service

**Interface: MessagingAPI**

```typescript
interface MessagingAPI {
  // Message operations
  sendMessage(senderId: string, recipientId: string, content: MessageContent): Promise<Message>
  getThread(userId1: string, userId2: string, limit: number, offset: number): Promise<Message[]>
  getThreads(userId: string): Promise<Thread[]>
  markAsRead(messageId: string): Promise<void>
  getUnreadCount(userId: string): Promise<number>
}

interface Message {
  messageId: string
  senderId: string
  recipientId: string
  content: MessageContent
  status: 'sent' | 'delivered' | 'read'
  createdAt: Date
}

interface MessageContent {
  type: 'text' | 'photo' | 'video'
  text?: string
  mediaUrl?: string
}

interface Thread {
  otherUser: User
  lastMessage: Message
  unreadCount: number
}
```

### Media Processing Service

**Interface: MediaProcessingAPI**

```typescript
interface MediaProcessingAPI {
  // Upload and process
  uploadAndProcess(file: File, type: 'post' | 'story' | 'profile' | 'message'): Promise<MediaResult>
  
  // Validation
  validateMedia(file: File, type: 'photo' | 'video'): Promise<ValidationResult>
}

interface MediaResult {
  originalUrl: string
  thumbnailUrl: string
  displayUrl: string
  width: number
  height: number
  processingTime: number
}

interface ValidationResult {
  valid: boolean
  error?: string
}
```

**Implementation Details**
- Image processing: Sharp library for resizing and compression
- Video processing: FFmpeg for thumbnail extraction and compression
- Storage: S3-compatible object storage with CDN integration
- Processing queue: Bull queue with Redis backend for async processing
- Thumbnail size: 150x150 pixels (cropped to square)
- Display size: Max 1080px on longest dimension (aspect ratio preserved)
- Compression: JPEG quality 85%, WebP where supported

### Notification Service

**Interface: NotificationAPI**

```typescript
interface NotificationAPI {
  // Notification creation
  createNotification(userId: string, type: NotificationType, data: NotificationData): Promise<void>
  
  // Notification retrieval
  getNotifications(userId: string, limit: number, offset: number): Promise<Notification[]>
  getUnreadCount(userId: string): Promise<number>
  markAsRead(notificationId: string): Promise<void>
  markAllAsRead(userId: string): Promise<void>
}

type NotificationType = 'like' | 'comment' | 'follow' | 'message'

interface NotificationData {
  actorId: string
  postId?: string
  commentId?: string
}

interface Notification {
  notificationId: string
  userId: string
  type: NotificationType
  actors: User[]  // Grouped actors for same action
  postId?: string
  commentId?: string
  isRead: boolean
  createdAt: Date
}
```

**Implementation Details**
- Grouping: Multiple likes on same post within 1 hour grouped into single notification
- Real-time delivery: WebSocket push for instant notification display
- Push notifications: Integration with FCM (Firebase Cloud Messaging) for mobile
- Notification retention: Keep for 30 days, then archive

### Search Service

**Interface: SearchAPI**

```typescript
interface SearchAPI {
  searchUsers(query: string, limit: number): Promise<User[]>
}
```

**Implementation Details**
- Search algorithm: PostgreSQL ILIKE for case-insensitive partial matching
- Indexing: B-tree index on username column for fast lookups
- Result ranking: Exact matches first, then prefix matches, then contains matches
- Limit: Maximum 50 results per query

## Data Models

### Database Schema

**Users Table**
```sql
CREATE TABLE users (
  user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  username VARCHAR(30) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  bio VARCHAR(150),
  profile_picture_url VARCHAR(500),
  is_private BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT username_format CHECK (username ~ '^[a-zA-Z0-9._]+$'),
  CONSTRAINT email_format CHECK (email ~ '^[^@]+@[^@]+\.[^@]+$')
);

CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_email ON users(email);
```

**Posts Table**
```sql
CREATE TABLE posts (
  post_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  caption TEXT CHECK (LENGTH(caption) <= 2200),
  like_count INTEGER DEFAULT 0,
  comment_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT like_count_positive CHECK (like_count >= 0),
  CONSTRAINT comment_count_positive CHECK (comment_count >= 0)
);

CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);
```

**Post Media Table**
```sql
CREATE TABLE post_media (
  media_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  post_id UUID NOT NULL REFERENCES posts(post_id) ON DELETE CASCADE,
  media_url VARCHAR(500) NOT NULL,
  thumbnail_url VARCHAR(500) NOT NULL,
  display_url VARCHAR(500) NOT NULL,
  media_type VARCHAR(10) NOT NULL CHECK (media_type IN ('photo', 'video')),
  width INTEGER NOT NULL,
  height INTEGER NOT NULL,
  position INTEGER NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT position_positive CHECK (position >= 0)
);

CREATE INDEX idx_post_media_post_id ON post_media(post_id, position);
```

**Follows Table**
```sql
CREATE TABLE follows (
  follower_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  followee_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  PRIMARY KEY (follower_id, followee_id),
  CONSTRAINT no_self_follow CHECK (follower_id != followee_id)
);

CREATE INDEX idx_follows_follower ON follows(follower_id);
CREATE INDEX idx_follows_followee ON follows(followee_id);
```

**Likes Table**
```sql
CREATE TABLE likes (
  post_id UUID NOT NULL REFERENCES posts(post_id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  PRIMARY KEY (post_id, user_id)
);

CREATE INDEX idx_likes_post_id ON likes(post_id);
CREATE INDEX idx_likes_user_id ON likes(user_id);
```

**Comments Table**
```sql
CREATE TABLE comments (
  comment_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  post_id UUID NOT NULL REFERENCES posts(post_id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  text VARCHAR(500) NOT NULL,
  like_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT like_count_positive CHECK (like_count >= 0),
  CONSTRAINT text_not_empty CHECK (LENGTH(TRIM(text)) > 0)
);

CREATE INDEX idx_comments_post_id ON comments(post_id, created_at);
CREATE INDEX idx_comments_user_id ON comments(user_id);
```

**Comment Likes Table**
```sql
CREATE TABLE comment_likes (
  comment_id UUID NOT NULL REFERENCES comments(comment_id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  PRIMARY KEY (comment_id, user_id)
);

CREATE INDEX idx_comment_likes_comment_id ON comment_likes(comment_id);
```

**Stories Table**
```sql
CREATE TABLE stories (
  story_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  media_url VARCHAR(500) NOT NULL,
  thumbnail_url VARCHAR(500) NOT NULL,
  media_type VARCHAR(10) NOT NULL CHECK (media_type IN ('photo', 'video')),
  view_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  expires_at TIMESTAMP NOT NULL,
  
  CONSTRAINT view_count_positive CHECK (view_count >= 0)
);

CREATE INDEX idx_stories_user_id ON stories(user_id);
CREATE INDEX idx_stories_expires_at ON stories(expires_at);
```

**Story Views Table**
```sql
CREATE TABLE story_views (
  story_id UUID NOT NULL REFERENCES stories(story_id) ON DELETE CASCADE,
  viewer_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  viewed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  PRIMARY KEY (story_id, viewer_id)
);

CREATE INDEX idx_story_views_story_id ON story_views(story_id);
```

**Messages Table**
```sql
CREATE TABLE messages (
  message_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sender_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  recipient_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  content_type VARCHAR(10) NOT NULL CHECK (content_type IN ('text', 'photo', 'video')),
  text_content TEXT CHECK (LENGTH(text_content) <= 1000),
  media_url VARCHAR(500),
  status VARCHAR(10) DEFAULT 'sent' CHECK (status IN ('sent', 'delivered', 'read')),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  read_at TIMESTAMP,
  
  CONSTRAINT content_validation CHECK (
    (content_type = 'text' AND text_content IS NOT NULL) OR
    (content_type IN ('photo', 'video') AND media_url IS NOT NULL)
  )
);

CREATE INDEX idx_messages_sender ON messages(sender_id, created_at DESC);
CREATE INDEX idx_messages_recipient ON messages(recipient_id, created_at DESC);
CREATE INDEX idx_messages_thread ON messages(
  LEAST(sender_id, recipient_id),
  GREATEST(sender_id, recipient_id),
  created_at DESC
);
```

**Notifications Table**
```sql
CREATE TABLE notifications (
  notification_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  type VARCHAR(20) NOT NULL CHECK (type IN ('like', 'comment', 'follow', 'message')),
  actor_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  post_id UUID REFERENCES posts(post_id) ON DELETE CASCADE,
  comment_id UUID REFERENCES comments(comment_id) ON DELETE CASCADE,
  is_read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT no_self_notification CHECK (user_id != actor_id)
);

CREATE INDEX idx_notifications_user_id ON notifications(user_id, created_at DESC);
CREATE INDEX idx_notifications_grouping ON notifications(user_id, type, post_id, created_at DESC);
```

**Moderation Log Table**
```sql
CREATE TABLE moderation_log (
  log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  admin_id UUID NOT NULL REFERENCES users(user_id),
  action_type VARCHAR(50) NOT NULL,
  target_type VARCHAR(20) NOT NULL CHECK (target_type IN ('post', 'comment', 'user')),
  target_id UUID NOT NULL,
  reason TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_moderation_log_admin ON moderation_log(admin_id);
CREATE INDEX idx_moderation_log_target ON moderation_log(target_type, target_id);
```

**Reported Content Table**
```sql
CREATE TABLE reported_content (
  report_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  reporter_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  content_type VARCHAR(20) NOT NULL CHECK (content_type IN ('post', 'comment', 'user')),
  content_id UUID NOT NULL,
  reason TEXT,
  status VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending', 'reviewed', 'actioned', 'dismissed')),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  reviewed_at TIMESTAMP
);

CREATE INDEX idx_reported_content_status ON reported_content(status, created_at);
```

### Redis Cache Schema

**Feed Cache**
```
Key: feed:{user_id}
Type: List
Value: [post_id1, post_id2, ...]
TTL: 300 seconds (5 minutes)
```

**Active Stories Cache**
```
Key: active_stories
Type: Sorted Set
Score: expires_at timestamp
Value: story_id
```

**Rate Limiting**
```
Key: rate_limit:{user_id}
Type: String (counter)
TTL: 60 seconds
```

**Unread Counts**
```
Key: unread_notifications:{user_id}
Type: String (counter)
TTL: None (persistent)

Key: unread_messages:{user_id}
Type: String (counter)
TTL: None (persistent)
```

