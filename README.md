# PlacementPanic - AI Mock Interview Platform

## Overview

PlacementPanic is a full-stack web application designed to help computer science students overcome interview anxiety through realistic mock interviews, AI-powered feedback, and comprehensive performance analytics. The platform provides timed interview sessions across multiple categories (DSA, Web Development, Java, System Design, HR) with self-rating and detailed performance reports.

## Live Demo
- **Replit Live**: https://replit.com/@nayankamal305/VividCanvas
- **GitHub Repo**: https://github.com/nayankamal305-hub/placement-panic

## Key Features

- **User Authentication**: Secure signup/login with JWT and bcrypt password hashing
- **Question Bank**: 54+ pre-seeded questions across multiple difficulty levels and categories
- **Mock Interview Sessions**: Timed interviews with category/difficulty selection
- **Self-Rating System**: Rate your responses (1-5 scale) after each question
- **Performance Analytics**: Track total interviews, average score, best score, and panic reduction metrics
- **AI Feedback Generator**: Get personalized strengths and improvement suggestions
- **Responsive Design**: Beautiful glassmorphism UI with smooth animations
- **Fast Performance**: Optimized with Vite and modern web technologies

## Project Structure

```
placement-panic/
├── client/                   # React Frontend
│   ├── src/
│   │   ├── components/       # Reusable UI components
│   │   ├── pages/            # Route pages (Auth, Dashboard, InterviewSetup, etc.)
│   │   ├── lib/              # Utilities, API client, state management
│   │   └── index.css         # Global styles + Tailwind
│   ├── index.html
│   └── vite.config.ts
├── server/                   # Express Backend
│   ├── index.ts              # Server entry point
│   ├── routes.ts             # API route definitions
│   ├── storage.ts            # In-memory data persistence
│   ├── static.ts             # Static file serving
│   └── vite.ts               # Dev server configuration
├── shared/                   # Shared types and schemas
│   └── schema.ts             # Zod validation schemas
├── package.json              # Dependencies and scripts
├── tailwind.config.ts        # Tailwind CSS configuration
├── tsconfig.json             # TypeScript configuration
├── vite.config.ts            # Vite configuration
└── replit.md                 # Detailed architecture documentation
```

## Tech Stack

### Frontend
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite (lightning-fast bundling)
- **State Management**: Zustand (auth state, persisted to localStorage)
- **Server State**: TanStack Query (React Query) for caching
- **Styling**: Tailwind CSS with glassmorphism effects
- **UI Components**: shadcn/ui + Radix UI primitives
- **Routing**: Wouter (lightweight client-side routing)
- **Forms**: React Hook Form with Zod validation
- **Icons**: Lucide React

### Backend
- **Runtime**: Node.js with Express.js
- **Language**: TypeScript
- **Authentication**: JWT (jsonwebtoken) + bcryptjs
- **Database/ORM**: Drizzle ORM with @neondatabase/serverless (PostgreSQL support)
- **Data Storage**: In-memory (MongoDB-ready architecture)
- **Validation**: Drizzle + Zod schemas

## Getting Started

### Prerequisites
- **Node.js** 18 or higher
- **npm** or **yarn**
- **Git** (for cloning)

### Installation

1. **Clone the repository**:
```bash
git clone https://github.com/nayankamal305-hub/placement-panic.git
cd placement-panic
```

2. **Install dependencies**:
```bash
npm install
```

3. **Set up environment** (optional, for PostgreSQL):
Create a `.env` file:
```
DATABASE_URL=postgresql://user:password@localhost:5432/placement_panic
NODE_ENV=development
```

4. **Run the development server**:
```bash
npm run dev
```

The application will start at `http://localhost:5173` (frontend) and `http://localhost:5173/api` (backend).

## Available Scripts

```bash
# Development (HMR enabled)
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Type checking
npm run tsc

# Lint and format checks
npm run check
```

## API Endpoints

### Authentication
- `POST /api/auth/signup` - Register new user
- `POST /api/auth/login` - Login and get JWT token

### Questions
- `GET /api/questions` - Fetch all questions with optional filters
- `GET /api/questions?category=DSA&difficulty=2` - Filtered questions

