
# 🍃 MongoDB Basics

## Table of Contents
- [What is MongoDB?](#what-is-mongodb)
- [Installation and Setup](#installation-and-setup)
- [Database Concepts](#database-concepts)
- [CRUD Operations](#crud-operations)
- [Data Modeling](#data-modeling)
- [Indexing](#indexing)
- [Aggregation](#aggregation)
- [MongoDB with Node.js](#mongodb-with-nodejs)
- [Best Practices](#best-practices)

---

## 🎯 What is MongoDB?

**MongoDB** is a NoSQL document database that stores data in flexible, JSON-like documents called BSON (Binary JSON).

### Key Features:
- **Document-oriented**: Stores data as documents
- **Schema-flexible**: No rigid schema requirements
- **Scalable**: Horizontal scaling with sharding
- **High Performance**: Fast read/write operations
- **Rich Queries**: Powerful query language

### MongoDB vs SQL Databases

| Feature | MongoDB | SQL Database |
|---------|---------|--------------|
| **Data Model** | Documents | Tables/Rows |
| **Schema** | Flexible | Fixed |
| **Relationships** | Embedded/References | Foreign Keys |
| **Scaling** | Horizontal | Vertical |
| **Query Language** | MQL | SQL |

---

## 🛠 Installation and Setup

### Install MongoDB

**Using MongoDB Atlas (Cloud - Recommended):**
1. Visit [MongoDB Atlas](https://www.mongodb.com/atlas)
2. Create free account
3. Create cluster
4. Get connection string

**Local Installation:**

```bash
# macOS with Homebrew
brew tap mongodb/brew
brew install mongodb-community

# Ubuntu
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org

# Windows
# Download from https://www.mongodb.com/try/download/community
```

### Start MongoDB

```bash
# Start MongoDB service
sudo systemctl start mongod

# Start MongoDB shell
mongosh

# Or connect to Atlas
mongosh "mongodb+srv://your-connection-string"
```

---

## 📚 Database Concepts

### Hierarchy Structure

```
MongoDB Server
├── Database 1
│   ├── Collection 1
│   │   ├── Document 1
│   │   ├── Document 2
│   │   └── Document 3
│   └── Collection 2
└── Database 2
```

### Basic Commands

```javascript
// Show databases
show dbs

// Create/switch to database
use myapp

// Show current database
db

// Show collections
show collections

// Create collection
db.createCollection("users")

// Drop database
db.dropDatabase()

// Drop collection
db.users.drop()
```

### Document Structure

```javascript
// Example document
{
  _id: ObjectId("64f1a2b3c4d5e6f7g8h9i0j1"),
  name: "John Doe",
  email: "john@example.com",
  age: 30,
  address: {
    street: "123 Main St",
    city: "New York",
    zipCode: "10001"
  },
  hobbies: ["reading", "swimming", "coding"],
  createdAt: ISODate("2023-01-15T10:30:00Z"),
  isActive: true
}
```

---

## 🔄 CRUD Operations

### Create (Insert)

```javascript
// Insert one document
db.users.insertOne({
  name: "Alice Johnson",
  email: "alice@example.com",
  age: 28,
  city: "San Francisco"
});

// Insert multiple documents
db.users.insertMany([
  {
    name: "Bob Smith",
    email: "bob@example.com",
    age: 35,
    city: "Chicago"
  },
  {
    name: "Carol Wilson",
    email: "carol@example.com",
    age: 32,
    city: "Seattle"
  }
]);

// Insert with custom _id
db.users.insertOne({
  _id: "user_001",
  name: "David Brown",
  email: "david@example.com",
  age: 29
});
```

### Read (Find)

```javascript
// Find all documents
db.users.find()

// Find with pretty formatting
db.users.find().pretty()

// Find one document
db.users.findOne()

// Find by criteria
db.users.find({ age: 30 })

// Find with multiple criteria
db.users.find({
  age: { $gte: 25 },
  city: "New York"
})

// Find with projection (select specific fields)
db.users.find(
  { age: { $gte: 25 } },
  { name: 1, email: 1, _id: 0 }
)

// Find with sorting
db.users.find().sort({ age: 1 })  // 1 for ascending, -1 for descending

// Find with limit and skip
db.users.find().limit(5).skip(10)

// Count documents
db.users.countDocuments({ age: { $gte: 25 } })
```

### Query Operators

```javascript
// Comparison operators
db.users.find({ age: { $eq: 30 } })     // Equal to
db.users.find({ age: { $ne: 30 } })     // Not equal to
db.users.find({ age: { $gt: 25 } })     // Greater than
db.users.find({ age: { $gte: 25 } })    // Greater than or equal
db.users.find({ age: { $lt: 35 } })     // Less than
db.users.find({ age: { $lte: 35 } })    // Less than or equal
db.users.find({ age: { $in: [25, 30, 35] } })    // In array
db.users.find({ age: { $nin: [25, 30] } })       // Not in array

// Logical operators
db.users.find({
  $and: [
    { age: { $gte: 25 } },
    { city: "New York" }
  ]
})

db.users.find({
  $or: [
    { age: { $lt: 25 } },
    { city: "Chicago" }
  ]
})

db.users.find({ age: { $not: { $eq: 30 } } })

// Element operators
db.users.find({ email: { $exists: true } })
db.users.find({ age: { $type: "number" } })

// Array operators
db.users.find({ hobbies: "reading" })              // Contains element
db.users.find({ hobbies: { $all: ["reading", "coding"] } })  // Contains all
db.users.find({ hobbies: { $size: 3 } })           // Array size

// Regular expressions
db.users.find({ name: /^John/ })                   // Starts with "John"
db.users.find({ email: /@gmail\.com$/ })           // Ends with "@gmail.com"
```

### Update

```javascript
// Update one document
db.users.updateOne(
  { name: "John Doe" },
  { $set: { age: 31, city: "Boston" } }
)

// Update multiple documents
db.users.updateMany(
  { city: "New York" },
  { $set: { timezone: "EST" } }
)

// Replace entire document
db.users.replaceOne(
  { name: "John Doe" },
  {
    name: "John Doe",
    email: "john.doe@example.com",
    age: 31,
    city: "Boston"
  }
)

// Update operators
db.users.updateOne(
  { name: "Alice Johnson" },
  {
    $set: { city: "Los Angeles" },     // Set field
    $inc: { age: 1 },                  // Increment number
    $push: { hobbies: "yoga" },        // Add to array
    $pull: { hobbies: "reading" },     // Remove from array
    $unset: { oldField: "" }           // Remove field
  }
)

// Upsert (update or insert)
db.users.updateOne(
  { email: "new@example.com" },
  {
    $set: {
      name: "New User",
      email: "new@example.com",
      age: 25
    }
  },
  { upsert: true }
)

// Update with array filters
db.users.updateOne(
  { "scores.subject": "math" },
  { $set: { "scores.$.grade": "A" } }
)
```

### Delete

```javascript
// Delete one document
db.users.deleteOne({ name: "John Doe" })

// Delete multiple documents
db.users.deleteMany({ age: { $lt: 18 } })

// Delete all documents in collection
db.users.deleteMany({})
```

---

## 🏗 Data Modeling

### Embedded Documents

```javascript
// User with embedded address
{
  _id: ObjectId("..."),
  name: "John Doe",
  email: "john@example.com",
  address: {
    street: "123 Main St",
    city: "New York",
    state: "NY",
    zipCode: "10001",
    coordinates: {
      latitude: 40.7128,
      longitude: -74.0060
    }
  },
  phoneNumbers: [
    { type: "home", number: "555-1234" },
    { type: "work", number: "555-5678" }
  ]
}

// Blog post with embedded comments
{
  _id: ObjectId("..."),
  title: "Introduction to MongoDB",
  content: "MongoDB is a document database...",
  author: "Alice Johnson",
  publishedAt: ISODate("2023-01-15T10:00:00Z"),
  tags: ["database", "nosql", "mongodb"],
  comments: [
    {
      _id: ObjectId("..."),
      author: "Bob Smith",
      content: "Great article!",
      createdAt: ISODate("2023-01-16T09:30:00Z")
    },
    {
      _id: ObjectId("..."),
      author: "Carol Wilson",
      content: "Very informative, thanks!",
      createdAt: ISODate("2023-01-16T14:15:00Z")
    }
  ]
}
```

### References

```javascript
// Users collection
{
  _id: ObjectId("64f1a2b3c4d5e6f7g8h9i0j1"),
  name: "John Doe",
  email: "john@example.com"
}

// Posts collection with user reference
{
  _id: ObjectId("64f1a2b3c4d5e6f7g8h9i0j2"),
  title: "My First Post",
  content: "This is my first blog post...",
  authorId: ObjectId("64f1a2b3c4d5e6f7g8h9i0j1"),  // Reference to user
  createdAt: ISODate("2023-01-15T10:00:00Z")
}

// Query with $lookup (join)
db.posts.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "authorId",
      foreignField: "_id",
      as: "author"
    }
  },
  {
    $unwind: "$author"
  }
])
```

### Schema Design Patterns

```javascript
// One-to-One (Embedded)
{
  _id: ObjectId("..."),
  name: "John Doe",
  profile: {
    bio: "Software developer",
    avatar: "avatar.jpg",
    socialMedia: {
      twitter: "@johndoe",
      linkedin: "john-doe"
    }
  }
}

// One-to-Many (Embedded Array)
{
  _id: ObjectId("..."),
  name: "John Doe",
  addresses: [
    {
      type: "home",
      street: "123 Main St",
      city: "New York"
    },
    {
      type: "work", 
      street: "456 Office Blvd",
      city: "New York"
    }
  ]
}

// Many-to-Many (References)
// Users collection
{
  _id: ObjectId("..."),
  name: "John Doe"
}

// Groups collection
{
  _id: ObjectId("..."),
  name: "JavaScript Developers",
  memberIds: [
    ObjectId("..."),  // John's ID
    ObjectId("..."),  // Other user IDs
  ]
}
```

---

## 📇 Indexing

### Basic Indexes

```javascript
// Create single field index
db.users.createIndex({ email: 1 })

// Create compound index
db.users.createIndex({ city: 1, age: -1 })

// Create text index for search
db.posts.createIndex({ title: "text", content: "text" })

// Create unique index
db.users.createIndex({ email: 1 }, { unique: true })

// Create sparse index (only for documents with the field)
db.users.createIndex({ phone: 1 }, { sparse: true })

// Create TTL index (automatic expiration)
db.sessions.createIndex(
  { createdAt: 1 }, 
  { expireAfterSeconds: 3600 }  // Expire after 1 hour
)
```

### Index Management

```javascript
// List indexes
db.users.getIndexes()

// Drop index
db.users.dropIndex({ email: 1 })
db.users.dropIndex("email_1")

// Drop all indexes (except _id)
db.users.dropIndexes()

// Explain query execution
db.users.find({ email: "john@example.com" }).explain("executionStats")

// Index statistics
db.users.stats()
```

---

## 📊 Aggregation

### Aggregation Pipeline

```javascript
// Basic aggregation pipeline
db.orders.aggregate([
  // Stage 1: Match documents
  { $match: { status: "completed" } },
  
  // Stage 2: Group and calculate
  { 
    $group: {
      _id: "$customerId",
      totalAmount: { $sum: "$amount" },
      orderCount: { $sum: 1 },
      avgAmount: { $avg: "$amount" }
    }
  },
  
  // Stage 3: Sort results
  { $sort: { totalAmount: -1 } },
  
  // Stage 4: Limit results
  { $limit: 10 }
])

// Complex aggregation example
db.sales.aggregate([
  // Match sales from last year
  {
    $match: {
      date: {
        $gte: ISODate("2023-01-01"),
        $lt: ISODate("2024-01-01")
      }
    }
  },
  
  // Add computed fields
  {
    $addFields: {
      month: { $month: "$date" },
      year: { $year: "$date" }
    }
  },
  
  // Group by month
  {
    $group: {
      _id: { year: "$year", month: "$month" },
      totalSales: { $sum: "$amount" },
      avgSale: { $avg: "$amount" },
      minSale: { $min: "$amount" },
      maxSale: { $max: "$amount" },
      salesCount: { $sum: 1 }
    }
  },
  
  // Project final structure
  {
    $project: {
      _id: 0,
      period: {
        $concat: [
          { $toString: "$_id.year" },
          "-",
          { $toString: "$_id.month" }
        ]
      },
      totalSales: 1,
      avgSale: { $round: ["$avgSale", 2] },
      salesCount: 1
    }
  },
  
  // Sort by period
  { $sort: { period: 1 } }
])
```

### Aggregation Operators

```javascript
// Arithmetic operators
db.products.aggregate([
  {
    $project: {
      name: 1,
      price: 1,
      discountedPrice: { $multiply: ["$price", 0.8] },
      tax: { $multiply: ["$price", 0.1] },
      total: {
        $add: [
          { $multiply: ["$price", 0.8] },
          { $multiply: ["$price", 0.1] }
        ]
      }
    }
  }
])

// Array operators
db.posts.aggregate([
  {
    $project: {
      title: 1,
      tagCount: { $size: "$tags" },
      hasTag: { $in: ["mongodb", "$tags"] },
      firstTag: { $arrayElemAt: ["$tags", 0] }
    }
  }
])

// String operators
db.users.aggregate([
  {
    $project: {
      name: 1,
      email: 1,
      domain: {
        $arrayElemAt: [
          { $split: ["$email", "@"] },
          1
        ]
      },
      initials: {
        $concat: [
          { $substr: ["$firstName", 0, 1] },
          { $substr: ["$lastName", 0, 1] }
        ]
      }
    }
  }
])
```

---

## 🔗 MongoDB with Node.js

### Using MongoDB Driver

```javascript
const { MongoClient } = require('mongodb');

// Connection URL
const url = 'mongodb://localhost:27017';
const dbName = 'myapp';

// Connect to MongoDB
async function connectToMongoDB() {
  const client = new MongoClient(url);
  
  try {
    await client.connect();
    console.log('Connected to MongoDB');
    
    const db = client.db(dbName);
    const collection = db.collection('users');
    
    // Insert document
    const result = await collection.insertOne({
      name: 'John Doe',
      email: 'john@example.com',
      age: 30
    });
    
    console.log('Inserted document:', result.insertedId);
    
    // Find documents
    const users = await collection.find({ age: { $gte: 25 } }).toArray();
    console.log('Found users:', users);
    
  } catch (error) {
    console.error('Error:', error);
  } finally {
    await client.close();
  }
}
```

### Using Mongoose (ODM)

```javascript
const mongoose = require('mongoose');

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/myapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// Define schema
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true
  },
  age: {
    type: Number,
    min: 0,
    max: 120
  },
  isActive: {
    type: Boolean,
    default: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

// Create model
const User = mongoose.model('User', userSchema);

// CRUD operations with Mongoose
async function userOperations() {
  try {
    // Create user
    const user = new User({
      name: 'John Doe',
      email: 'john@example.com',
      age: 30
    });
    
    await user.save();
    console.log('User created:', user);
    
    // Find users
    const users = await User.find({ age: { $gte: 25 } });
    console.log('Found users:', users);
    
    // Update user
    const updatedUser = await User.findByIdAndUpdate(
      user._id,
      { age: 31 },
      { new: true }
    );
    console.log('Updated user:', updatedUser);
    
    // Delete user
    await User.findByIdAndDelete(user._id);
    console.log('User deleted');
    
  } catch (error) {
    console.error('Error:', error);
  }
}
```

### Express + MongoDB API

```javascript
const express = require('express');
const mongoose = require('mongoose');

const app = express();
app.use(express.json());

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/api', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// User schema and model
const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  age: { type: Number, required: true }
}, { timestamps: true });

const User = mongoose.model('User', userSchema);

// Routes
app.get('/api/users', async (req, res) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.post('/api/users', async (req, res) => {
  try {
    const user = new User(req.body);
    await user.save();
    res.status(201).json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.get('/api/users/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.put('/api/users/:id', async (req, res) => {
  try {
    const user = await User.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    );
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.delete('/api/users/:id', async (req, res) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json({ message: 'User deleted successfully' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(5000, '0.0.0.0', () => {
  console.log('Server running on port 5000');
});
```

---

## ✅ Best Practices

### Schema Design

```javascript
// Good: Embedded documents for one-to-few relationships
const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  addresses: [{
    type: { type: String, enum: ['home', 'work'] },
    street: String,
    city: String,
    zipCode: String
  }]
});

// Good: References for one-to-many relationships
const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  authorId: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }
});

// Good: Denormalization for read performance
const orderSchema = new mongoose.Schema({
  items: [{
    productId: mongoose.Schema.Types.ObjectId,
    productName: String,  // Denormalized for quick access
    price: Number,
    quantity: Number
  }],
  total: Number,
  customerId: mongoose.Schema.Types.ObjectId,
  customerName: String   // Denormalized
});
```

### Performance Optimization

```javascript
// Use indexes for frequently queried fields
db.users.createIndex({ email: 1 })
db.posts.createIndex({ authorId: 1, createdAt: -1 })

// Use projection to limit returned fields
db.users.find({}, { name: 1, email: 1, _id: 0 })

// Use limit and skip for pagination
db.posts.find().sort({ createdAt: -1 }).limit(10).skip(20)

// Use aggregation for complex queries
db.orders.aggregate([
  { $match: { status: 'completed' } },
  { $group: { _id: '$customerId', total: { $sum: '$amount' } } }
])
```

### Security

```javascript
// Validate input data
const userSchema = new mongoose.Schema({
  email: {
    type: String,
    required: true,
    validate: {
      validator: function(v) {
        return /\S+@\S+\.\S+/.test(v);
      },
      message: 'Invalid email format'
    }
  },
  age: {
    type: Number,
    min: [0, 'Age cannot be negative'],
    max: [120, 'Age cannot exceed 120']
  }
});

// Use environment variables for connection strings
const mongoURI = process.env.MONGODB_URI || 'mongodb://localhost:27017/myapp';

// Enable authentication in production
// Connection with authentication
mongoose.connect('mongodb://username:password@host:port/database');
```

## 📚 Quick Reference

### Common Commands

| Operation | MongoDB Shell | Mongoose |
|-----------|---------------|----------|
| **Insert** | `db.collection.insertOne({})` | `new Model({}).save()` |
| **Find** | `db.collection.find({})` | `Model.find({})` |
| **Update** | `db.collection.updateOne({}, {$set: {}})` | `Model.findByIdAndUpdate()` |
| **Delete** | `db.collection.deleteOne({})` | `Model.findByIdAndDelete()` |

### Query Operators

| Operator | Purpose | Example |
|----------|---------|---------|
| `$eq` | Equal | `{ age: { $eq: 25 } }` |
| `$gt` | Greater than | `{ age: { $gt: 18 } }` |
| `$in` | In array | `{ status: { $in: ['active', 'pending'] } }` |
| `$and` | Logical AND | `{ $and: [{ age: { $gt: 18 } }, { city: 'NYC' }] }` |
| `$or` | Logical OR | `{ $or: [{ age: { $lt: 18 } }, { age: { $gt: 65 } }] }` |

### Aggregation Stages

| Stage | Purpose | Example |
|-------|---------|---------|
| `$match` | Filter documents | `{ $match: { status: 'active' } }` |
| `$group` | Group and aggregate | `{ $group: { _id: '$category', count: { $sum: 1 } } }` |
| `$sort` | Sort documents | `{ $sort: { createdAt: -1 } }` |
| `$project` | Select/transform fields | `{ $project: { name: 1, email: 1 } }` |
| `$lookup` | Join collections | `{ $lookup: { from: 'users', localField: 'userId', foreignField: '_id', as: 'user' } }` |

### Next Steps:
1. Learn advanced aggregation pipelines
2. Study MongoDB performance optimization
3. Explore MongoDB Atlas features
4. Learn about replica sets and sharding
5. Practice with real-world projects

This MongoDB guide provides the foundation for working with document databases in modern applications!
