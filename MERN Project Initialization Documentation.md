# Project Structure Documentation

### Create a Main Folder with the Project_name
```
cd Project_name
```
```
npm init -y
```
```
npm install concurrently --save-dev
```
#### Add the script to the package.json file
```
"scripts": {
  "start": "concurrently \"cd frontend && npm install && npm start\" \"cd Admin && npm install && npm start\" \"cd Backend && npm install && nodemon server.js\"",
  "build": "cd frontend && npm install && npm run build && sudo rm -rf /var/www/html/* && sudo cp -r dist/* /var/www/html && cd ../Admin && npm install && npm run build && sudo rm -rf /var/www/admin_html/* && sudo cp -r dist/* /var/www/admin_html && cd ../Backend && npm install && pm2 start server.js --watch"
}
```
```
git init
```
Ensure `.gitignore` is created at the root level to avoid pushing `node_modules` and sensitive files like `.env`
```
node_modules
.env
```
### Publish the Main_Folder as git repo with Readme.md


## Main Folder Structure
```
Website Name
├── Admin
│   │   
│   ├── src
|   |   |── configs
|   |   |      |── Constant.js
│   │   ├── components
│   │   │   └── [ReusableComponents.js]
│   │   ├── pages
│   │   │   └── [PageName.js]
│   │   ├── services
│   │   │   └── [ApiService.js]
│   │   ├── styles
│   │   │   └── [Styles.css]
│   │   └── utils
│   │       └── [HelperFunctions.js]
│   ├── public
│   │   └── assets
│   │       └── [Images, Fonts, Icons]
│   ├── package.json
│   └── README.md
├── Backend
│   ├── configs
│   │   └── database.js
│   ├── controllers
│   │   └── [ControllerName.js]
│   ├── middlewares
│   │   └── [MiddlewareName.js]
│   ├── models
│   │   └── [ModelName.js]
│   ├── routes
│   │   └── [RouteName.js]
│   ├── services
│   │   └── [ServiceName.js]
│   ├── utils
│   │   └── [UtilityFunctions.js]
│   ├── tests
│   │   └── [TestFiles.js]
│   ├── server.js
│   ├── package.json
│   └── README.md
├── Frontend
│   ├── src
|   |   |── configs
|   |   |      |── Constant.js
│   │   ├── components
│   │   │   └── [ReusableComponents.js]
│   │   ├── pages
│   │   │   └── [PageName.js]
│   │   ├── services
│   │   │   └── [ApiService.js]
│   │   ├── styles
│   │   │   └── [Styles.css]
│   │   └── utils
│   │       └── [HelperFunctions.js]
│   ├── public
│   │   └── assets
│   │       └── [Images, Fonts, Icons]
│   ├── package.json
│   └── README.md
├── scripts
│   └── [DeploymentScripts.js]
└── README.md
```

## Folder Descriptions

### Admin
- **configs/Constant.js**: Holds app-wide constants such as API URLs, keys, or feature flags.
- **src/components**: Reusable UI components.
- **src/pages**: Individual pages or routes (e.g., Dashboard, Login).
- **src/services**: API interaction or service layer functions.
- **src/styles**: CSS/SCSS files for styling.
- **src/utils**: Utility/helper functions like formatters or validators.
- **public/assets**: Static assets such as images, fonts, and icons.

### Backend
- **configs/database.js**: Database connection settings.
- **controllers**: Handles business logic for requests and responses.
- **middlewares**: Functions for logging, validation, or authentication.
- **models**: Database schemas or ORM models.
- **routes**: API route definitions.
- **services**: Backend-specific services or external integrations.
- **utils**: Backend utility functions.
- **tests**: Unit or integration tests.
- **server.js**: Entry point for the backend server.

### Frontend
- **configs/Constant.js**: Similar to the Admin configuration file.
- **src/components**: Reusable UI components.
- **src/pages**: Individual pages or routes (e.g., Home, Profile).
- **src/services**: API interaction or service layer functions.
- **src/styles**: CSS/SCSS files for styling.
- **src/utils**: Utility/helper functions.
- **public/assets**: Static assets such as images, fonts, and icons.

### Root-Level Files
- **scripts/DeploymentScripts.js**: Automation scripts for building and deploying.
- **README.md**: Overview of the project, including setup, usage, and deployment instructions.

# Setting Up a Vite Project with Tailwind CSS

## Step 1: Create a Vite Project
To create a new Vite project, run the following commands:

```bash
npm create vite
```

```bash
cd Frontend
```

```bash
npm install
```
## Step 2: Setup Tailwind CSS

