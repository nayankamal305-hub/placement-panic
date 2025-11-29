# Backend Setup Guide - PlacementPanic

Detailed instructions for running the PlacementPanic backend locally and deploying to production.

## Quick Start

```bash
# Clone and install
git clone https://github.com/nayankamal305-hub/placement-panic.git
cd placement-panic
npm install

# Start development server
npm run dev

# Server runs on port 5173 (full-stack)
```

## Backend Architecture

### Technology Stack
- **Runtime**: Node.js (v18+)
- **Framework**: Express.js
- **Language**: TypeScript
- **ORM**: Drizzle ORM
- **Database**: PostgreSQL (with Neon serverless option)
- **Authentication**: JWT + bcryptjs
- **Validation**: Zod + Drizzle schemas

### Server Files Structure

```
server/
├── index.ts           # Express server setup, middleware, port listening
├── routes.ts          # All API route definitions and controllers
├── storage.ts         # Database layer (currently in-memory, Drizzle-ready)
├── static.ts          # Static file serving configuration
└── vite.ts            # Vite dev server integration
```

## Backend Code Overview

### 1. server/index.ts
Main server entry point that:
- Creates Express app instance
- Sets up middleware (CORS, JSON parsing)
- Mounts API routes
- Serves static files
- Logs requests with timestamps
- Listens on port 5173

### 2. server/routes.ts
Defines all API endpoints:

**Auth Routes**:
- `POST /api/auth/signup` - Register user
  - Body: `{ email, password, name, college?, year?, targetRole? }`
  - Returns: `{ token: string, user: { id, email, name } }`
  
- `POST /api/auth/login` - Login user
  - Body: `{ email, password }`
  - Returns: `{ token: string, user: { id, email, name } }`

**Question Routes**:
- `GET /api/questions` - Fetch questions
  - Query: `?category=DSA&difficulty=2`
  - Returns: `{ questions: [] }`

**Interview Routes**:
- `POST /api/interviews/start` - Start new interview
  - Body: `{ category, difficulty, duration }`
  - Auth: Required (Bearer token)
  - Returns: `{ interviewId, questions: [] }`
  
- `POST /api/interviews/complete` - Save interview results
  - Body: `{ interviewId, questionsAnswered, ratings[], averageRating }`
  - Auth: Required
  - Returns: `{ interviewId, feedback, metrics }`
  
- `GET /api/interviews` - Get user's interviews
  - Auth: Required
  - Returns: `{ interviews: [] }`

### 3. server/storage.ts
Data persistence layer:

**Current Implementation** (In-Memory):
- Stores users, questions, and interview records in RAM
- Perfect for development and testing
- Data resets on server restart

**Structure**:
```typescript
interface User {
  id: string;
  email: string;
  password: string; // bcrypt hashed
  name: string;
  college?: string;
  year?: string;
  targetRole?: string;
}

interface Question {
  id: string;
  text: string;
  category: 'DSA' | 'Web' | 'Java' | 'System Design' | 'HR';
  difficulty: 1 | 2 | 3 | 4 | 5;
}

interface Interview {
  id: string;
  userId: string;
  category: string;
  difficulty: number;
  duration: number;
  questionsAnswered: number;
  totalQuestions: number;
  averageRating: number;
  ratings: number[];
  completedAt: Date;
}
```

## Key Backend Functions

### Authentication

```typescript
// Password Hashing (bcrypt with 10 salt rounds)
const hashedPassword = await bcrypt.hash(password, 10);
const isValid = await bcrypt.compare(password, hashedPassword);

// JWT Token Generation
const token = jwt.sign(
  { userId: user.id, email: user.email },
  process.env.JWT_SECRET || 'fallback-secret',
  { expiresIn: '7d' }
);
```

### Protected Route Middleware

```typescript
function authenticateToken(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) return res.sendStatus(401);
  
  try {
    const user = jwt.verify(token, process.env.JWT_SECRET);
    req.user = user;
    next();
  } catch (err) {
    res.sendStatus(403);
  }
}
```

### Question Filtering

