# Building APIs for Complete Beginners
## Understanding Every Step and Why We Do It

---

## 🎯 What We'll Learn Today

By the end of this lesson, you'll understand:
- What an API actually is (in simple terms)
- Why we need APIs for our ecommerce site
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

### Building Product Controllers (CRUD Operations)

Let's build product controllers step by step. These are the most important controllers for any ecommerce site because they handle the products that customers will buy.

**CRUD** stands for:
- **C**reate - Add new products
- **R**ead - Get/view products  
- **U**pdate - Edit existing products
- **D**elete - Remove products

### 1. Get All Products Controller

This is probably the most important endpoint - it's what customers use to browse your store.

```javascript
// controllers/productController.js
const Product = require('../models/Product');

// @desc    Get all products with filtering and pagination
// @route   GET /api/products
// @access  Public (anyone can view products)
const getProducts = async (req, res) => {
  try {
    // Step 1: Extract query parameters from URL
    const {
      page = 1,        // Which page of results? Default to page 1
      limit = 12,      // How many products per page? Default to 12
      category,        // Filter by category
      minPrice,        // Filter by minimum price
      maxPrice,        // Filter by maximum price
      search,          // Search in product names/descriptions
      sortBy = 'createdAt',  // What field to sort by? Default to newest first
      order = 'desc'   // Sort direction? desc = newest first, asc = oldest first
    } = req.query;
```

#### What are query parameters?
Query parameters come after the `?` in URLs:
```
GET /api/products?page=2&limit=10&category=electronics&minPrice=100&maxPrice=500
```

This URL means: "Get page 2, show 10 products per page, only electronics category, price between $100-$500"

#### Why extract with default values?
If the user doesn't specify these parameters, we need sensible defaults:
- **page = 1:** Start at the first page
- **limit = 12:** Show 12 products (3 rows of 4 is common in ecommerce)
- **sortBy = 'createdAt':** Show newest products first (customers usually want to see new items)

```javascript
    // Step 2: Build filter object for database query
    const filter = { isActive: true };  // Only show active products
```

#### Why start with isActive: true?
We don't want to show products that are:
- Discontinued
- Out of stock permanently
- Draft products (not ready for sale)

The `isActive` field lets us "soft delete" products - hide them without actually removing them from the database.

```javascript
    // Step 3: Add category filter if provided
    if (category) {
      filter.category = category;
    }
```

#### Why check "if (category)"?
Only add the category filter if the user actually wants to filter by category. If they don't specify a category, show products from all categories.

```javascript
    // Step 4: Add price range filter if provided
    if (minPrice || maxPrice) {
      filter.price = {};
      if (minPrice) filter.price.$gte = Number(minPrice);  // Greater than or equal
      if (maxPrice) filter.price.$lte = Number(maxPrice);  // Less than or equal
    }
```

#### What does $gte and $lte mean?
These are MongoDB operators:
- **$gte** = "greater than or equal to"
- **$lte** = "less than or equal to"

**Example:**
```javascript
// User wants products between $100 and $500
filter.price = {
  $gte: 100,  // price >= 100
  $lte: 500   // price <= 500
}
```

#### Why Number(minPrice)?
Query parameters always come as strings: `"100"` instead of `100`. We need to convert them to numbers for proper comparison.

```javascript
    // Step 5: Add search filter if provided
    if (search) {
      filter.$or = [
        { name: { $regex: search, $options: 'i' } },
        { description: { $regex: search, $options: 'i' } }
      ];
    }
```

#### What does this search logic do?
**$or** means "match if ANY of these conditions are true":
1. Product name contains the search term (case-insensitive)
2. Product description contains the search term (case-insensitive)

**Example:** User searches for "phone"
- Matches: "iPhone 14", "Samsung Phone", "Bluetooth headphones for phone"
- **$regex** = regular expression (pattern matching)
- **$options: 'i'** = case-insensitive ("Phone" matches "phone")

```javascript
    // Step 6: Build sort object
    const sortOrder = order === 'desc' ? -1 : 1;  // MongoDB uses -1 for descending, 1 for ascending
    const sort = {};
    sort[sortBy] = sortOrder;
```

#### Why this sorting setup?
MongoDB expects sort objects like this:
```javascript
// Sort by price, lowest to highest
{ price: 1 }

// Sort by creation date, newest first  
{ createdAt: -1 }
```

