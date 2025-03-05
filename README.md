<!---# Project Outline: ByteStack  
**Name**: ByteStack  
**Slogan**: "From Bits to Epic Bytes"  
**Stack**: React (Frontend, TypeScript), Node.js + Express (Backend, TypeScript), MongoDB (Database), Stripe (Payments), Cloudinary (File Storage), Nodemailer (Email Notifications), Socket.IO (Real-Time Notifications), Oauth(github, google)

**Objective**: Build a scalable, tech-focused blog platform with site-wide subscriptions, SEO-optimized rich content, real-time discussions, advanced search/recommendations, and coin-based monetization for premium reads.

## Core Components
- **Portals**:  
  - **User**: Access free/premium blogs, subscribe, join discussions, follow bloggers, receive real-time notifications, get personalized recommendations based on tech stack interests.  
  - **Blogger**: Create/edit SEO-friendly blogs, view analytics (views, discussions), manage discussions, earn coins.  
  - **Admin**: Monitor revenue/analytics, manage users/blogs, approve coin withdrawals, control content visibility/recommendations.  

- **Rich Editor**: Editor with image uploads (Cloudinary), code blocks (copy button), GIFs, headlines/subheadlines, different fonts, category based, and real-time SEO feedback (title length, meta tags).  

- **Subscriptions**:  
  - Site-wide plans: $15/month or $120/year via Stripe, 7-day free trial on sign-up.  
  - Access: Premium blogs locked unless subscribed or in trial.  

- **Discussions**:  
  - Types: Public (threaded, all users) and Private (blogger-only feedback).  
  - Real-Time: Socket.IO for live discussion updates and notifications (e.g., new replies).  

- **Social Features**:  
  - Follow: Users follow bloggers, tracked in profile.  
  - Notifications: Real-time via Socket.IO (new blogs, discussion replies), plus email backups (Nodemailer).  

- **SEO Optimization**:  
  - Blog fields: `metaTitle`, `metaDescription`, `slug` for search engine visibility.  
  - Checker: Real-time SEO analysis in editor (e.g., keyword density, tag usage).  

- **Search & Recommendations**:  
  - **Sign-Up Tech Stack**: Prompt users to select interests during sign-up (e.g., JavaScript, Python, AI, DevOps) from a predefined list, stored as `techInterests: [String]`.  
  - **Optimized Search**:  
    - By: Title, content, tags (MongoDB text index).  
    - Filters: Free/Premium (`isPremium`), Recommended (`isRecommended`), Tech Stack (`tags`), Date (`createdAt`), Popularity (`views`).  
    - UI: Search bar with dropdown filters (e.g., “JavaScript + Free + Last 30 Days”).  
    - History: Search history(10 resend hestory useing caped collection).
  - **Personalized Recommendations**:  
    - Logic: Match `techInterests` with blog `tags`, prioritize `isRecommended` and high `views`.  
    - UI: “Recommended for You” section on homepage/profile.  
    - Frequency: Update on login or new blog publish (Socket.IO trigger).  
  - Deliverable: Smart search bar + recommendation engine tied to user interests.  

- **Analytics (Blogger)**: Views, discussion counts, follower growth displayed in a dashboard.  

- **Coin System**:  
  - Earn: earn coin per premium reads, awarded to blogger based on an algorithm.  
  - Withdraw: 1000 coins = $100, manual admin approval.  