#### 1. Install Tailwind CSS and its dependencies:

```bash
npm install tailwindcss @tailwindcss/vite
```

#### 2. Configure your template paths by updating the `vite.config.js` file:

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

// https://vite.dev/config/
export default defineConfig({
  plugins: [react(),tailwindcss()],
})
```

#### 3. Add Tailwind Directives to CSS

Add the `@tailwind` directives for each layer to your `./src/app.css` file:

```css
@import "tailwindcss";
```

#### 4. Start the Development Server

Add the Script
```bash
"start": "vite"
```

Run the development server:

```bash
npm start
```
#### 5. Start Using Tailwind CSS

You can now start using Tailwind CSS classes in your project. Example:

```jsx
<div className="bg-blue-500 text-white p-4">
  Hello, Tailwind CSS!
</div>
```
#### Note -- Don't Forget to add BASE_URL in configs directory using the Constant.js
---

# Setting Up a Node.js Backend with Express

#### 1. Initialize the Project

Create a new Node.js project and install Express:

```bash
mkdir Backend
```
```bash
cd backend
```
```bash
npm init -y
```


```bash
npm install bcrypt connect-mongo cors dotenv ejs express express-session mongoose nodemon passport passport-local uuid swagger-ui-express yamljs morgan
```

#### 2. Create the Folder Structure

Organize your backend project as follows:

```
backend/
├── config/
│   └── constants.js
├── controllers/
│   └── exampleController.js
├── models/
│   └── userSchema.js
├── routes/
│   └── exampleRoutes.js
├── middleware/
│   └── authMiddleware.js
├── utils/
│   └── helper.js
├── server.js
├── swagger.yaml
├── passportconfig.js
|-- .env
└── package.json
```

## Step 3: Create the Entry Point

In `server.js`, set up the basic Express server:

```javascript
// Import dependencies
const express = require('express');
const path = require('path');
const dotenv = require('dotenv');
const morgan = require('morgan');
const session = require('express-session');
const MongoStore = require('connect-mongo');
const swaggerUi = require('swagger-ui-express');
const YAML = require('yamljs');
const swaggerDocument = YAML.load('./swagger.yaml');

// Load environment variables
dotenv.config();

// Initialize Express app
const app = express();

// Set port from environment variables or default to 8080
const PORT = process.env.PORT || 8080;

//EJS Setup
app.set("view engine", "ejs");
app.use(express.static(path.join(__dirname, "/views")));

// Middleware setup
app.use(morgan('dev')); // Log requests to the console

// Enable CORS
const cors = require("cors");
var corsOptions = {
  origin: ["http://localhost:5173", "http://localhost:5174", "http://localhost:5175"],
  // origin: [""], // Allow specific domains
  credentials: true,
  methods: ["GET", "POST", "PUT", "DELETE"],
  allowedHeaders: ["Content-Type", "Authorization", "Origin", "Accept"],
  optionsSuccessStatus: 200, // some legacy browsers (IE11, various SmartTVs) choke on 204
};
app.use(cors(corsOptions));

// Parse incoming JSON requests
app.use(express.json());

// Enable URL-encoded form data parsing
app.use(express.urlencoded({ extended: true }));

// Setup session store (for example using MongoDB)
const passport = require("passport");
const { initializingPassport, isAuthenticated } = require("./passportconfig");
initializingPassport(passport);
app.set("trust proxy", 1); // trust first proxy
app.use(
  session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    store: MongoStore.create({
      mongoUrl: process.env.DB_URI,
      collectionName: "sessions",
    }),
    cookie: {
      httpOnly: true,
      // sameSite: 'None', // 'none' will work with secure: true
      // secure: true, // set to true if your using https
      maxAge: 1000 * 60 * 60 * 24,
    }, // 1 day
  })
);

// Initialize Passport for authentication
app.use(passport.initialize());
app.use(passport.session());

//Swagger Documentation Route
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument));

// Basic route to test server
app.get('/', (req, res) => {
  res.send('Server is running!');
});

// Example API route
app.get('/api', (req, res) => {
  res.json({ message: 'API is working!' });
});

// Add API routes for different services (Admin, Backend, etc.)
// app.use('/api/admin', require('./Backend/routes/adminRoutes')); // Example backend route
// app.use('/api/frontend', require('./Frontend/routes/frontendRoutes')); // Example frontend route

// Handle 404 errors
app.use((req, res, next) => {
  res.status(404).json({ error: 'Not Found' });
});

// Global error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ message: 'Something went wrong!' });
});