We build this dynamically based on user input.

```javascript
    // Step 7: Execute the database query
    const products = await Product.find(filter)
      .populate('category', 'name description')  // Include category info
      .sort(sort)
      .limit(limit * 1)           // Convert to number and limit results
      .skip((page - 1) * limit);  // Skip products from previous pages
```

#### What does each part do?

**`.find(filter)`** - Find products matching our filter criteria

**`.populate('category', 'name description')`** - Instead of just showing the category ID, show the actual category name and description:
```javascript
// Without populate
{ name: "iPhone", category: "507f1f77bcf86cd799439011" }

// With populate  
{ name: "iPhone", category: { name: "Electronics", description: "Electronic devices" } }
```

**`.limit(limit * 1)`** - Only return this many products (convert string to number)

**`.skip((page - 1) * limit)`** - Skip products from previous pages
- Page 1: skip 0 products (0 = (1-1) * 12)
- Page 2: skip 12 products (12 = (2-1) * 12)  
- Page 3: skip 24 products (24 = (3-1) * 12)

```javascript
    // Step 8: Get total count for pagination info
    const total = await Product.countDocuments(filter);
```

#### Why count separately?
We need to know the total number of products (matching the filter) to calculate:
- How many pages total?
- Is there a next page?
- Is there a previous page?

```javascript
    // Step 9: Send response with products and pagination info
    res.json({
      success: true,
      count: products.length,  // How many products in this response
      data: {
        products,
        pagination: {
          currentPage: Number(page),
          totalPages: Math.ceil(total / limit),      // Round up: 25 products ÷ 12 per page = 3 pages
          totalProducts: total,
          hasNextPage: page < Math.ceil(total / limit),
          hasPrevPage: page > 1
        }
      }
    });
```

#### Why include pagination info?
The frontend needs this information to build:
- Page numbers: "Page 2 of 5"
- Next/Previous buttons
- "Showing 13-24 of 57 products"

```javascript
  } catch (error) {
    res.status(500).json({
      success: false,
      message: 'Error fetching products'
    });
  }
};
```

### 2. Get Single Product Controller

When a customer clicks on a product, they need detailed information about that specific product.

```javascript
// @desc    Get single product by ID
// @route   GET /api/products/:id
// @access  Public
const getProduct = async (req, res) => {
  try {
    // Step 1: Get product ID from URL parameter
    const productId = req.params.id;
```

#### What are URL parameters?
URL parameters are parts of the URL marked with `:`:
```
Route: /api/products/:id
URL: /api/products/64a5f8b2c9d4e1f2a3b4c5d6
req.params.id = "64a5f8b2c9d4e1f2a3b4c5d6"
```

```javascript
    // Step 2: Find product by ID and include category info
    const product = await Product.findById(productId)
      .populate('category', 'name description');
```

#### Why findById()?
`findById()` is a shortcut for `findOne({ _id: productId })`. It's more readable and slightly faster.

```javascript
    // Step 3: Check if product exists
    if (!product) {
      return res.status(404).json({
        success: false,
        message: 'Product not found'
      });
    }

    // Step 4: Send product data
    res.json({
      success: true,
      data: { product }
    });

  } catch (error) {
    // Handle invalid ID format
    if (error.name === 'CastError') {
      return res.status(404).json({
        success: false,
        message: 'Product not found'
      });
    }

    res.status(500).json({
      success: false,
      message: 'Error fetching product'
    });
  }
};
```

#### What is a CastError?
MongoDB IDs have a specific format (24-character hex string). If someone tries to access `/api/products/invalid-id`, MongoDB throws a CastError because "invalid-id" isn't a valid ID format.

**Example:**
- Valid ID: `64a5f8b2c9d4e1f2a3b4c5d6`
- Invalid ID: `123` or `invalid-id` or `abc`

### 3. Create Product Controller

This is for admins to add new products to the store.

```javascript
// @desc    Create new product
// @route   POST /api/products
// @access  Private/Admin only (we'll add auth later)
const createProduct = async (req, res) => {
  try {
    // Step 1: Extract product data from request body
    const {
      name,
      description,
      price,
      category,
      brand,
      stock,
      images
    } = req.body;
```

