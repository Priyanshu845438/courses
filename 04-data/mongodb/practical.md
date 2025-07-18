
# 🍃 Intermediate MongoDB

## Table of Contents
- [Advanced Aggregation Pipeline](#advanced-aggregation-pipeline)
- [Data Modeling Patterns](#data-modeling-patterns)
- [Indexing Strategies](#indexing-strategies)
- [Transactions and ACID](#transactions-and-acid)
- [Schema Validation](#schema-validation)
- [Change Streams](#change-streams)
- [GridFS for File Storage](#gridfs-for-file-storage)
- [Performance Optimization](#performance-optimization)
- [Replica Sets](#replica-sets)

---

## 🔄 Advanced Aggregation Pipeline

### Complex Aggregation Operations

```javascript
// Multi-stage aggregation with facets
db.orders.aggregate([
    {
        $match: {
            orderDate: { $gte: new Date("2024-01-01") }
        }
    },
    {
        $facet: {
            "salesByMonth": [
                {
                    $group: {
                        _id: {
                            year: { $year: "$orderDate" },
                            month: { $month: "$orderDate" }
                        },
                        totalSales: { $sum: "$totalAmount" },
                        orderCount: { $sum: 1 }
                    }
                },
                { $sort: { "_id.year": 1, "_id.month": 1 } }
            ],
            "topCustomers": [
                {
                    $group: {
                        _id: "$customerId",
                        totalSpent: { $sum: "$totalAmount" },
                        orderCount: { $sum: 1 }
                    }
                },
                { $sort: { totalSpent: -1 } },
                { $limit: 10 },
                {
                    $lookup: {
                        from: "customers",
                        localField: "_id",
                        foreignField: "_id",
                        as: "customerInfo"
                    }
                }
            ],
            "productPerformance": [
                { $unwind: "$items" },
                {
                    $group: {
                        _id: "$items.productId",
                        totalSold: { $sum: "$items.quantity" },
                        revenue: { $sum: { $multiply: ["$items.quantity", "$items.price"] } }
                    }
                },
                { $sort: { revenue: -1 } },
                { $limit: 20 }
            ]
        }
    }
]);

// Window functions in aggregation
db.sales.aggregate([
    {
        $setWindowFields: {
            partitionBy: "$department",
            sortBy: { date: 1 },
            output: {
                cumulativeRevenue: {
                    $sum: "$revenue",
                    window: {
                        documents: ["unbounded", "current"]
                    }
                },
                movingAverage: {
                    $avg: "$revenue",
                    window: {
                        documents: [-2, 2]
                    }
                },
                revenueRank: {
                    $rank: {}
                }
            }
        }
    }
]);
```

### Advanced Lookup Operations

```javascript
// Multiple lookups with pipeline
db.orders.aggregate([
    {
        $lookup: {
            from: "customers",
            let: { customerId: "$customerId" },
            pipeline: [
                { $match: { $expr: { $eq: ["$_id", "$$customerId"] } } },
                {
                    $lookup: {
                        from: "addresses",
                        localField: "addressIds",
                        foreignField: "_id",
                        as: "addresses"
                    }
                }
            ],
            as: "customer"
        }
    },
    {
        $lookup: {
            from: "products",
            let: { items: "$items" },
            pipeline: [
                {
                    $match: {
                        $expr: {
                            $in: ["$_id", "$$items.productId"]
                        }
                    }
                },
                {
                    $addFields: {
                        orderQuantity: {
                            $let: {
                                vars: {
                                    item: {
                                        $arrayElemAt: [
                                            {
                                                $filter: {
                                                    input: "$$items",
                                                    cond: { $eq: ["$$this.productId", "$_id"] }
                                                }
                                            },
                                            0
                                        ]
                                    }
                                },
                                in: "$$item.quantity"
                            }
                        }
                    }
                }
            ],
            as: "productDetails"
        }
    }
]);
```

---

## 🏗️ Data Modeling Patterns

### Embedded vs Referenced Documents

```javascript
// Embedded pattern (One-to-Few relationship)
{
    _id: ObjectId("..."),
    title: "MongoDB Comprehensive Guide",
    author: "John Doe",
    comments: [
        {
            _id: ObjectId("..."),
            author: "Alice",
            content: "Great article!",
            createdAt: new Date(),
            likes: 5
        },
        {
            _id: ObjectId("..."),
            author: "Bob", 
            content: "Very helpful",
            createdAt: new Date(),
            likes: 3
        }
    ],
    tags: ["mongodb", "database", "nosql"],
    createdAt: new Date(),
    updatedAt: new Date()
}

// Referenced pattern (One-to-Many relationship)
// Users collection
{
    _id: ObjectId("user123"),
    username: "john_doe",
    email: "john@example.com",
    profile: {
        firstName: "John",
        lastName: "Doe",
        bio: "Software developer"
    },
    preferences: {
        theme: "dark",
        notifications: true
    }
}

// Posts collection
{
    _id: ObjectId("post123"),
    title: "My First Blog Post",
    content: "Content here...",
    authorId: ObjectId("user123"),
    categoryId: ObjectId("category123"),
    publishedAt: new Date(),
    metadata: {
        views: 150,
        shares: 25,
        featured: false
    }
}
```

### Polymorphic Pattern

```javascript
// Products with different types
{
    _id: ObjectId("..."),
    name: "Gaming Laptop",
    price: 1299.99,
    category: "electronics",
    type: "laptop",
    specifications: {
        processor: "Intel i7",
        memory: "16GB",
        storage: "512GB SSD",
        graphics: "NVIDIA RTX 3070"
    },
    warranty: "2 years",
    createdAt: new Date()
}

{
    _id: ObjectId("..."),
    name: "Organic Cotton T-Shirt",
    price: 29.99,
    category: "clothing",
    type: "shirt",
    specifications: {
        material: "100% Organic Cotton",
        sizes: ["S", "M", "L", "XL"],
        colors: ["black", "white", "navy"],
        care: "Machine wash cold"
    },
    seasonalDiscount: 0.15,
    createdAt: new Date()
}

// Query polymorphic data
db.products.aggregate([
    {
        $match: { type: "laptop" }
    },
    {
        $addFields: {
            processorBrand: {
                $arrayElemAt: [
                    { $split: ["$specifications.processor", " "] },
                    0
                ]
            }
        }
    }
]);
```

---

## 🚀 Indexing Strategies

### Compound and Partial Indexes

```javascript
// Compound index for multiple fields
db.orders.createIndex({ 
    customerId: 1, 
    orderDate: -1, 
    status: 1 
});

// Partial index for specific conditions
db.products.createIndex(
    { name: 1, category: 1 },
    { 
        partialFilterExpression: { 
            price: { $gt: 100 },
            inStock: true 
        } 
    }
);

// Sparse index (only documents with the field)
db.users.createIndex(
    { "profile.socialSecurityNumber": 1 },
    { sparse: true }
);

// TTL index for automatic document expiration
db.sessions.createIndex(
    { createdAt: 1 },
    { expireAfterSeconds: 3600 } // 1 hour
);

// Text index for full-text search
db.articles.createIndex({
    title: "text",
    content: "text",
    tags: "text"
}, {
    weights: {
        title: 10,
        content: 5,
        tags: 1
    },
    name: "article_text_index"
});

// Use text search
db.articles.find({
    $text: { $search: "mongodb database tutorial" }
}).sort({ score: { $meta: "textScore" } });
```

### Index Performance Analysis

```javascript
// Explain query execution
db.orders.find({ customerId: 123, status: "shipped" }).explain("executionStats");

// Get index usage statistics
db.orders.aggregate([{ $indexStats: {} }]);

// Hint to use specific index
db.orders.find({ customerId: 123 }).hint({ customerId: 1, orderDate: -1 });

// Check slow queries
db.setProfilingLevel(2, { slowms: 100 });
db.system.profile.find().sort({ ts: -1 }).limit(5);
```

---

## 🔐 Transactions and ACID

### Multi-Document Transactions

```javascript
// Transaction example with error handling
async function transferFunds(fromAccount, toAccount, amount) {
    const session = client.startSession();
    
    try {
        await session.withTransaction(async () => {
            // Check source account balance
            const sourceAccount = await db.accounts.findOne(
                { _id: fromAccount },
                { session }
            );
            
            if (sourceAccount.balance < amount) {
                throw new Error("Insufficient funds");
            }
            
            // Debit source account
            await db.accounts.updateOne(
                { _id: fromAccount },
                { 
                    $inc: { balance: -amount },
                    $push: { 
                        transactions: {
                            type: "debit",
                            amount: amount,
                            to: toAccount,
                            timestamp: new Date()
                        }
                    }
                },
                { session }
            );
            
            // Credit target account
            await db.accounts.updateOne(
                { _id: toAccount },
                { 
                    $inc: { balance: amount },
                    $push: { 
                        transactions: {
                            type: "credit",
                            amount: amount,
                            from: fromAccount,
                            timestamp: new Date()
                        }
                    }
                },
                { session }
            );
            
            // Log transaction
            await db.transactionLogs.insertOne({
                from: fromAccount,
                to: toAccount,
                amount: amount,
                status: "completed",
                timestamp: new Date()
            }, { session });
            
        }, {
            readConcern: { level: "majority" },
            writeConcern: { w: "majority" },
            readPreference: "primary"
        });
        
    } finally {
        await session.endSession();
    }
}
```

### Transaction with Retry Logic

```javascript
async function performTransactionWithRetry(operations, maxRetries = 3) {
    let retryCount = 0;
    
    while (retryCount < maxRetries) {
        const session = client.startSession();
        
        try {
            await session.withTransaction(async () => {
                for (const operation of operations) {
                    await operation(session);
                }
            });
            
            await session.endSession();
            return; // Success
            
        } catch (error) {
            await session.endSession();
            
            if (error.hasErrorLabel('TransientTransactionError')) {
                retryCount++;
                console.log(`Transaction failed, retrying... (${retryCount}/${maxRetries})`);
                await new Promise(resolve => setTimeout(resolve, 100 * retryCount));
            } else {
                throw error; // Non-transient error
            }
        }
    }
    
    throw new Error(`Transaction failed after ${maxRetries} retries`);
}
```

---

## ✅ Schema Validation

### JSON Schema Validation

```javascript
// Create collection with validation rules
db.createCollection("users", {
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["email", "username", "createdAt"],
            properties: {
                email: {
                    bsonType: "string",
                    pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$",
                    description: "Must be a valid email address"
                },
                username: {
                    bsonType: "string",
                    minLength: 3,
                    maxLength: 30,
                    pattern: "^[a-zA-Z0-9_]+$",
                    description: "Username must be 3-30 characters, alphanumeric and underscore only"
                },
                age: {
                    bsonType: "int",
                    minimum: 13,
                    maximum: 120,
                    description: "Age must be between 13 and 120"
                },
                profile: {
                    bsonType: "object",
                    properties: {
                        firstName: { bsonType: "string" },
                        lastName: { bsonType: "string" },
                        bio: { 
                            bsonType: "string",
                            maxLength: 500 
                        }
                    }
                },
                preferences: {
                    bsonType: "object",
                    properties: {
                        notifications: { bsonType: "bool" },
                        theme: { 
                            enum: ["light", "dark", "auto"] 
                        }
                    }
                },
                createdAt: { bsonType: "date" }
            }
        }
    },
    validationLevel: "strict",
    validationAction: "error"
});

// Add validation to existing collection
db.runCommand({
    collMod: "products",
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["name", "price", "category"],
            properties: {
                name: { bsonType: "string" },
                price: { 
                    bsonType: "double",
                    minimum: 0 
                },
                category: { 
                    enum: ["electronics", "clothing", "books", "home"] 
                },
                tags: {
                    bsonType: "array",
                    items: { bsonType: "string" },
                    maxItems: 10
                }
            }
        }
    }
});
```

This intermediate MongoDB guide covers advanced aggregation pipelines, data modeling patterns, indexing strategies, transactions, schema validation, and performance optimization techniques for building robust applications.
