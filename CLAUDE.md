# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

IDURAR is an open-source ERP/CRM system built on the MERN stack (MongoDB, Express.js, React, Node.js) with Ant Design (AntD) and Redux. It handles Invoice Management, Payment Management, Quote Management, and Customer Management.

## Technology Stack

### Backend
- **Runtime**: Node.js 20.9.0, npm 10.2.4
- **Framework**: Express.js with module aliasing (`@` → `src`)
- **Database**: MongoDB with Mongoose ODM
- **Authentication**: JWT with bcryptjs
- **File Handling**: express-fileupload, multer
- **PDF Generation**: html-pdf with Pug templates
- **Email**: Resend API
- **Storage**: AWS S3 support

### Frontend
- **Framework**: React 18 with Vite
- **UI Library**: Ant Design 5
- **State Management**: Redux Toolkit with local storage persistence
- **Routing**: React Router v6
- **API Client**: Axios with custom request handlers
- **Module Aliasing**: `@` → `src`

## Development Commands

### Backend (from `backend/` directory)
```bash
npm install              # Install dependencies
npm run setup            # Run initial setup (creates admin user and default data)
npm run dev              # Start development server with nodemon (port 8888)
npm start                # Start production server
npm run upgrade          # Run upgrade script
npm run reset            # Reset database
```

### Frontend (from `frontend/` directory)
```bash
npm install              # Install dependencies
npm run dev              # Start dev server (port 3000, proxies /api to backend)
npm run dev:remote       # Start dev server with remote backend
npm run build            # Build for production
npm run lint             # Run ESLint
npm run preview          # Preview production build
```

### Default Admin Credentials (after setup)
- Email: `admin@admin.com`
- Password: `admin123`

## Docker Deployment

### Prerequisites
- Docker Engine 20.10+
- Docker Compose 2.0+

### Quick Start with Docker

1. **Clone and Configure**:
   ```bash
   git clone https://github.com/idurar/idurar-erp-crm.git
   cd idurar-erp-crm

   # Create .env file from example
   cp .env.example .env

   # Edit .env and set required variables (especially JWT_SECRET)
   # For Windows: notepad .env
   # For Linux/Mac: nano .env
   ```

