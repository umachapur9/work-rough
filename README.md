QUESTION DESCRIPTION
Issue: Post Creation
This is a blogging platform where users can discover, read, and share blog posts.
However, users are unable to create and publish new blog posts, despite correctly filling all required fields, which prevents the post creation functionality from working as expected.
Steps to Reproduce:
• Log in as a user with credentials:
Email: john@exampLe.com
Password: password123
• Click on "Write" in the navbar to navigate to the create post page.
BlogPlatform
m All Posts
B My Posts
Discover Stories
Explore articles from our community of writers
Q
Search posts...
Y
*040
Technology
Lifestyle
Travel
Business
Health
@ Write
John Doe A
Education
Entertainment

Technologi
© 8 min read
Getting Started with React Hooks
Discover how React Hooks revolutionize component development, simplifying state management and lifecycle methods.
Technology
© 10 min read
The Future of Web Development
Explore emerging trends shaping web development in 2025, from Al tools to WebAssembly and edge computing.
John Doe
C Oot 9, 2025
JavaScript
John Doe
B out 9 2025
Web Development
• Fill all required details (title, content, category) and optional fields (excerpt, tags).
BlogPlatiorm
@ All Posts
© My Posts
Create New Post
Share your thoughts with the world

• Fill all required details (title, content, category) and optional fields (excerpt, tags).
BlogPlatform
All Posts
© My Posts
1 Write
John Doe E
Create New Post
Share your thoughts with the world
Title *
Enter an engaging title...
Category *
Technology
Excerpt
Brief description of your post (optional)
Tags
React, JavaScript, Web Dev (comma-separated)
0/150 characters
Content
Write your post content here... (Markdown supported)

Click "Publish Post" to submit the form.
• Observe that the post creation fails with an error, preventing users from publishing their content. Rather, it should have been published successfully and shown up right away in the "My Posts" section.
Blog Platform
@ All Posts
图
My Posts
My Posts
Manage your published articles
© 5 min read
Test Blog Post
This is a test excerpt
© Published on Oct 9, 2025
E Write
John Doe G
@ Write New Post
Technology
• 8 min read
Getting Started with React Hooks
Discover how React Hooks revolutionize component development, simplifying state management and lifecycle methods.
Published on Oct 9, 2025
向
70°F
Sunny
Technology
© 10 min read
The Future of Web Development
•040
Explore emerging trends shaping web development in 2025, from Al tools to WebAssembly and edge computing.

Test Blog Post
This is a test excerpt
& Published on Oct 9, 2025
Technology
© 8 min read
Getting Started with React Hooks
Discover how React Hooks revolutionize component development, simplifying state management and lifecycle methods.
Published on Oct 9, 2025
Technology
© 10 min read
The Future of Web Development
Explore emerging trends shaping web development in 2025, from Al tools to WebAssembly and edge computing.
@ Published on Oct 9, 2025
Your task is to fix this backend issue. Refer to the README.md file for more details.
Note: The acceptance criteria for this task require that your solution pass all predefined unit tests.

READMEmd 8 x
PROJECT_FILES_INSTRUCTIONS.md
Markdown
Preview
Blog Platform(Django+React): Post Creation
Overview
This is a blogging platform where users can discover, read, and share blog posts.
However, users are unable to create and publish new blog posts, even when they correctly fill in all required fields. Your task is to fix this backend issue.
Expected API Behavior
POST /api/posts/
Purpose Creates a new blog post for the authenticated user and returns the created post.
Request Headers
x-user-id: <user _id›
Request Body
"title": "string",
"content": "string",
// Required: Post title
// Required: Full post content

Request Body
"title": "string",
"content": "string",
"excerpt": "string",
"category": "string"
"tags": ["string"),
"readTime": "number"
// Required: Post title
// Required: Full post content
// Optional: Short description
// Required: Technology or Lifestyle or so on
// Optional: Array of tag strings
// Optional: Estimated read time in minutes
Success Response (201)
"message": "Post created successfully".
"post": {
"id": "string",
/ Id
"title": "string",
// Provided title
"content": "string",
// Provided content
"excerpt": "string",
// Provided or auto-generated
"category"; "string",
// Provided category
"tags": ["string"),
// Provided tags or empty array
"readTime"; "number",
// Provided
"published": True,
// Always True
"author: {
" 1d"; "string"
// User Id
"name"; "string",
// User name
"email": "string",
"bio": "string"
PROJECT_FILES_INSTRUCTIONS.md
// Id
// Provided title
// Provided content
// Provided or auto-generated
// Provided category
// Provided tags or empty array
// Provided
// Always True
// User Id
// User name
// User email
/ User bio
"createdAt": "ISO string", // Creation timestamp
"updatedAt": "ISO string" // Update timestamp
Marki
I
Additional Information
• The post should be saved to the database as they are created.
• The published status should be set to True when a post is created.
• If an excerpt isn't provided, the first 150 characters of the content should be used automatically.
• To manually reset the database, stop the running server and then restart it.

README.md
* PROJECT_FILES_INSTRUCTIONS.md ×
Markdown
Preview


Al Assistant
Preview
• Add Context..
@ to add context
?
•ll
Save & Proceed
Run
V
Run Tests (
yes
Ask the Al Assistant
ameni
• Explain the problem statement
• Clarify errors
• Answer questions about the platform
• Provide you help with syntax
Ehtml &
Note: The Al Assistant will not provide solutions
