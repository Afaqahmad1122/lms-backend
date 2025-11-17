# Learning Management System (LMS) - Technical Plan

## Technology Stack

### Backend

- **Framework**: NestJS (TypeScript)
- **Database**: PostgreSQL
- **ORM**: TypeORM / Prisma
- **Authentication**: JWT (Passport.js)
- **File Storage**: AWS S3 / Local Storage
- **Email**: Nodemailer / SendGrid
- **Validation**: class-validator, class-transformer

### Frontend

- **Framework**: Next.js 14+ (App Router)
- **Styling**: Tailwind CSS
- **UI Components**: shadcn/ui
- **State Management**: React Query (TanStack Query)
- **Forms**: React Hook Form + Zod
- **HTTP Client**: Axios / Fetch
- **Video Player**: Video.js / Plyr

---

## Project Structure

```
lms/
├── backend/                 # NestJS Backend
│   ├── src/
│   │   ├── auth/           # Authentication module
│   │   ├── users/          # User management
│   │   ├── courses/        # Course management
│   │   ├── lessons/        # Lesson/Module management
│   │   ├── enrollments/    # Enrollment system
│   │   ├── quizzes/        # Quiz/Assessment system
│   │   ├── certificates/   # Certificate generation
│   │   ├── notifications/  # Notifications & announcements
│   │   ├── files/          # File upload handling
│   │   └── common/         # Shared modules (guards, decorators, etc.)
│   └── prisma/             # Database schema (if using Prisma)
│
└── frontend/               # Next.js Frontend
    ├── app/
    │   ├── (auth)/         # Auth routes (login, signup)
    │   ├── (dashboard)/    # Dashboard routes
    │   │   ├── student/    # Student dashboard
    │   │   ├── teacher/    # Teacher dashboard
    │   │   └── admin/      # Admin dashboard
    │   ├── courses/        # Course listing & details
    │   └── api/            # Next.js API routes (if needed)
    ├── components/         # React components
    │   ├── ui/             # shadcn components
    │   ├── course/         # Course-related components
    │   ├── quiz/           # Quiz components
    │   └── dashboard/      # Dashboard components
    ├── lib/                # Utilities
    │   ├── api/            # API client setup
    │   ├── hooks/          # Custom React hooks
    │   └── utils/          # Helper functions
    └── types/              # TypeScript types
```

---

## Database Schema Design

### Core Tables

#### Users

```sql
- id (UUID, Primary Key)
- email (String, Unique)
- password (String, Hashed)
- firstName (String)
- lastName (String)
- role (Enum: STUDENT, TEACHER, ADMIN)
- avatar (String, URL)
- isEmailVerified (Boolean)
- createdAt (Timestamp)
- updatedAt (Timestamp)
```

#### Courses

```sql
- id (UUID, Primary Key)
- title (String)
- description (Text)
- thumbnail (String, URL)
- price (Decimal, default 0)
- isFree (Boolean)
- categoryId (UUID, Foreign Key -> Categories)
- instructorId (UUID, Foreign Key -> Users)
- status (Enum: DRAFT, PUBLISHED, ARCHIVED)
- level (Enum: BEGINNER, INTERMEDIATE, ADVANCED)
- duration (Integer, minutes)
- createdAt (Timestamp)
- updatedAt (Timestamp)
```

#### Categories

```sql
- id (UUID, Primary Key)
- name (String, Unique)
- slug (String, Unique)
- description (Text)
- createdAt (Timestamp)
```

#### Modules (Lessons)

```sql
- id (UUID, Primary Key)
- courseId (UUID, Foreign Key -> Courses)
- title (String)
- description (Text)
- order (Integer)
- type (Enum: VIDEO, TEXT, QUIZ)
- content (Text/JSON)
- videoUrl (String, nullable)
- duration (Integer, minutes)
- createdAt (Timestamp)
- updatedAt (Timestamp)
```

#### Enrollments

```sql
- id (UUID, Primary Key)
- studentId (UUID, Foreign Key -> Users)
- courseId (UUID, Foreign Key -> Courses)
- progress (Integer, percentage, default 0)
- completedAt (Timestamp, nullable)
- enrolledAt (Timestamp)
- status (Enum: ACTIVE, COMPLETED, DROPPED)
```

#### Module Progress

```sql
- id (UUID, Primary Key)
- enrollmentId (UUID, Foreign Key -> Enrollments)
- moduleId (UUID, Foreign Key -> Modules)
- isCompleted (Boolean, default false)
- completedAt (Timestamp, nullable)
- lastAccessedAt (Timestamp)
```

#### Quizzes

```sql
- id (UUID, Primary Key)
- moduleId (UUID, Foreign Key -> Modules)
- title (String)
- description (Text)
- timeLimit (Integer, minutes, nullable)
- passingScore (Integer, percentage)
- isRandomized (Boolean)
- createdAt (Timestamp)
```

#### Questions

```sql
- id (UUID, Primary Key)
- quizId (UUID, Foreign Key -> Quizzes)
- question (Text)
- type (Enum: MCQ, TRUE_FALSE, SHORT_ANSWER)
- options (JSON) - for MCQ
- correctAnswer (Text/JSON)
- points (Integer)
- order (Integer)
```

#### Quiz Attempts

```sql
- id (UUID, Primary Key)
- studentId (UUID, Foreign Key -> Users)
- quizId (UUID, Foreign Key -> Quizzes)
- score (Integer, percentage)
- totalQuestions (Integer)
- correctAnswers (Integer)
- startedAt (Timestamp)
- submittedAt (Timestamp)
- answers (JSON) - student answers
```

#### Certificates

```sql
- id (UUID, Primary Key)
- enrollmentId (UUID, Foreign Key -> Enrollments)
- certificateNumber (String, Unique)
- issuedAt (Timestamp)
- pdfUrl (String)
- template (String, default template name)
```

#### Announcements

```sql
- id (UUID, Primary Key)
- courseId (UUID, Foreign Key -> Courses)
- authorId (UUID, Foreign Key -> Users)
- title (String)
- content (Text)
- isPinned (Boolean)
- createdAt (Timestamp)
- updatedAt (Timestamp)
```

#### Notifications

```sql
- id (UUID, Primary Key)
- userId (UUID, Foreign Key -> Users)
- type (Enum: ANNOUNCEMENT, QUIZ_REMINDER, COURSE_UPDATE)
- title (String)
- message (Text)
- isRead (Boolean, default false)
- link (String, nullable)
- createdAt (Timestamp)
```

---

## API Endpoints Structure

### Authentication (`/api/auth`)

```
POST   /auth/register          # User registration
POST   /auth/login             # User login
POST   /auth/refresh           # Refresh JWT token
POST   /auth/logout            # User logout
POST   /auth/forgot-password   # Request password reset
POST   /auth/reset-password    # Reset password
GET    /auth/me                # Get current user
```

### Users (`/api/users`)

```
GET    /users                  # List users (Admin)
GET    /users/:id              # Get user by ID
PUT    /users/:id              # Update user
DELETE /users/:id              # Delete user (Admin)
PUT    /users/:id/profile      # Update profile
PUT    /users/:id/avatar       # Update avatar
```

### Courses (`/api/courses`)

```
GET    /courses                # List all courses (with filters)
GET    /courses/:id            # Get course details
POST   /courses                # Create course (Teacher/Admin)
PUT    /courses/:id            # Update course (Owner/Admin)
DELETE /courses/:id            # Delete course (Owner/Admin)
GET    /courses/:id/modules    # Get course modules
POST   /courses/:id/enroll     # Enroll in course (Student)
GET    /courses/my-courses     # Get user's courses (by role)
```

### Modules/Lessons (`/api/modules`)

```
GET    /modules/:id            # Get module details
POST   /modules                # Create module (Teacher/Admin)
PUT    /modules/:id            # Update module
DELETE /modules/:id            # Delete module
PUT    /modules/:id/complete   # Mark module as complete
```

### Enrollments (`/api/enrollments`)

```
GET    /enrollments            # Get user enrollments
GET    /enrollments/:id        # Get enrollment details
GET    /enrollments/:id/progress # Get enrollment progress
PUT    /enrollments/:id/progress # Update progress
```

### Quizzes (`/api/quizzes`)

```
GET    /quizzes/:id            # Get quiz details
POST   /quizzes                # Create quiz (Teacher/Admin)
PUT    /quizzes/:id            # Update quiz
DELETE /quizzes/:id            # Delete quiz
POST   /quizzes/:id/attempt    # Submit quiz attempt
GET    /quizzes/:id/results    # Get quiz results
GET    /quizzes/attempts/:id   # Get attempt details
```

### Certificates (`/api/certificates`)

```
GET    /certificates           # Get user certificates
GET    /certificates/:id       # Get certificate details
GET    /certificates/:id/download # Download certificate PDF
POST   /certificates/generate  # Generate certificate (on completion)
```

### Announcements (`/api/announcements`)

```
GET    /announcements          # Get announcements (by course)
POST   /announcements          # Create announcement (Teacher/Admin)
PUT    /announcements/:id      # Update announcement
DELETE /announcements/:id      # Delete announcement
```

### Notifications (`/api/notifications`)

```
GET    /notifications          # Get user notifications
PUT    /notifications/:id/read # Mark as read
PUT    /notifications/read-all # Mark all as read
DELETE /notifications/:id      # Delete notification
```

### Files (`/api/files`)

```
POST   /files/upload           # Upload file (course material, video, etc.)
DELETE /files/:id              # Delete file
```

---

## Frontend Components Structure

### Pages/Routes

#### Authentication

- `/login` - Login page
- `/register` - Registration page
- `/forgot-password` - Password reset request

#### Student Dashboard (`/student`)

- `/student/dashboard` - Overview, enrolled courses, progress
- `/student/courses` - All enrolled courses
- `/student/courses/[id]` - Course player with modules
- `/student/certificates` - Earned certificates
- `/student/profile` - Profile management

#### Teacher Dashboard (`/teacher`)

- `/teacher/dashboard` - Overview, courses created, students
- `/teacher/courses` - Manage courses
- `/teacher/courses/[id]` - Course editor
- `/teacher/courses/[id]/students` - Enrolled students
- `/teacher/analytics` - Course analytics

#### Admin Dashboard (`/admin`)

- `/admin/dashboard` - System overview, analytics
- `/admin/users` - User management
- `/admin/courses` - All courses management
- `/admin/categories` - Category management
- `/admin/settings` - System settings

#### Public Pages

- `/` - Homepage with featured courses
- `/courses` - Course catalog with filters
- `/courses/[id]` - Course details page
- `/categories/[slug]` - Category page

### Key Components

#### Course Components

- `CourseCard` - Course preview card
- `CoursePlayer` - Video/text lesson player
- `CourseSidebar` - Module navigation
- `ProgressBar` - Course progress indicator
- `EnrollButton` - Enrollment CTA

#### Quiz Components

- `QuizContainer` - Quiz wrapper
- `QuestionCard` - Individual question display
- `QuizTimer` - Time limit countdown
- `QuizResults` - Results display
- `AnswerOptions` - MCQ/True-False options

#### Dashboard Components

- `StatsCard` - Dashboard statistics
- `CourseProgressChart` - Progress visualization
- `RecentActivity` - Activity feed
- `NotificationsDropdown` - Notification bell

#### UI Components (shadcn)

- Button, Input, Card, Dialog, Dropdown, etc.

---

## Feature Implementation Details

### 1. User Authentication & Roles

**Backend (NestJS)**

- JWT strategy with Passport
- Role-based guards (RBAC)
- Password hashing with bcrypt
- Email verification flow

**Frontend (Next.js)**

- Protected routes with middleware
- Role-based route access
- Auth context/provider
- Login/signup forms with validation

**API Integration**

- React Query mutations for login/register
- Token storage in httpOnly cookies or localStorage
- Auto token refresh

---

### 2. Course Management

**Backend**

- CRUD operations for courses
- File upload for thumbnails/materials
- Category/tag system
- Course search & filtering

**Frontend**

- Course creation/editing form
- Rich text editor for descriptions
- File upload component
- Course listing with filters (category, price, level)

**Database Relations**

- Course → Category (Many-to-One)
- Course → User/Instructor (Many-to-One)
- Course → Modules (One-to-Many)

---

### 3. Lesson/Module Management

**Backend**

- Module CRUD with ordering
- Video upload/streaming support
- Text content storage
- Progress tracking endpoints

**Frontend**

- Module editor (drag & drop ordering)
- Video player integration
- Text lesson viewer
- Progress tracking UI

**Features**

- Sequential module unlocking
- Module completion tracking
- Last accessed position

---

### 4. Enrollment System

**Backend**

- Enrollment creation (free/paid)
- Payment integration (Stripe/PayPal) - future
- Enrollment validation
- Progress calculation

**Frontend**

- Enrollment button/flow
- Enrollment confirmation
- Enrollment history page

**Business Logic**

- Prevent duplicate enrollments
- Check course availability
- Handle free vs paid courses

---

### 5. Dashboard (Role-Based)

**Student Dashboard**

- Enrolled courses grid
- Progress overview
- Upcoming deadlines
- Recent certificates
- Quick stats (courses completed, in progress)

**Teacher Dashboard**

- Courses created
- Total students enrolled
- Course performance metrics
- Student list per course

**Admin Dashboard**

- System-wide analytics
- User management table
- Course approval workflow
- Revenue metrics (if paid)

**Components**

- Use shadcn Card, Table, Chart components
- React Query for data fetching
- Real-time updates with polling/WebSocket

---

### 6. Quizzes/Assessments

**Backend**

- Quiz CRUD operations
- Question management
- Answer validation
- Auto-grading logic
- Randomized question selection

**Frontend**

- Quiz taking interface
- Timer component
- Answer submission
- Results page with feedback
- Question review

**Features**

- One attempt per quiz (configurable)
- Immediate feedback for MCQ/True-False
- Manual grading for short answers
- Question randomization

---

### 7. Certificates

**Backend**

- Certificate generation on course completion
- PDF generation (PDFKit/Puppeteer)
- Certificate template system
- Unique certificate numbers

**Frontend**

- Certificate display page
- Download PDF button
- Certificate verification page

**Implementation**

- Trigger on 100% course completion
- Generate PDF server-side
- Store PDF in S3/local storage
- Display certificate with verification link

---

### 8. Announcements & Notifications

**Backend**

- Announcement CRUD
- Notification creation triggers
- Email notification service
- In-app notification storage

**Frontend**

- Announcement feed
- Notification bell with badge
- Email notification preferences
- Real-time notification updates

**Notification Types**

- Course announcements
- Quiz due date reminders
- Course completion
- New course published (for enrolled students)

---

### 9. Responsive UI

**Design System**

- Tailwind CSS for styling
- shadcn/ui components (mobile-friendly)
- Dark mode support (optional)
- Accessible components (ARIA labels)

**Breakpoints**

- Mobile: < 640px
- Tablet: 640px - 1024px
- Desktop: > 1024px

**Key Pages**

- Mobile-first course player
- Responsive dashboard layouts
- Touch-friendly quiz interface

---

## React Query Setup

### API Client Configuration

```typescript
// lib/api/client.ts
- Axios instance with base URL
- Request interceptors (add token)
- Response interceptors (handle errors)
- Error handling
```

### Query Hooks

```typescript
// hooks/useCourses.ts
- useCourses() - List courses
- useCourse(id) - Get course details
- useCreateCourse() - Mutation
- useUpdateCourse() - Mutation
- useDeleteCourse() - Mutation
```

### Query Keys

- Consistent key structure: `['courses']`, `['courses', id]`
- Invalidation strategies
- Optimistic updates where applicable

---

## Implementation Phases

### Phase 1: Foundation (Week 1-2)

- [ ] Project setup (NestJS + Next.js)
- [ ] Database schema & migrations
- [ ] Authentication system
- [ ] Basic user management
- [ ] Role-based access control

### Phase 2: Core Features (Week 3-4)

- [ ] Course CRUD operations
- [ ] Category system
- [ ] File upload functionality
- [ ] Module/lesson management
- [ ] Enrollment system

### Phase 3: Learning Features (Week 5-6)

- [ ] Course player (video/text)
- [ ] Progress tracking
- [ ] Quiz system
- [ ] Quiz taking interface
- [ ] Results & grading

### Phase 4: Advanced Features (Week 7-8)

- [ ] Certificate generation
- [ ] Announcements
- [ ] Notifications (in-app + email)
- [ ] Dashboard implementation (all roles)
- [ ] Analytics

### Phase 5: Polish & Testing (Week 9-10)

- [ ] UI/UX improvements
- [ ] Responsive design refinement
- [ ] Error handling
- [ ] Performance optimization
- [ ] Testing (unit + integration)

---

## Additional Considerations

### Security

- Input validation & sanitization
- SQL injection prevention (ORM)
- XSS protection
- CSRF tokens
- Rate limiting
- File upload validation

### Performance

- Database indexing
- Image optimization
- Video streaming/CDN
- Caching strategies
- Lazy loading components

### Scalability

- Database connection pooling
- File storage (S3) for production
- Background jobs (Bull/BullMQ)
- Caching layer (Redis) - optional

### Future Enhancements

- Payment integration (Stripe)
- Live classes/video conferencing
- Discussion forums
- Assignment submissions
- Gamification (badges, points)
- Multi-language support

---

## Getting Started Checklist

### Backend Setup

1. Initialize NestJS project
2. Setup PostgreSQL database
3. Configure TypeORM/Prisma
4. Setup authentication module
5. Create base entities
6. Setup file upload module

### Frontend Setup

1. Initialize Next.js project
2. Install Tailwind CSS
3. Setup shadcn/ui
4. Configure React Query
5. Setup API client
6. Create auth context
7. Setup protected routes

---

_This plan provides a comprehensive roadmap for building a production-ready LMS. Adjust timelines and features based on your specific requirements._