// Start the server
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```
In `.env` use this variables
```
DB_URI = 
PORT = 8080
SECRET_KEY = 
SESSION_SECRET = 
```
In `passportconfig.js` update the code
```
const passport = require("passport");
const LocalStrategy = require("passport-local").Strategy;
const bcrypt = require("bcrypt");
const { session } = require("passport");
const User = require("./models/userSchema"); // Assuming your schema is named "userschema"

exports.initializingPassport = (passport) => {
passport.use(
    'passport-local',
  new LocalStrategy(
    async (username, password, done) => {
      try {
        // Use async/await to find the user by username
        const user = await User.findOne({ username });
        
        // If user not found
        if (!user) {
          return done(null, false, { message: 'Invalid Username' });
        }

        // Compare passwords using bcrypt
        const isMatch = await bcrypt.compare(password, user.password);

        // If password does not match
        if (!isMatch) {
          return done(null, false, { message: 'Invalid Password' });
        }

        // If user found and password matches, pass the user object to the done callback
        return done(null, user);
      } catch (err) {
        // Catch any error and pass it to the done callback
        return done(err);
      }
    }
  )
);

passport.serializeUser(function (user, done) {
  // console.log("Serializing user:", user);
  done(null, user.id); // Only store the user ID in the session
});

passport.deserializeUser(async function (id, done) {
  try {
    // console.log("Deserializing user with ID:", id);
    const user = await User.findById(id); // Retrieve the user from the database
    // console.log("deserializing user");
    done(null, user); // Attach the user object to the session
  } catch (err) {
    // console.log(err, " deserializer lafda");
    done(err); // Handle any error that occurs
  }
});
};

// Middleware to check if the user is authenticated
exports.isAuthenticated = (req, res, next) => {
  console.log('Checking if user is authenticated');
  // console.log(req.user);
  // console.log(req.session);
  // console.log(req.isAuthenticated());
  console.log(req.isAuthenticated());
  if (req.isAuthenticated()) {
    return next(); // If authenticated, proceed to the next middleware or route
  }
  // console.log(req.user);
  
  // If not authenticated, send a JSON response with a 401 status code
  return res.status(401).json({
    success: false,
    message: 'User is not authenticated',
  });
};

```
In `db.js` file at root level update the code
```
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const MongoStore = require('connect-mongo');

// Load environment variables from .env
dotenv.config();

// MongoDB connection URI from .env
const MONGO_URI = process.env.DB_URI;
// console.log(process.env.DB_URI);

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(MONGO_URI, {
    //   useNewUrlParser: true,
    //   useUnifiedTopology: true,
    });

    console.log(`✅ MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error(`❌ Error connecting to MongoDB: ${error.message}`);
    process.exit(1); // Exit the process with failure
  }
};

module.exports = connectDB;
```

In `userSchema.js` file under `Models` folder update the code
```
// models/userSchema.js

const mongoose = require('mongoose');
const bcrypt = require('bcrypt');

// Define the schema
const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    unique: true,
    trim: true,
  },
  password: {
    type: String,
    required: true,
  },
}, { timestamps: true });

// Hash password before saving
userSchema.pre('save', async function (next) {
  if (!this.isModified('password')) return next();

  try {
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (err) {
    next(err);
  }
});

// Method to compare password
userSchema.methods.comparePassword = function (candidatePassword) {
  return bcrypt.compare(candidatePassword, this.password);
};

// Export model
module.exports = mongoose.model('User', userSchema);
```

In `swagger.yaml` file update
```
openapi: 3.0.0
info:
  title: Simple API
  version: 1.0.0
  description: A basic API that confirms it's working.
paths:
  /api:
    get:
      summary: Check API status
      description: Returns a simple message to confirm the API is working.
      responses:
        '200':
          description: API is working response
          content:
            application/json:
              schema:
                type: object
                properties:
                  message:
                    type: string
                    example: API is working!
```

#### 4. Start the Server
Add the Script
```
"start": "nodemon server.js"
```

Run the server:

```bash
npm start
```
### Note - Don't forget to Maintain the API documentation on Swagger | Added SwaggerUI By Default at `/api-docs` route.

## Admin Panel
Add the Script
```bash
"start": "vite"
```

Run the development server:

```bash
npm start
```
Ensure to create a `configs` folder within the Admin directory and include a `Constants.js` file. This file should define a `BASE_URL` constant, which can be used throughout the application.

---
## Starting all servers at a time
### Locate to the Main Folder
```
npm start
```
---

This setup creates a structured Node.js backend with Express, ready for further customization and development. 🚀


