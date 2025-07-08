# Building APIs for Our Product Store
## Understanding Every Step and Why We Do It

---

## 🎯 What We'll Learn Today

By the end of this lesson, you'll understand:
- What an API actually is (in simple terms)
- Why we need APIs for our product store site
- How to create data blueprints (schemas)
- How to build endpoints that your frontend can talk to
- Why we organize code the way we do

**Remember**: Every line of code we write has a purpose. I'll explain not just HOW to do something, but WHY we're doing it.

---

## 🤔 What is an API? (Really Simple Explanation)

### Think of a Restaurant 🍽️

Imagine you're hungry and want to order food:

1. **You (Customer)** = Your React frontend
2. **The Menu** = API documentation (what you can order)
3. **The Waiter** = The API (takes your order, brings your food)
4. **The Kitchen** = Your backend/database (prepares the food)

**The Process:**
- You look at the menu to see what's available
- You tell the waiter what you want
- The waiter takes your order to the kitchen
- The kitchen prepares your food
- The waiter brings it back to you

![Figure: Real-world Analogy of an API](/Figure_Real_world_Analogy_of_an_API_2f5cee9677.png)

### In Web Terms:

```javascript
// You (Frontend) make a request
"Hey API, can I get a list of all products?"

// API processes your request
"Sure! Let me check the database..."

// API responds with data
"Here are all the products: [iPhone, Samsung, iPad...]"
```

### Why Do We Need APIs?

**Problem**: Your React app (frontend) and your database can't talk directly to each other. It's like trying to order food by walking into the kitchen - it's chaos!

**Solution**: APIs act as organized waiters that handle requests properly, check if you're allowed to order certain things, and make sure the kitchen gets clear instructions.

---

## 🛣️ What is REST? (The Rules for Good APIs)

**REST** stands for "Representational State Transfer" - but forget that fancy name. Think of it as **"Rules for Building Good APIs"**.

### Why Do We Need Rules?

Imagine if every restaurant had different ways to order:
- Restaurant A: "Give me food type 1"
- Restaurant B: "I want to consume edible item number 3"
- Restaurant C: "Bring me sustenance of the third variety"

**Confusing, right?** REST gives us standard rules so all APIs work similarly.

### The REST Rules:

#### Rule 1: Use Clear URLs (Like Street Addresses)
```
❌ Bad: /getProducts
❌ Bad: /prod
❌ Bad: /fetchAllProductsFromDatabase

✅ Good: /api/products
```

**Why?** Just like house addresses, URLs should be clear and predictable.

#### Rule 2: Use HTTP Methods Like Verbs
Think of HTTP methods as action words:

```
GET    = "Please show me..." (looking at something)
POST   = "Please create..." (making something new)
PUT    = "Please replace..." (changing everything)
DELETE = "Please remove..." (throwing away)
```

**Real Examples:**
```
GET    /api/products     = "Show me all products"
POST   /api/products     = "Create a new product"
GET    /api/products/123 = "Show me product #123"
PUT    /api/products/123 = "Replace product #123"
DELETE /api/products/123 = "Delete product #123"
```

#### Rule 3: Use JSON for Data
**JSON** = JavaScript Object Notation (fancy name for "data format that looks like JavaScript objects")

```javascript
// This is JSON - easy to read!
{
  "name": "iPhone 14",
  "price": 999,
  "inStock": true
}
```

**Why JSON?** It's like a universal language that both JavaScript and most other programming languages can understand.

---

## 🗃️ Understanding Data Models (Schemas)

### What is a Schema?

A **schema** is like a blueprint for your data. Just like building a house, you need a plan before you start.

### Why Do We Need Schemas?

**Without Schema (Chaos):**
```javascript
// User 1
{ name: "John", email: "john@email.com", age: 25 }

// User 2  
{ fullName: "Jane", emailAddress: "jane@email.com", years: 30, city: "Accra" }

// User 3
{ username: "Bob", contact: "bob@email.com" }
```

**Problems:**
- Different field names (name vs fullName vs username)
- Missing required information
- Inconsistent data types
- Hard to write code that works with all variations

**With Schema (Organized):**
```javascript
// Every user MUST have these fields in this format
{
  name: "String, required",
  email: "String, required, must be valid email",
  age: "Number, optional"
}
```

