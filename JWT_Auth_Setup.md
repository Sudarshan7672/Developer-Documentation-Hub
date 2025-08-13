# JWT Authentication Backend - Complete Implementation Guide

A complete Node.js backend implementation with JWT (JSON Web Token) authentication, built with Express.js and MongoDB. This guide contains all the code and setup instructions needed to build the application from scratch.

## 🚀 Features

- **JWT Authentication**: Secure token-based authentication system
- **User Management**: Complete user registration and login system
- **Protected Routes**: Middleware-based route protection
- **Password Security**: Encrypted password storage with bcrypt
- **API Documentation**: Swagger/OpenAPI documentation
- **Environment Configuration**: Secure environment variable management
- **Session Management**: MongoDB session store integration

## 📁 Project Structure

```
backend/
├── config/
│   ├── constants.js          # Application constants and configuration
│   └── db.js                 # MongoDB connection configuration
├── controllers/
│   └── authController.js     # Authentication route handlers
├── models/
│   └── userSchema.js         # MongoDB user schema definition
├── routes/
│   └── authRoutes.js         # Authentication API routes
├── middleware/
│   └── authMiddleware.js     # JWT authentication middleware
├── utils/
│   └── helper.js             # Utility functions and helpers
├── server.js                 # Application entry point
├── swagger.yaml              # API documentation
├── .env                      # Environment variables (create this)
└── package.json              # Dependencies and scripts
```

## 🛠️ Complete Setup Guide

### Step 1: Initialize Project

Create a new directory and initialize the project:

```bash
mkdir jwt-auth-backend
cd jwt-auth-backend
npm init -y
```

### Step 2: Install Dependencies

```bash
npm install express mongoose bcryptjs jsonwebtoken dotenv cors helmet morgan express-rate-limit connect-mongo express-session passport passport-local swagger-ui-express yamljs
npm install -D nodemon
```

### Step 3: Create Package.json Scripts

Update your `package.json` with these scripts:

```json
{
  "name": "jwt-auth-backend",
  "version": "1.0.0",
  "description": "JWT Authentication Backend with Node.js and MongoDB",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": ["jwt", "authentication", "nodejs", "mongodb", "express"],
  "author": "Your Name",
  "license": "MIT"
}
```

### Step 4: Environment Configuration

Create a `.env` file in the root directory:

```env
# Server Configuration
PORT=3000
NODE_ENV=development

# Database Configuration
DB_URI=mongodb://localhost:27017/jwt-auth-db

# JWT Configuration
JWT_SECRET=your-super-secret-jwt-key-make-it-long-and-complex
JWT_EXPIRES_IN=7d

# Session Configuration
SESSION_SECRET=your-session-secret-key

# Security
BCRYPT_ROUNDS=12
```

## 📄 Complete Code Implementation

### server.js - Application Entry Point

```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');
const rateLimit = require('express-rate-limit');
const session = require('express-session');
const MongoStore = require('connect-mongo');
const swaggerUi = require('swagger-ui-express');
const YAML = require('yamljs');
const dotenv = require('dotenv');

// Load environment variables
dotenv.config();

// Import custom modules
const connectDB = require('./config/db');
const authRoutes = require('./routes/authRoutes');
const { errorHandler } = require('./middleware/authMiddleware');

// Initialize Express app
const app = express();

// Connect to MongoDB
connectDB();

// Load Swagger documentation
const swaggerDocument = YAML.load('./swagger.yaml');

// Security middleware
app.use(helmet());
app.use(cors({
  origin: process.env.CLIENT_URL || 'http://localhost:3000',
  credentials: true
}));

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP, please try again later.'
});
app.use(limiter);

// Body parsing middleware
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Logging middleware
app.use(morgan('combined'));

// Session configuration
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  store: MongoStore.create({
    mongoUrl: process.env.DB_URI,
    touchAfter: 24 * 3600 // lazy session update
  }),
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  }
}));

// Routes
app.use('/api/auth', authRoutes);

// API Documentation
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument));

// Health check endpoint
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'OK',
    message: 'Server is running',
    timestamp: new Date().toISOString()
  });
});

// 404 handler
app.use('*', (req, res) => {
  res.status(404).json({
    success: false,
    message: 'Route not found'
  });
});

// Global error handler
app.use(errorHandler);

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`🚀 Server running on port ${PORT}`);
  console.log(`📚 API Documentation: http://localhost:${PORT}/api-docs`);
});

