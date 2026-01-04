<h1 align="center">🦷 Dentwise – Dental Appointment Platform 🦷</h1>

A modern dental appointment booking platform built with Next.js 15, featuring AI voice assistant, appointment management, and secure authentication.

## ✨ Features

- 🏠 Modern Landing Page with Tailwind CSS & Shadcn UI
- 🔐 Authentication via Clerk (Google, GitHub, Email & Password)
- 📅 Comprehensive Appointment Booking System
- 🦷 3-Step Booking Flow (Select Doctor → Choose Time → Confirm)
- 📩 Email Notifications using Resend
- 📊 Admin Dashboard for Managing Appointments & Doctors
- 🗣️ AI Voice Agent powered by Vapi
- 💳 Subscription Payments with Clerk (Free + Premium Plans)
- 📂 PostgreSQL Database with Prisma ORM
- ⚡ Optimistic Updates with TanStack Query
- 🎨 Responsive Design with Tailwind CSS

## 🚀 Tech Stack

- **Framework:** Next.js 15 (App Router, Turbopack)
- **Language:** TypeScript
- **Database:** PostgreSQL with Prisma ORM
- **Authentication:** Clerk
- **Styling:** Tailwind CSS, Shadcn UI
- **State Management:** TanStack Query
- **Email:** Resend
- **AI Voice:** Vapi
- **Code Quality:** Biome

## 📋 Prerequisites

- Node.js 18+ 
- PostgreSQL database
- Clerk account
- Resend API key (for emails)
- Vapi account (optional, for voice features)

## 🛠️ Installation

1. Clone the repository:
```bash
git clone <your-repo-url>
cd dentwise
```

2. Install dependencies:
```bash
npm install
```

3. Set up your environment variables (see below)

4. Initialize the database:
```bash
npx prisma generate
npx prisma db push
```

5. Run the development server:
```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

## 🔐 Environment Variables

Create a `.env.local` file in the root directory:

```bash
# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
CLERK_SECRET_KEY=your_clerk_secret_key

# Database
DATABASE_URL=your_postgres_database_url

# Vapi AI Voice (Optional)
NEXT_PUBLIC_VAPI_ASSISTANT_ID=your_vapi_assistant_id
NEXT_PUBLIC_VAPI_API_KEY=your_vapi_api_key

# Admin Configuration
ADMIN_EMAIL=your_admin_email

# Email Service
RESEND_API_KEY=your_resend_api_key

# App URL
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## 📦 Available Scripts

```bash
npm run dev      # Start development server with Turbopack
npm run build    # Build for production
npm run start    # Start production server
npm run lint     # Run Biome linter
npm run format   # Format code with Biome
```

## 📁 Project Structure

```
├── prisma/              # Database schema
├── public/              # Static assets
├── src/
│   ├── app/            # Next.js app router pages
│   ├── components/     # React components
│   ├── hooks/          # Custom React hooks
│   └── lib/            # Utility functions & configurations
```

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is open source and available under the MIT License.
