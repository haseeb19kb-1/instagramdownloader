# Instagram Video Downloader — React Project Specification

## 1. Project Goal

Build a modern, responsive Instagram video downloader web application using **React** for the frontend and a secure **Node.js/Express backend** for server-side processing.

The application should allow users to paste an Instagram URL and retrieve downloadable media **only when the user owns the content or has permission to download it**.

> Important: Do not implement mechanisms intended to bypass Instagram authentication, private-account restrictions, DRM, access controls, or other platform protections. Respect Instagram's Terms of Use, copyright, and applicable laws.

---

# 2. Recommended Technology Stack

## Frontend

- React
- Vite
- TypeScript
- React Router
- Tailwind CSS
- Axios or native `fetch`
- Lucide React icons
- React Hook Form + Zod for validation
- ESLint
- Prettier

## Backend

- Node.js
- Express
- TypeScript
- Zod
- Helmet
- CORS
- express-rate-limit
- Pino/Winston for logging
- UUID
- Native `fetch` or Axios
- Optional Redis for rate limiting/caching
- Optional PostgreSQL if persistent application data is required

## Infrastructure

- HTTPS
- Reverse proxy such as Nginx or a managed platform
- Environment variables
- Docker
- Temporary storage with automatic cleanup
- Optional object storage such as S3-compatible storage

---

# 3. Project Structure

```text
instagram-downloader/
│
├── client/
│   ├── public/
│   ├── src/
│   │   ├── assets/
│   │   ├── components/
│   │   │   ├── Navbar.tsx
│   │   │   ├── Footer.tsx
│   │   │   ├── UrlInput.tsx
│   │   │   ├── DownloadForm.tsx
│   │   │   ├── MediaPreview.tsx
│   │   │   ├── DownloadButton.tsx
│   │   │   ├── QualitySelector.tsx
│   │   │   ├── LoadingState.tsx
│   │   │   └── ErrorMessage.tsx
│   │   │
│   │   ├── pages/
│   │   │   ├── Home.tsx
│   │   │   ├── Privacy.tsx
│   │   │   ├── Terms.tsx
│   │   │   └── NotFound.tsx
│   │   │
│   │   ├── hooks/
│   │   ├── lib/
│   │   │   ├── api.ts
│   │   │   └── validation.ts
│   │   ├── types/
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   └── index.css
│   │
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
│
├── server/
│   ├── src/
│   │   ├── config/
│   │   │   └── env.ts
│   │   │
│   │   ├── controllers/
│   │   │   └── media.controller.ts
│   │   │
│   │   ├── routes/
│   │   │   ├── health.routes.ts
│   │   │   └── media.routes.ts
│   │   │
│   │   ├── services/
│   │   │   ├── media.service.ts
│   │   │   ├── validation.service.ts
│   │   │   └── cleanup.service.ts
│   │   │
│   │   ├── middleware/
│   │   │   ├── error.middleware.ts
│   │   │   ├── rateLimit.middleware.ts
│   │   │   └── security.middleware.ts
│   │   │
│   │   ├── utils/
│   │   │   ├── logger.ts
│   │   │   └── url.ts
│   │   │
│   │   ├── app.ts
│   │   └── server.ts
│   │
│   ├── package.json
│   └── tsconfig.json
│
├── .env.example
├── .gitignore
├── docker-compose.yml
├── Dockerfile
├── README.md
└── package.json
```

---

# 4. Main User Flow

```text
User opens website
        ↓
Pastes Instagram URL
        ↓
Frontend validates URL
        ↓
POST /api/media/analyze
        ↓
Backend validates URL again
        ↓
Security checks
        ↓
Authorized media retrieval/processing
        ↓
Return safe metadata
        ↓
Frontend displays preview
        ↓
User selects available format
        ↓
POST /api/media/download
        ↓
Backend generates/returns temporary download
        ↓
Temporary resource expires
        ↓
Cleanup
```

---

# 5. Frontend Requirements

## Home Page

