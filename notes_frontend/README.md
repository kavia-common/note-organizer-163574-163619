# Notes Frontend (Svelte)

A minimalistic, responsive notes web app built with SvelteKit. Features:
- User authentication (login/register)
- Create, view, edit, and delete notes
- Organize notes with folders and tags
- Clean light theme with primary/secondary/accent colors
- RESTful API integration with .env configuration

## Quickstart

1. Install dependencies
   npm install

2. Configure environment
   cp .env.example .env
   # Edit .env to set VITE_API_BASE_URL to your backend URL

3. Run the dev server
   npm run dev

The app will be available on port 3000.

## Environment variables

- VITE_API_BASE_URL: Backend base URL (e.g., http://localhost:8000)
- VITE_APP_TITLE: Optional app title
- VITE_USE_MOCK: Set to true to enable mock mode (API calls will throw)

## Structure

- src/lib/services/api.ts: REST API client
- src/lib/stores/auth.ts: Authentication store with localStorage persistence
- src/lib/stores/notes.ts: Notes state, filters, derived stores
- src/lib/components/*: UI components (Header, Sidebar, NoteList, NoteEditor, Auth forms)
- src/routes/*: Pages (+layout, dashboard, login, register)

## Notes

- The app expects a backend exposing:
  - POST /auth/login, POST /auth/register, GET /auth/me
  - GET/POST /notes, GET/PUT/DELETE /notes/:id
  - GET/POST /folders, DELETE /folders/:id
- Authentication uses Bearer token in Authorization header plus include credentials.

## Theming

- Colors used:
  - primary: #3366ff
  - secondary: #3d3d3d
  - accent: #ffd600

## License

MIT