- **Infrastructure**:  
  - **Database**: MongoDB for data storage, leveraging flexible schemas and text indexing for search.  
  - **Hosting**: AWS (EC2 for compute, S3 for static assets if needed), ensuring scalability and reliability.  
  - **Payments**: Stripe for subscription processing and webhook integration.  
  - **File Storage**: Cloudinary for image/GIF uploads from the rich editor.  
  - **Architecture**: Clean Architecture with SOLID principles (Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion), implemented in TypeScript for type safety and maintainability.  
    - **Structure**: Organized into layers (e.g., Presentation, Application, Domain, Infrastructure), with interfaces defining contracts between layers.  
    - **TypeScript**: Strong typing for models (e.g., `User`, `Blog`), APIs, and services to catch errors at compile time.


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
**Objective**: Enable blog creation, editing, viewing, SEO optimization, and analytics with premium access control.  
**Features**:  
- **2.1 Blog Creation**  
  - Editor: React component with Markdown parsing, image uploads (Cloudinary), code blocks (copy button via `react-copy-to-clipboard`), GIF support, headlines/subheadlines.  
  - Toggle: Free or premium (`isPremium: Boolean`).  
  - SEO Fields: Add `metaTitle`, `metaDescription`, `slug` (auto-generated from title, editable).  
  - Deliverable: Rich text editor UI + blog creation API with SEO inputs.  
- **2.2 Blog Editing**  
  - Update: Modify content, toggle free/premium, edit SEO fields, save changes.  
  - Publish: Make blog live (set `isPublished: true`).  
  - Deliverable: Edit form + PATCH endpoint.  
- **2.3 Blog Viewing**  
  - Free: Open to all users.  
  - Premium: Locked unless `subType: 'active'` or `trialEndDate > now` (middleware check).  
  - SEO: Serve `<meta>` tags (`metaTitle`, `metaDescription`) in HTML head, use `slug` in URL (e.g., `/blogs/:slug`).  
  - Deliverable: Blog page with access logic and SEO metadata.  
- **2.4 Profile Integration**  
  - List: Show user’s blogs on profile (filter by `userId`).  
  - Deliverable: Blog list component.  
- **2.5 SEO Checker** 
  - Check: Analyze blog for SEO (e.g., title length < 60 chars, metaDescription 120-160 chars, at least one H1, image alt tags).  
  - Feedback: Show pass/fail in editor UI (e.g., “Add more keywords” or “Good to go”).  
  - Tools: Use a lightweight library (e.g., custom JS logic or `seo-analyzer`).  
  - Deliverable: Real-time SEO feedback in editor.  
- **2.6 Blogger Analytics**
  - Metrics: Views (`views` count), discussion count (aggregate `discussions` by `blogId`), follower growth.  
  - UI: Dashboard tab on profile (e.g., “Your Stats”).  
  - API: `GET /user/:id/analytics` (auth required, `userId` match).  
  - Deliverable: Analytics dashboard for bloggers.  
**Dependencies**: User Module (auth), Subscription Module (access), Cloudinary, MongoDB (analytics).  
**APIs**:  
  - `POST /blogs`  
  - `PATCH /blogs/:id`  
  - `GET /blogs/:slug` (SEO-friendly URL)  
  - `GET /user/:id/blogs`  
  - `GET /user/:id/analytics`  
<!---
**Updated Schema**:  
```json
{
  "blog": {
    "_id": "ObjectId",
    "userId": "ObjectId",
    "title": "String",
    "content": "String (Markdown)",
    "metaTitle": "String",
    "metaDescription": "String",
    "slug": "String",
    "isPremium": "Boolean",
    "isPublished": "Boolean",
    "isHidden": "Boolean (admin)",
    "isRecommended": "Boolean (admin)",
    "views": "Number",
    "createdAt": "Date",
    "updatedAt": "Date"
  }
}
```

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

### 7. Coin & Withdrawal Module
**Portal**: Blogger + Admin  
**Objective**: Reward bloggers with coins for premium reads and manage withdrawals with a scalable, secure system.  
**Features**:  
- **7.1 Coin Earning**  
  - **Logic**: Award 1 coin per premium blog read, tracked via a unique read system (`views` increment only for subscribed/trial users).  
  - **Field**: Add `coins: Number` to user document for tracking earnings.  
  - **Trigger**: On `GET /blogs/:id` when a premium blog is accessed by a valid subscriber/trial user.  
  - **Explanation**: Ensures coins are awarded only for unique premium reads by subscribed/trial users, prevents duplicates with a `reads` collection, and updates `views` and `coins` atomically.  
  - **Deliverable**: Coin accrual system integrated into blog view endpoint.  

