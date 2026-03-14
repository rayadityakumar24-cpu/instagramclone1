# Implementation Plan: Instagram Clone

## Overview

This implementation plan breaks down the Instagram Clone feature into discrete, incremental coding tasks. The system will be built using TypeScript for the backend API, PostgreSQL for the database, Redis for caching, and S3-compatible storage for media. Each task builds on previous work, with testing integrated throughout to ensure correctness and reliability.

## Tasks

- [x] 1. Set up project infrastructure and core configuration
  - Initialize TypeScript Node.js project with Express framework
  - Configure PostgreSQL database connection with connection pooling
  - Configure Redis client for caching
  - Set up environment variables for database, Redis, S3, and JWT secrets
  - Create base project structure (routes, controllers, services, models, middleware)
  - Configure TypeScript compiler options and build scripts
  - _Requirements: 14.1 (HTTPS), 14.2 (rate limiting infrastructure)_

- [x] 2. Implement database schema and migrations
  - [x] 2.1 Create database migration system
    - Set up migration tool (e.g., node-pg-migrate or Knex)
    - Create initial migration for all tables from design document
    - _Requirements: All requirements depend on data persistence_
  
  - [x] 2.2 Create Users table with constraints
    - Implement users table with username, email, password_hash, bio, profile_picture_url, is_private
    - Add indexes on username and email for fast lookups
    - Add CHECK constraints for username and email format validation
    - _Requirements: 1.1, 1.2, 2.1, 14.4_
  
  - [x] 2.3 Create Posts and Post Media tables
    - Implement posts table with user_id, caption, like_count, comment_count
    - Implement post_media table for multiple images per post
    - Add foreign key constraints and indexes
    - _Requirements: 3.1, 3.2, 3.3, 3.7_
  
  - [x] 2.4 Create social interaction tables
    - Implement follows table with composite primary key
    - Implement likes table for post likes
    - Implement comments table with like_count
    - Implement comment_likes table
    - Add appropriate indexes for query performance
    - _Requirements: 5.1-5.5, 6.1-6.7, 7.1-7.5_
  
  - [x] 2.5 Create Stories and Story Views tables
    - Implement stories table with expiration timestamp
    - Implement story_views table for view tracking
    - Add indexes on expires_at for cleanup job
    - _Requirements: 8.1-8.7_
  
  - [x] 2.6 Create Messages table
    - Implement messages table with sender_id, recipient_id, content fields
    - Add composite index for thread queries
    - Add CHECK constraints for content validation
    - _Requirements: 11.1-11.7_
  
  - [x] 2.7 Create Notifications table
    - Implement notifications table with type, actor_id, post_id, comment_id
    - Add indexes for user queries and grouping
    - _Requirements: 10.1-10.5_
  
  - [x] 2.8 Create moderation and reporting tables
    - Implement moderation_log table for admin actions
    - Implement reported_content table for user reports
    - _Requirements: 13.1-13.4_

