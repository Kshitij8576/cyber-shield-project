# CyberShield Project Documentation

## 1. Project Overview

**Project name:** Project CyberShield

**Purpose:** A content-driven Next.js site that promotes cyber awareness, resources, events, articles, and enables admin-posting of blog articles with image upload.

**Stack:**

- Frontend: Next.js (Pages Router)
- Backend: Next.js API routes
- Data: Firebase Firestore (blogs collection)
- File storage: Firebase Storage (via `pages/api/upload.js`)

## 2. Repository Structure

- `pages/` - Next.js page routes
- `pages/api/` - API endpoints
- `pages/admin/` - Admin content creation and management pages
- `components/` - UI widgets (Navbar, Hero, Blogs, Footer, etc.)
- `styles/` - CSS stylesheets
- `lib/firebase.js` and `firebase/client.js` - Firebase client helper functions
- `public/images/` - static image assets

## 3. Core Pages

| Route               | File                        | Functionality                                           |
| ------------------- | --------------------------- | ------------------------------------------------------- |
| `/`                 | `pages/index.js`            | Home with hero, about, blogs and opportunities sections |
| `/about`            | `pages/about.js`            | About page                                              |
| `/blog`             | `pages/blog.js`             | Blog listing page                                       |
| `/blog/[id]`        | `pages/blog/[id].js`        | Blog detail for selected article                        |
| `/resources-events` | `pages/resources-events.js` | Resources + events listing                              |
| `/contact`          | `pages/contact.js`          | Contact form (UI only)                                  |
| `/joinus`           | `pages/joinus.js`           | Join us form / mission statement                        |
| `/wedo`             | `pages/wedo.js`             | "What we do" info page                                  |

### Admin pages

| Route                 | File                          | Functionality                                                |
| --------------------- | ----------------------------- | ------------------------------------------------------------ |
| `/admin/login`        | `pages/admin/login.js`        | Admin sign-in UI (Firebase auth)                             |
| `/admin/dashboard`    | `pages/admin/dashboard.js`    | Admin panel overview                                         |
| `/admin/manage-blogs` | `pages/admin/manage-blogs.js` | Manage existing blog posts (edit/delete)                     |
| `/admin/create-blog`  | `pages/admin/create-blog.js`  | Create new blog entries with content editor and cover upload |
| `/admin/edit-blog`    | `pages/admin/edit-blog.js`    | Edit existing blog content                                   |

## 4. Firebase Data Model

### Firestore: `blogs` collection

Each blog document contains:

- `title`: string
- `category`: string (`resource` or `event`)
- `content`: string (HTML content from editor)
- `coverImage`: string (URL to uploaded image or null)
- `createdAt`: timestamp (serverTimestamp)
- `createdAtLocal`: string (local date/time value)

## 5. API Endpoints

All API endpoints are under `/api`.

### 5.1 `GET /api/hello`

- Purpose: test endpoint
- File: `pages/api/hello.js`
- Method: `GET`
- Request: none

#### Response

- `200`: `{ "name": "John Doe" }`

#### Example (curl)

```bash
curl -X GET http://localhost:3000/api/hello
```

### 5.2 `POST /api/upload`

- Purpose: upload blog cover image to Firebase Storage
- File: `pages/api/upload.js`
- Method: `POST`
- `Content-Type`: multipart/form-data
- Body field: `file`

#### Behavior

1. Uses `formidable` to parse multipart form data.
2. Requires exactly one file under `file`.
3. Uploads file to `blog-covers/<timestamp>-<originalFilename>` in Firebase Storage.
4. Makes the file public.
5. Responds with `url` pointing to `https://storage.googleapis.com/<bucket>/<storedPath>`.

#### Response

- `200`: `{ "url": "https://storage.googleapis.com/<bucket>/blog-covers/...." }`
- `400`: `{ "error": "No file received" }`
- `405`: `{ "error": "Method not allowed" }`
- `500`: `{ "error": "Upload failed" }` or `"Form parse failed"`

#### Example (curl)

```bash
curl -X POST http://localhost:3000/api/upload \
  -F "file=@cover.jpg"
```

#### Example (JavaScript)

```js
const formData = new FormData();
formData.append("file", fileInput.files[0]);

const res = await fetch("/api/upload", {
  method: "POST",
  body: formData,
});

if (!res.ok) throw new Error("Upload failed");
const json = await res.json();
console.log(json.url);
```

## 6. Firebase Admin Configuration

### File: `pages/api/_firebaseAdmin.js`

```js
import admin from "firebase-admin";

if (!admin.apps.length) {
  admin.initializeApp({
    credential: admin.credential.cert({
      projectId: process.env.FIREBASE_PROJECT_ID,
      clientEmail: process.env.FIREBASE_CLIENT_EMAIL,
      privateKey: process.env.FIREBASE_PRIVATE_KEY.replace(/\\n/g, "\n"),
    }),
    storageBucket: `${process.env.FIREBASE_PROJECT_ID}.appspot.com`,
  });
}

export const bucket = admin.storage().bucket();
export default admin;
```

### Required environment variables

- `FIREBASE_PROJECT_ID`
- `FIREBASE_CLIENT_EMAIL`
- `FIREBASE_PRIVATE_KEY`

## 7. Client-Side Firestore Usage

In `pages/admin/create-blog.js`, Firestore is initialized directly on client:

- `addDoc(collection(db, "blogs"), {...})` sends blog data.
- `getDocs`, `doc`, `updateDoc`, `deleteDoc` are used in admin pages for CRUD operations.

#### Note

The project currently uses open client-side Firestore access; to enforce security use Firestore Rules and auth.

## 8. Running Locally

1. Install dependencies:
   - `npm install` or `pnpm install` or `yarn`
2. Set environment variables in `.env.local` as above.
3. Run:
   - `npm run dev`
4. Open:
   - `http://localhost:3000`

## 9. Deployment

- Compatible with Vercel.
- Add environment variables in Vercel dashboard.

## 10. Additional Notes

- UI uses static blog content in `pages/blog/[id].js` to render blog detail pages (hardcoded articles r1, r2, r3). This can be adapted to use Firestore queries.
- Upload function is a centralized endpoint for preparing image storage (used by admin create/edit flows).
- Add other API endpoints as needed for true CRUD (blog list, fetch by ID, update, delete).

---

### Appendix: Recommended API extensions

1. `GET /api/blogs` - list all blogs from Firestore
2. `GET /api/blogs/:id` - single blog by document ID
3. `PUT /api/blogs/:id` - update blog metadata
4. `DELETE /api/blogs/:id` - remove blog post

These are not yet in the repo but strongly recommended for complete REST coverage.