#### Why extract specific fields?
**Security:** We only accept the fields we expect. If someone tries to send malicious data like `{ isActive: false, role: 'admin' }`, we ignore it.

```javascript
    // Step 2: Create product in database
    const product = await Product.create({
      name,
      description,
      price,
      category,
      brand,
      stock,
      images
    });
```

#### Why Product.create() instead of new Product()?
Both work, but `create()` is cleaner:

**Longer way:**
```javascript
const product = new Product({ name, description, price });
await product.save();
```

**Shorter way:**
```javascript
const product = await Product.create({ name, description, price });
```

```javascript
    // Step 3: Populate category info for response
    await product.populate('category', 'name description');

    // Step 4: Send success response
    res.status(201).json({
      success: true,
      message: 'Product created successfully',
      data: { product }
    });

  } catch (error) {
    // Handle validation errors
    if (error.name === 'ValidationError') {
      const errors = Object.values(error.errors).map(err => err.message);
      return res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors
      });
    }

    res.status(500).json({
      success: false,
      message: 'Error creating product'
    });
  }
};
```

#### Why status 201?
HTTP status codes have specific meanings:
- **200** = "OK" (general success)
- **201** = "Created" (something new was successfully created)
- **400** = "Bad Request" (client error, like invalid data)
- **500** = "Internal Server Error" (our server broke)

### 4. Update Product Controller

For editing existing products.

```javascript
// @desc    Update product
// @route   PUT /api/products/:id
// @access  Private/Admin only
const updateProduct = async (req, res) => {
  try {
    // Step 1: Check if product exists
    const product = await Product.findById(req.params.id);

    if (!product) {
      return res.status(404).json({
        success: false,
        message: 'Product not found'
      });
    }
```

#### Why check if product exists first?
**Better user experience:** Instead of a generic error, we give a specific "Product not found" message.

```javascript
    // Step 2: Update product with new data
    const updatedProduct = await Product.findByIdAndUpdate(
      req.params.id,     // Which product to update
      req.body,          // New data to update with
      {
        new: true,       // Return the updated product (not the old one)
        runValidators: true  // Run schema validation on the update
      }
    ).populate('category', 'name description');
```

#### What do these options mean?

**`new: true`** - By default, `findByIdAndUpdate` returns the OLD product data. We want the NEW (updated) data.

**`runValidators: true`** - Run our schema validation rules. Without this, you could update a product with invalid data like `{ price: -100 }`.

```javascript
    // Step 3: Send updated product
    res.json({
      success: true,
      message: 'Product updated successfully',
      data: { product: updatedProduct }
    });

  } catch (error) {
    // Handle validation errors and invalid IDs
    if (error.name === 'ValidationError') {
      const errors = Object.values(error.errors).map(err => err.message);
      return res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors
      });
    }

    if (error.name === 'CastError') {
      return res.status(404).json({
        success: false,
        message: 'Product not found'
      });
    }

    res.status(500).json({
      success: false,
      message: 'Error updating product'
    });
  }
};
```

### 5. Delete Product Controller

For removing products from the store.

```javascript
// @desc    Delete product
// @route   DELETE /api/products/:id
// @access  Private/Admin only
const deleteProduct = async (req, res) => {
  try {
    // Step 1: Find the product
    const product = await Product.findById(req.params.id);

    if (!product) {
      return res.status(404).json({
        success: false,
        message: 'Product not found'
      });
    }

    // Step 2: Delete the product
    await product.deleteOne();

    // Step 3: Send confirmation
    res.json({
      success: true,
      message: 'Product deleted successfully'
    });

  } catch (error) {
    if (error.name === 'CastError') {
      return res.status(404).json({
        success: false,
        message: 'Product not found'
      });
    }

    res.status(500).json({
      success: false,
      message: 'Error deleting product'
    });
  }
};
```

#### Why deleteOne() instead of findByIdAndDelete()?
Both work, but `deleteOne()` on the found product is clearer about what we're doing. It also triggers any middleware we might have set up for deletions.

```javascript
// Export all our controller functions
module.exports = {
  getProducts,
  getProduct,
  createProduct,
  updateProduct,
  deleteProduct
};
```

---

## 🛣️ Setting Up Product Routes