Create a professional SaaS-style landing page.

### Header

- Logo
- Home
- How It Works
- Privacy
- Terms
- Responsive mobile menu

### Hero Section

Headline:

> Download Instagram Videos Easily

Subtitle:

> Save videos you own or have permission to download.

URL input:

```text
Paste Instagram URL here...
```

Primary button:

```text
Analyze Video
```

Supported URL examples can be shown without implying access to private content.

### Features

Display cards for:

- Fast processing
- Mobile friendly
- Secure processing
- No unnecessary registration
- Temporary download links
- Responsive design

### How It Works

```text
1. Paste URL
2. Analyze
3. Preview
4. Download
```

### Footer

Include:

- Privacy Policy
- Terms of Service
- Copyright notice
- Contact
- Responsible-use statement

---

# 6. UI Design

Use a modern dark interface.

Suggested design:

- Deep navy/purple background
- Purple/blue gradients
- Glassmorphism cards
- Rounded corners
- Subtle shadows
- Smooth hover animations
- Clean typography
- Responsive layout
- Mobile-first design

Do not overload the page with animations.

The application must remain fast.

---

# 7. API Design

## Health Check

```http
GET /api/health
```

Response:

```json
{
  "status": "ok"
}
```

---

## Analyze Media

```http
POST /api/media/analyze
```

Request:

```json
{
  "url": "https://www.instagram.com/..."
}
```

Response example:

```json
{
  "success": true,
  "media": {
    "id": "temporary-id",
    "type": "video",
    "title": "Example",
    "thumbnail": "temporary-thumbnail-url",
    "duration": 15,
    "formats": [
      {
        "id": "format-1",
        "quality": "720p",
        "format": "mp4"
      }
    ]
  }
}
```

Do not expose internal server paths, credentials, cookies, tokens, or provider-specific secrets.

---

# 8. Download API

```http
POST /api/media/download
```

Request:

```json
{
  "mediaId": "temporary-id",
  "formatId": "format-1"
}
```

Response:

```json
{
  "success": true,
  "downloadUrl": "temporary-download-url",
  "expiresIn": 300
}
```

Download URLs should expire.

---

# 9. URL Validation

The backend must NEVER trust frontend validation.

Validate the URL on the server.

Accept only the expected Instagram URL formats.

Example validation rules:

- Must use HTTPS
- Must have an allowed hostname
- Reject localhost
- Reject private IP addresses
- Reject internal network addresses
- Reject unsupported protocols
- Reject malformed URLs
- Limit URL length
- Normalize URLs before processing

Example allowed hostname policy:

```text
instagram.com
www.instagram.com
```

Only allow additional official domains when there is a clear technical requirement.

---

# 10. SSRF Protection

This is one of the most important security requirements.

The backend must not become an unrestricted URL-fetching proxy.

Protect against:

```text
localhost
127.0.0.1
0.0.0.0
::1
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
169.254.0.0/16
internal hostnames
cloud metadata endpoints
```

Also:

- Resolve DNS safely
- Validate the resolved IP
- Re-check redirects
- Do not blindly follow redirects to arbitrary hosts
- Restrict outbound requests to approved domains/providers
- Set request timeouts

---

# 11. Rate Limiting

Add rate limits to prevent abuse.

Example:

```text
General API:
100 requests / 15 minutes / IP

Analyze:
20 requests / 15 minutes / IP

Download:
10 requests / 15 minutes / IP
```

These are starting values and should be adjusted after observing real traffic.

For production at scale, consider Redis-backed rate limiting.

---

# 12. Request Limits

Set strict limits.

Example:

```text
Maximum URL length: 2,048 characters
Request timeout: 30 seconds
Maximum metadata response size: reasonable fixed limit
Maximum download size: configurable
Maximum concurrent jobs per user/IP: limited
```

Never allow unlimited downloads.

---

# 13. Security Middleware

Use:

```text
Helmet
CORS
Rate limiting
Request size limits
Security headers
Centralized error handling
Structured logging
```