- **7.2 Wallet View**  
  - **Display**: Show current `coins` balance and lifetime earnings (sum of approved payouts) on blogger profile.  
  - **API**: `GET /user/:id/wallet` (auth required, returns `{ coins: number, lifetimeEarnings: number }`).  
  - **Deliverable**: Wallet UI component with real-time balance display (TypeScript-typed response).  

- **7.3 Withdrawal**  
  - **Threshold**: 1000 coins = ₹200 (updated from 100 coins = ₹250).  
  - **Process**: Blogger submits withdrawal request when `coins >= 1000`, admin reviews and approves manually.  
  - **Action**: Deduct `coins` from user, create a `payout` record, notify blogger via Socket.IO/email (Nodemailer).  
  - **APIs**:  
    - `POST /withdraw` (auth required, payload: `{ coinAmount: number }`, validates `coins >= 1000`).  
    - `GET /admin/payouts` (admin-only, lists pending/approved/rejected requests).  
    - `PATCH /admin/payouts/:id` (admin-only, updates `status` to `approved` or `rejected`, deducts coins on approval).  
  - **Deliverable**: Withdrawal form for bloggers + admin payout queue UI with approval workflow.  

**Dependencies**: Blog Module (views tracking), User Module (auth and coin storage), Admin Module (approval process).  
  
<!---**Updated Schema**:  
```json
{
  "user": { "coins": "Number" },
  "payout": {
    "_id": "ObjectId",
    "userId": "ObjectId",
    "amount": "Number (INR)",
    "status": "String ('pending' | 'approved' | 'rejected')",
    "createdAt": "Date",
    "approvedAt": "Date (null if pending/rejected)"
  }
}
```


---

### Your 6-Week v1 Plan (Updated)
- **Week 1: Planning & Setup**  
  - Repo, MongoDB Atlas, wireframes (add SEO + analytics UI).  
  - Modules: 1.1-1.3.  
- **Week 2: User + Blog Basics**  
  - Auth, profile, blog CRUD (editor + SEO fields).  
  - Modules: 1.4-1.6, 2.1-2.4.  
- **Week 3: Subs + SEO Checker**  
  - Stripe, trial, premium lock, SEO feedback.  
  - Modules: 3, 2.5.  
- **Week 4: Discussions**  
  - Public/private threads, deletion.  
  - Module: 4.  
- **Week 5: Follow + Analytics + Admin**  
  - Follow, notifications, blogger analytics, admin basics.  
  - Modules: 5, 2.6, 6.  
- **Week 6: Testing & Deploy**  
  - Test all features, deploy to Heroku/AWS.  

---

### Next Steps
- **Slogan**: “Stack Your Tech Wisdom” locked—want another option?  
- **Markdown/PDF**: I’ll assume you’ll convert this updated spec—copy it into `ByteStack_DFS.md` and use `pandoc` or VS Code for PDF.  
- **Code**: Start Week 1? I can drop an Express + Mongo setup to kick off **ByteStack**.
--->
Project Outline: ByteStack  
---
  
**Project Name**: ByteStack  
**Slogan**: "Stack Your Tech Wisdom"  
**Date**: March 3, 2025  
**Stack**: React (Frontend, TypeScript), Node.js + Express (Backend, TypeScript), MongoDB (Database), Elasticsearch or MongoDB Atlas Search (Search & Analytics), Redis (Caching & Real-Time), Stripe (Payments), Cloudinary (File Storage), Nodemailer (Email Notifications), Socket.IO (Real-Time Notifications)  
**Objective**: Build a scalable, dev-focused blog platform with site-wide subscriptions, SEO-optimized rich content, real-time discussions, advanced search/recommendations, and coin-based monetization.