- [x] 3. Implement Authentication Service
  - [x] 3.1 Create authentication middleware and utilities
    - Implement bcrypt password hashing with salt rounds = 12
    - Implement JWT token generation and verification with 24-hour expiration
    - Create authentication middleware to validate JWT tokens on protected routes
    - _Requirements: 1.4, 1.5_
  
  - [x] 3.2 Implement user registration endpoint
    - Create POST /api/auth/register endpoint
    - Validate email format using RFC 5322 regex
    - Validate password strength (min 8 chars, uppercase, lowercase, number)
    - Hash password before storing in database
    - Return JWT token and user object on success
    - _Requirements: 1.1, 1.2, 1.3, 1.4_
  
  - [ ]* 3.3 Write unit tests for registration validation
    - Test email format validation with valid and invalid emails
    - Test password strength validation with weak and strong passwords
    - Test duplicate username/email handling
    - _Requirements: 1.2_
  
  - [x] 3.4 Implement user login endpoint
    - Create POST /api/auth/login endpoint
    - Validate credentials against database
    - Return generic error message for invalid credentials (don't reveal which field)
    - Return JWT token and user object on success
    - _Requirements: 1.5, 1.6_
  
  - [x] 3.5 Implement password reset flow
    - Create POST /api/auth/request-reset endpoint to send reset email
    - Create POST /api/auth/reset-password endpoint with token validation
    - Generate secure reset tokens with expiration
    - _Requirements: 1.7_
  
  - [x] 3.6 Implement rate limiting middleware
    - Create rate limiting middleware using token bucket algorithm
    - Set limit to 100 requests per minute per user
    - Store rate limit counters in Redis with 60-second TTL
    - Return 429 error when limit exceeded
    - _Requirements: 14.2, 14.3_

- [x] 4. Checkpoint - Ensure authentication tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [x] 5. Implement User Management Service
  - [x] 5.1 Create user profile endpoints
    - Create GET /api/users/:userId endpoint to retrieve profile
    - Create PUT /api/users/:userId endpoint to update bio and privacy settings
    - Include post count, follower count, following count in response
    - Validate bio length (max 150 characters)
    - _Requirements: 2.1, 2.3, 2.4, 14.4_
  
  - [x] 5.2 Implement profile picture upload
    - Create POST /api/users/:userId/profile-picture endpoint
    - Validate image format (JPEG, PNG) and size (max 5MB)
    - Integrate with media processing service for upload
    - Update user profile_picture_url in database
    - _Requirements: 2.2_
  
  - [x] 5.3 Implement follow/unfollow functionality
    - Create POST /api/users/:userId/follow endpoint
    - Create DELETE /api/users/:userId/follow endpoint
    - Insert/delete record in follows table
    - Update follower/following counts
    - Trigger notification on follow action
    - _Requirements: 7.1, 7.2, 7.3, 7.4_
  
  - [x] 5.4 Implement followers and following list endpoints
    - Create GET /api/users/:userId/followers endpoint
    - Create GET /api/users/:userId/following endpoint
    - Return paginated list of users
    - _Requirements: 7.5_
  
  - [x] 5.5 Implement data export and account deletion
    - Create GET /api/users/:userId/export endpoint to download all user data as JSON
    - Create DELETE /api/users/:userId endpoint to permanently delete account
    - Cascade delete all user posts, comments, likes, follows, messages
    - _Requirements: 14.6, 14.7_

- [ ] 6. Implement Media Processing Service
  - [x] 6.1 Set up S3-compatible storage client
    - Configure AWS SDK or compatible library for object storage
    - Create helper functions for upload, download, delete operations
    - Generate signed URLs for secure media access
    - _Requirements: 3.1, 3.2, 12.1-12.6_
  
  - [x] 6.2 Implement media validation
    - Create validation function for photo formats (JPEG, PNG) and size (max 10MB)
    - Create validation function for video format (MP4), size (max 100MB), duration (max 60s)
    - Return detailed error messages for validation failures
    - _Requirements: 3.1, 3.2_
  
  - [x] 6.3 Implement image processing
    - Use Sharp library to generate thumbnail (150x150 cropped square)
    - Generate display version with max dimension 1080px, preserve aspect ratio
    - Compress images to JPEG quality 85%
    - Upload all versions to S3 and return URLs
    - _Requirements: 12.1, 12.2, 12.4, 12.5_
  
  - [x] 6.4 Implement video processing
    - Use FFmpeg to extract first frame as thumbnail
    - Generate thumbnail at 150x150 pixels
    - Compress video if needed to reduce file size
    - Upload video and thumbnail to S3
    - _Requirements: 12.3, 12.4_
  
  - [x] 6.5 Create async processing queue
    - Set up Bull queue with Redis backend
    - Create job processor for media processing tasks
    - Implement retry logic for failed processing
    - Ensure processing completes within 10 seconds for valid files
    - _Requirements: 12.6_
  
  - [ ]* 6.6 Write unit tests for media validation
    - Test file size limits for photos and videos
    - Test format validation for supported and unsupported formats
    - Test video duration validation
    - _Requirements: 3.1, 3.2_

- [ ] 7. Implement Post Management Service
  - [x] 7.1 Create post creation endpoint
    - Create POST /api/posts endpoint accepting multiple media files (up to 10)
    - Validate caption length (max 2200 characters)
    - Trigger media processing for each uploaded file
    - Create post record and post_media records in database
    - Invalidate feed cache for all followers
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.7_
  
  - [x] 7.2 Create post retrieval endpoints
    - Create GET /api/posts/:postId endpoint for single post
    - Create GET /api/users/:userId/posts endpoint for user's posts with pagination
    - Include like count, comment count, and isLikedByCurrentUser flag
    - Order posts by created_at descending
    - _Requirements: 2.5, 3.5_
  
  - [x] 7.3 Implement post deletion
    - Create DELETE /api/posts/:postId endpoint
    - Verify requesting user owns the post
    - Cascade delete post_media, likes, comments
    - Delete media files from S3
    - _Requirements: 13.2_
  
  - [x] 7.4 Implement like/unlike functionality
    - Create POST /api/posts/:postId/like endpoint
    - Create DELETE /api/posts/:postId/like endpoint
    - Insert/delete record in likes table
    - Update like_count on posts table atomically
    - Ensure operations complete within 500ms
    - Trigger notification on like action
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_
  
  - [ ]* 7.5 Write unit tests for like operations
    - Test like count increments correctly
    - Test unlike decrements correctly
    - Test double-like doesn't increment twice
    - Test concurrent like operations maintain consistency
    - _Requirements: 5.1, 5.2_
  
  - [x] 7.6 Implement comment functionality
    - Create POST /api/posts/:postId/comments endpoint
    - Validate comment text length (max 500 characters)
    - Create comment record and increment post comment_count
    - Ensure comment appears within 2 seconds
    - Trigger notification on comment action
    - _Requirements: 6.1, 6.2, 6.3, 6.5, 6.6_
  
  - [x] 7.7 Create comment retrieval endpoint
    - Create GET /api/posts/:postId/comments endpoint
    - Return comments in chronological order
    - Include author username and timestamp
    - _Requirements: 6.4, 6.5_
  
  - [x] 7.8 Implement comment like functionality
    - Create POST /api/comments/:commentId/like endpoint
    - Create DELETE /api/comments/:commentId/like endpoint
    - Update like_count on comments table
    - _Requirements: 6.7_

- [x] 8. Checkpoint - Ensure post management tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 9. Implement Feed Generation Service
  - [x] 9.1 Create personalized feed endpoint
    - Create GET /api/feed endpoint
    - Query posts from users that current user follows
    - Order by created_at descending
    - Implement cursor-based pagination with limit of 20 posts per page
    - Cache feed in Redis with 5-minute TTL
    - _Requirements: 4.1, 4.2, 4.3, 4.4_
  
  - [x] 9.2 Implement explore feed endpoint
    - Create GET /api/explore endpoint
    - Query posts with high engagement (likes + comments) from last 24 hours
    - Exclude posts from users current user already follows
    - Implement pagination with limit of 20 posts
    - Cache explore feed with 30-minute TTL
    - _Requirements: 4.5, 9.3, 9.4_
  
  - [x] 9.3 Implement feed cache invalidation
    - Create helper function to invalidate user feed cache
    - Call on new post creation to invalidate followers' feeds
    - Call on follow/unfollow to invalidate user's feed
    - _Requirements: 3.6_
  
  - [ ]* 9.4 Write integration tests for feed generation
    - Test feed shows posts from followed users only
    - Test feed ordering by timestamp
    - Test pagination returns correct posts
    - Test explore feed excludes followed users
    - _Requirements: 4.1, 4.2, 4.3, 4.5_

- [ ] 10. Implement Story Service
  - [x] 10.1 Create story creation endpoint
    - Create POST /api/stories endpoint
    - Accept single photo or video with same size limits as posts
    - Set expires_at to 24 hours from creation
    - Process media and create story record
    - Add story_id to active_stories sorted set in Redis
    - _Requirements: 8.1, 8.2_
  
  - [x] 10.2 Create story retrieval endpoints
    - Create GET /api/stories endpoint to get active stories from followed users
    - Create GET /api/stories/:storyId endpoint for single story
    - Filter stories where expires_at > current time
    - Return stories with view count for creator
    - _Requirements: 8.2, 8.4, 8.7_
  
  - [x] 10.3 Implement story view tracking
    - Create POST /api/stories/:storyId/view endpoint
    - Insert record in story_views table
    - Increment view_count on stories table
    - _Requirements: 8.7_
  
  - [x] 10.4 Create story viewers endpoint
    - Create GET /api/stories/:storyId/viewers endpoint
    - Return list of users who viewed the story
    - Only allow story creator to access this endpoint
    - _Requirements: 8.7_
  
  - [x] 10.5 Implement story expiration cleanup job
    - Create cron job that runs every hour
    - Delete stories where expires_at < current time
    - Delete associated story_views records
    - Delete media files from S3
    - Remove from active_stories Redis set
    - _Requirements: 8.3_

- [ ] 11. Implement Messaging Service
  - [x] 11.1 Create message sending endpoint
    - Create POST /api/messages endpoint
    - Accept text messages (max 1000 chars) or media (photo/video)
    - Validate content based on type
    - Create message record with status 'sent'
    - Ensure delivery within 2 seconds
    - Increment unread count in Redis for recipient
    - _Requirements: 11.1, 11.2, 11.3_
  
  - [x] 11.2 Create message thread retrieval endpoint
    - Create GET /api/messages/thread/:userId endpoint
    - Query messages between current user and specified user
    - Order by created_at descending
    - Implement pagination
    - _Requirements: 11.4_
  
  - [x] 11.3 Create threads list endpoint
    - Create GET /api/messages/threads endpoint
    - Group messages by conversation partner
    - Show last message and unread count for each thread
    - Order by last message timestamp descending
    - _Requirements: 11.4, 11.5_
  
  - [x] 11.4 Implement message read receipts
    - Create POST /api/messages/:messageId/read endpoint
    - Update message status to 'read' and set read_at timestamp
    - Decrement unread count in Redis
    - _Requirements: 11.6_
  
  - [x] 11.5 Create unread count endpoint
    - Create GET /api/messages/unread-count endpoint
    - Return total unread message count from Redis
    - _Requirements: 11.5_

- [ ] 12. Implement Notification Service
  - [x] 12.1 Create notification creation helper
    - Create helper function to create notifications
    - Support types: like, comment, follow, message
    - Store actor_id, post_id, comment_id as appropriate
    - Ensure notifications created within 5 seconds of action
    - Increment unread count in Redis
    - _Requirements: 10.1, 5.5, 6.6, 7.4_
  
  - [x] 12.2 Implement notification grouping logic
    - Group multiple likes on same post within 1 hour into single notification
    - Store multiple actor_ids for grouped notifications
    - _Requirements: 10.5_
  
  - [x] 12.3 Create notification retrieval endpoint
    - Create GET /api/notifications endpoint
    - Return notifications in reverse chronological order
    - Include actor information and related post/comment data
    - Implement pagination
    - _Requirements: 10.3_
  
  - [x] 12.4 Create unread count endpoint
    - Create GET /api/notifications/unread-count endpoint
    - Return unread notification count from Redis
    - _Requirements: 10.2_
  
  - [x] 12.5 Implement mark as read functionality
    - Create POST /api/notifications/:notificationId/read endpoint
    - Create POST /api/notifications/read-all endpoint
    - Update is_read flag in database
    - Decrement unread count in Redis
    - _Requirements: 10.4_

- [ ] 13. Implement WebSocket Service for Real-time Features
  - [x] 13.1 Set up WebSocket server
    - Configure Socket.io or ws library
    - Implement authentication for WebSocket connections using JWT
    - Maintain map of user_id to socket connections
    - _Requirements: 11.7, 10.1_
  
  - [x] 13.2 Implement real-time message delivery
    - Emit 'new_message' event to recipient when message sent
    - Update message status to 'delivered' when recipient connected
    - _Requirements: 11.1, 11.6, 11.7_
  
  - [x] 13.3 Implement real-time notification delivery
    - Emit 'new_notification' event when notification created
    - Push notification to connected users instantly
    - _Requirements: 10.1_
  
  - [x] 13.4 Implement story update notifications
    - Emit 'new_story' event to followers when story created
    - Update active stories list in real-time
    - _Requirements: 8.4_

- [ ] 14. Implement Search Service
  - [x] 14.1 Create user search endpoint
    - Create GET /api/search/users endpoint with query parameter
    - Use PostgreSQL ILIKE for case-insensitive partial matching
    - Rank results: exact matches first, then prefix matches, then contains
    - Limit results to 50 users
    - Ensure results returned within 1 second
    - _Requirements: 9.1, 9.2, 9.5_
  
  - [ ]* 14.2 Write unit tests for search ranking
    - Test exact match appears first
    - Test prefix match appears before contains match
    - Test case-insensitive matching
    - _Requirements: 9.2_

- [ ] 15. Implement Content Moderation Features
  - [x] 15.1 Create content reporting endpoint
    - Create POST /api/reports endpoint
    - Accept content_type (post, comment, user) and content_id
    - Create record in reported_content table with status 'pending'
    - _Requirements: 13.1_
  
  - [x] 15.2 Implement admin post/comment deletion
    - Add admin role check middleware
    - Allow admins to delete any post or comment via existing endpoints
    - Log all deletions in moderation_log table
    - _Requirements: 13.2, 13.4_
  
  - [x] 15.3 Implement admin user suspension
    - Create POST /api/admin/users/:userId/suspend endpoint
    - Add is_suspended flag to users table
    - Block suspended users from authentication
    - Log suspension in moderation_log table
    - _Requirements: 13.3, 13.4_

- [x] 16. Implement Privacy Controls
  - [x] 16.1 Add privacy checks to feed generation
    - Filter out posts from private accounts unless user is approved follower
    - Modify feed query to check is_private flag and follow relationship
    - _Requirements: 14.5_
  
  - [x] 16.2 Add privacy checks to profile viewing
    - Hide post list on private profiles for non-followers
    - Show follower/following counts but not lists for private profiles
    - _Requirements: 14.5_

- [ ] 17. Add HTTPS and security configurations
  - [~] 17.1 Configure HTTPS
    - Set up SSL/TLS certificates
    - Configure Express to use HTTPS
    - Redirect HTTP requests to HTTPS
    - _Requirements: 14.1_
  
  - [~] 17.2 Add security headers
    - Configure helmet middleware for security headers
    - Set up CORS with appropriate origins
    - Add CSRF protection for state-changing operations
    - _Requirements: 14.1_

- [ ] 18. Final integration and testing
  - [~] 18.1 Wire all services together
    - Ensure all endpoints are registered in main router
    - Verify authentication middleware applied to protected routes
    - Verify rate limiting applied to all routes
    - Test end-to-end flows for major features
    - _Requirements: All requirements_
  
  - [ ]* 18.2 Write integration tests for critical flows
    - Test user registration → login → create post → like → comment flow
    - Test follow user → see posts in feed flow
    - Test create story → view story → story expires flow
    - Test send message → receive notification → read message flow
    - _Requirements: All requirements_
  
  - [~] 18.3 Performance testing and optimization
    - Test feed generation performance with large datasets
    - Verify all time-based requirements are met (response times)
    - Optimize database queries with EXPLAIN ANALYZE
    - Add additional indexes if needed
    - _Requirements: 1.3, 1.5, 2.4, 3.4, 4.3, 5.1, 5.2, 6.3, 7.1, 9.1, 11.1_

- [~] 19. Final checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP delivery
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation throughout development
- The implementation uses TypeScript, PostgreSQL, Redis, and S3-compatible storage as specified in the design
- All time-based performance requirements should be validated during implementation
- Security and privacy controls are integrated throughout rather than added at the end