Recommended security headers include:

```text
Content-Security-Policy
X-Content-Type-Options
Referrer-Policy
Strict-Transport-Security
X-Frame-Options
```

Configure CSP carefully according to the frontend's actual requirements.

---

# 14. CORS

Do NOT use:

```text
Access-Control-Allow-Origin: *
```

in production when authenticated or sensitive APIs are involved.

Use an environment variable:

```env
FRONTEND_URL=https://yourdomain.com
```

Only allow the required frontend origin.

---

# 15. Environment Variables

Create:

```text
.env.example
```

Example:

```env
NODE_ENV=development
PORT=5000

FRONTEND_URL=http://localhost:5173

API_BASE_URL=http://localhost:5000

RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

DOWNLOAD_EXPIRY_SECONDS=300

MAX_URL_LENGTH=2048
REQUEST_TIMEOUT_MS=30000

TEMP_DIRECTORY=./tmp
MAX_DOWNLOAD_SIZE_MB=100

REDIS_URL=

DATABASE_URL=

STORAGE_ENDPOINT=
STORAGE_BUCKET=
STORAGE_ACCESS_KEY=
STORAGE_SECRET_KEY=
```

Never commit:

```text
.env
```

to Git.

---

# 16. Secret Management

Never put private secrets inside React.

Bad:

```text
VITE_SECRET_KEY=...
```

Frontend environment variables can become visible to users.

Secrets must stay on the backend.

Examples:

- API keys
- Storage credentials
- Database credentials
- Provider credentials
- Encryption secrets

Use server-side environment variables or a proper secrets manager.

---

# 17. Temporary Files

If files are temporarily stored:

```text
/tmp
```

must be cleaned automatically.

Recommended lifecycle:

```text
Create temporary file
        ↓
Process file
        ↓
Return temporary download
        ↓
Expire after 5 minutes
        ↓
Delete file
```

Also run periodic cleanup in case a request crashes.

Never keep user media permanently unless there is a legitimate documented reason.

---

# 18. File Security

If the backend creates downloadable files:

- Generate random filenames
- Never use user-supplied filenames directly
- Prevent path traversal
- Validate MIME types
- Limit file size
- Store temporary files outside the public web root
- Set safe `Content-Type`
- Set safe `Content-Disposition`
- Delete files after expiration

Reject filenames containing:

```text
../
..\
absolute paths
null bytes
```

---

# 19. Error Handling

Never expose stack traces in production.

Bad:

```json
{
  "error": "/home/server/src/download/service.js:125..."
}
```

Good:

```json
{
  "success": false,
  "error": {
    "code": "MEDIA_UNAVAILABLE",
    "message": "The requested media could not be processed."
  }
}
```

Use consistent error codes.

Example:

```text
INVALID_URL
UNSUPPORTED_URL
MEDIA_UNAVAILABLE
PRIVATE_CONTENT
PROCESSING_TIMEOUT
FILE_TOO_LARGE
RATE_LIMITED
SERVER_ERROR
```

---

# 20. Logging

Use structured server-side logs.

Log:

- Request ID
- Timestamp
- Endpoint
- Processing time
- Status code
- Error code
- Rate-limit events

Do NOT log:

- Passwords
- API keys
- Cookies
- Authentication tokens
- Private media
- Sensitive user information

Use a request ID for debugging.

---

# 21. Abuse Prevention

Implement:

- IP rate limiting
- Request limits
- Download limits
- Concurrent job limits
- Timeouts
- Temporary-file expiration
- Bot protection if necessary
- Abuse monitoring

For high traffic, consider:

```text
Cloudflare
Redis
Queue system
Object storage
```

---

# 22. Database

A database is NOT required for the first MVP.

For an initial version, avoid storing unnecessary user data.

If a database is later required, PostgreSQL can store:

```text
users
download_jobs
rate_limit_records
usage_statistics
```

Do not store downloaded media permanently unless required.

---

# 23. Optional Job Queue

For large media processing, use a queue.