---

## Module Breakdown

### 1. User Management Module
**Portal**: User  
**Objective**: Handle user authentication, profiles, and subscription status with optimized queries and caching.  
**Features**:  
- **1.1 Authentication**  
  - Sign-up: Email/password or GitHub OAuth (Passport.js), prompt for `techInterests`.  
  - Login: Email/password or GitHub token validation.  
  - Logout: Clear JWT.  
  - Deliverable: Secure JWT-based auth with tech stack selection UI.  
- **1.2 Profile Management**  
  - View: Display bio, avatar, blog list, follower count, follow button.  
  - Edit: Update bio, avatar (Cloudinary upload).  
  - Auto-Blogger: Set `isBlogger: true` on first blog publish.  
  - Deliverable: Profile page with CRUD APIs, cached in Redis (`user:${id}:profile`).  
- **1.3 Subscription Status**  
  - Track: Store `subType`, `subEndDate`, `trialEndDate`.  
  - Trial: 7-day free trial on sign-up (set `trialEndDate`).  
  - Cache: Store sub status in Redis (`user:${id}:sub`, TTL 1h).  
  - Deliverable: Sub status API with Redis caching.  
**Dependencies**: MongoDB (`users`), Redis (caching), Stripe (subs), Cloudinary (avatar).  
**APIs**:  
  - `POST /auth/signup`  
  - `POST /auth/login`  
  - `GET /user/:id` (cached)  
  - `PATCH /user/:id`  
  - `GET /user/subscription` (cached)  

---

### 2. Blog Management Module
**Portal**: Blogger (User with `isBlogger: true`)  
**Objective**: Enable structured blog creation, editing, and viewing with SEO, analytics, and real-time features.  
**Features**:  
- **2.1 Blog Creation**  
  - Editor: React component with structured input (title, subtitle, text sections with subheaders/paragraphs/images).  
  - Fields: `title`, `subtitle`, `text` (array of sections), `tags`, `isPremium`.  
  - Images: Upload via Cloudinary (`src`, `height`, `width`, `onRight`).  
  - Deliverable: Rich editor UI + creation API, sync to Elasticsearch/Atlas.  
- **2.2 Blog Editing**  
  - Update: Modify all fields, toggle `isPremium`/`isPublished`, save changes.  
  - Publish: Set `isPublished: true`, update `datePosted`.  
  - Deliverable: Edit form + PATCH endpoint, resync to search.  
- **2.3 Blog Viewing**  
  - Free: Open to all.  
  - Premium: Locked unless `subType` active or `trialEndDate > now`.  
  - SEO: Serve `metaTitle`, `metaDescription`, `slug` in HTML head.  
  - Cache: Views in Redis (`blog:${id}:views`, TTL 5m).  
  - Deliverable: Blog page with access logic, cached views.  
- **2.4 Profile Integration**  
  - List: Show user’s blogs (`userId`), cached in Redis (`user:${id}:blogs`).  
  - Deliverable: Blog list component.  
- **2.5 SEO Checker**  
  - Check: Real-time analysis (title length, meta tags, keywords in `text`).  
  - Deliverable: Editor feedback UI.  
- **2.6 Blogger Analytics**  
  - Metrics: Views (`views`), discussion count, likes (`likes.length`), cached in Redis (`blog:${id}:stats`).  
  - Deliverable: Analytics dashboard on profile.  
**Dependencies**: User Module (auth), Subscription Module (access), Cloudinary, MongoDB (`blogs`), Redis (caching), Elasticsearch/Atlas (search).  
**APIs**:  
  - `POST /blogs`  
  - `PATCH /blogs/:id`  
  - `GET /blogs/:slug` (cached views)  
  - `GET /user/:id/blogs` (cached)  
  - `GET /user/:id/analytics` (cached)  

---