### Creating Our First Schema: User

Let's build a User schema step by step, explaining every decision:

```javascript
// models/User.js
const mongoose = require('mongoose');

// Why mongoose? It's like a translator between JavaScript and MongoDB
const userSchema = new mongoose.Schema({
  // User's name
  name: {
    type: String,              // Why String? Names are text
    required: [true, 'Name is required'],  // Why required? Every user needs a name
    trim: true,                // Why trim? Removes extra spaces ("  John  " becomes "John")
    minlength: [2, 'Name must be at least 2 characters'],  // Why? Single letters aren't real names
    maxlength: [50, 'Name cannot exceed 50 characters']    // Why? Prevents spam/abuse
  },
```

**Let me explain each part:**

#### `type: String`
**Why?** Names are text, not numbers. If someone tries to save a number as a name, Mongoose will reject it.

#### `required: [true, 'Custom error message']`
**Why?** Every user must have a name. The custom message tells the user exactly what went wrong.

#### `trim: true`
**Why?** Users often accidentally add spaces. "  John  " becomes "John". This prevents database inconsistencies.

#### `minlength` and `maxlength`
**Why?** Prevents abuse. Someone can't register with name "A" or copy-paste a whole book as their name.

Let's continue building the User schema:

```javascript
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,              // Why? No two users can have the same email
    lowercase: true,           // Why? "JOHN@EMAIL.COM" becomes "john@email.com"
    match: [                   // Why? Validates email format
      /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/,
      'Please enter a valid email'
    ]
  },
```

#### `unique: true`
**Why?** Emails are like usernames - only one person can have each email address. This prevents duplicate accounts.

#### `lowercase: true`
**Why?** "John@Email.Com" and "john@email.com" are the same email, but computers see them as different. This fixes that.

#### `match: [regex, error message]`
**Why?** Ensures the email looks like an email (has @ symbol, domain, etc.). The regex (regular expression) is a pattern that checks email format.

```javascript
  password: {
    type: String,
    required: [true, 'Password is required'],
    minlength: [6, 'Password must be at least 6 characters'],
    select: false              // Why? NEVER send passwords back to frontend
  },
```

#### `select: false`
**Why?** This is CRUCIAL for security. When we fetch user data, the password should NEVER be included in the response. Even though it's encrypted, it's safer to never send it.

```javascript
  role: {
    type: String,
    enum: {                    // Why enum? Limits choices
      values: ['user', 'admin'],
      message: 'Role must be either user or admin'
    },
    default: 'user'            // Why default? Most people are regular users, not admins
  },
```

#### `enum` (enumeration)
**Why?** Limits choices to prevent mistakes. Someone can't accidentally set their role to "superman" or "hacker". Only "user" or "admin" are allowed.

#### `default: 'user'`
**Why?** If no role is specified, assume they're a regular user. This is safer than making them an admin by accident.

Now let's add timestamps:

```javascript
}, {
  timestamps: true             // Why? Automatically adds createdAt and updatedAt
});
```

#### `timestamps: true`
**Why?** MongoDB automatically adds `createdAt` (when the user was created) and `updatedAt` (when the user was last modified). This is useful for tracking user activity and debugging.

### Adding Middleware to Hash Passwords

```javascript
// This runs BEFORE saving a user to the database
userSchema.pre('save', async function(next) {
  // Only hash the password if it was modified
  if (!this.isModified('password')) return next();
  
  // Hash the password
  this.password = await bcrypt.hash(this.password, 12);
  next();
});
```

**Let me explain this step by step:**

#### Why do we need to hash passwords?
**Problem:** If someone hacks our database and sees this:
```javascript
{ name: "John", email: "john@email.com", password: "mypassword123" }
```
They immediately know John's password!

**Solution:** Hash the password so it looks like this:
```javascript
{ name: "John", email: "john@email.com", password: "$2b$12$xyz123randomhash..." }
```
Even if hackers see this, they can't figure out the original password.

#### `userSchema.pre('save')`
**Why "pre"?** This runs BEFORE saving to the database. It's like a checkpoint that transforms the data before storing it.

#### `if (!this.isModified('password')) return next()`
**Why this check?** If a user updates their name but not their password, we don't want to hash the already-hashed password again. That would break it!