Example:

```text
React
  ↓
API
  ↓
Job Queue
  ↓
Worker
  ↓
Temporary Storage
  ↓
Download
```

Possible technologies:

- Redis
- BullMQ

This prevents long-running jobs from blocking the main API server.

---

# 24. Performance

Optimize for:

- Fast initial page load
- Lazy-loaded components
- Compressed assets
- CDN for static assets
- HTTP caching where appropriate
- Efficient API requests
- Short processing timeouts
- Cleanup of abandoned jobs

Do not download/process the same resource repeatedly when safe caching is possible.

---

# 25. Accessibility

The frontend must support:

- Keyboard navigation
- Visible focus states
- Semantic HTML
- Proper labels
- Accessible buttons
- Good color contrast
- Screen-reader-friendly error messages
- Mobile touch targets

---

# 26. Responsive Design

Test:

```text
320px
375px
414px
768px
1024px
1280px
1440px+
```

The URL input and Analyze button should work properly on mobile.

---

# 27. Loading States

When analyzing:

```text
Analyzing URL...
```

Show:

- Spinner
- Progress-style visual feedback
- Disabled submit button

Prevent duplicate submissions.

---

# 28. Download Result Card

After successful analysis show:

```text
Thumbnail
Title
Duration
Media Type

Available Quality:
720p
1080p
2K
```

Only display formats actually returned by the backend.

Do not promise 1080p/2K/4K if the source does not provide them.

---

# 29. Legal / Responsible Use

Add a visible responsible-use notice.

Example:

> This service is intended for downloading content that you own or have permission to download. Respect copyright, privacy, and Instagram's Terms of Use.

Do not provide features intended to:

- Access private accounts
- Bypass login restrictions
- Circumvent DRM
- Evade platform security
- Download content without authorization

---

# 30. Privacy Policy Requirements

Explain:

- What data is collected
- Whether URLs are logged
- How long temporary data is stored
- How temporary files are deleted
- Whether analytics are used
- Third-party services
- Contact information
- User rights where applicable

Avoid collecting unnecessary personal information.

---

# 31. Terms of Service

Include:

- Authorized-use requirement
- Copyright responsibility
- Prohibited activities
- Service availability
- Abuse restrictions
- Limitation of liability
- Contact information

Have the final legal text reviewed for the jurisdiction where the service operates.

---

# 32. Docker

Create a production-ready Docker setup.

Suggested structure:

```text
Dockerfile
docker-compose.yml
.dockerignore
```

Do not run the application as root inside the production container when avoidable.

Use a multi-stage build for the React frontend.

---

# 33. Production Architecture

Recommended:

```text
                    INTERNET
                       │
                       ▼
                  CDN / WAF
                       │
                       ▼
                 Reverse Proxy
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
        React Frontend       API Server
                                 │
                    ┌────────────┼────────────┐
                    ▼            ▼            ▼
                 Queue        Storage       Redis
                    │
                    ▼
                  Worker
```

---

# 34. Testing

Create tests for:

## Frontend

- URL validation
- Button states
- Loading state
- Error state
- Result rendering
- Mobile layout

## Backend

- Valid URL
- Invalid URL
- Unsupported domain
- SSRF attempts
- Rate limiting
- Timeout
- File-size limits
- Expired download
- Error responses

Security tests are mandatory before production.

---

# 35. Example Frontend API Client

```ts
const API_URL = import.meta.env.VITE_API_URL;

export async function analyzeMedia(url: string) {
  const response = await fetch(`${API_URL}/api/media/analyze`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ url }),
  });

  if (!response.ok) {
    throw new Error("Unable to analyze media");
  }

  return response.json();
}
```

Do not put secret API keys in this file.

---

# 36. Example Backend Environment Validation

Use Zod to validate environment variables when the server starts.

Required values should fail fast if missing.

Example:

```ts
const envSchema = z.object({
  NODE_ENV: z.enum(["development", "production", "test"]),
  PORT: z.coerce.number().default(5000),
  FRONTEND_URL: z.string().url(),
});

export const env = envSchema.parse(process.env);
```