```typescript
function getQuestions(category?, difficulty?) {
  let filtered = QUESTIONS;
  
  if (category) {
    filtered = filtered.filter(q => q.category === category);
  }
  
  if (difficulty) {
    filtered = filtered.filter(q => q.difficulty === Number(difficulty));
  }
  
  // Shuffle and return random 5
  return filtered.sort(() => Math.random() - 0.5).slice(0, 5);
}
```

## Switching to PostgreSQL

### Step 1: Install Drizzle PostgreSQL
```bash
npm install drizzle-orm @neondatabase/serverless
npm install -D drizzle-kit
```

### Step 2: Update server/storage.ts
```typescript
import { drizzle } from 'drizzle-orm/neon-http';
import { neon } from '@neondatabase/serverless';

const sql = neon(process.env.DATABASE_URL!);
const db = drizzle(sql);
```

### Step 3: Create Drizzle Migrations
```bash
npx drizzle-kit generate:pg
npx drizzle-kit migrate
```

### Step 4: Configure drizzle.config.ts
```typescript
import type { Config } from 'drizzle-kit';

export default {
  schema: './server/schema.ts',
  out: './server/migrations',
  driver: 'pg',
  dbCredentials: {
    connectionString: process.env.DATABASE_URL!,
  },
} satisfies Config;
```

## Environment Variables

Create `.env` file:
```
# Development
NODE_ENV=development
DATABASE_URL=postgresql://user:password@localhost:5432/placement_panic
JWT_SECRET=your-super-secret-jwt-key-here

# Optional: For production
PORT=5173
LOG_LEVEL=info
```

## Running Locally

### Development Mode (with hot reload)
```bash
npm run dev

# Logs:
# ✓ Express listening on port 5173
# ✓ Frontend: http://localhost:5173
# ✓ API: http://localhost:5173/api
```

### Production Build
```bash
npm run build
npm start

# Logs:
# ✓ Server built and running on port 5173
```

## Testing the Backend

### Using cURL

**Signup**:
```bash
curl -X POST http://localhost:5173/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{
    "email": "student@example.com",
    "password": "secure123",
    "name": "John Doe",
    "college": "IIT Delhi",
    "year": "3rd",
    "targetRole": "SDE"
  }'
```

**Login**:
```bash
curl -X POST http://localhost:5173/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "student@example.com",
    "password": "secure123"
  }'
```

**Get Questions** (with auth):
```bash
curl -X GET "http://localhost:5173/api/questions?category=DSA&difficulty=2" \
  -H "Authorization: Bearer <JWT_TOKEN>"
```

## Performance Tuning

### TypeScript Compilation
- Uses `tsx` for fast TypeScript execution in development
- `esbuild` for optimized production builds
- No additional compilation overhead

### Database Optimization (when switching to PostgreSQL)
```typescript
// Add indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_questions_category ON questions(category, difficulty);
CREATE INDEX idx_interviews_user_id ON interviews(user_id);

// Connection pooling (handled by Neon serverless)
```

## Deployment

### Deploy to Vercel (Recommended)
```bash
npm install -g vercel
vercel
```

### Deploy to Railway
```bash
npm install -g @railway/cli
railway link
railway up
```

### Deploy to Render
1. Connect GitHub repo
2. Create new Web Service
3. Set environment variables
4. Deploy

## Troubleshooting

### Port 5173 already in use
```bash
# Kill process on port 5173
lsof -ti:5173 | xargs kill -9

# Or use different port
PORT=3000 npm run dev
```

### JWT verification fails
- Ensure `JWT_SECRET` environment variable is set
- Check token expiration (default: 7 days)
- Verify Authorization header format: `Bearer <token>`

### Database connection errors
- Verify `DATABASE_URL` format
- Check PostgreSQL is running (if local)
- Ensure network access allowed to Neon database

## API Rate Limiting

For production, add rate limiting:
```bash
npm install express-rate-limit
```

```typescript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});

app.use('/api/', limiter);
```

## Additional Resources

- [Express.js Docs](https://expressjs.com/)
- [Drizzle ORM Docs](https://orm.drizzle.team/)
- [JWT.io](https://jwt.io/)
- [Neon PostgreSQL](https://neon.tech/)
- [bcryptjs Docs](https://www.npmjs.com/package/bcryptjs)

---

For questions or issues, please open a GitHub Issue or contact nayankamal305@example.com