#### `bcrypt.hash(this.password, 12)`
**Why bcrypt?** It's a special algorithm designed for hashing passwords. The "12" is the "salt rounds" - higher numbers are more secure but slower.

### Adding Instance Methods

```javascript
// Method to check if entered password matches hashed password
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};
```

#### Why do we need this method?
When a user logs in, they enter their plain text password. We need to check if it matches the hashed password in our database. This method does that comparison.

**How it works:**
1. User enters: "mypassword123"
2. Database has: "$2b$12$xyz123randomhash..."
3. `comparePassword` checks if they match
4. Returns true or false

---

## 🛒 Building the Product Schema

Now let's create a schema for products. I'll explain every decision:

```javascript
// models/Product.js
const mongoose = require('mongoose');

const productSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Product name is required'],  // Why? Products need names
    trim: true,                                    // Why? Remove extra spaces
    maxlength: [100, 'Product name cannot exceed 100 characters']  // Why? Prevent abuse
  },
```

#### Why maxlength for product names?
Someone could try to save a 10,000 character product name, which would break our UI and waste database space. 100 characters is enough for any reasonable product name.

```javascript
  description: {
    type: String,
    required: [true, 'Product description is required'],
    maxlength: [2000, 'Description cannot exceed 2000 characters']
  },
```

#### Why require descriptions?
Customers need to know what they're buying. A product without a description is useless.

```javascript
  price: {
    type: Number,                    // Why Number? Prices are mathematical values
    required: [true, 'Product price is required'],
    min: [0, 'Price cannot be negative'],         // Why? Free is ok, but negative prices don't make sense
    max: [1000000, 'Price too high']            // Why? Prevents mistakes (like $999,999,999)
  },
```

#### Why validate price ranges?
- **Minimum 0:** Negative prices don't make business sense
- **Maximum 1,000,000:** Prevents accidental entry of crazy prices like $999,999,999 for a phone

```javascript
  category: {
    type: mongoose.Schema.Types.ObjectId,        // Why ObjectId? It's a reference to another document
    ref: 'Category',                             // Why ref? Points to the Category model
    required: [true, 'Product category is required']
  },
```

#### What is ObjectId and why use it?
Instead of storing the full category information in every product:

**❌ Bad way (repetitive):**
```javascript
// Product 1
{ name: "iPhone", category: { name: "Electronics", description: "Electronic devices..." } }

// Product 2  
{ name: "Samsung", category: { name: "Electronics", description: "Electronic devices..." } }
```

**✅ Good way (efficient):**
```javascript
// Product 1
{ name: "iPhone", category: "507f1f77bcf86cd799439011" }  // Just an ID reference

// Product 2
{ name: "Samsung", category: "507f1f77bcf86cd799439011" }  // Same ID reference
```

The ObjectId points to a separate Category document. This saves space and keeps data consistent.

```javascript
  stock: {
    type: Number,
    required: [true, 'Stock quantity is required'],
    min: [0, 'Stock cannot be negative'],        // Why? Can't have -5 items
    default: 0                                   // Why default 0? Start with no stock until items arrive
  },
```

#### Why track stock?
- Prevents selling items we don't have
- Helps with inventory management
- Shows "out of stock" to customers

```javascript
  images: [{                                     // Why array? Products can have multiple photos
    type: String,                                // Why String? We store image URLs, not actual images
    required: true
  }],
```

#### Why store image URLs instead of actual images?
**Images are big files.** Storing them directly in the database would make it huge and slow. Instead, we:
1. Upload images to a file storage service (like AWS S3 or Cloudinary)
2. Store just the URL in our database
3. Frontend uses the URL to display the image

```javascript
  ratings: {
    average: {
      type: Number,
      default: 0,                                // Why default 0? New products have no ratings yet
      min: [0, 'Rating cannot be less than 0'],
      max: [5, 'Rating cannot be more than 5']  // Why 5? Standard 5-star rating system
    },
    count: {
      type: Number,
      default: 0                                 // Why track count? Shows how many people rated it
    }
  },
```

#### Why separate average and count?
**Example:**
- Product A: 4.5 stars (2 reviews)
- Product B: 4.5 stars (500 reviews)