2. **Build and Start All Services**:
   ```bash
   docker-compose up -d
   ```
   This will start:
   - MongoDB on internal network (not exposed)
   - Backend API on internal port 8888 (not exposed)
   - Frontend on **port 8085** (http://localhost:8085)

3. **Initialize Database** (first time only):
   ```bash
   docker-compose exec backend npm run setup
   ```

4. **Access the Application**:
   - Open browser: http://localhost:8085
   - Login with default credentials (see above)

### Docker Commands

```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# View logs
docker-compose logs -f
docker-compose logs -f backend    # Backend logs only
docker-compose logs -f frontend   # Frontend logs only

# Restart services
docker-compose restart

# Rebuild and restart after code changes
docker-compose up -d --build

# Stop and remove all data (including database)
docker-compose down -v

# Execute commands in containers
docker-compose exec backend npm run setup     # Run setup
docker-compose exec backend npm run upgrade   # Run upgrade
docker-compose exec mongodb mongosh           # Access MongoDB shell
```

### Docker Architecture

- **mongodb**: Official MongoDB 7.0 image with persistent volume
- **backend**: Node.js 20.9.0 Alpine, runs Express API
- **frontend**: Multi-stage build (Node builder + Nginx server)
- **Network**: All services on isolated bridge network
- **Volumes**:
  - `mongodb_data`: Database persistence
  - `mongodb_config`: MongoDB configuration
  - `backend_uploads`: Uploaded files

### EasyPanel Deployment

For EasyPanel deployment:
1. Create new project in EasyPanel
2. Connect to your Git repository
3. Use the docker-compose.yml file provided
4. Set environment variables in EasyPanel dashboard
5. Deploy the stack

The application will be accessible on port 8085.

### Environment Variables for Docker

See `.env.example` for all available variables. Key variables:
```bash
DATABASE=mongodb://mongodb:27017/idurar_db    # MongoDB connection string
JWT_SECRET=your_secret_key                     # REQUIRED: Change in production
PUBLIC_SERVER_FILE=http://localhost:8085/      # Frontend URL
PORT=8888                                      # Backend port (internal)
```

## Architecture Patterns

### Backend Architecture

The backend follows a modular MVC-style architecture with automatic route generation:

**Core vs App Separation**:
- `coreModels/`, `coreControllers/`, `coreRoutes/` - Framework-level entities (Admin, Settings)
- `appModels/`, `appControllers/`, `appRoutes/` - Business logic entities (Invoice, Quote, Payment, Client)

**Automatic CRUD Pattern**:
1. Create a model in `backend/src/models/appModels/{EntityName}.js`
2. The system automatically:
   - Generates controller via `createCRUDController` (unless custom controller exists in `appControllers/{entityName}Controller/`)
   - Creates routes: `/{entity}/create`, `/read/:id`, `/update/:id`, `/delete/:id`, `/search`, `/list`, `/listAll`, `/filter`, `/summary`
   - Maps entity name to controller through `models/utils/index.js`

**Custom Controllers**:
Place in `backend/src/controllers/appControllers/{controllerName}/` to override auto-generated CRUD. Existing custom controllers:
- `clientController`, `invoiceController`, `paymentController`, `paymentModeController`, `quoteController`, `taxesController`

**Route Structure** (backend/src/app.js):
- `/api/*` (auth) → `coreAuth` routes (login, register, etc.)
- `/api/*` (protected) → `coreApi` + `erpApiRouter` (requires JWT via `adminAuth.isValidAuthToken`)
- `/download/*` → File downloads
- `/public/*` → Public files

### Frontend Architecture

**Redux State Management**:
- Feature-based slices: `auth`, `crud`, `erp`, `settings`, `adavancedCrud`
- State persistence via `storePersist.js` (auth state persisted to localStorage)
- Reselect for memoized selectors

**Module Structure**:
- `modules/` - Reusable module components (AuthModule, CrudModule, DashboardModule, ErpPanelModule, InvoiceModule, PaymentModule, etc.)
- `pages/` - Route-level page components that compose modules
- `apps/` - Main app shells (ErpApp, IdurarOs)

**Request Layer**:
- Custom Axios instance in `request/request.js`
- Global success/error handlers
- Error code mapping in `codeMessage.js`

**Routing**:
Defined in `router/` using React Router v6 with protected routes and authentication guards.

## Environment Configuration

### Backend (.env)
Required variables:
```
DATABASE=mongodb://localhost:27017  # or MongoDB Atlas URI
JWT_SECRET=your_private_jwt_secret_key
NODE_ENV=production
PUBLIC_SERVER_FILE=http://localhost:8888/
RESEND_API=your_resend_api           # Optional: for email
OPENAI_API_KEY=your_openai_key       # Optional: for AI features
```

### Frontend
Uses Vite environment variables (`VITE_*` prefix):
- `VITE_DEV_REMOTE=remote` enables remote backend mode
- `VITE_BACKEND_SERVER` for remote backend URL

## Key Implementation Notes

1. **Module Aliases**: Both backend and frontend use `@` to reference the `src` directory

2. **Authentication Flow**:
   - JWT tokens stored in cookies (backend) and managed via Redux (frontend)
   - Protected routes check `adminAuth.isValidAuthToken` middleware

3. **Adding New Entities**:
   - Backend: Create model in `appModels/`, routes auto-generate
   - Frontend: Create page in `pages/`, module in `modules/`, Redux slice if needed

4. **PDF Generation**: Uses Pug templates in `backend/src/pdf/` with `html-pdf`

5. **File Uploads**: Configured in `backend/src/middlewares/uploadMiddleware`

6. **Email Templates**: Stored in `backend/src/emailTemplate/`

7. **Localization**: Translation files in `backend/src/locale/` and `frontend/src/locale/`

## Common Development Workflow

1. **Initial Setup**:
   ```bash
   # Backend
   cd backend
   npm install
   # Edit .env with MongoDB URI and JWT_SECRET
   npm run setup
   npm run dev

   # Frontend (separate terminal)
   cd frontend
   npm install
   npm run dev
   ```

2. **Adding a New Business Entity**:
   - Create Mongoose model in `backend/src/models/appModels/`
   - Create frontend page in `frontend/src/pages/`
   - Add route in frontend router
   - Routes and basic CRUD auto-generated

3. **Custom Business Logic**:
   - Create controller directory in `backend/src/controllers/appControllers/{entityName}Controller/`
   - Export methods: `create`, `read`, `update`, `delete`, `search`, `list`, `listAll`, `filter`, `summary`

## Project Structure

```
├── backend/
│   ├── src/
│   │   ├── models/          # Mongoose models (coreModels, appModels)
│   │   ├── controllers/     # Request handlers (auto-generated or custom)
│   │   ├── routes/          # Express routes (coreRoutes, appRoutes)
│   │   ├── middlewares/     # Auth, validation, file upload
│   │   ├── handlers/        # Error handlers
│   │   ├── pdf/             # PDF templates
│   │   ├── emailTemplate/   # Email templates
│   │   ├── setup/           # Setup/migration scripts
│   │   └── server.js        # Entry point
│   └── package.json
│
├── frontend/
│   ├── src/
│   │   ├── apps/            # Main application shells
│   │   ├── modules/         # Feature modules (Invoice, Payment, etc.)
│   │   ├── pages/           # Route pages
│   │   ├── redux/           # State management
│   │   ├── router/          # React Router config
│   │   ├── request/         # API client layer
│   │   ├── components/      # Reusable UI components
│   │   └── main.jsx         # Entry point
│   ├── vite.config.js
│   └── package.json
│
└── INSTALLATION-INSTRUCTIONS.md
```