module.exports = app;
```

### config/db.js - Database Connection

```javascript
const mongoose = require('mongoose');
const dotenv = require('dotenv');

// Load environment variables from .env
dotenv.config();

// MongoDB connection URI from .env
const MONGO_URI = process.env.DB_URI;

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(MONGO_URI, {
      // useNewUrlParser: true,     // No longer needed in Mongoose 6+
      // useUnifiedTopology: true,  // No longer needed in Mongoose 6+
    });

    console.log(`✅ MongoDB Connected: ${conn.connection.host}`);
    
    // Connection event handlers
    mongoose.connection.on('error', (err) => {
      console.error(`❌ MongoDB connection error: ${err}`);
    });

    mongoose.connection.on('disconnected', () => {
      console.log('📤 MongoDB disconnected');
    });

    // Graceful shutdown
    process.on('SIGINT', async () => {
      await mongoose.connection.close();
      console.log('📤 MongoDB connection closed through app termination');
      process.exit(0);
    });

  } catch (error) {
    console.error(`❌ Error connecting to MongoDB: ${error.message}`);
    process.exit(1); // Exit the process with failure
  }
};

module.exports = connectDB;
```

### config/constants.js - Application Constants

```javascript
module.exports = {
  // HTTP Status Codes
  STATUS_CODES: {
    OK: 200,
    CREATED: 201,
    BAD_REQUEST: 400,
    UNAUTHORIZED: 401,
    FORBIDDEN: 403,
    NOT_FOUND: 404,
    CONFLICT: 409,
    INTERNAL_SERVER_ERROR: 500
  },

  // JWT Configuration
  JWT: {
    SECRET: process.env.JWT_SECRET,
    EXPIRES_IN: process.env.JWT_EXPIRES_IN || '7d',
    REFRESH_EXPIRES_IN: '30d'
  },

  // User Roles
  USER_ROLES: {
    USER: 'user',
    ADMIN: 'admin',
    MODERATOR: 'moderator'
  },

  // Validation Rules
  VALIDATION: {
    PASSWORD_MIN_LENGTH: 6,
    USERNAME_MIN_LENGTH: 3,
    USERNAME_MAX_LENGTH: 30
  },

  // Rate Limiting
  RATE_LIMIT: {
    WINDOW_MS: 15 * 60 * 1000, // 15 minutes
    MAX_REQUESTS: 100
  },

  // Error Messages
  ERROR_MESSAGES: {
    INVALID_CREDENTIALS: 'Invalid credentials',
    USER_EXISTS: 'User already exists',
    USER_NOT_FOUND: 'User not found',
    INVALID_TOKEN: 'Invalid token',
    TOKEN_EXPIRED: 'Token expired',
    ACCESS_DENIED: 'Access denied',
    VALIDATION_ERROR: 'Validation error'
  }
};
```

### models/userSchema.js - User Model

```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const { USER_ROLES } = require('../config/constants');

const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: [true, 'Username is required'],
    unique: true,
    trim: true,
    minlength: [3, 'Username must be at least 3 characters long'],
    maxlength: [30, 'Username cannot exceed 30 characters']
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    trim: true,
    lowercase: true,
    match: [
      /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/,
      'Please enter a valid email address'
    ]
  },
  password: {
    type: String,
    required: [true, 'Password is required'],
    minlength: [6, 'Password must be at least 6 characters long'],
    select: false // Don't include password in queries by default
  },
  role: {
    type: String,
    enum: Object.values(USER_ROLES),
    default: USER_ROLES.USER
  },
  isActive: {
    type: Boolean,
    default: true
  },
  lastLogin: {
    type: Date,
    default: null
  },
  refreshToken: {
    type: String,
    default: null,
    select: false
  }
}, {
  timestamps: true // Adds createdAt and updatedAt fields
});

