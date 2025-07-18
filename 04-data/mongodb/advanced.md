
# 🚀 Advanced MongoDB

## Table of Contents
- [Sharding and Horizontal Scaling](#sharding-and-horizontal-scaling)
- [Advanced Replica Set Management](#advanced-replica-set-management)
- [MongoDB Atlas and Cloud Operations](#mongodb-atlas-and-cloud-operations)
- [Performance Monitoring and Optimization](#performance-monitoring-and-optimization)
- [Security Hardening](#security-hardening)
- [Backup and Disaster Recovery](#backup-and-disaster-recovery)
- [MongoDB with Microservices](#mongodb-with-microservices)
- [Time Series Collections](#time-series-collections)
- [Advanced Operations](#advanced-operations)

---

## ⚖️ Sharding and Horizontal Scaling

### Sharding Configuration

```javascript
// Enable sharding on database
sh.enableSharding("ecommerce");

// Shard a collection using hashed sharding
sh.shardCollection("ecommerce.orders", { customerId: "hashed" });

// Shard using range-based sharding
sh.shardCollection("ecommerce.products", { category: 1, _id: 1 });

// Create compound shard key for better distribution
sh.shardCollection("ecommerce.analytics", { 
    date: 1, 
    userId: 1 
});

// Add shard zones for geographical distribution
sh.addShardToZone("shard0000", "US-EAST");
sh.addShardToZone("shard0001", "US-WEST");
sh.addShardToZone("shard0002", "EUROPE");

// Configure zone ranges
sh.updateZoneKeyRange(
    "ecommerce.users",
    { region: "us-east", _id: MinKey },
    { region: "us-east", _id: MaxKey },
    "US-EAST"
);

sh.updateZoneKeyRange(
    "ecommerce.users",
    { region: "europe", _id: MinKey },
    { region: "europe", _id: MaxKey },
    "EUROPE"
);
```

### Shard Key Selection Strategies

```javascript
// Example: E-commerce platform sharding strategies

// 1. User-centric sharding (good for user-specific queries)
{
    shardKey: { userId: "hashed" },
    pros: ["Even distribution", "User queries hit single shard"],
    cons: ["Cross-user analytics require scatter-gather"]
}

// 2. Time-based sharding (good for time-series data)
{
    shardKey: { timestamp: 1, _id: 1 },
    pros: ["Range queries efficient", "Natural data lifecycle"],
    cons: ["Hotspotting on recent data", "Uneven write distribution"]
}

// 3. Compound sharding (balanced approach)
{
    shardKey: { tenantId: 1, category: 1, _id: 1 },
    pros: ["Multi-tenant isolation", "Category-based queries efficient"],
    cons: ["More complex", "Requires careful design"]
}

// Monitor shard distribution
db.orders.getShardDistribution();

// Check chunk distribution
sh.status();

// Manually split chunks if needed
sh.splitAt("ecommerce.orders", { customerId: ObjectId("...") });
```

---

## 🔄 Advanced Replica Set Management

### Replica Set Configuration

```javascript
// Advanced replica set configuration
rs.initiate({
    _id: "myReplicaSet",
    members: [
        { 
            _id: 0, 
            host: "primary:27017",
            priority: 2,
            tags: { region: "us-east", datacenter: "dc1" }
        },
        { 
            _id: 1, 
            host: "secondary1:27017",
            priority: 1,
            tags: { region: "us-east", datacenter: "dc2" }
        },
        { 
            _id: 2, 
            host: "secondary2:27017",
            priority: 0,
            hidden: true,
            tags: { region: "us-west", datacenter: "dc3" }
        },
        { 
            _id: 3, 
            host: "arbiter:27017",
            arbiterOnly: true
        }
    ],
    settings: {
        chainingAllowed: true,
        heartbeatIntervalMillis: 2000,
        heartbeatTimeoutSecs: 10,
        electionTimeoutMillis: 10000,
        catchUpTimeoutMillis: 60000,
        getLastErrorModes: {
            "multiDataCenter": { 
                "datacenter": 2 
            }
        },
        getLastErrorDefaults: {
            w: "majority",
            wtimeout: 5000
        }
    }
});

// Custom read preferences
db.collection.find().readPref("secondary", [
    { region: "us-east" },
    { datacenter: "dc1" }
]);

// Write concern for multi-datacenter writes
db.orders.insertOne(
    { /* document */ },
    { 
        writeConcern: { 
            w: "multiDataCenter",
            wtimeout: 5000 
        } 
    }
);
```

### Replica Set Monitoring

```javascript
// Monitor replica set status
rs.status();
rs.isMaster();
rs.printReplicationInfo();
rs.printSlaveReplicationInfo();

// Check oplog size and timing
db.oplog.rs.find().sort({ ts: -1 }).limit(1);
db.oplog.rs.stats();

// Monitor lag between nodes
rs.printSlaveReplicationInfo();

// Add member with specific configuration
rs.add({
    _id: 4,
    host: "delayed-secondary:27017",
    priority: 0,
    hidden: true,
    slaveDelay: 3600,  // 1 hour delay for point-in-time recovery
    tags: { backup: "delayed" }
});
```

---

## ☁️ MongoDB Atlas and Cloud Operations

### Atlas Configuration and Management

```javascript
// Atlas connection with connection pooling
const { MongoClient } = require('mongodb');

const client = new MongoClient(uri, {
    maxPoolSize: 10,
    serverSelectionTimeoutMS: 5000,
    socketTimeoutMS: 45000,
    bufferMaxEntries: 0,
    retryWrites: true,
    retryReads: true,
    w: 'majority',
    readPreference: 'secondaryPreferred',
    readConcern: { level: 'majority' }
});

// Atlas Data API usage
const atlasDataAPI = {
    endpoint: 'https://data.mongodb-api.com/app/<app-id>/endpoint/data/beta',
    headers: {
        'Content-Type': 'application/json',
        'api-key': process.env.ATLAS_API_KEY
    }
};

// Find documents via Data API
async function findDocuments(collection, filter) {
    const response = await fetch(`${atlasDataAPI.endpoint}/action/find`, {
        method: 'POST',
        headers: atlasDataAPI.headers,
        body: JSON.stringify({
            collection: collection,
            database: 'ecommerce',
            filter: filter
        })
    });
    
    return response.json();
}

// Atlas Search integration
db.products.createSearchIndex(
    "product_search",
    {
        mappings: {
            dynamic: false,
            fields: {
                name: {
                    type: "string",
                    analyzer: "lucene.standard"
                },
                description: {
                    type: "string",
                    analyzer: "lucene.standard"
                },
                category: {
                    type: "stringFacet"
                },
                price: {
                    type: "number"
                },
                tags: {
                    type: "string",
                    multi: true
                }
            }
        }
    }
);

// Use Atlas Search
db.products.aggregate([
    {
        $search: {
            index: "product_search",
            compound: {
                must: [
                    {
                        text: {
                            query: "laptop gaming",
                            path: ["name", "description"],
                            fuzzy: { maxEdits: 1 }
                        }
                    }
                ],
                filter: [
                    {
                        range: {
                            path: "price",
                            gte: 500,
                            lte: 2000
                        }
                    }
                ],
                should: [
                    {
                        text: {
                            query: "RGB mechanical",
                            path: "tags",
                            score: { boost: { value: 2 } }
                        }
                    }
                ]
            }
        }
    },
    {
        $addFields: {
            score: { $meta: "searchScore" }
        }
    }
]);
```

---

## 📊 Performance Monitoring and Optimization

### Advanced Performance Monitoring

```javascript
// Enable profiling for slow operations
db.setProfilingLevel(2, { slowms: 100, sampleRate: 0.1 });

// Custom profiling with operations filter
db.setProfilingLevel(2, {
    filter: {
        ts: { $gte: new Date(Date.now() - 3600000) }, // Last hour
        ns: /^myapp\./,
        millis: { $gte: 100 }
    }
});

// Analyze profiling data
db.system.profile.aggregate([
    {
        $match: {
            ts: { $gte: new Date(Date.now() - 3600000) }
        }
    },
    {
        $group: {
            _id: "$command.find",
            count: { $sum: 1 },
            avgDuration: { $avg: "$millis" },
            maxDuration: { $max: "$millis" },
            totalDuration: { $sum: "$millis" }
        }
    },
    { $sort: { totalDuration: -1 } }
]);

// Monitor index usage
db.orders.aggregate([{ $indexStats: {} }]);

// Real-time performance monitoring
function monitorPerformance() {
    const stats = db.runCommand({ serverStatus: 1 });
    
    console.log({
        connections: stats.connections,
        operations: stats.opcounters,
        memory: stats.mem,
        locks: stats.locks,
        network: stats.network,
        wiredTiger: stats.wiredTiger.cache
    });
}

// Set up monitoring interval
setInterval(monitorPerformance, 10000);
```

### Query Optimization Techniques

```javascript
// Optimize aggregation pipelines
// Bad: Early $lookup without filtering
db.orders.aggregate([
    {
        $lookup: {
            from: "customers",
            localField: "customerId",
            foreignField: "_id",
            as: "customer"
        }
    },
    {
        $match: {
            "customer.region": "US",
            orderDate: { $gte: new Date("2024-01-01") }
        }
    }
]);

// Good: Filter first, then lookup
db.orders.aggregate([
    {
        $match: {
            orderDate: { $gte: new Date("2024-01-01") }
        }
    },
    {
        $lookup: {
            from: "customers",
            let: { customerId: "$customerId" },
            pipeline: [
                {
                    $match: {
                        $expr: { $eq: ["$_id", "$$customerId"] },
                        region: "US"
                    }
                }
            ],
            as: "customer"
        }
    },
    {
        $match: {
            customer: { $ne: [] }
        }
    }
]);

// Use $addFields for computed fields instead of $project
// This preserves existing fields and is more performant
db.orders.aggregate([
    {
        $addFields: {
            totalWithTax: { 
                $multiply: ["$total", 1.08] 
            },
            year: { $year: "$orderDate" }
        }
    }
]);
```

---

## 🔒 Security Hardening

### Advanced Authentication and Authorization

```javascript
// Create custom roles with fine-grained permissions
db.runCommand({
    createRole: "orderManager",
    privileges: [
        {
            resource: { db: "ecommerce", collection: "orders" },
            actions: ["find", "insert", "update"]
        },
        {
            resource: { db: "ecommerce", collection: "products" },
            actions: ["find"]
        },
        {
            resource: { db: "ecommerce", collection: "customers" },
            actions: ["find"]
        }
    ],
    roles: []
});

// Create user with multiple roles and restrictions
db.createUser({
    user: "orderProcessor",
    pwd: passwordPrompt(),
    roles: [
        { role: "orderManager", db: "ecommerce" },
        { role: "read", db: "analytics" }
    ],
    authenticationRestrictions: [
        {
            clientSource: ["192.168.1.0/24"],
            serverAddress: ["10.0.0.0/8"]
        }
    ]
});

// Field-level redaction
db.users.aggregate([
    {
        $redact: {
            $cond: {
                if: {
                    $eq: [
                        { $ifNull: ["$owner", ""] },
                        "$$ROOT.currentUser"
                    ]
                },
                then: "$$KEEP",
                else: {
                    $cond: {
                        if: { $in: ["$$CURRENT", ["$public", "$email"]] },
                        then: "$$KEEP",
                        else: "$$REDACT"
                    }
                }
            }
        }
    }
]);
```

### Encryption and Compliance

```javascript
// Client-side field level encryption setup
const autoEncryptionOpts = {
    keyVaultNamespace: 'encryption.__keyVault',
    kmsProviders: {
        aws: {
            accessKeyId: process.env.AWS_ACCESS_KEY_ID,
            secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
            region: 'us-east-1'
        }
    },
    schemaMap: {
        'ecommerce.customers': {
            bsonType: 'object',
            encryptMetadata: {
                keyId: [new Binary(Buffer.from(process.env.DEK_UUID, 'hex'), 4)]
            },
            properties: {
                ssn: {
                    encrypt: {
                        bsonType: 'string',
                        algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic'
                    }
                },
                creditCard: {
                    encrypt: {
                        bsonType: 'string',
                        algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Random'
                    }
                }
            }
        }
    }
};

const encryptedClient = new MongoClient(uri, {
    autoEncryption: autoEncryptionOpts
});
```

---

## ⏰ Time Series Collections

### Time Series Collection Setup

```javascript
// Create time series collection for IoT data
db.createCollection("sensor_data", {
    timeseries: {
        timeField: "timestamp",
        metaField: "metadata",
        granularity: "minutes"
    }
});

// Insert time series data
db.sensor_data.insertMany([
    {
        timestamp: new Date(),
        temperature: 22.5,
        humidity: 45.0,
        pressure: 1013.25,
        metadata: {
            sensorId: "sensor_001",
            location: "warehouse_a",
            floor: 1,
            zone: "north"
        }
    },
    {
        timestamp: new Date(Date.now() - 60000),
        temperature: 22.3,
        humidity: 44.8,
        pressure: 1013.20,
        metadata: {
            sensorId: "sensor_001",
            location: "warehouse_a",
            floor: 1,
            zone: "north"
        }
    }
]);

// Query time series data with aggregation
db.sensor_data.aggregate([
    {
        $match: {
            "metadata.sensorId": "sensor_001",
            timestamp: {
                $gte: new Date(Date.now() - 24 * 60 * 60 * 1000) // Last 24 hours
            }
        }
    },
    {
        $group: {
            _id: {
                hour: { $hour: "$timestamp" },
                sensorId: "$metadata.sensorId"
            },
            avgTemperature: { $avg: "$temperature" },
            maxTemperature: { $max: "$temperature" },
            minTemperature: { $min: "$temperature" },
            readings: { $sum: 1 }
        }
    },
    {
        $sort: { "_id.hour": 1 }
    }
]);

// Window functions for time series analysis
db.sensor_data.aggregate([
    {
        $match: {
            "metadata.sensorId": "sensor_001"
        }
    },
    {
        $setWindowFields: {
            partitionBy: "$metadata.sensorId",
            sortBy: { timestamp: 1 },
            output: {
                movingAvgTemp: {
                    $avg: "$temperature",
                    window: {
                        range: [-10, 0],
                        unit: "minute"
                    }
                },
                tempDelta: {
                    $subtract: [
                        "$temperature",
                        { $shift: { output: "$temperature", by: -1 } }
                    ]
                }
            }
        }
    }
]);
```

This advanced MongoDB guide covers sharding, replica set management, Atlas operations, performance optimization, security hardening, and time series collections for enterprise-scale applications.
