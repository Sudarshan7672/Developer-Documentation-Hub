# JWT Authentication Backend

A robust Node.js backend implementation with JWT (JSON Web Token) authentication, built with Express.js and MongoDB.

## 🚀 Features

- **JWT Authentication**: Secure token-based authentication system
- **User Management**: Complete user registration and login system
- **Protected Routes**: Middleware-based route protection
- **Password Security**: Encrypted password storage
- **API Documentation**: Swagger/OpenAPI documentation
- **Environment Configuration**: Secure environment variable management
- **Passport Integration**: Advanced authentication strategies

## 📁 Project Structure

```
backend/
├── config/
│   ├── constants.js          # Application constants and configuration
│   └── db.js                 # MongoDB connection configuration
├── controllers/
│   └── exampleController.js  # Route handlers and business logic
├── models/
│   └── userSchema.js         # MongoDB user schema definition
├── routes/
│   └── exampleRoutes.js      # API route definitions
├── middleware/
│   └── authMiddleware.js     # JWT authentication middleware
├── utils/
│   └── helper.js             # Utility functions and helpers
├── server.js                 # Application entry point
├── swagger.yaml              # API documentation
├── passportconfig.js         # Passport.js configuration
├── .env                      # Environment variables (not in version control)
└── package.json              # Dependencies and scripts
```

## 🛠️ Installation & Setup

### Prerequisites

- Node.js (v14 or higher)
- MongoDB (local or cloud instance)
- npm or yarn package manager

### Installation Steps

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd backend
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Environment Configuration**
   
   Create a `.env` file in the root directory with the following variables:
   ```env
   # Server Configuration
   PORT=3000
   NODE_ENV=development
   
   # Database Configuration
   DB_URI=mongodb://localhost:27017/your-database-name
   
   # JWT Configuration
   JWT_SECRET=your-super-secret-jwt-key
   JWT_EXPIRES_IN=7d
   
   # Passport Configuration (if using OAuth)
   GOOGLE_CLIENT_ID=your-google-client-id
   GOOGLE_CLIENT_SECRET=your-google-client-secret
   ```

4. **Start the server**
   
   Make sure to initialize the database connection in your `server.js`:
   ```javascript
   const connectDB = require('./config/db');
   
   // Connect to MongoDB
   connectDB();
   ```

   Then start the server:
   ```bash
   # Development mode
   npm run dev
   
   # Production mode
   npm start
   ```

## 🔐 Authentication Flow

### User Registration
```http
POST /api/auth/register
Content-Type: application/json

{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "securePassword123"
}
```

### User Login
```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "john@example.com",
  "password": "securePassword123"
}
```

### Response Format
```json
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "user_id",
    "username": "john_doe",
    "email": "john@example.com"
  }
}
```

### Accessing Protected Routes
```http
GET /api/protected-route
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## 🔧 Key Components

### Database Connection (`config/db.js`)
- MongoDB connection using Mongoose
- Environment-based URI configuration
- Connection error handling with process exit
- Connect-mongo store integration for sessions
- Connection status logging with emojis

### Authentication Middleware (`middleware/authMiddleware.js`)
- Validates JWT tokens
- Extracts user information from tokens
- Protects routes from unauthorized access

### User Schema (`models/userSchema.js`)
- MongoDB user model
- Password encryption
- User validation rules

### Controllers (`controllers/exampleController.js`)
- Business logic for authentication
- User management operations
- Error handling

### Routes (`routes/exampleRoutes.js`)
- API endpoint definitions
- Route-level middleware application
- Request/response handling

## 📚 API Documentation

Access the interactive API documentation at:
```
http://localhost:3000/api-docs
```

The documentation is generated from `swagger.yaml` and provides:
- Endpoint descriptions
- Request/response examples
- Authentication requirements
- Schema definitions

## 🛡️ Security Features

- **Password Hashing**: Bcrypt for secure password storage
- **JWT Tokens**: Stateless authentication tokens
- **Environment Variables**: Sensitive data protection
- **CORS Configuration**: Cross-origin request handling
- **Input Validation**: Request data validation
- **Error Handling**: Secure error responses

## 🧪 Testing

### Run Tests
```bash
# Run all tests
npm test

# Run tests with coverage
npm run test:coverage

# Run tests in watch mode
npm run test:watch
```

### Test Authentication Endpoints

1. **Register a new user**
   ```bash
   curl -X POST http://localhost:3000/api/auth/register \
     -H "Content-Type: application/json" \
     -d '{"username":"testuser","email":"test@example.com","password":"password123"}'
   ```

2. **Login with credentials**
   ```bash
   curl -X POST http://localhost:3000/api/auth/login \
     -H "Content-Type: application/json" \
     -d '{"email":"test@example.com","password":"password123"}'
   ```

3. **Access protected route**
   ```bash
   curl -X GET http://localhost:3000/api/protected \
     -H "Authorization: Bearer YOUR_JWT_TOKEN"
   ```

## 🚀 Deployment

### Environment Variables for Production
```env
NODE_ENV=production
PORT=80
   DB_URI=mongodb+srv://username:password@cluster.mongodb.net/database
JWT_SECRET=your-production-secret-key
```

### Docker Deployment (Optional)
```dockerfile
FROM node:16-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

## 🔄 Common Use Cases

### Protecting Routes
```javascript
const { authenticateToken } = require('./middleware/authMiddleware');

// Apply to specific routes
router.get('/profile', authenticateToken, getUserProfile);

// Apply to all routes in a group
router.use('/admin', authenticateToken);
```

### Generating JWT Tokens
```javascript
const jwt = require('jsonwebtoken');

const generateToken = (payload) => {
  return jwt.sign(payload, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRES_IN
  });
};
```

## 🐛 Troubleshooting

### Common Issues

1. **JWT Token Expired**
   - Check token expiration time
   - Implement token refresh mechanism

2. **Database Connection Failed**
   - Verify DB_URI in .env file
   - Check MongoDB service is running
   - Ensure network connectivity
   - Check MongoDB Atlas whitelist (if using cloud)

3. **Environment Variables Not Found**
   - Ensure .env file exists
   - Verify variable names match

4. **CORS Issues**
   - Configure CORS middleware
   - Check allowed origins

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-feature`)
3. Commit your changes (`git commit -am 'Add new feature'`)
4. Push to the branch (`git push origin feature/new-feature`)
5. Create a Pull Request

---

**Note**: Remember to keep your JWT secret secure and never commit your `.env` file to version control.