// Hash password before saving
userSchema.pre('save', async function(next) {
  // Only hash the password if it has been modified (or is new)
  if (!this.isModified('password')) return next();

  try {
    // Hash password with cost of 12
    const salt = await bcrypt.genSalt(12);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (error) {
    next(error);
  }
});

// Instance method to check password
userSchema.methods.comparePassword = async function(candidatePassword) {
  try {
    return await bcrypt.compare(candidatePassword, this.password);
  } catch (error) {
    throw new Error('Password comparison failed');
  }
};

// Instance method to generate user object for responses (without sensitive data)
userSchema.methods.toPublicJSON = function() {
  const userObject = this.toObject();
  delete userObject.password;
  delete userObject.refreshToken;
  delete userObject.__v;
  return userObject;
};

// Static method to find user by email with password
userSchema.statics.findByEmailWithPassword = function(email) {
  return this.findOne({ email }).select('+password');
};

// Index for better performance
userSchema.index({ email: 1 });
userSchema.index({ username: 1 });

module.exports = mongoose.model('User', userSchema);
```

### controllers/authController.js - Authentication Logic

```javascript
const jwt = require('jsonwebtoken');
const User = require('../models/userSchema');
const { STATUS_CODES, JWT, ERROR_MESSAGES } = require('../config/constants');
const { generateToken, generateRefreshToken } = require('../utils/helper');

// Register new user
const register = async (req, res) => {
  try {
    const { username, email, password } = req.body;

    // Check if user already exists
    const existingUser = await User.findOne({
      $or: [{ email }, { username }]
    });

    if (existingUser) {
      return res.status(STATUS_CODES.CONFLICT).json({
        success: false,
        message: ERROR_MESSAGES.USER_EXISTS
      });
    }

    // Create new user
    const user = new User({
      username,
      email,
      password
    });

    await user.save();

    // Generate tokens
    const token = generateToken(user._id);
    const refreshToken = generateRefreshToken(user._id);

    // Save refresh token to user
    user.refreshToken = refreshToken;
    await user.save();

    // Update last login
    user.lastLogin = new Date();
    await user.save();

    res.status(STATUS_CODES.CREATED).json({
      success: true,
      message: 'User registered successfully',
      data: {
        user: user.toPublicJSON(),
        token,
        refreshToken
      }
    });
  } catch (error) {
    console.error('Registration error:', error);
    res.status(STATUS_CODES.INTERNAL_SERVER_ERROR).json({
      success: false,
      message: 'Registration failed',
      error: error.message
    });
  }
};

// Login user
const login = async (req, res) => {
  try {
    const { email, password } = req.body;

    // Find user with password
    const user = await User.findByEmailWithPassword(email);

    if (!user) {
      return res.status(STATUS_CODES.UNAUTHORIZED).json({
        success: false,
        message: ERROR_MESSAGES.INVALID_CREDENTIALS
      });
    }

    // Check if user is active
    if (!user.isActive) {
      return res.status(STATUS_CODES.UNAUTHORIZED).json({
        success: false,
        message: 'Account is deactivated'
      });
    }

    // Verify password
    const isPasswordValid = await user.comparePassword(password);

    if (!isPasswordValid) {
      return res.status(STATUS_CODES.UNAUTHORIZED).json({
        success: false,
        message: ERROR_MESSAGES.INVALID_CREDENTIALS
      });
    }

    // Generate tokens
    const token = generateToken(user._id);
    const refreshToken = generateRefreshToken(user._id);

    // Save refresh token to user
    user.refreshToken = refreshToken;
    user.lastLogin = new Date();
    await user.save();

    res.status(STATUS_CODES.OK).json({
      success: true,
      message: 'Login successful',
      data: {
        user: user.toPublicJSON(),
        token,
        refreshToken
      }
    });
  } catch (error) {
    console.error('Login error:', error);
    res.status(STATUS_CODES.INTERNAL_SERVER_ERROR).json({
      success: false,
      message: 'Login failed',
      error: error.message
    });
  }
};

// Get user profile
const getProfile = async (req, res) => {
  try {
    const user = await User.findById(req.user.id);

    if (!user) {
      return res.status(STATUS_CODES.NOT_FOUND).json({
        success: false,
        message: ERROR_MESSAGES.USER_NOT_FOUND
      });
    }

    res.status(STATUS_CODES.OK).json({
      success: true,
      data: {
        user: user.toPublicJSON()
      }
    });
  } catch (error) {
    console.error('Profile fetch error:', error);
    res.status(STATUS_CODES.INTERNAL_SERVER_ERROR).json({
      success: false,
      message: 'Failed to fetch profile',
      error: error.message
    });
  }
};

// Refresh token
const refreshToken = async (req, res) => {
  try {
    const { refreshToken } = req.body;

    if (!refreshToken) {
      return res.status(STATUS_CODES.UNAUTHORIZED).json({
        success: false,
        message: 'Refresh token required'
      });
    }

    // Verify refresh token
    const decoded = jwt.verify(refreshToken, JWT.SECRET);
    const user = await User.findById(decoded.id).select('+refreshToken');

    if (!user || user.refreshToken !== refreshToken) {
      return res.status(STATUS_CODES.UNAUTHORIZED).json({
        success: false,
        message: ERROR_MESSAGES.INVALID_TOKEN
      });
    }

    // Generate new tokens
    const newToken = generateToken(user._id);
    const newRefreshToken = generateRefreshToken(user._id);

    // Update refresh token
    user.refreshToken = newRefreshToken;
    await user.save();

    res.status(STATUS_CODES.OK).json({
      success: true,
      data: {
        token: newToken,
        refreshToken: newRefreshToken
      }
    });
  } catch (error) {
    console.error('Token refresh error:', error);
    res.status(STATUS_CODES.UNAUTHORIZED).json({
      success: false,
      message: ERROR_MESSAGES.INVALID_TOKEN
    });
  }
};

// Logout user
const logout = async (req, res) => {
  try {
    const user = await User.findById(req.user.id);

    if (user) {
      user.refreshToken = null;
      await user.save();
    }

    res.status(STATUS_CODES.OK).json({
      success: true,
      message: 'Logout successful'
    });
  } catch (error) {
    console.error('Logout error:', error);
    res.status(STATUS_CODES.INTERNAL_SERVER_ERROR).json({
      success: false,
      message: 'Logout failed',
      error: error.message
    });
  }
};

module.exports = {
  register,
  login,
  getProfile,
  refreshToken,
  logout
};
```

### middleware/authMiddleware.js - Authentication Middleware

```javascript
const jwt = require('jsonwebtoken');
const User = require('../models/userSchema');
const { STATUS_CODES, JWT, ERROR_MESSAGES } = require('../config/constants');

// Authenticate JWT token
const authenticateToken = async (req, res, next) => {
  try {
    const authHeader = req.headers.authorization;
    const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN

    if (!token) {
      return res.status(STATUS_CODES.UNAUTHORIZED).json({
        success: false,
        message: ERROR_MESSAGES.ACCESS_DENIED
      });
    }

    // Verify token
    const decoded = jwt.verify(token, JWT.SECRET);
    
    // Find user
    const user = await User.findById(decoded.id);
    
    if (!user) {
      return res.status(STATUS_CODES.UNAUTHORIZED).json({
        success: false,
        message: ERROR_MESSAGES.USER_NOT_FOUND
      });
    }

    if (!user.isActive) {
      return res.status(STATUS_CODES.UNAUTHORIZED).json({
        success: false,
        message: 'Account is deactivated'
      });
    }

    // Add user to request object
    req.user = {
      id: user._id,
      username: user.username,
      email: user.email,
      role: user.role
    };

    next();
  } catch (error) {
    console.error('Token verification error:', error);
    
    if (error.name === 'JsonWebTokenError') {
      return res.status(STATUS_CODES.UNAUTHORIZED).json({
        success: false,
        message: ERROR_MESSAGES.INVALID_TOKEN
      });
    }
    
    if (error.name === 'TokenExpiredError') {
      return res.status(STATUS_CODES.UNAUTHORIZED).json({
        success: false,
        message: ERROR_MESSAGES.TOKEN_EXPIRED
      });
    }

    res.status(STATUS_CODES.INTERNAL_SERVER_ERROR).json({
      success: false,
      message: 'Authentication failed',
      error: error.message
    });
  }
};

// Check if user has required role
const requireRole = (roles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(STATUS_CODES.UNAUTHORIZED).json({
        success: false,
        message: ERROR_MESSAGES.ACCESS_DENIED
      });
    }

    const userRole = req.user.role;
    const allowedRoles = Array.isArray(roles) ? roles : [roles];

    if (!allowedRoles.includes(userRole)) {
      return res.status(STATUS_CODES.FORBIDDEN).json({
        success: false,
        message: 'Insufficient permissions'
      });
    }

    next();
  };
};