Now let's connect our product controllers to actual routes that can be accessed via URLs.

### Why Separate Route Files?

**Imagine all routes in one file:**
```javascript
// server.js - Would become HUGE and messy!
app.get('/api/products', /* get all products logic - 50 lines */);
app.get('/api/products/:id', /* get single product logic - 30 lines */);
app.post('/api/products', /* create product logic - 40 lines */);
app.put('/api/products/:id', /* update product logic - 35 lines */);
app.delete('/api/products/:id', /* delete product logic - 25 lines */);
// ... plus 20 more product routes
// ... plus user routes, order routes, etc.
```

**Better approach - separate files:**
```javascript
// server.js - Clean and organized!
app.use('/api/products', productRoutes);  // All product routes in separate file
```

### Creating Product Routes

```javascript
// routes/productRoutes.js
const express = require('express');
const {
  getProducts,     // Controller function to get all products
  getProduct,      // Controller function to get single product
  createProduct,   // Controller function to create new product
  updateProduct,   // Controller function to update product
  deleteProduct    // Controller function to delete product
} = require('../controllers/productController');

const router = express.Router();
```

#### What is express.Router()?
Think of `express.Router()` as a mini Express app that handles just the routes for one feature (products in this case). It keeps routes organized and modular.

```javascript
// Public routes (anyone can access these)
router.route('/')
  .get(getProducts)      // GET /api/products - Browse all products
  .post(createProduct);  // POST /api/products - Add new product (will add auth later)

router.route('/:id')
  .get(getProduct)       // GET /api/products/123 - View specific product
  .put(updateProduct)    // PUT /api/products/123 - Edit product (will add auth later)
  .delete(deleteProduct); // DELETE /api/products/123 - Remove product (will add auth later)

module.exports = router;
```

#### Why group routes with router.route()?
Instead of writing:
```javascript
router.get('/', getProducts);
router.post('/', createProduct);
router.get('/:id', getProduct);
router.put('/:id', updateProduct);
router.delete('/:id', deleteProduct);
```

We can write:
```javascript
router.route('/')
  .get(getProducts)
  .post(createProduct);

router.route('/:id')
  .get(getProduct)
  .put(updateProduct)
  .delete(deleteProduct);
```

**Benefits:**
- **Less repetition:** Don't repeat the same URL path
- **Cleaner code:** All operations on the same endpoint grouped together
- **Easier to see:** All CRUD operations for one resource in one place

### Understanding Route Parameters

The `:id` in our routes is a **route parameter**:

```javascript
// Route definition
router.get('/:id', getProduct);

// When someone visits /api/products/64a5f8b2c9d4e1f2a3b4c5d6
// req.params.id = "64a5f8b2c9d4e1f2a3b4c5d6"
```

**Real examples:**
- `/api/products/64a5f8b2c9d4e1f2a3b4c5d6` → Get product with ID "64a5f8b2c9d4e1f2a3b4c5d6"
- `/api/products/507f1f77bcf86cd799439011` → Get product with ID "507f1f77bcf86cd799439011"

### Connecting Routes to Main Server

Now let's connect our product routes to the main server:

```javascript
// server.js
require('dotenv').config();
const express = require('express');
const connectDB = require('./config/database');

// Import route files
const productRoutes = require('./routes/productRoutes');

const app = express();

// Connect to database
connectDB();

// Middleware
app.use(express.json());  // Parse JSON request bodies

// Mount routes
app.use('/api/products', productRoutes);

// Basic route
app.get('/', (req, res) => {
  res.json({ 
    message: 'Ecommerce API is running!',
    endpoints: {
      products: '/api/products',
      singleProduct: '/api/products/:id'
    }
  });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`🚀 Server running on port ${PORT}`);
  console.log(`📍 API available at http://localhost:${PORT}/api/products`);
});
```

#### What does app.use('/api/products', productRoutes) do?

It tells Express: "For any request that starts with '/api/products', use the productRoutes file to handle it."

**How URL mapping works:**
```
Request: GET /api/products
1. Express sees it starts with '/api/products' 
2. Sends it to productRoutes
3. productRoutes sees the remaining path is '/'
4. Matches router.route('/').get(getProducts)
5. Calls getProducts controller

