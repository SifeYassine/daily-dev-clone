# Daily.dev Clone

A full-stack clone of **[daily.dev](https://daily.dev)**, the developer news feed. Users share links to articles, the feed updates **in real time** for everyone, and posts can be discussed in live comment threads. It's built with a **Laravel 11** API and **Laravel Reverb** WebSockets on the backend, and **Next.js 15**, **NextAuth**, and **Tailwind CSS** on the frontend.

<p align="center">
  <img src="https://raw.githubusercontent.com/SifeYassine/portfolio/main/assets/images/daily_dev1.png" alt="Daily.dev clone feed" width="48%" />
  <img src="https://raw.githubusercontent.com/SifeYassine/portfolio/main/assets/images/daily_dev2.png" alt="Daily.dev clone post" width="48%" />
</p>

## Features

- **Authentication**: register (with password confirmation) and login via Laravel Sanctum, with sessions handled on the frontend by NextAuth; a verification code is emailed on sign-up
- **Profile picture upload**
- **Share posts**: paste a URL and the title, description, and cover image are pulled automatically from the page's metadata (`link-preview-js`)
- **Real-time feed**: new posts appear instantly for all users through Laravel Reverb and Laravel Echo
- **Live comments**: new comments and comment counts update in real time
- Responsive layout with navbar, sidebar, mobile sidebar, profile menu, and search input
- Toast notifications (react-toastify)

## Tech Stack

| Layer      | Technology |
| ---------- | ---------- |
| Backend    | PHP 8.2+, Laravel 11, Laravel Sanctum, Laravel Reverb |
| Database   | MySQL |
| Frontend   | Next.js 15 (App Router), React 19, TypeScript |
| Auth (FE)  | NextAuth.js |
| Realtime   | Laravel Echo + pusher-js (Reverb protocol) |
| UI         | Tailwind CSS, Radix UI (shadcn/ui), lucide-react |
| Utilities  | Axios, Moment.js, use-immer, link-preview-js |

## Project Structure

```
daily-dev-clone/
├── backend/                     # Laravel API
│   ├── app/
│   │   ├── Events/              # PostBroadCastEvent, CommentBroadCastEvent, CommentIncrement
│   │   ├── Http/Controllers/api/
│   │   │   ├── auth/  users/  posts/  comments/
│   │   ├── Mail/VerificationCodeMail.php
│   │   └── Models/              # User, Post, Comment
│   ├── database/migrations/
│   └── routes/api.php
└── frontend/                    # Next.js app
    └── src/
        ├── app/
        │   ├── (daily_dev)/     # Main feed
        │   ├── login/
        │   └── api/
        │       ├── auth/[...nextauth]/   # NextAuth config
        │       └── image-preview/        # Link-metadata extraction
        ├── components/          # auth, base (layout), post, comment, common
        ├── lib/                 # axios, echo, env, API endpoints
        ├── providers/AuthProvider.tsx
        └── middleware.ts        # Route protection
```

## Getting Started

### Prerequisites

- PHP 8.2+ and [Composer](https://getcomposer.org/)
- [Node.js](https://nodejs.org/) 18.18+ and npm
- MySQL

```bash
git clone https://github.com/SifeYassine/daily-dev-clone.git
cd daily-dev-clone
```

### 1. Backend

```bash
cd backend
composer install
npm install
cp .env.example .env
php artisan key:generate
```

Configure `backend/.env`:

```env
DB_CONNECTION=mysql
DB_DATABASE=daily_dev
DB_USERNAME=root
DB_PASSWORD=

BROADCAST_CONNECTION=reverb

REVERB_APP_ID=my-app-id
REVERB_APP_KEY=my-app-key
REVERB_APP_SECRET=my-app-secret
REVERB_HOST=localhost
REVERB_PORT=8080
REVERB_SCHEME=http

# Mail is needed for the sign-up verification code (use "log" to just write emails to the log)
MAIL_MAILER=log
```

Migrate and link storage:

```bash
php artisan migrate
php artisan storage:link
```

Run the API and the WebSocket server in separate terminals:

```bash
php artisan serve        # http://127.0.0.1:8000
php artisan reverb:start # ws://localhost:8080
```

### 2. Frontend

```bash
cd frontend
npm install
```

Create `frontend/.env.local`:

```env
NEXT_PUBLIC_API_URL=http://127.0.0.1:8000
NEXT_PUBLIC_NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-random-secret

NEXT_PUBLIC_REVERB_APP_KEY=<REVERB_APP_KEY from backend/.env>
NEXT_PUBLIC_REVERB_HOST=localhost
NEXT_PUBLIC_REVERB_PORT=8080
NEXT_PUBLIC_REVERB_SCHEME=http
```

Run:

```bash
npm run dev              # http://localhost:3000
```

## API Overview

All routes are prefixed with `/api`. Except for `register` and `login`, send `Authorization: Bearer <token>`.

| Method | Endpoint                    | Description |
| ------ | --------------------------- | ----------- |
| POST   | `/auth/register`            | `username`, `email`, `password`, `password_confirmation` |
| POST   | `/auth/login`               | `username`, `password`; returns a Sanctum token |
| GET    | `/auth/user`                | Current user |
| POST   | `/auth/logout`              | Revoke the token |
| POST   | `/users/update-profile`     | Upload `profile_image` (png/jpg/webp/svg, max 2 MB) |
| POST   | `/posts/create`             | `title`, `description`, `url`, `image_url` (broadcasts `PostBroadCastEvent`) |
| GET    | `/posts/index`              | List posts |
| POST   | `/comments/create`          | Add a comment (broadcasts `CommentBroadCastEvent` and `CommentIncrement`) |
| GET    | `/comments/index/{postId}`  | Comments for a post |

### Broadcast channels

| Channel             | Events |
| ------------------- | ------ |
| `post-broadcast`    | `PostBroadCastEvent`, `CommentIncrement` |
| `comment-broadcast` | `CommentBroadCastEvent` |

## Disclaimer

This is a learning project and is not affiliated with daily.dev.

## Author

**Sifeddine Yassine** · [@SifeYassine](https://github.com/SifeYassine)