### Interviews
- `POST /api/interviews/start` - Begin new interview session
- `POST /api/interviews/complete` - Save interview results and generate feedback
- `GET /api/interviews` - Get user's interview history (protected)
- `GET /api/interviews/:id` - Get specific interview details (protected)

## Database Schema

### Users Table
- `id` (UUID, Primary Key)
- `email` (String, Unique)
- `password` (Hashed)
- `name` (String)
- `college` (String, Optional)
- `year` (String, Optional)
- `targetRole` (String, Optional)

### Questions Table
- `id` (UUID, Primary Key)
- `text` (String)
- `category` (String: DSA, Web, Java, System Design, HR)
- `difficulty` (Number: 1-5)

### Interviews Table
- `id` (UUID, Primary Key)
- `userId` (Foreign Key → Users)
- `category` (String)
- `difficulty` (Number)
- `duration` (Number - seconds)
- `questionsAnswered` (Number)
- `totalQuestions` (Number)
- `averageRating` (Number: 1-5)
- `ratings` (JSON Array)
- `completedAt` (DateTime)

## Authentication Flow

1. User signs up/logs in via the frontend form
2. Server validates credentials and hashes password with bcrypt (10 salt rounds)
3. JWT token is generated and sent to client
4. Client stores token in Zustand store (persisted to localStorage)
5. Subsequent requests include token in `Authorization: Bearer <token>` header
6. Server middleware validates token before allowing access to protected routes

## Running Locally

### Quick Start (Development)

```bash
# Clone and setup
git clone https://github.com/nayankamal305-hub/placement-panic.git
cd placement-panic
npm install

# Start dev server
npm run dev

# Open browser
# Frontend: http://localhost:5173
# API: http://localhost:5173/api
```

### Using PostgreSQL (Production-Ready)

1. Set up a PostgreSQL database (or use Neon Serverless)
2. Update `.env` with your database URL
3. Modify `server/storage.ts` to use Drizzle with Neon driver
4. Run migrations (if implemented)
5. Start the server

## How It Works

1. **User Registration**: Create account with college, year, and target role
2. **Dashboard**: View interview history, statistics, and trending performance
3. **Interview Setup**: Select category (DSA, Web, etc.), difficulty level, and duration
4. **Interview Session**: Answer questions one by one with a countdown timer
5. **Self-Rating**: Rate your response quality (1-5) after answering
6. **Performance Report**: View detailed metrics and AI-generated feedback
7. **Analytics**: Track progress over multiple interviews

## Customization

### Add More Questions
Edit `server/storage.ts` and add questions to the QUESTIONS array:
```typescript
{
  id: "q123",
  text: "Your question here?",
  category: "DSA",
  difficulty: 3
}
```

### Modify UI Colors
Update `tailwind.config.ts` to customize the color palette:
```typescript
colors: {
  primary: '#6366F1',   // Indigo
  secondary: '#8B5CF6', // Purple
  accent: '#06B6D4'     // Cyan
}
```

### Switch to PostgreSQL
1. Install Neon client: `npm install @neondatabase/serverless`
2. Update `server/storage.ts` imports
3. Configure connection in `drizzle.config.ts`

## Performance Optimizations

- **Vite**: Ultra-fast build times and HMR
- **Code Splitting**: Automatic route-based code splitting
- **TanStack Query**: Automatic caching and request deduplication
- **Tailwind CSS**: Minimal CSS output with PurgeCSS
- **Image Optimization**: Hero images from Replicate AI
- **Type Safety**: TypeScript catches errors at build time

## Future Enhancements

- [ ] Video recording of responses
- [ ] Real-time AI feedback during interviews
- [ ] Peer comparison and leaderboards
- [ ] Email notifications and progress reports
- [ ] Integration with LeetCode problems
- [ ] Mobile app (React Native)
- [ ] Live interviewer feature
- [ ] Export reports as PDF

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

MIT License - feel free to use this for personal or commercial projects.

## Support

For issues, feature requests, or questions:
- Open a GitHub Issue
- Contact: nayankamal305@example.com

## Acknowledgments

- Built with ❤️ for CS students preparing for interviews
- Inspired by platforms like Stripe, Apple's design, and modern web standards
- Special thanks to the React, Express, and TypeScript communities

---

**Made with love by** [@nayankamal305](https://github.com/nayankamal305-hub)