Both have the same average, but Product B is more trustworthy because more people rated it.

```javascript
}, {
  timestamps: true                               // Why? Track when products were added/updated
});
```

### Adding Pre-save Middleware for SEO

```javascript
// Create a URL-friendly version of the product name
productSchema.pre('save', function(next) {
  if (this.isModified('name')) {
    this.slug = this.name
      .toLowerCase()                             // "iPhone 14 Pro" -> "iphone 14 pro"
      .replace(/[^a-zA-Z0-9 ]/g, '')            // Remove special characters
      .replace(/\s+/g, '-');                    // Replace spaces with dashes -> "iphone-14-pro"
  }
  next();
});
```

#### What is a slug and why do we need it?
A **slug** is a URL-friendly version of the product name.

**Example:**
- Product name: "iPhone 14 Pro (256GB) - Space Gray!"
- Slug: "iphone-14-pro-256gb-space-gray"
- URL: `/products/iphone-14-pro-256gb-space-gray`

**Why?** Clean URLs are better for SEO and user experience than `/products/507f1f77bcf86cd799439011`.

---

## 🏗️ Building Controllers (The Business Logic)

### What is a Controller?

Think of controllers as **smart waiters** in our restaurant analogy:
- They understand what customers want
- They validate requests ("Sorry, we don't serve ice cream for breakfast")
- They talk to the kitchen (database)
- They format the response nicely

### Why Separate Controllers from Routes?

**❌ Bad way (everything mixed together):**
```javascript
app.get('/api/products', (req, res) => {
  // 50 lines of business logic here
  // Database queries
  // Validation
  // Error handling
  // Response formatting
});
```

**✅ Good way (separated):**
```javascript
// Route file - just defines the endpoint
app.get('/api/products', productController.getProducts);

// Controller file - handles the logic
const getProducts = async (req, res) => {
  // All the business logic here
};
```

**Why separate?** 
- **Reusability:** Same logic can be used by different routes
- **Testing:** Easier to test business logic separately
- **Organization:** Cleaner, more maintainable code

### Building User Registration Controller

Let's build a user registration endpoint step by step:

```javascript
// controllers/userController.js
const User = require('../models/User');
const jwt = require('jsonwebtoken');

const registerUser = async (req, res) => {
  try {
    // Step 1: Extract data from request
    const { name, email, password } = req.body;
```

#### Why extract from req.body?
When someone makes a POST request to register, they send data in the request body:
```javascript
// Frontend sends this
fetch('/api/users/register', {
  method: 'POST',
  body: JSON.stringify({
    name: "John Doe",
    email: "john@email.com", 
    password: "mypassword123"
  })
});
```

`req.body` contains that data: `{ name: "John Doe", email: "john@email.com", password: "mypassword123" }`

```javascript
    // Step 2: Check if user already exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(400).json({
        success: false,
        message: 'User with this email already exists'
      });
    }
```

#### Why check for existing users?
**Problem:** Without this check, someone could register multiple accounts with the same email, causing confusion and database issues.

**Solution:** Before creating a new user, search the database to see if that email is already taken.

#### Why `await User.findOne({ email })`?
- `findOne()` searches for ONE document that matches the criteria
- `{ email }` is shorthand for `{ email: email }` - find user with this email
- `await` waits for the database to respond (database operations take time)

```javascript
    // Step 3: Create new user
    const user = await User.create({
      name,
      email,
      password  // Remember: our schema middleware will hash this automatically
    });
```

#### Why `User.create()` instead of `new User().save()`?
Both work, but `create()` is shorter and clearer. It creates the document and saves it to the database in one step.

```javascript
    // Step 4: Generate JWT token
    const token = jwt.sign(
      { userId: user._id },        // Payload: what to store in the token
      process.env.JWT_SECRET,      // Secret key: used to verify token later
      { expiresIn: '30d' }         // Options: token expires in 30 days
    );
```

#### What is JWT and why do we need it?
**JWT (JSON Web Token)** is like a VIP wristband at a concert:

1. **User logs in** → We give them a wristband (JWT token)
2. **User makes requests** → They show their wristband
3. **We verify the wristband** → If valid, we serve their request

