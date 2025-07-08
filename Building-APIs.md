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
Image source: zilliz.com

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

---

### Creating Our First Schema

## 🛒 Building the Product Schema

Now let's create a schema for products. I'll explain every decision:

```javascript
// models/product.model.js
import mongoose from 'mongoose';

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