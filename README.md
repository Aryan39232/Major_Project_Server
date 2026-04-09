# BrandCollab Platform - Backend API

This is the backend API server for the BrandCollab Platform, built with Node.js, Express, and MongoDB.

## 🚀 Quick Start

### Prerequisites

- Node.js v16 or higher
- MongoDB (local installation or MongoDB Atlas account)
- npm or yarn

### Installation

1. **Install dependencies**
   ```bash
   npm install
   ```

2. **Set up environment variables**
   ```bash
   cp .env.example .env
   # Edit .env with your actual values
   ```

3. **Start the server**
   ```bash
   # Development mode with auto-reload
   npm run dev
   
   # Production mode
   npm start
   ```

The server will start on `http://localhost:3500`

## 📁 Project Structure

```
Major_Project_Server_Enhanced/
├── config/              # Configuration files
│   ├── corsOptions.js   # CORS settings
│   └── dbConn.js        # Database connection
├── controllers/         # Route controllers
│   ├── authController.js
│   ├── projectController.js
│   └── userController.js
├── middleware/          # Custom middleware
│   ├── auth.js          # Authentication middleware
│   ├── errorHandler.js  # Error handling
│   └── logger.js        # Request logging
├── models/              # Mongoose models
│   ├── User.js
│   ├── Project.js
│   └── Payment.js
├── routes/              # API routes
│   ├── authRoutes.js
│   ├── projectRoutes.js
│   └── userRoutes.js
├── utils/               # Utility functions
│   ├── s3Upload.js      # AWS S3 helper
│   └── emailService.js  # Email utilities
├── server.js            # Entry point
└── package.json
```

## 🔌 API Endpoints

### Authentication

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/register` | Register new user | No |
| POST | `/api/auth` | Login user | No |
| GET | `/api/refresh` | Refresh access token | No (Refresh token in cookie) |
| POST | `/api/logout` | Logout user | Yes |

### Users

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/api/users` | Get all users | Yes (Admin) |
| GET | `/api/users/:id` | Get user by ID | Yes |
| PATCH | `/api/users/:id` | Update user | Yes |
| DELETE | `/api/users/:id` | Delete user | Yes |

### Projects

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/api/projects` | Get all projects | Yes |
| GET | `/api/projects/:id` | Get project by ID | Yes |
| POST | `/api/projects` | Create project | Yes (Brand) |
| PATCH | `/api/projects/:id` | Update project | Yes |
| DELETE | `/api/projects/:id` | Delete project | Yes |

### File Upload

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/upload` | Upload file to S3 | Yes |

### Payments

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/payments/create-intent` | Create payment intent | Yes |
| POST | `/api/payments/webhook` | Stripe webhook handler | No |

## 🔐 Authentication

The API uses JWT (JSON Web Tokens) for authentication with a dual-token system:

- **Access Token**: Short-lived (15 minutes), sent in Authorization header
- **Refresh Token**: Long-lived (7 days), sent as httpOnly cookie

### Example Request

```javascript
// Login
const response = await fetch('http://localhost:3500/api/auth', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    email: 'user@example.com',
    password: 'password123'
  }),
  credentials: 'include' // Important for cookies
});

const { accessToken } = await response.json();