// Global error handler
const errorHandler = (err, req, res, next) => {
  console.error('Global error:', err);

  // Mongoose validation error
  if (err.name === 'ValidationError') {
    const errors = Object.values(err.errors).map(e => e.message);
    return res.status(STATUS_CODES.BAD_REQUEST).json({
      success: false,
      message: ERROR_MESSAGES.VALIDATION_ERROR,
      errors
    });
  }

  // Mongoose duplicate key error
  if (err.code === 11000) {
    const field = Object.keys(err.keyValue)[0];
    return res.status(STATUS_CODES.CONFLICT).json({
      success: false,
      message: `${field} already exists`
    });
  }

  // Default error
  res.status(STATUS_CODES.INTERNAL_SERVER_ERROR).json({
    success: false,
    message: 'Internal server error',
    ...(process.env.NODE_ENV === 'development' && { error: err.message })
  });
};

module.exports = {
  authenticateToken,
  requireRole,
  errorHandler
};
```

### routes/authRoutes.js - Authentication Routes

```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');
const { body, validationResult } = require('express-validator');
const {
  register,
  login,
  getProfile,
  refreshToken,
  logout
} = require('../controllers/authController');
const { authenticateToken } = require('../middleware/authMiddleware');

const router = express.Router();

// Rate limiting for auth routes
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // limit each IP to 5 requests per windowMs
  message: {
    success: false,
    message: 'Too many authentication attempts, please try again later.'
  }
});