### 3. Subscription Module
**Portal**: User  
**Objective**: Manage site-wide subscriptions and trial periods with caching for performance.  
**Features**:  
- **3.1 Subscription Plans**  
  - Monthly: $15/month (Stripe).  
  - Yearly: $120/year (Stripe).  
  - Action: Update `subType`/`subEndDate`, cache in Redis (`user:${id}:sub`).  
  - Deliverable: Checkout page + Stripe integration.  
- **3.2 Free Trial**  
  - 7 Days: Set `trialEndDate` on sign-up.  
  - Auto-End: Clear access unless subbed (middleware).  
  - Deliverable: Trial logic in auth flow.  
- **3.3 Access Control**  
  - Check: Middleware validates `subType`/`trialEndDate` (Redis first, then MongoDB).  
  - Redirect: Non-subbed to `/subscribe`.  
  - Deliverable: Secure content gate.  
**Dependencies**: User Module (status), Stripe, Redis (caching).  
**APIs**:  
  - `POST /subscribe`  
  - `GET /user/subscription` (cached)  
**Webhooks**:  
  - `payment_succeeded`: Update `subEndDate`, refresh Redis.  
  - `subscription_canceled`: Clear `subType`, invalidate cache.  

---

### 4. Discussion Module
**Portal**: User + Blogger  
**Objective**: Facilitate real-time public/private discussions under blogs.  
**Features**:  
- **4.1 Posting**  
  - Public: `type: 'public'`, visible to all.  
  - Private: `type: 'private'`, blogger-only.  
  - Real-Time: Socket.IO + Redis pub/sub (`discussion:${blogId}`).  
  - Deliverable: Discussion form + POST endpoint.  
- **4.2 Threading**  
  - Replies: Nest with `parentId` in `blogs.discussions`.  
  - Deliverable: Threaded UI component.  
- **4.3 Deletion**  
  - Remove: Owner (`userId`) or blogger (`blog.userId`).  
  - Deliverable: Delete button + API.  
- **4.4 Blogger View**  
  - Filter: Show `type: 'private'` for `blog.userId`.  
  - Deliverable: Private discussion tab.  
**Dependencies**: Blog Module (`blogs.discussions`), User Module (auth), Redis (real-time), Socket.IO.  
**APIs**:  
  - `POST /blogs/:id/discussions`  
  - `GET /blogs/:id/discussions`  
  - `DELETE /blogs/:id/discussions/:discussionId`  

---

### 5. Follow & Notification Module
**Portal**: User + Blogger  
**Objective**: Enable following and real-time notifications with caching.  
**Features**:  
- **5.1 Follow System**  
  - Follow: Add to `followers`/`following` arrays.  
  - Unfollow: Remove from arrays.  
  - Cache: Follower list in Redis (`user:${id}:followers`).  
  - Deliverable: Follow button + API.  
- **5.2 Notifications**  
  - Trigger: New blog (`POST /blogs`), new discussion (`POST /discussions`).  
  - Real-Time: Socket.IO + Redis pub/sub (`notify:${userId}`).  
  - Email: Fallback via Nodemailer.  
  - View: List in UI, mark read.  
  - Deliverable: Notification system + UI component.  
**Dependencies**: User Module (`followers`, `notifications`), Blog Module (triggers), Redis, Socket.IO.  
**APIs**:  
  - `POST /follow/:userId`  
  - `DELETE /follow/:userId`  
  - `GET /user/notifications`  
  - `PATCH /notifications/:id` (mark read)  

---

### 6. Admin Dashboard Module
**Portal**: Admin  
**Objective**: Oversee platform with analytics and content control.  
**Features**:  
- **6.1 Analytics**  
  - Revenue: Sum Stripe payments (MongoDB).  
  - User Stats: Total, subs, trials (MongoDB + Elasticsearch/Atlas aggregations).  
  - Top Bloggers: Rank by views/likes (cached in Redis).  
  - Deliverable: Dashboard UI with charts.  