// Authenticated Request
const projects = await fetch('http://localhost:3500/api/projects', {
  headers: {
    'Authorization': `Bearer ${accessToken}`
  }
});
```

## 🗄️ Database Models

### User Model

```javascript
{
  username: String,
  email: String,
  password: String (hashed),
  role: String, // 'influencer', 'brand', 'admin'
  profile: {
    bio: String,
    avatar: String,
    socialMedia: {
      instagram: String,
      tiktok: String,
      youtube: String
    }
  },
  createdAt: Date,
  updatedAt: Date
}
```

### Project Model

```javascript
{
  title: String,
  description: String,
  brandId: ObjectId,
  influencerId: ObjectId,
  status: String, // 'draft', 'pending', 'active', 'completed'
  budget: Number,
  deadline: Date,
  deliverables: [{
    title: String,
    description: String,
    status: String,
    files: [String]
  }],
  payments: [{
    amount: Number,
    status: String,
    stripePaymentId: String
  }],
  createdAt: Date,
  updatedAt: Date
}
```

## 🔧 Environment Variables

See `.env.example` for all required environment variables.

### Critical Variables

- `DATABASE_URI`: MongoDB connection string
- `ACCESS_TOKEN_SECRET`: Secret for access tokens (use a strong random string)
- `REFRESH_TOKEN_SECRET`: Secret for refresh tokens (use a different strong random string)
- `AWS_ACCESS_KEY_ID`: AWS access key for S3
- `AWS_SECRET_ACCESS_KEY`: AWS secret key for S3
- `STRIPE_SECRET_KEY`: Stripe secret key for payments

### Generate Secure Secrets

```bash
# Generate random secrets
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

## 📦 Dependencies

### Production Dependencies

- `express`: Web framework
- `mongoose`: MongoDB ODM
- `bcryptjs`: Password hashing
- `jsonwebtoken`: JWT authentication
- `cors`: Cross-origin resource sharing
- `dotenv`: Environment variables
- `multer`: File upload handling
- `aws-sdk`: AWS S3 integration
- `stripe`: Payment processing
- `express-rate-limit`: Rate limiting
- `helmet`: Security headers

### Development Dependencies

- `nodemon`: Auto-restart server on changes

## 🧪 Testing

```bash
# Run tests (if implemented)
npm test

# Run with coverage
npm run test:coverage
```

## 🚀 Deployment

### Heroku Deployment

```bash
# Login to Heroku
heroku login

# Create app
heroku create your-app-name

# Set environment variables
heroku config:set DATABASE_URI="your_mongodb_uri"
heroku config:set ACCESS_TOKEN_SECRET="your_secret"
# ... set all variables

# Deploy
git push heroku main
```

### Railway Deployment

```bash
# Install Railway CLI
npm install -g @railway/cli

# Login and deploy
railway login
railway init
railway up
```

## 🔒 Security Best Practices

1. **Never commit `.env` files** - Already in .gitignore
2. **Use strong JWT secrets** - Generate with crypto.randomBytes
3. **Hash passwords** - Using bcryptjs with salt rounds >= 10
4. **Validate input** - Sanitize all user inputs
5. **Rate limiting** - Prevent brute force attacks
6. **CORS configuration** - Only allow trusted origins
7. **HTTPS only in production** - Force SSL
8. **Keep dependencies updated** - Run `npm audit` regularly

## 📊 Monitoring & Logging

```javascript
// Logs are written to console
// In production, consider using:
// - Winston for structured logging
// - Sentry for error tracking
// - New Relic for performance monitoring
```

## 🐛 Troubleshooting

### Database Connection Failed

```
Error: MongooseServerSelectionError
```

**Solutions:**
- Check MongoDB is running locally
- Verify DATABASE_URI is correct
- Ensure IP is whitelisted in MongoDB Atlas
- Check network connectivity

### JWT Authentication Errors

```
Error: jwt malformed / jwt expired
```

**Solutions:**
- Verify ACCESS_TOKEN_SECRET is set
- Check token expiration time
- Ensure Authorization header format: `Bearer <token>`

### File Upload Errors

```
Error: S3 upload failed
```

**Solutions:**
- Verify AWS credentials
- Check S3 bucket permissions
- Ensure bucket CORS is configured
- Check file size limits

## 📝 API Response Format

### Success Response

```json
{
  "success": true,
  "data": { /* response data */ }
}
```

### Error Response

```json
{
  "success": false,
  "error": "Error message",
  "details": { /* optional error details */ }
}
```

## 🤝 Contributing

See [CONTRIBUTING.md](../CONTRIBUTING.md) for contribution guidelines.

## 📄 License

This project is licensed under the MIT License - see [LICENSE](../LICENSE) file.

## 🆘 Support

For issues or questions:
- Create an issue on GitHub
- Email: support@brandcollab.com
- Documentation: [Link to docs]
