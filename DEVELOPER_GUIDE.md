# Tea World AI - Developer Guide

## Getting Started

This guide will help you set up and start working on the Tea World AI project, a comprehensive management system for milk tea shops that integrates AI capabilities using Supabase as the complete backend solution.

### Prerequisites

- Node.js (v18+)
- npm or yarn
- Supabase account
- OpenAI API key

### Project Structure

The project consists of two main components:
- `client/`: Next.js application for customers
- `admin/`: Next.js application for shop management

## Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/tea-world-ai.git
cd tea-world-ai
```

### 2. Set Up Supabase Project

1. Create a new Supabase project at https://app.supabase.io
2. Note your project URL and anon/service keys
3. Set up database schema (see DATABASE_SETUP.sql in the repo)
4. Enable necessary extensions: pgvector for AI embeddings 

### 3. Set Up Environment Variables

Create `.env.local` files in each directory:

**client/.env.local**
```
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
```

**admin/.env.local**
```
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
```

### 4. Deploy Edge Functions

```bash
# Install Supabase CLI if you haven't already
npm install -g supabase

# Login to Supabase
supabase login

# Link to your project
supabase link --project-ref your-project-ref

# Deploy Edge Functions (after navigating to the supabase/functions directory)
cd supabase/functions
supabase functions deploy ai-recommend
supabase functions deploy ai-chat
supabase functions deploy ai-generate-description
supabase functions deploy ai-marketing-content
supabase functions deploy dashboard-summary
supabase functions deploy auth-anonymous-session
supabase functions deploy order-anonymous
```

### 5. Install Dependencies

```bash
# Client setup
cd client
npm install

# Admin setup
cd ../admin
npm install
```

### 6. Set Up Authentication

1. Configure authentication providers in Supabase dashboard
2. Set up email templates for confirmation, reset password, etc.
3. Configure Row Level Security (RLS) policies
4. Set up anonymous session handling

### 7. Run Development Servers

```bash
# Start client (in client directory)
npm run dev

# Start admin (in admin directory)
npm run dev
```

The client will run on http://localhost:3000
The admin will run on http://localhost:3001

## Development Guidelines

### Code Style

- Follow the established project conventions
- Use TypeScript for type safety
- Implement component-based architecture
- Write unit tests for critical functionality

### Database & Supabase Best Practices

- Use Row Level Security (RLS) for all tables
- Create appropriate policies for different user roles
- Use foreign key relationships for data integrity
- Keep Edge Functions concise and focused on single responsibilities
- Use Supabase Realtime for live updates where appropriate

### Implementing Anonymous Sessions

1. **Session Initialization**:
   ```typescript
   // Initialize anonymous session when user first visits
   export const initAnonymousSession = async () => {
     const { data, error } = await supabase.functions.invoke('auth-anonymous-session', {
       body: { action: 'create' }
     });
     
     if (data?.guestId) {
       localStorage.setItem('guestId', data.guestId);
       return data.guestId;
     }
     return null;
   };
   ```

2. **Attaching Guest ID to Requests**:
   ```typescript
   // Utility to include guest ID in all Supabase requests
   export const getAuthContext = () => {
     // Check for authenticated user first
     const user = supabase.auth.user();
     if (user) return { userId: user.id };
     
     // Fall back to guest ID if not authenticated
     const guestId = localStorage.getItem('guestId');
     return guestId ? { guestId } : {};
   };
   
   // Example usage
   const makeRequest = async () => {
     const authContext = getAuthContext();
     const { data, error } = await supabase.functions.invoke('some-function', {
       body: {
         ...otherParams,
         ...authContext
       }
     });
   };
   ```

3. **Anonymous Order Checkout**:
   ```typescript
   export const createAnonymousOrder = async (orderDetails, contactInfo) => {
     const guestId = localStorage.getItem('guestId');
     if (!guestId) {
       // Handle missing guest ID
       return { error: "Session expired" };
     }
     
     const { data, error } = await supabase.functions.invoke('order-anonymous', {
       body: {
         orderDetails,
         contactInfo,
         guestId
       }
     });
     
     return { data, error };
   };
   ```

4. **Converting Anonymous to Registered User**:
   ```typescript
   export const convertGuestToUser = async (email, password, name) => {
     const guestId = localStorage.getItem('guestId');
     
     // Register new user
     const { user, error } = await supabase.auth.signUp({
       email,
       password,
     });
     
     if (user) {
       // Associate guest data with new user
       await supabase.functions.invoke('auth-anonymous-session', {
         body: {
           action: 'convert',
           guestId,
           userId: user.id
         }
       });
       
       // Clear guest ID from storage
       localStorage.removeItem('guestId');
     }
     
     return { user, error };
   };
   ```

### AI Integration

When working with AI features:

1. Always use environment variables for API keys in Edge Functions
2. Implement request caching to reduce OpenAI API costs
3. Include fallbacks for when AI services are unavailable
4. Follow AI ethics guidelines in the AI_FEATURES.md document
5. Store embeddings using pgvector for efficient similarity searches
6. Handle both authenticated and anonymous users in AI endpoints

### Branching Strategy

- `main`: Production-ready code
- `develop`: Integration branch for features
- `feature/feature-name`: For new features
- `bugfix/bug-description`: For bug fixes

### Pull Request Process

1. Create a PR against the develop branch
2. Fill out the PR template thoroughly
3. Ensure all tests pass
4. Request reviews from at least one team member
5. Address review comments before merging

## Available Scripts

### Client & Admin

- `npm run dev`: Start development server
- `npm run build`: Build for production
- `npm run start`: Run production build
- `npm run lint`: Check code style
- `npm run test`: Run tests

### Supabase Edge Functions

- `supabase functions serve`: Run functions locally for development
- `supabase functions deploy`: Deploy functions to production
- `supabase functions delete`: Remove deployed functions

## Supabase Management

- Use the Supabase dashboard to monitor database usage
- Check function logs for debugging Edge Functions
- Set up monitoring for database performance
- Create database backups regularly
- Monitor anonymous session metrics for optimization

## Resources

- [Project Structure](./PROJECT_STRUCTURE.md)
- [AI Features Documentation](./AI_FEATURES.md)
- [UI Mockups](./UI_MOCKUPS.md)
- [Next.js Documentation](https://nextjs.org/docs)
- [Supabase Documentation](https://supabase.io/docs)
- [Supabase Edge Functions](https://supabase.com/docs/guides/functions)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [pgvector Documentation](https://github.com/pgvector/pgvector)