// Validation middleware
const validateRegistration = [
  body('username')
    .trim()
    .isLength({ min: 3, max: 30 })
    .withMessage('Username must be between 3 and 30 characters')
    .matches(/^[a-zA-Z0-9_]+$/)
    .withMessage('Username can only contain letters, numbers, and underscores'),
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Please enter a valid email address'),
  body('password')
    .isLength({ min: 6 })
    .withMessage('Password must be at least 6 characters long')
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
    .withMessage('Password must contain at least one lowercase letter, one uppercase letter, and one number')
];

const validateLogin = [
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Please enter a valid email address'),
  body('password')
    .notEmpty()
    .withMessage('Password is required')
];

// Handle validation errors
const handleValidationErrors = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      message: 'Validation failed',
      errors: errors.array()
    });
  }
  next();
};

// Routes
router.post('/register', authLimiter, validateRegistration, handleValidationErrors, register);
router.post('/login', authLimiter, validateLogin, handleValidationErrors, login);
router.post('/refresh-token', refreshToken);
router.get('/profile', authenticateToken, getProfile);
router.post('/logout', authenticateToken, logout);

// Test protected route
router.get('/test', authenticateToken, (req, res) => {
  res.json({
    success: true,
    message: 'Access granted to protected route',
    user: req.user
  });
});

module.exports = router;
```

### utils/helper.js - Utility Functions

```javascript
const jwt = require('jsonwebtoken');
const crypto = require('crypto');
const { JWT } = require('../config/constants');

// Generate JWT access token
const generateToken = (userId) => {
  return jwt.sign(
    { 
      id: userId,
      type: 'access'
    },
    JWT.SECRET,
    { 
      expiresIn: JWT.EXPIRES_IN,
      issuer: 'jwt-auth-backend',
      audience: 'jwt-auth-client'
    }
  );
};