---

# 37. Gitignore

Create:

```gitignore
node_modules/
dist/
.env
.env.*
!.env.example
tmp/
logs/
coverage/
.DS_Store
```

---

# 38. Development Commands

Frontend:

```bash
cd client
npm install
npm run dev
```

Backend:

```bash
cd server
npm install
npm run dev
```

Production:

```bash
npm run build
npm start
```

---

# 39. Environment Setup

Frontend:

```env
VITE_API_URL=http://localhost:5000
```

Backend:

```env
PORT=5000
FRONTEND_URL=http://localhost:5173
```

Keep `.env` files private.

---

# 40. MVP Development Order

Build in this order:

### Phase 1 — Project Setup

- React + Vite + TypeScript
- Express + TypeScript
- ESLint
- Prettier
- Environment variables

### Phase 2 — UI

- Navbar
- Hero
- URL input
- Analyze button
- Result card
- Loading state
- Error state
- Footer

### Phase 3 — API

- Health endpoint
- Analyze endpoint
- Download endpoint
- Validation
- Error handling

### Phase 4 — Security

- Helmet
- CORS
- Rate limiting
- SSRF protection
- Request limits
- Timeouts
- File limits
- Secure temporary storage

### Phase 5 — Testing

- Unit tests
- API tests
- Security tests
- Mobile testing

### Phase 6 — Deployment

- Docker
- HTTPS
- Reverse proxy
- Environment variables
- Monitoring
- Cleanup worker

---

# 41. AI Coding Agent Instructions

When giving this project to an AI coding agent such as Antigravity, use these rules:

```text
Build the project according to this specification.

Use React + Vite + TypeScript for the frontend.

Use Node.js + Express + TypeScript for the backend.

Do not put secrets in frontend code.

Implement backend-side URL validation.

Implement strict Instagram hostname validation.

Implement SSRF protection.

Implement rate limiting.

Implement request timeouts.

Implement file-size limits.

Implement secure temporary storage.

Implement automatic cleanup of temporary files.

Implement Helmet security headers.

Implement restrictive CORS.

Implement centralized error handling.

Never expose stack traces or secrets to the client.

Do not store user media permanently.

Do not implement private-account access.

Do not bypass authentication, DRM, or platform security.

Only support content the user owns or is authorized to download.

Create a clean responsive UI.

Make all buttons functional.

Add loading, success, and error states.

Add API validation using Zod.

Create .env.example.

Create README.md with setup instructions.

Create tests for the main API and security validation.

Before finishing, run the build and tests and fix all errors.
```

---

# 42. Important Security Checklist

Before production, verify:

```text
[ ] HTTPS enabled
[ ] Secrets are not in React
[ ] .env is not committed
[ ] CORS restricted
[ ] Helmet enabled
[ ] Rate limiting enabled
[ ] SSRF protection tested
[ ] Private IP blocking tested
[ ] Redirect validation implemented
[ ] Request timeout enabled
[ ] Download size limit enabled
[ ] Temporary files automatically deleted
[ ] Path traversal prevented
[ ] Random temporary filenames used
[ ] Error messages sanitized
[ ] Stack traces disabled in production
[ ] Sensitive data excluded from logs
[ ] API input validated
[ ] Frontend input validated
[ ] Backend validation also implemented
[ ] Authentication bypass is not implemented
[ ] Private content access is not implemented
[ ] Copyright/authorized-use notice added
[ ] Privacy Policy added
[ ] Terms added
[ ] Security tests completed
[ ] Production build passes
```

---

# 43. Final Goal

The finished website should feel like a professional SaaS product:

```text
Fast
Modern
Responsive
Secure
Simple
Mobile-friendly
Easy to understand
```

The architecture should keep the React frontend separate from server-side processing so that security-sensitive operations remain on the backend.

Do not launch the service until the security, legal, platform-compliance, and authorization requirements have been reviewed.