Request: GET /api/products/64a5f8b2c9d4e1f2a3b4c5d6
1. Express sees it starts with '/api/products'
2. Sends it to productRoutes  
3. productRoutes sees the remaining path is '/64a5f8b2c9d4e1f2a3b4c5d6'
4. Matches router.route('/:id').get(getProduct)
5. Calls getProduct controller with req.params.id = "64a5f8b2c9d4e1f2a3b4c5d6"
```

---

## 🧪 Testing Our Product API

### Using Postman (The Best Tool for Testing APIs)

**Postman** is like a playground for testing APIs. Instead of building a frontend first, we can test our backend directly to make sure it works.

### Test 1: Get All Products

**Request:**
```
Method: GET
URL: http://localhost:5000/api/products
```

**Expected Response:**
```json
{
  "success": true,
  "count": 0,
  "data": {
    "products": [],
    "pagination": {
      "currentPage": 1,
      "totalPages": 0,
      "totalProducts": 0,
      "hasNextPage": false,
      "hasPrevPage": false
    }
  }
}
```

**Why empty?** We haven't created any products yet!

### Test 2: Create a Product

**Request:**
```
Method: POST
URL: http://localhost:5000/api/products
Headers:
  Content-Type: application/json
Body (JSON):
{
  "name": "iPhone 14 Pro",
  "description": "Latest iPhone with advanced camera system and A16 Bionic chip",
  "price": 999,
  "category": "64a5f8b2c9d4e1f2a3b4c5d6",
  "brand": "Apple",
  "stock": 50,
  "images": ["https://example.com/iphone14pro.jpg"]
}
```

**Expected Response:**
```json
{
  "success": true,
  "message": "Product created successfully",
  "data": {
    "product": {
      "_id": "64a5f8b2c9d4e1f2a3b4c5d7",
      "name": "iPhone 14 Pro",
      "description": "Latest iPhone with advanced camera system and A16 Bionic chip",
      "price": 999,
      "category": {
        "_id": "64a5f8b2c9d4e1f2a3b4c5d6",
        "name": "Electronics"
      },
      "brand": "Apple",
      "stock": 50,
      "images": ["https://example.com/iphone14pro.jpg"],
      "ratings": {
        "average": 0,
        "count": 0
      },
      "slug": "iphone-14-pro",
      "isActive": true,
      "createdAt": "2023-07-06T10:30:00.000Z",
      "updatedAt": "2023-07-06T10:30:00.000Z"
    }
  }
}
```

### Test 3: Get Single Product

**Request:**
```
Method: GET
URL: http://localhost:5000/api/products/64a5f8b2c9d4e1f2a3b4c5d7
```

**Expected Response:**
```json
{
  "success": true,
  "data": {
    "product": {
      "_id": "64a5f8b2c9d4e1f2a3b4c5d7",
      "name": "iPhone 14 Pro",
      "description": "Latest iPhone with advanced camera system and A16 Bionic chip",
      "price": 999,
      // ... rest of product data
    }
  }
}
```

### Test 4: Update Product

**Request:**
```
Method: PUT
URL: http://localhost:5000/api/products/64a5f8b2c9d4e1f2a3b4c5d7
Headers:
  Content-Type: application/json
Body (JSON):
{
  "price": 899,
  "stock": 45
}
```

**Expected Response:**
```json
{
  "success": true,
  "message": "Product updated successfully",
  "data": {
    "product": {
      "_id": "64a5f8b2c9d4e1f2a3b4c5d7",
      "name": "iPhone 14 Pro",
      "price": 899,
      "stock": 45,
      // ... other fields unchanged
      "updatedAt": "2023-07-06T11:15:00.000Z"
    }
  }
}
```

### Test 5: Delete Product

**Request:**
```
Method: DELETE
URL: http://localhost:5000/api/products/64a5f8b2c9d4e1f2a3b4c5d7
```

**Expected Response:**
```json
{
  "success": true,
  "message": "Product deleted successfully"
}
```

### Test 6: Advanced Filtering

**Request:**
```
Method: GET
URL: http://localhost:5000/api/products?category=electronics&minPrice=500&maxPrice=1500&page=1&limit=5&sortBy=price&order=asc
```

This URL tests:
- **category=electronics** → Only electronics
- **minPrice=500&maxPrice=1500** → Price between $500-$1500
- **page=1&limit=5** → First page, 5 products per page
- **sortBy=price&order=asc** → Sort by price, lowest first

---

## 🚨 Common Errors and How to Fix Them

### Error 1: "Cannot GET /api/products"

**Problem:** Your route isn't registered properly.

**Solution:** Make sure you have:
```javascript
// In server.js
app.use('/api/products', productRoutes);