// Generate JWT refresh token
const generateRefreshToken = (userId) => {
  return jwt.sign(
    { 
      id: userId,
      type: 'refresh'
    },
    JWT.SECRET,
    { 
      expiresIn: JWT.REFRESH_EXPIRES_IN,
      issuer: 'jwt-auth-backend',
      audience: 'jwt-auth-client'
    }
  );
};

// Generate random string
const generateRandomString = (length = 32) => {
  return crypto.randomBytes(length).toString('hex');
};

// Format validation errors
const formatValidationError = (error) => {
  if (error.name === 'ValidationError') {
    const errors = {};
    Object.keys(error.errors).forEach(key => {
      errors[key] = error.errors[key].message;
    });
    return errors;
  }
  return null;
};

// Sanitize user input
const sanitizeInput = (input) => {
  if (typeof input !== 'string') return input;
  
  return input
    .trim()
    .replace(/[<>]/g, '') // Remove potential HTML tags
    .substring(0, 1000); // Limit length
};

// Check if email is valid format
const isValidEmail = (email) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

// Generate API response format
const apiResponse = (success, message, data = null, errors = null) => {
  const response = {
    success,
    message,
    timestamp: new Date().toISOString()
  };

  if (data) response.data = data;
  if (errors) response.errors = errors;

  return response;
};

// Password strength checker
const checkPasswordStrength = (password) => {
  const minLength = 6;
  const hasLowerCase = /[a-z]/.test(password);
  const hasUpperCase = /[A-Z]/.test(password);
  const hasNumbers = /\d/.test(password);
  const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);

  const strength = {
    isValid: password.length >= minLength,
    length: password.length >= minLength,
    hasLowerCase,
    hasUpperCase,
    hasNumbers,
    hasSpecialChar,
    score: 0
  };

  // Calculate strength score
  if (strength.length) strength.score += 1;
  if (strength.hasLowerCase) strength.score += 1;
  if (strength.hasUpperCase) strength.score += 1;
  if (strength.hasNumbers) strength.score += 1;
  if (strength.hasSpecialChar) strength.score += 1;

  strength.level = strength.score < 3 ? 'weak' : 
                   strength.score < 4 ? 'medium' : 'strong';

  return strength;
};

module.exports = {
  generateToken,
  generateRefreshToken,
  generateRandomString,
  formatValidationError,
  sanitizeInput,
  isValidEmail,
  apiResponse,
  checkPasswordStrength
};
```

### swagger.yaml - API Documentation

```yaml
openapi: 3.0.0
info:
  title: JWT Authentication API
  description: A complete JWT authentication system with user management
  version: 1.0.0
  contact:
    name: API Support
    email: support@example.com

servers:
  - url: http://localhost:3000/api
    description: Development server

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  
  schemas:
    User:
      type: object
      properties:
        _id:
          type: string
          description: User ID
        username:
          type: string
          description: Username
        email:
          type: string
          format: email
          description: User email
        role:
          type: string
          enum: [user, admin, moderator]
          description: User role
        isActive:
          type: boolean
          description: Account status
        lastLogin:
          type: string
          format: date-time
          description: Last login timestamp
        createdAt:
          type: string
          format: date-time
          description: Account creation timestamp
        updatedAt:
          type: string
          format: date-time
          description: Last update timestamp

    AuthResponse:
      type: object
      properties:
        success:
          type: boolean
        message:
          type: string
        data:
          type: object
          properties:
            user:
              $ref: '#/components/schemas/User'
            token:
              type: string
              description: JWT access token
            refreshToken:
              type: string
              description: JWT refresh token

    ErrorResponse:
      type: object
      properties:
        success:
          type: boolean
          example: false
        message:
          type: string
        errors:
          type: array
          items:
            type: string

paths:
  /auth/register:
    post:
      summary: Register a new user
      tags:
        - Authentication
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - username
                - email
                - password
              properties:
                username:
                  type: string
                  minLength: 3
                  maxLength: 30
                  example: johndoe
                email:
                  type: string
                  format: email
                  example: john@example.com
                password:
                  type: string
                  minLength: 6
                  example: Password123
      responses:
        '201':
          description: User registered successfully