**Why JWT?**
- **Stateless:** We don't need to store sessions on the server
- **Secure:** Tokens are encrypted and tamper-proof
- **Scalable:** Works across multiple servers

#### Why include userId in the token?
When the user makes future requests, we'll decode the token to see which user it is. The userId tells us who they are.

```javascript
    // Step 5: Send response
    res.status(201).json({
      success: true,
      message: 'User registered successfully',
      data: {
        user: {
          id: user._id,
          name: user.name,
          email: user.email,
          role: user.role
          // Notice: NO PASSWORD in the response!
        },
        token
      }
    });
```

#### Why this response format?
- **status(201):** "201 Created" means something new was successfully created
- **success: true:** Helps frontend know if the request worked
- **message:** Human-readable feedback
- **data:** The actual useful information
- **No password:** NEVER send passwords back to the frontend, even hashed ones

```javascript
  } catch (error) {
    // Handle validation errors specially
    if (error.name === 'ValidationError') {
      const errors = Object.values(error.errors).map(err => err.message);
      return res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors
      });
    }

    // Handle other errors
    res.status(500).json({
      success: false,
      message: 'Server error during registration'
    });
  }
};
```

#### Why different error handling?
**Validation errors** (400 Bad Request): User's fault - they sent invalid data
**Server errors** (500 Internal Server Error): Our fault - something broke on our end

**Example validation error response:**
```javascript
{
  success: false,
  message: 'Validation failed',
  errors: [
    'Name must be at least 2 characters',
    'Please enter a valid email'
  ]
}
```

This helps the frontend show specific error messages to the user.

### Building User Login Controller

```javascript
const loginUser = async (req, res) => {
  try {
    const { email, password } = req.body;

    // Step 1: Validate input
    if (!email || !password) {
      return res.status(400).json({
        success: false,
        message: 'Please provide email and password'
      });
    }
```

#### Why validate input first?
**Early validation** saves time and resources. Instead of querying the database and then realizing we're missing data, we check immediately.

```javascript
    // Step 2: Find user and include password
    const user = await User.findOne({ email }).select('+password');
```

#### Why `.select('+password')`?
Remember we set `select: false` on the password field in our schema. By default, passwords are NOT included in queries. We need to explicitly ask for it during login.

```javascript
    if (!user) {
      return res.status(401).json({
        success: false,
        message: 'Invalid email or password'
      });
    }

    // Step 3: Check password
    const isPasswordValid = await user.comparePassword(password);
    
    if (!isPasswordValid) {
      return res.status(401).json({
        success: false,
        message: 'Invalid email or password'
      });
    }
```

#### Why the same error message for both cases?
**Security principle:** Don't tell attackers whether an email exists or not.

**❌ Bad (gives attackers information):**
- "User not found" → Attacker knows this email isn't registered
- "Wrong password" → Attacker knows this email IS registered

**✅ Good (reveals nothing):**
- "Invalid email or password" → Attacker doesn't know which is wrong

```javascript
    // Step 4: Generate token and send response
    const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET, {
      expiresIn: '30d'
    });

    res.json({
      success: true,
      message: 'Login successful',
      data: {
        user: {
          id: user._id,
          name: user.name,
          email: user.email,
          role: user.role
        },
        token
      }
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: 'Server error during login'
    });
  }
};
```

---

## 🛡️ Authentication Middleware (The Security Guard)

### What is Middleware?

Middleware functions run **between** receiving a request and sending a response. Think of them as security guards at a building:

1. **Request comes in** → "I want to enter the VIP area"
2. **Security guard checks** → "Do you have a VIP pass?"
3. **If yes** → "Go ahead" (next())
4. **If no** → "Access denied" (return error)

### Building the Authentication Middleware

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');
const User = require('../models/User');