// In routes/productRoutes.js
router.get('/', getProducts);
```

### Error 2: "ValidationError: Product validation failed"

**Problem:** You're trying to create a product with invalid data.

**Example error:**
```json
{
  "success": false,
  "message": "Validation failed",
  "errors": [
    "Product name is required",
    "Price cannot be negative"
  ]
}
```

**Solution:** Check your request body has all required fields with valid values.

### Error 3: "CastError: Cast to ObjectId failed"

**Problem:** You're using an invalid MongoDB ID.

**Bad:** `/api/products/123`
**Good:** `/api/products/64a5f8b2c9d4e1f2a3b4c5d7`

### Error 4: "Cannot read property 'name' of undefined"

**Problem:** Trying to access a property on undefined data.

**Common cause:** Forgetting to check if data exists:
```javascript
// ❌ Bad
res.json({ name: product.name });  // What if product is null?

// ✅ Good
if (!product) {
  return res.status(404).json({ message: 'Product not found' });
}
res.json({ name: product.name });
```

---

## 📁 Complete Project Structure

After implementing everything, your project should look like this:

```
ecommerce-backend/
├── config/
│   └── database.js          # Database connection
├── controllers/
│   └── productController.js # Product business logic
├── models/
│   ├── Product.js          # Product schema
│   └── User.js             # User schema (for later)
├── routes/
│   └── productRoutes.js    # Product endpoints
├── middleware/             # For future auth middleware
├── node_modules/           # Installed packages (auto-generated)
├── .env                    # Environment variables
├── .gitignore             # Files to ignore in git
├── package.json           # Project configuration
├── package-lock.json      # Exact dependency versions
└── server.js              # Main server file
```

---

## 🎯 What We've Accomplished

### ✅ Complete CRUD API for Products
- **C**reate new products → `POST /api/products`
- **R**ead products → `GET /api/products` and `GET /api/products/:id`
- **U**pdate products → `PUT /api/products/:id`
- **D**elete products → `DELETE /api/products/:id`

### ✅ Advanced Features
- **Filtering** by category, price range
- **Search** in product names and descriptions
- **Pagination** for large product lists
- **Sorting** by different fields
- **Validation** to ensure data quality
- **Error handling** for better user experience

### ✅ Professional API Design
- **RESTful URLs** that make sense
- **Consistent response format** across all endpoints
- **Proper HTTP status codes** (200, 201, 400, 404, 500)
- **Descriptive error messages** to help developers

---

## 🚀 Next Steps

Now that you understand how to build a complete API, here's what we'll cover in future lessons:

### 1. **User Authentication & Authorization** (Next lesson)
- User registration and login
- JWT tokens for security
- Protecting routes (only admins can create/edit products)
- User roles and permissions

### 2. **Advanced Features**
- File upload for product images
- Shopping cart functionality
- Order management
- Email notifications
- Payment integration

### 3. **Production Deployment**
- Environment setup for production
- Database hosting (MongoDB Atlas)
- Server deployment (Heroku, DigitalOcean)
- API documentation with Swagger

---

## 🤔 Review Questions for Students

To make sure students understand the concepts:

### 1. **Conceptual Questions**
- Why do we separate controllers from routes?
- What's the difference between PUT and PATCH?
- Why do we use query parameters for filtering instead of request body?
- What happens if we don't validate user input?

### 2. **Practical Questions**
- How would you add a "featured products" endpoint?
- What would you change to add product reviews?
- How would you implement a "low stock" warning system?
- What additional validation would you add for product prices?

### 3. **Debugging Scenarios**
- A student gets "Product not found" for a valid product ID - what could be wrong?
- Products are being created but the category information isn't showing - what's missing?
- The API works in Postman but not from the frontend - what could be the issue?

This foundation gives students a solid understanding of building APIs that can handle real-world ecommerce requirements while explaining every design decision along the way.

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