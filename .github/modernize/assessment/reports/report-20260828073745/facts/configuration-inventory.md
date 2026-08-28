# Configuration & Externalized Settings Inventory

The repository has a compact configuration surface: `package.json`, `package-lock.json`, `nodemon.json`, `Procfile`, `.gitignore`, and inline Express configuration in `app.js`. Secrets are currently supplied through environment variables for production-style runtime, with local development values present in `nodemon.json`.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| `package.json` | npm package manifest | `/package.json` | Declares runtime dependencies, development dependency, entry point, and start/debug scripts |
| `package-lock.json` | npm lockfile | `/package-lock.json` | Pins resolved transitive dependency versions using lockfile version 1 |
| `nodemon.json` | Local development environment | `/nodemon.json` | Provides local environment variables for MongoDB, port, and Stripe; sensitive values should be treated as secrets and are not reproduced here |
| `Procfile` | Process startup declaration | `/Procfile` | Defines web process startup as `node app.js` |
| `app.js` | Inline runtime configuration | `/app.js` | Configures Express middleware, static paths, view engine, session store, MongoDB connection, upload destination, and listen port |
| `.gitignore` | Repository ignore configuration | `/.gitignore` | Ignores IDE files, `node_modules`, runtime data, and access logs |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| default | `npm install` and `npm start` | Install dependencies and run the Express application | npm with dependencies from `package.json` |
| debug | `npm run debug` | Start application under nodemon with inspector enabled | nodemon 2.0.4 development dependency |

## Runtime Profiles

| Profile | Config Files | Key Overrides |
|---|---|---|
| default or hosted | Environment variables and `Procfile` | Uses environment-provided MongoDB credentials, MongoDB database name, Stripe key, and optional port |
| local debug | `nodemon.json` and `npm run debug` | Supplies MongoDB, Stripe, and port settings to nodemon for local execution |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `MONGO_USER` | None in application code | default or hosted, local debug | Environment variable; local debug value exists in `nodemon.json` and is sensitive |
| `MONGO_PWD` | None in application code | default or hosted, local debug | Environment variable; local debug value exists in `nodemon.json` and is sensitive |
| `MONGO_DB` | None in application code | default or hosted, local debug | Environment variable used to build the MongoDB database name |
| `STRIPE_KEY` | None in application code | default or hosted, local debug | Environment variable; local debug value exists in `nodemon.json` and is sensitive |
| `PORT` | `3000` | default or hosted, local debug | Environment variable controls Express listen port |
| Express view engine | `ejs` | all | `app.js` inline configuration |
| Express views directory | `views` | all | `app.js` inline configuration |
| Static assets path | `public` | all | `app.js` inline configuration |
| Uploaded image path | `images` | all | Multer storage configuration in `app.js` |
| Session collection | `sessions` | all | MongoDB session store configuration in `app.js` |
| Session secret | Hardcoded string in source | all | `app.js`; should be externalized before production use |
| Access log path | `access.log` | all | Morgan file stream configuration in `app.js` |
| Invoice output path | `data/invoices` | all | Invoice generation path in `controllers/shop.js` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| Node.js shopping web app | `node app.js` for normal startup; `nodemon --inspect app.js` for debug startup | Not specified | Not specified |

## Startup Dependency Chain

1. MongoDB Atlas must be reachable before the Express application begins listening; server startup occurs only after the Mongoose connection succeeds.
2. The Express process initializes middleware, routes, session storage, CSRF handling, and static file serving before accepting traffic.
3. Stripe and SMTP availability are not checked at startup; related workflows fail at request time if those external services are unavailable.
4. No Docker Compose `depends_on`, health checks, Kubernetes readiness probes, or startup timeout configuration were detected.

## Secrets & Sensitive Configuration

### Critical Security Findings

| Finding | Severity | Evidence | Recommended Action |
|---|---|---|---|
| Hardcoded session secret | High | `app.js` configures a literal session signing secret | Move the session secret to an environment variable or managed secret store and rotate the deployed value |
| Hardcoded SMTP credentials | High | `controllers/auth.js` constructs the mail transporter with literal credentials | Move SMTP username and password to environment variables or a managed secret store and rotate the exposed credentials |
| Local debug secrets committed in configuration | High | `nodemon.json` contains local MongoDB and Stripe secret values | Replace local secret values with documented placeholders or untracked local environment files and rotate exposed credentials |

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `MONGO_USER` | Database username | Environment variable and local `nodemon.json` value `[MASKED]` |
| `MONGO_PWD` | Database password | Environment variable and local `nodemon.json` value `[MASKED]` |
| `STRIPE_KEY` | Payment API key | Environment variable and local `nodemon.json` value `[MASKED]` |
| SMTP username | Email service username | Literal hardcoded credential in `controllers/auth.js` value `[MASKED]`; high-priority externalization candidate |
| SMTP password | Email service password | Literal hardcoded credential in `controllers/auth.js` value `[MASKED]`; high-priority externalization candidate |
| Express session secret | Session signing secret | Hardcoded in `app.js` value `[MASKED]` |

### Secrets Provisioning Workflow

For hosted execution, secrets are expected to be injected as environment variables before the Node.js process starts. Local debug execution loads values from `nodemon.json`. No managed identity, cloud secret store, encrypted property mechanism, or CI-based secret provisioning workflow was detected. The application binds database and Stripe secrets during module initialization and uses literal hardcoded SMTP credentials when constructing the mail transporter; these SMTP values should be moved to environment variables or a managed secret store before production use.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | No feature flag framework, conditional configuration toggle, or runtime flag source was identified |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---:|---|
| Node.js | 14 documented | `README.md` |
| npm | lockfile version 1 | `package-lock.json` |
| Express | 4.17.1 | `package.json` |
| EJS | 3.1.3 | `package.json` |
| Mongoose | 5.9.17 | `package.json` |
| MongoDB driver | 3.5.8 | `package.json` |
| Stripe SDK | 8.60.0 | `package.json` |
| Helmet | 3.23.0 | `package.json` |
| csurf | 1.11.0 | `package.json` |
| express-session | 1.17.1 | `package.json` |
| Nodemailer | 6.4.8 | `package.json` |
| Multer | 1.4.2 | `package.json` |
| PDFKit | 0.11.0 | `package.json` |