const protect = async (req, res, next) => {
  let token;

  try {
    // Step 1: Extract token from headers
    if (req.headers.authorization && req.headers.authorization.startsWith('Bearer')) {
      token = req.headers.authorization.split(' ')[1];
    }
```

#### Why check headers.authorization?
When making authenticated requests, frontends send the token in the Authorization header:
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

We extract just the token part (after "Bearer ").

```javascript
    // Step 2: Check if token exists
    if (!token) {
      return res.status(401).json({
        success: false,
        message: 'Access denied. No token provided.'
      });
    }

    // Step 3: Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
```

#### Why verify the token?
**Tokens can be:**
- **Expired:** User logged in 2 months ago, token expired
- **Tampered:** Someone tried to modify the token
- **Fake:** Someone created a fake token

`jwt.verify()` checks all of these issues.

```javascript
    // Step 4: Get user from token
    const user = await User.findById(decoded.userId);

    if (!user) {
      return res.status(401).json({
        success: false,
        message: 'Token is not valid - user not found'
      });
    }
```

#### Why look up the user again?
**Edge cases:**
- User account was deleted after login
- User account was deactivated
- User changed their password (in advanced systems, this invalidates old tokens)

We need to make sure the user still exists and is active.

```javascript
    // Step 5: Add user to request object
    req.user = user;
    next();  // Continue to the next middleware or route handler
```

#### Why add user to req?
Now any route that uses this middleware can access the current user via `req.user`. This is super convenient:

```javascript
// In any protected route
app.get('/api/profile', protect, (req, res) => {
  // req.user is available here!
  res.json({ user: req.user });
});
```

---

## 🛣️ Setting Up Routes (Connecting Everything)

### Why Separate Route Files?

**Imagine all routes in one file:**
```javascript
// server.js - Would become HUGE!
app.post('/api/users/register', /* register logic */);
app.post('/api/users/login', /* login logic */);
app.get('/api/users/profile', /* profile logic */);
app.get('/api/products', /* get products logic */);
app.post('/api/products', /* create product logic */);
app.get('/api/orders', /* get orders logic */);
// ... 50 more routes
```

**Better approach - separate files:**
```javascript
// server.js - Clean and organized
app.use('/api/users', userRoutes);
app.use('/api/products', productRoutes);
app.use('/api/orders', orderRoutes);
```

### Creating User Routes

```javascript
// routes/userRoutes.js
const express = require('express');
const {
  registerUser,
  loginUser,
  getUserProfile,
  updateUserProfile
} = require('../controllers/userController');
const { protect } = require('../middleware/auth');

const router = express.Router();

// Public routes (no authentication needed)
router.post('/register', registerUser);
router.post('/login', loginUser);

// Protected routes (authentication required)
router.route('/profile')
  .get(protect, getUserProfile)      // GET /api/users/profile
  .put(protect, updateUserProfile);  // PUT /api/users/profile

module.exports = router;
```

#### Why group routes with router.route()?
Instead of writing:
```javascript
router.get('/profile', protect, getUserProfile);
router.put('/profile', protect, updateUserProfile);
```

We can write:
```javascript
router.route('/profile')
  .get(protect, getUserProfile)
  .put(protect, updateUserProfile);
```

**Benefits:**
- **Less repetition:** Don't repeat '/profile'
- **Cleaner code:** Related operations grouped together
- **Easier to maintain:** All profile operations in one place

### Connecting Routes to Main Server

```javascript
// server.js
require('dotenv').config();
const express = require('express');
const connectDB = require('./config/database');

// Import route files
const userRoutes = require('./routes/userRoutes');
const productRoutes = require('./routes/productRoutes');

const app = express();

// Connect to database
connectDB();

// Middleware
app.use(express.json());

// Mount routes
app.use('/api/users', userRoutes);
app.use('/api/products', productRoutes);

// Basic route
app.get('/', (req, res) => {
  res.json({ message: 'Ecommerce API is running!' });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

#### What does app.use('/api/users', userRoutes) do?

It tells Express: "For any request that starts with '/api/users', use the userRoutes file to handle it."

**Examples:**
- `POST /api/users/register` → Goes to userRoutes → Handled by registerUser
- `POST /api/users/login` → Goes to userRoutes → Handled by loginUser
- `GET /api/users/profile` → Goes to userRoutes → Handled by getUserProfile (with protect middleware)

---

## 🧪 Testing Our API

### Using Postman (Recommended Tool)

**Postman** is like a playground for testing APIs. Instead of building a frontend, we can test our backend directly.

### Test 1: User Registration

**Request:**
```
Method: POST
URL: http://localhost:5000/api/users/register
Headers:
  Content-Type: application/json
Body (JSON):
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123"
}
```

**Expected Response:**
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {