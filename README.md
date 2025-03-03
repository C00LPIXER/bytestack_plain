<!---
# Detailed Feature Specification (DFS)  
**Project Name**: ByteStack  
**Version**: v1.0 (Core) + v2.0 (Coins & Extras)  
**Date**: March 3, 2025  
**Stack**: React (Frontend), Node.js + Express (Backend), MongoDB (Database), Stripe (Payments), Cloudinary (File Storage), Nodemailer (Notifications)  
**Timeline**: 6 Weeks (v1, Solo), v2 TBD  
**Objective**: Build a scalable, dev-focused blog platform with site-wide subscriptions, rich content, discussions, social features, and future monetization via coins.
--->

## Module Breakdown

### 1. User Management Module
**Portal**: User  
**Objective**: Handle user authentication, profiles, and subscription status.  
**Features**:  
- **1.1 Authentication**  
  - Sign-up: Email/password or GitHub OAuth (Passport.js).  
  - Login: Email/password or GitHub token validation.  
  - Logout: Clear session/token.  
  - Deliverable: Secure JWT-based auth system.  
- **1.2 Profile Management**  
  - View: Display bio, blog list, follower count, follow button.  
  - Edit: Update bio, avatar (Cloudinary upload).  
  - Auto-Blogger: Set `isBlogger: true` on first blog publish (MongoDB update).  
  - Deliverable: Profile page with CRUD API endpoints.  
- **1.3 Subscription Status**  
  - Track: Store `subType: 'monthly' | 'yearly' | null`, `subEndDate: Date`, `trialEndDate: Date`.  
  - Trial: 7-day free trial on sign-up (set `trialEndDate`).  
  - Deliverable: User doc with sub/trial fields, API to check status.  
**Dependencies**: MongoDB, Stripe (for sub integration), Cloudinary.  
**APIs**:  
  - `POST /auth/signup`  
  - `POST /auth/login`  
  - `GET /user/:id`  
  - `PATCH /user/:id`  

---

### 2. Blog Management Module
**Portal**: Blogger (User with `isBlogger: true`)  
**Objective**: Enable blog creation, editing, and viewing with premium access control.  
**Features**:  
- **2.1 Blog Creation**  
  - Editor: React component with Markdown parsing, image uploads (Cloudinary), code blocks (copy button via `react-copy-to-clipboard`), GIF support, headlines/subheadlines.  
  - Toggle: Free or premium (`isPremium: Boolean`).  
  - Deliverable: Rich text editor UI + blog creation API.  
- **2.2 Blog Editing**  
  - Update: Modify content, toggle free/premium, save changes.  
  - Publish: Make blog live (set `isPublished: true`).  
  - Deliverable: Edit form + PATCH endpoint.  
- **2.3 Blog Viewing**  
  - Free: Open to all users.  
  - Premium: Locked unless `subType: 'active'` or `trialEndDate > now` (middleware check).  
  - Deliverable: Blog page with access logic.  
- **2.4 Profile Integration**  
  - List: Show user’s blogs on profile (filter by `userId`).  
  - Deliverable: Blog list component.  
**Dependencies**: User Module (auth), Subscription Module (access), Cloudinary.  
**APIs**:  
  - `POST /blogs`  
  - `PATCH /blogs/:id`  
  - `GET /blogs/:id`  
  - `GET /user/:id/blogs`  

---

### 3. Subscription Module
**Portal**: User  
**Objective**: Manage site-wide subscriptions and trial periods.  
**Features**:  
- **3.1 Subscription Plans**  
  - Monthly: $15/month (Stripe recurring payment).  
  - Yearly: $120/year (Stripe recurring payment).  
  - Store: Update `subType` and `subEndDate` on success.  
  - Deliverable: Checkout page + Stripe integration.  
- **3.2 Free Trial**  
  - 7 Days: Set `trialEndDate = now + 7 days` on sign-up.  
  - Auto-End: Clear access unless subbed (checked via middleware).  
  - Deliverable: Trial logic in auth flow.  
- **3.3 Access Control**  
  - Check: Middleware validates `subType` or `trialEndDate` for premium content.  
  - Redirect: Non-subbed users to sub page if trial expired.  
  - Deliverable: Secure content gate.  
**Dependencies**: User Module (status), Stripe (payments).  
**APIs**:  
  - `POST /subscribe` (Stripe checkout session)  
  - `GET /user/subscription` (status check)  
**Webhooks**:  
  - Stripe: `payment_succeeded` (update `subEndDate`), `subscription_canceled` (clear `subType`).  

---

### 4. Discussion Module
**Portal**: User + Blogger  
**Objective**: Facilitate public and private discussions under blogs.  
**Features**:  
- **4.1 Posting**  
  - Public: Post to threaded discussion (`type: 'public'`).  
  - Private: Send feedback to blogger (`type: 'private'`).  
  - Deliverable: Discussion form + POST endpoint.  
- **4.2 Threading**  
  - Replies: Nest comments with `parentId` (MongoDB ref).  
  - Deliverable: Threaded UI component.  
- **4.3 Deletion**  
  - Remove: Owner (`userId`) or blogger (`bloggerId`) can delete (auth check).  
  - Deliverable: Delete button + API.  
- **4.4 Blogger View**  
  - Private Access: Filter `type: 'private'` by `bloggerId`.  
  - Deliverable: Private discussion tab on blog page.  
**Dependencies**: Blog Module (blog IDs), User Module (auth).  
**APIs**:  
  - `POST /discussions`  
  - `GET /blogs/:id/discussions`  
  - `DELETE /discussions/:id`  

---

### 5. Follow & Notification Module
**Portal**: User + Blogger  
**Objective**: Enable following and notify users of new blog posts.  
**Features**:  
- **5.1 Follow System**  
  - Follow: Add `userId` to `followers: [userId]` array.  
  - Unfollow: Remove from array.  
  - Deliverable: Follow button + API.  
- **5.2 Notifications**  
  - Trigger: On blog publish (`POST /blogs`), notify followers.  
  - Email: Send via Nodemailer (`"New post by [user]!"`).  
  - In-App: Add to `notifications: [{ blogId, read: Boolean, createdAt }]` in user doc.  
  - View: Show unread count + list in UI.  
  - Mark Read: Update `read: true` on click.  
  - Deliverable: Notification system + UI component.  
**Dependencies**: Blog Module (publish event), User Module (followers).  
**APIs**:  
  - `POST /follow/:userId`  
  - `DELETE /follow/:userId`  
  - `GET /user/notifications`  
  - `PATCH /notifications/:id` (mark read)  

---

### 6. Admin Dashboard Module
**Portal**: Admin  
**Objective**: Provide oversight and control over platform users and content.  
**Features**:  
- **6.1 Analytics**  
  - Revenue: Aggregate Stripe payments.  
  - User Stats: Count total, active subs, trial conversions (MongoDB queries).  
  - Top Bloggers: Rank by views or discussion count (aggregation pipeline).  
  - Deliverable: Dashboard UI with charts.  
- **6.2 User Management**  
  - View: List all users (filter by `isBlogger`, `subType`).  
  - Ban: Set `isBanned: true`.  
  - Manual Sub: Assign `subType` and `subEndDate`.  
  - Deliverable: User table + controls.  
- **6.3 Blog Management**  
  - Hide: Set `isHidden: true`.  
  - Delete: Remove blog doc.  
  - Recommend: Toggle `isRecommended: Boolean` for visibility.  
  - Deliverable: Blog list + action buttons.  
**Dependencies**: All modules (full access).  
**APIs**:  
  - `GET /admin/stats`  
  - `GET /admin/users`  
  - `PATCH /admin/users/:id`  
  - `GET /admin/blogs`  
  - `PATCH /admin/blogs/:id`  

---

### 7. Infrastructure Module
**Portal**: Shared  
**Objective**: Support all modules with robust backend and deployment.  
**Features**:  
- **7.1 Database**  
  - MongoDB: Collections (`users`, `blogs`, `discussions`, `notifications`).  
  - Schema: Defined with Mongoose (e.g., `UserSchema`, `BlogSchema`).  
  - Deliverable: Connected DB with indexes (e.g., `userId`, `blogId`).  
- **7.2 Hosting**  
  - Platform: Heroku or AWS EC2 (scalable, solo-friendly).  
  - Deliverable: Deployed app with environment vars (Mongo URI, Stripe keys).  
- **7.3 Payments**  
  - Stripe: Monthly/yearly subs, trial setup, webhooks for updates.  
  - Deliverable: Payment flow + webhook listener.  
- **7.4 File Storage**  
  - Cloudinary: Image/GIF uploads from editor.  
  - Deliverable: Integrated upload API.  

---

<!-- ## 6-Week Development Plan (Solo)
- **Week 1: Planning & Setup**  
  - Repo: Initialize React + Express, connect MongoDB Atlas.  
  - Wireframes: Sketch User, Blogger, Admin portals.  
  - Deliverable: Running skeleton app.  
- **Week 2: User + Blog**  
  - Auth, profile, blog CRUD (editor setup).  
  - Deliverable: Users can sign up, write blogs.  
- **Week 3: Subscription**  
  - Stripe integration, trial logic, premium lock.  
  - Deliverable: Subs work, trial enforces.  
- **Week 4: Discussion**  
  - Public/private threads, delete functionality.  
  - Deliverable: Discussions live under blogs.  
- **Week 5: Follow + Notifications + Admin**  
  - Follow system, email/in-app notifications, admin stats + controls.  
  - Deliverable: Social features + basic admin.  
- **Week 6: Testing & Deployment**  
  - Test: All APIs, UI flows, Stripe live mode.  
  - Deploy: Heroku/AWS, domain TBD.  

--- -->