- **6.2 User Management**  
  - View: List users, filter by `isBlogger`, `subType`.  
  - Ban: Set `isBanned: true`.  
  - Manual Sub: Set `subType`/`subEndDate`.  
  - Deliverable: User table + controls.  
- **6.3 Blog Management**  
  - Hide: `isHidden: true`.  
  - Recommend: `isRecommended: true`.  
  - Delete: Remove blog, update search index.  
  - Deliverable: Blog list + action buttons.  
**Dependencies**: All modules, MongoDB, Redis, Elasticsearch/Atlas.  
**APIs**:  
  - `GET /admin/stats`  
  - `GET /admin/users`  
  - `PATCH /admin/users/:id`  
  - `GET /admin/blogs`  
  - `PATCH /admin/blogs/:id`  
  - `DELETE /admin/blogs/:id`  

---

### 7. Coin & Withdrawal Module (v2)
**Portal**: Blogger + Admin  
**Objective**: Reward bloggers with coins for premium reads and manage withdrawals.  
**Features**:  
- **7.1 Coin Earning**  
  - Logic: 1 coin per unique premium read (`reads` collection).  
  - Trigger: `GET /blogs/:id`, check `reads`.  
  - Algorithm: See earlier (`awardCoinForPremiumRead`).  
  - Deliverable: Coin accrual system.  
- **7.2 Wallet View**  
  - Display: `coins` + lifetime earnings (Redis cache: `user:${id}:wallet`).  
  - API: `GET /user/:id/wallet`.  
  - Deliverable: Wallet UI.  
- **7.3 Withdrawal**  
  - Threshold: 1000 coins = ₹200.  
  - Process: Submit request, admin approves via `payouts`.  
  - APIs: `POST /withdraw`, `GET /admin/payouts`, `PATCH /admin/payouts/:id`.  
  - Deliverable: Withdrawal form + admin queue.  
**Dependencies**: Blog Module (`views`), User Module (`coins`), Admin Module, MongoDB (`reads`, `payouts`), Redis (caching).  

---

### 8. Search & Discovery Module
**Portal**: User  
**Objective**: Enhance content discovery with scalable search and recommendations.  
**Features**:  
- **8.1 Blog Search**  
  - By: Title, `text.paragraphs`, tags (Elasticsearch/Atlas Search).  
  - Filters: `isPremium`, `tags`, `createdAt`, `views` (MongoDB compound index).  
  - API: `GET /blogs?search=term&filter=premium|tag|date`.  
  - Deliverable: Search bar with filters.  
- **8.2 Recommendations**  
  - Logic: Match `techInterests` with `tags`, boost `isRecommended`/`views`.  
  - Cache: Redis (`recommendations:${userId}`, TTL 24h).  
  - API: `GET /user/recommendations`.  
  - Deliverable: “Recommended for You” section.  
**Dependencies**: User Module (`techInterests`), Blog Module (`tags`), Elasticsearch/Atlas, Redis.  

---

### 9. Infrastructure Module
**Portal**: Shared  
**Objective**: Support all features with a scalable, optimized stack.  
**Features**:  
- **9.1 Database**  
  - MongoDB: `users`, `blogs` (with discussions), `reads` (v2), `payouts` (v2).  
  - Indexes: Compound (e.g., `blogs: { tags: 1, isPremium: 1, createdAt: -1 }`).  
- **9.2 Search & Analytics**  
  - Elasticsearch/Atlas Search: Full-text search, aggregations (top blogs, tags).  
  - Sync: Blogs to search index on create/update.  
- **9.3 Caching & Real-Time**  
  - Redis: Cache views, subs, recommendations; pub/sub for notifications, discussions.  
- **9.4 Hosting**  
  - AWS: EC2 (compute), S3 (static assets if needed).  
- **9.5 Payments**  
  - Stripe: Subs, trial, webhooks.  
- **9.6 Files**  
  - Cloudinary: Image storage for `text.image.src`.  

---
