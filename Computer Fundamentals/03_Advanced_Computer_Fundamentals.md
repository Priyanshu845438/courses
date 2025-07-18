
# 🚀 Advanced Computer Fundamentals

## Table of Contents
- [Advanced Computer Architecture](#advanced-computer-architecture)
- [Parallel Computing](#parallel-computing)
- [Advanced Operating Systems](#advanced-operating-systems)
- [Distributed Systems](#distributed-systems)
- [Advanced Security](#advanced-security)
- [Performance Engineering](#performance-engineering)
- [Emerging Technologies](#emerging-technologies)

---

## 🏗️ Advanced Computer Architecture

### Modern CPU Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Modern CPU Core                      │
├─────────────┬─────────────┬─────────────┬─────────────┤
│   Fetch     │   Decode    │  Execute    │ Writeback   │
│    Unit     │    Unit     │    Unit     │    Unit     │
├─────────────┼─────────────┼─────────────┼─────────────┤
│ Instruction │ Instruction │    ALU      │   Result    │
│   Cache     │  Decoder    │    FPU      │   Buffer    │
│    (I$)     │             │   Vector    │             │
├─────────────┼─────────────┼─────────────┼─────────────┤
│             │ Branch      │ Out-of-     │ Register    │
│             │ Predictor   │ Order       │    File     │
│             │             │ Execution   │             │
└─────────────┴─────────────┴─────────────┴─────────────┘
```

### CPU Features Comparison

| Feature | Intel i9-13900K | AMD Ryzen 9 7950X | Apple M2 Max |
|---------|-----------------|-------------------|--------------|
| **Cores** | 24 (8P + 16E) | 16 | 12 |
| **Threads** | 32 | 32 | 12 |
| **Process** | Intel 7 (10nm) | TSMC 5nm | TSMC 5nm |
| **Cache L3** | 36MB | 64MB | 24MB |
| **TDP** | 125W | 170W | 30W |
| **Architecture** | Hybrid | Zen 4 | ARM64 |

### Cache Hierarchy Deep Dive

```python
# Cache simulation example
class CacheSimulator:
    def __init__(self, cache_size, block_size, associativity):
        self.cache_size = cache_size
        self.block_size = block_size
        self.associativity = associativity
        self.sets = cache_size // (block_size * associativity)
        self.cache = {}
        self.hits = 0
        self.misses = 0
    
    def access(self, address):
        block_address = address // self.block_size
        set_index = block_address % self.sets
        tag = block_address // self.sets
        
        cache_key = (set_index, tag)
        
        if cache_key in self.cache:
            self.hits += 1
            print(f"Cache HIT: Address {address}")
            return True
        else:
            self.misses += 1
            print(f"Cache MISS: Address {address}")
            # Simulate cache loading
            self.cache[cache_key] = True
            return False
    
    def hit_rate(self):
        total = self.hits + self.misses
        return self.hits / total if total > 0 else 0

# Example usage
cache = CacheSimulator(cache_size=1024, block_size=64, associativity=4)

# Simulate memory accesses
addresses = [100, 164, 100, 228, 164, 292]
for addr in addresses:
    cache.access(addr)

print(f"Hit rate: {cache.hit_rate():.2%}")
```

### SIMD (Single Instruction, Multiple Data)

```c
// AVX-512 example in C
#include <immintrin.h>
#include <stdio.h>

void vector_add_simd(float* a, float* b, float* result, int size) {
    // Process 16 floats at once with AVX-512
    for (int i = 0; i < size; i += 16) {
        __m512 va = _mm512_load_ps(&a[i]);
        __m512 vb = _mm512_load_ps(&b[i]);
        __m512 vresult = _mm512_add_ps(va, vb);
        _mm512_store_ps(&result[i], vresult);
    }
}

// Benchmark comparison
void vector_add_scalar(float* a, float* b, float* result, int size) {
    for (int i = 0; i < size; i++) {
        result[i] = a[i] + b[i];
    }
}
```

---

## ⚡ Parallel Computing

### Types of Parallelism

```
┌─────────────────────────────────────────┐
│           Parallelism Types             │
├─────────────────┬───────────────────────┤
│ Data Parallel   │ Task Parallel         │
├─────────────────┼───────────────────────┤
│ • Same operation│ • Different operations│
│   on different  │   on same/different   │
│   data          │   data                │
├─────────────────┼───────────────────────┤
│ Examples:       │ Examples:             │
│ • Vector ops    │ • Producer-Consumer   │
│ • Matrix mult   │ • Pipeline            │
│ • Map-Reduce    │ • Fork-Join           │
└─────────────────┴───────────────────────┘
```

### Parallel Programming Models

#### 1. Shared Memory (OpenMP)
```c
#include <omp.h>
#include <stdio.h>

void parallel_matrix_multiply() {
    const int N = 1000;
    double A[N][N], B[N][N], C[N][N];
    
    // Initialize matrices
    #pragma omp parallel for
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            A[i][j] = i + j;
            B[i][j] = i - j;
            C[i][j] = 0.0;
        }
    }
    
    // Parallel matrix multiplication
    #pragma omp parallel for
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            for (int k = 0; k < N; k++) {
                C[i][j] += A[i][k] * B[k][j];
            }
        }
    }
}
```

#### 2. Message Passing (MPI)
```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    MPI_Init(&argc, &argv);
    
    int rank, size;
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);
    
    const int N = 1000000;
    int local_n = N / size;
    double local_sum = 0.0;
    
    // Each process calculates partial sum
    for (int i = rank * local_n; i < (rank + 1) * local_n; i++) {
        local_sum += 1.0 / (1.0 + i * i);
    }
    
    // Reduce all partial sums
    double global_sum;
    MPI_Reduce(&local_sum, &global_sum, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD);
    
    if (rank == 0) {
        printf("Pi approximation: %f\n", 4.0 * global_sum);
    }
    
    MPI_Finalize();
    return 0;
}
```

#### 3. GPU Computing (CUDA)
```cuda
// CUDA kernel for vector addition
__global__ void vectorAdd(float* A, float* B, float* C, int N) {
    int i = blockDim.x * blockIdx.x + threadIdx.x;
    if (i < N) {
        C[i] = A[i] + B[i];
    }
}

// Host code
void cudaVectorAdd() {
    const int N = 1000000;
    size_t size = N * sizeof(float);
    
    // Allocate device memory
    float *d_A, *d_B, *d_C;
    cudaMalloc(&d_A, size);
    cudaMalloc(&d_B, size);
    cudaMalloc(&d_C, size);
    
    // Copy data to device
    cudaMemcpy(d_A, h_A, size, cudaMemcpyHostToDevice);
    cudaMemcpy(d_B, h_B, size, cudaMemcpyHostToDevice);
    
    // Launch kernel
    int threadsPerBlock = 256;
    int blocksPerGrid = (N + threadsPerBlock - 1) / threadsPerBlock;
    vectorAdd<<<blocksPerGrid, threadsPerBlock>>>(d_A, d_B, d_C, N);
    
    // Copy result back
    cudaMemcpy(h_C, d_C, size, cudaMemcpyDeviceToHost);
    
    // Cleanup
    cudaFree(d_A); cudaFree(d_B); cudaFree(d_C);
}
```

### Performance Analysis

| Metric | Formula | Example |
|--------|---------|---------|
| **Speedup** | S = T₁ / Tₚ | 8-core speedup = 6.4x |
| **Efficiency** | E = S / P | E = 6.4 / 8 = 80% |
| **Scalability** | S(2P) / S(P) | Strong vs weak scaling |
| **Amdahl's Law** | S ≤ 1 / (s + (1-s)/P) | Serial fraction limits |

---

## 🖥️ Advanced Operating Systems

### Modern OS Architecture

```
┌─────────────────────────────────────────┐
│              Microkernel                │
├─────────────┬───────────────────────────┤
│ User Space  │ • File System             │
│ Services    │ • Device Drivers          │
│             │ • Network Stack           │
├─────────────┼───────────────────────────┤
│ Kernel      │ • IPC                     │
│ Space       │ • Memory Management       │
│             │ • Process Scheduling      │
└─────────────┴───────────────────────────┘
```

### Advanced Scheduling Algorithms

#### Completely Fair Scheduler (CFS)
```python
class CFSScheduler:
    def __init__(self):
        self.tasks = []
        self.time_slice = 10  # milliseconds
    
    def add_task(self, task_id, nice_value=0):
        weight = 1024 / (1.25 ** nice_value)
        task = {
            'id': task_id,
            'weight': weight,
            'vruntime': 0,
            'runtime': 0
        }
        self.tasks.append(task)
    
    def schedule_next(self):
        if not self.tasks:
            return None
        
        # Select task with minimum vruntime
        next_task = min(self.tasks, key=lambda t: t['vruntime'])
        
        # Calculate time slice based on weight
        total_weight = sum(t['weight'] for t in self.tasks)
        time_slice = (next_task['weight'] / total_weight) * self.time_slice
        
        return next_task, time_slice
    
    def update_vruntime(self, task, elapsed_time):
        # Update virtual runtime
        task['vruntime'] += elapsed_time * (1024 / task['weight'])
        task['runtime'] += elapsed_time

# Example usage
scheduler = CFSScheduler()
scheduler.add_task('Task1', nice_value=0)   # Normal priority
scheduler.add_task('Task2', nice_value=5)   # Lower priority
scheduler.add_task('Task3', nice_value=-5)  # Higher priority

for _ in range(5):
    task, time_slice = scheduler.schedule_next()
    print(f"Running {task['id']} for {time_slice:.2f}ms")
    scheduler.update_vruntime(task, time_slice)
```

### Memory Management

#### Virtual Memory System
```c
// Page fault handler simulation
struct page_table_entry {
    unsigned int present : 1;
    unsigned int writable : 1;
    unsigned int user : 1;
    unsigned int accessed : 1;
    unsigned int dirty : 1;
    unsigned int frame : 20;
};

void handle_page_fault(uint32_t virtual_addr) {
    uint32_t page_number = virtual_addr >> 12;
    uint32_t offset = virtual_addr & 0xFFF;
    
    struct page_table_entry *pte = &page_table[page_number];
    
    if (!pte->present) {
        // Page not in memory - load from disk
        uint32_t frame = allocate_frame();
        load_page_from_disk(page_number, frame);
        
        pte->frame = frame;
        pte->present = 1;
        pte->accessed = 1;
        
        // Update TLB
        tlb_insert(page_number, frame);
    }
}
```

#### Advanced Page Replacement
```python
class LRUPageReplacement:
    def __init__(self, capacity):
        self.capacity = capacity
        self.pages = {}
        self.access_order = []
    
    def access_page(self, page_id):
        if page_id in self.pages:
            # Page hit - update access order
            self.access_order.remove(page_id)
            self.access_order.append(page_id)
            return "HIT"
        else:
            # Page miss
            if len(self.pages) >= self.capacity:
                # Evict least recently used page
                lru_page = self.access_order.pop(0)
                del self.pages[lru_page]
                print(f"Evicted page {lru_page}")
            
            # Load new page
            self.pages[page_id] = True
            self.access_order.append(page_id)
            return "MISS"

# Working Set Algorithm
class WorkingSetReplacement:
    def __init__(self, window_size):
        self.window_size = window_size
        self.access_history = []
        self.working_set = set()
    
    def access_page(self, page_id, timestamp):
        self.access_history.append((page_id, timestamp))
        
        # Remove old accesses outside window
        cutoff_time = timestamp - self.window_size
        self.access_history = [(p, t) for p, t in self.access_history if t > cutoff_time]
        
        # Update working set
        self.working_set = set(p for p, t in self.access_history)
        
        return len(self.working_set)
```

---

## 🌐 Distributed Systems

### Distributed System Characteristics

| Property | Description | Challenges |
|----------|-------------|------------|
| **Scalability** | Handle increasing load | Load balancing, partitioning |
| **Reliability** | Continue despite failures | Fault tolerance, redundancy |
| **Availability** | System remains accessible | High availability design |
| **Consistency** | Data consistency across nodes | CAP theorem trade-offs |
| **Partition Tolerance** | Function despite network failures | Split-brain scenarios |

### CAP Theorem

```
        Consistency
           /\
          /  \
         /    \
        /      \
   CA  /        \ CP
      /          \
     /            \
    /      AP      \
   /________________\
  Availability  Partition
                Tolerance

• CA: Traditional RDBMS (MySQL, PostgreSQL)
• CP: MongoDB, Redis
• AP: Cassandra, DynamoDB
```

### Consensus Algorithms

#### Raft Algorithm Implementation
```python
import random
import time
from enum import Enum

class NodeState(Enum):
    FOLLOWER = 1
    CANDIDATE = 2
    LEADER = 3

class RaftNode:
    def __init__(self, node_id, cluster_size):
        self.node_id = node_id
        self.cluster_size = cluster_size
        self.state = NodeState.FOLLOWER
        self.current_term = 0
        self.voted_for = None
        self.log = []
        self.commit_index = 0
        self.last_applied = 0
        
        # Leader state
        self.next_index = {}
        self.match_index = {}
        
        # Election timeout
        self.election_timeout = random.uniform(150, 300)  # ms
        self.last_heartbeat = time.time()
    
    def start_election(self):
        self.state = NodeState.CANDIDATE
        self.current_term += 1
        self.voted_for = self.node_id
        votes_received = 1  # Vote for self
        
        print(f"Node {self.node_id} starting election for term {self.current_term}")
        
        # Request votes from other nodes
        for node_id in range(self.cluster_size):
            if node_id != self.node_id:
                if self.request_vote(node_id):
                    votes_received += 1
        
        # Check if won election
        if votes_received > self.cluster_size // 2:
            self.become_leader()
        else:
            self.state = NodeState.FOLLOWER
    
    def become_leader(self):
        self.state = NodeState.LEADER
        print(f"Node {self.node_id} became leader for term {self.current_term}")
        
        # Initialize leader state
        for i in range(self.cluster_size):
            if i != self.node_id:
                self.next_index[i] = len(self.log)
                self.match_index[i] = 0
        
        # Send heartbeats
        self.send_heartbeats()
    
    def append_entry(self, entry):
        if self.state == NodeState.LEADER:
            self.log.append({
                'term': self.current_term,
                'entry': entry,
                'index': len(self.log)
            })
            
            # Replicate to followers
            self.replicate_to_followers()
    
    def replicate_to_followers(self):
        for node_id in range(self.cluster_size):
            if node_id != self.node_id:
                self.send_append_entries(node_id)

# Example usage
cluster = [RaftNode(i, 5) for i in range(5)]
leader = cluster[0]
leader.become_leader()
leader.append_entry("SET x = 10")
```

### Distributed Hash Tables (DHT)

```python
import hashlib
import bisect

class ConsistentHash:
    def __init__(self, nodes=None, replicas=3):
        self.replicas = replicas
        self.ring = {}
        self.sorted_keys = []
        
        if nodes:
            for node in nodes:
                self.add_node(node)
    
    def _hash(self, key):
        return int(hashlib.md5(key.encode()).hexdigest(), 16)
    
    def add_node(self, node):
        for i in range(self.replicas):
            virtual_key = self._hash(f"{node}:{i}")
            self.ring[virtual_key] = node
            bisect.insort(self.sorted_keys, virtual_key)
    
    def remove_node(self, node):
        for i in range(self.replicas):
            virtual_key = self._hash(f"{node}:{i}")
            if virtual_key in self.ring:
                del self.ring[virtual_key]
                self.sorted_keys.remove(virtual_key)
    
    def get_node(self, key):
        if not self.ring:
            return None
        
        hash_key = self._hash(key)
        idx = bisect.bisect_right(self.sorted_keys, hash_key)
        
        if idx == len(self.sorted_keys):
            idx = 0
        
        return self.ring[self.sorted_keys[idx]]

# Example usage
nodes = ['server1', 'server2', 'server3', 'server4']
ch = ConsistentHash(nodes)

# Test key distribution
keys = [f"key{i}" for i in range(100)]
distribution = {}

for key in keys:
    node = ch.get_node(key)
    distribution[node] = distribution.get(node, 0) + 1

print("Key distribution:", distribution)

# Add new node
ch.add_node('server5')
print("After adding server5:")

new_distribution = {}
for key in keys:
    node = ch.get_node(key)
    new_distribution[node] = new_distribution.get(node, 0) + 1

print("New distribution:", new_distribution)
```

---

## 🔐 Advanced Security

### Cryptographic Protocols

#### Elliptic Curve Cryptography (ECC)
```python
import hashlib
import secrets
from typing import Tuple

class EllipticCurve:
    def __init__(self, a: int, b: int, p: int):
        self.a = a
        self.b = b
        self.p = p  # Prime modulus
    
    def point_add(self, P: Tuple[int, int], Q: Tuple[int, int]) -> Tuple[int, int]:
        if P is None:
            return Q
        if Q is None:
            return P
        
        x1, y1 = P
        x2, y2 = Q
        
        if x1 == x2:
            if y1 == y2:
                # Point doubling
                s = (3 * x1 * x1 + self.a) * pow(2 * y1, -1, self.p) % self.p
            else:
                return None  # Point at infinity
        else:
            # Point addition
            s = (y2 - y1) * pow(x2 - x1, -1, self.p) % self.p
        
        x3 = (s * s - x1 - x2) % self.p
        y3 = (s * (x1 - x3) - y1) % self.p
        
        return (x3, y3)
    
    def scalar_mult(self, k: int, P: Tuple[int, int]) -> Tuple[int, int]:
        if k == 0:
            return None
        if k == 1:
            return P
        
        result = None
        addend = P
        
        while k:
            if k & 1:
                result = self.point_add(result, addend)
            addend = self.point_add(addend, addend)
            k >>= 1
        
        return result

# Digital Signature (ECDSA)
class ECDSA:
    def __init__(self, curve, generator, order):
        self.curve = curve
        self.G = generator
        self.n = order
    
    def generate_keypair(self):
        private_key = secrets.randbelow(self.n)
        public_key = self.curve.scalar_mult(private_key, self.G)
        return private_key, public_key
    
    def sign(self, message: bytes, private_key: int) -> Tuple[int, int]:
        z = int.from_bytes(hashlib.sha256(message).digest(), 'big')
        
        while True:
            k = secrets.randbelow(self.n)
            x, y = self.curve.scalar_mult(k, self.G)
            r = x % self.n
            
            if r == 0:
                continue
            
            s = (pow(k, -1, self.n) * (z + r * private_key)) % self.n
            
            if s != 0:
                return (r, s)
    
    def verify(self, message: bytes, signature: Tuple[int, int], public_key: Tuple[int, int]) -> bool:
        r, s = signature
        z = int.from_bytes(hashlib.sha256(message).digest(), 'big')
        
        w = pow(s, -1, self.n)
        u1 = (z * w) % self.n
        u2 = (r * w) % self.n
        
        point1 = self.curve.scalar_mult(u1, self.G)
        point2 = self.curve.scalar_mult(u2, public_key)
        point = self.curve.point_add(point1, point2)
        
        return point[0] % self.n == r
```

### Advanced Attack Techniques

#### Side-Channel Attack Simulation
```python
import time
import random
import numpy as np

class TimingAttack:
    def __init__(self):
        self.secret_key = "super_secret_password"
    
    def vulnerable_compare(self, user_input):
        """Vulnerable comparison that leaks timing information"""
        if len(user_input) != len(self.secret_key):
            return False
        
        for i in range(len(self.secret_key)):
            if user_input[i] != self.secret_key[i]:
                return False
            # Simulate processing time per character
            time.sleep(0.001)  # 1ms delay per correct character
        return True
    
    def secure_compare(self, user_input):
        """Constant-time comparison"""
        if len(user_input) != len(self.secret_key):
            return False
        
        result = 0
        for i in range(len(self.secret_key)):
            result |= ord(user_input[i]) ^ ord(self.secret_key[i])
            time.sleep(0.001)  # Fixed delay regardless of match
        
        return result == 0

def timing_attack_demo():
    target = TimingAttack()
    charset = "abcdefghijklmnopqrstuvwxyz_"
    discovered = ""
    
    print("Performing timing attack...")
    
    for position in range(len(target.secret_key)):
        best_char = None
        max_time = 0
        
        for char in charset:
            test_string = discovered + char + "x" * (len(target.secret_key) - position - 1)
            
            # Measure timing
            start_time = time.time()
            target.vulnerable_compare(test_string)
            elapsed = time.time() - start_time
            
            if elapsed > max_time:
                max_time = elapsed
                best_char = char
        
        discovered += best_char
        print(f"Position {position}: {best_char} (time: {max_time:.4f}s)")
        print(f"Discovered so far: {discovered}")
    
    print(f"Attack complete! Discovered: {discovered}")

# Power Analysis Attack Simulation
class PowerAnalysis:
    def simulate_aes_power_consumption(self, plaintext, key):
        """Simulate power consumption during AES operation"""
        # Hamming weight model - power proportional to number of 1s
        intermediate = plaintext ^ key
        hamming_weight = bin(intermediate).count('1')
        
        # Add noise
        noise = random.gauss(0, 0.1)
        return hamming_weight + noise
    
    def differential_power_analysis(self, traces, plaintexts, key_hypothesis):
        """Perform DPA attack"""
        group0 = []
        group1 = []
        
        for i, plaintext in enumerate(plaintexts):
            # Predict intermediate value
            predicted = plaintext ^ key_hypothesis
            
            # Group based on bit value (e.g., LSB)
            if predicted & 1:
                group1.append(traces[i])
            else:
                group0.append(traces[i])
        
        # Calculate difference of means
        if group0 and group1:
            mean0 = np.mean(group0)
            mean1 = np.mean(group1)
            return abs(mean1 - mean0)
        return 0

# Example usage
timing_attack_demo()
```

---

## ⚡ Performance Engineering

### Advanced Profiling Techniques

```python
import cProfile
import pstats
import functools
import time
from memory_profiler import profile

# CPU Profiling Decorator
def cpu_profile(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        profiler = cProfile.Profile()
        profiler.enable()
        result = func(*args, **kwargs)
        profiler.disable()
        
        stats = pstats.Stats(profiler)
        stats.sort_stats('cumulative')
        stats.print_stats(10)  # Top 10 functions
        
        return result
    return wrapper

# Memory Profiling Example
@profile
def memory_intensive_function():
    # Create large lists
    big_list = [i ** 2 for i in range(1000000)]
    another_list = [x * 2 for x in big_list]
    
    # Dictionary operations
    big_dict = {i: str(i) for i in range(100000)}
    
    return len(big_list) + len(another_list) + len(big_dict)

# Cache-Aware Programming
class CacheOptimizedMatrix:
    def __init__(self, size):
        self.size = size
        self.data = [[0] * size for _ in range(size)]
    
    def multiply_naive(self, other):
        """Cache-unfriendly matrix multiplication"""
        result = CacheOptimizedMatrix(self.size)
        for i in range(self.size):
            for j in range(self.size):
                for k in range(self.size):
                    result.data[i][j] += self.data[i][k] * other.data[k][j]
        return result
    
    def multiply_blocked(self, other, block_size=64):
        """Cache-friendly blocked matrix multiplication"""
        result = CacheOptimizedMatrix(self.size)
        
        for i0 in range(0, self.size, block_size):
            for j0 in range(0, self.size, block_size):
                for k0 in range(0, self.size, block_size):
                    # Process block
                    for i in range(i0, min(i0 + block_size, self.size)):
                        for j in range(j0, min(j0 + block_size, self.size)):
                            for k in range(k0, min(k0 + block_size, self.size)):
                                result.data[i][j] += self.data[i][k] * other.data[k][j]
        return result

# Lock-Free Data Structures
import threading
from queue import Queue

class LockFreeStack:
    def __init__(self):
        self.head = None
        self._lock = threading.Lock()  # For comparison
    
    def push_lockfree(self, value):
        """Compare-and-swap based push"""
        new_node = {'value': value, 'next': None}
        
        while True:
            current_head = self.head
            new_node['next'] = current_head
            
            # Atomic compare-and-swap simulation
            if self._cas(current_head, new_node):
                break
    
    def pop_lockfree(self):
        """Compare-and-swap based pop"""
        while True:
            current_head = self.head
            if current_head is None:
                return None
            
            next_node = current_head['next']
            
            # Atomic compare-and-swap simulation
            if self._cas(current_head, next_node):
                return current_head['value']
    
    def _cas(self, expected, new_value):
        """Simulate compare-and-swap operation"""
        with self._lock:  # In real implementation, this would be atomic
            if self.head == expected:
                self.head = new_value
                return True
            return False

# Performance Benchmarking
class PerformanceBenchmark:
    def __init__(self):
        self.results = {}
    
    def benchmark(self, func, name, *args, **kwargs):
        times = []
        for _ in range(10):  # Run 10 times
            start = time.perf_counter()
            result = func(*args, **kwargs)
            end = time.perf_counter()
            times.append(end - start)
        
        self.results[name] = {
            'mean': sum(times) / len(times),
            'min': min(times),
            'max': max(times),
            'std': (sum((t - sum(times)/len(times))**2 for t in times) / len(times))**0.5
        }
        
        return result
    
    def print_results(self):
        print("\nBenchmark Results:")
        print("-" * 60)
        print(f"{'Function':<20} {'Mean (s)':<12} {'Min (s)':<12} {'Max (s)':<12}")
        print("-" * 60)
        
        for name, stats in self.results.items():
            print(f"{name:<20} {stats['mean']:<12.6f} {stats['min']:<12.6f} {stats['max']:<12.6f}")

# Example usage
benchmark = PerformanceBenchmark()

# Test matrix multiplication
matrix_a = CacheOptimizedMatrix(500)
matrix_b = CacheOptimizedMatrix(500)

# Fill with random data
import random
for i in range(500):
    for j in range(500):
        matrix_a.data[i][j] = random.random()
        matrix_b.data[i][j] = random.random()

benchmark.benchmark(matrix_a.multiply_naive, "Naive Multiply", matrix_b)
benchmark.benchmark(matrix_a.multiply_blocked, "Blocked Multiply", matrix_b)
benchmark.print_results()
```

---

## 🔮 Emerging Technologies

### Quantum Computing Fundamentals

```python
import numpy as np
import cmath

class QuantumSimulator:
    def __init__(self, num_qubits):
        self.num_qubits = num_qubits
        self.num_states = 2 ** num_qubits
        # Initialize to |00...0⟩ state
        self.state = np.zeros(self.num_states, dtype=complex)
        self.state[0] = 1.0
    
    def apply_gate(self, gate_matrix, target_qubit):
        """Apply single-qubit gate"""
        new_state = np.zeros_like(self.state)
        
        for i in range(self.num_states):
            # Extract bit at target position
            bit = (i >> target_qubit) & 1
            
            # Calculate new amplitudes
            for new_bit in [0, 1]:
                new_i = i ^ (bit << target_qubit) ^ (new_bit << target_qubit)
                new_state[new_i] += gate_matrix[new_bit][bit] * self.state[i]
        
        self.state = new_state
    
    def apply_cnot(self, control, target):
        """Apply CNOT gate"""
        new_state = np.zeros_like(self.state)
        
        for i in range(self.num_states):
            control_bit = (i >> control) & 1
            target_bit = (i >> target) & 1
            
            if control_bit == 1:
                # Flip target bit
                new_i = i ^ (1 << target)
            else:
                new_i = i
            
            new_state[new_i] = self.state[i]
        
        self.state = new_state
    
    def measure(self, qubit):
        """Measure a specific qubit"""
        probabilities = np.abs(self.state) ** 2
        
        # Calculate probability of measuring 0 or 1
        prob_0 = sum(probabilities[i] for i in range(self.num_states) if not (i >> qubit) & 1)
        prob_1 = 1 - prob_0
        
        # Random measurement
        import random
        result = 1 if random.random() > prob_0 else 0
        
        # Collapse state
        new_state = np.zeros_like(self.state)
        for i in range(self.num_states):
            if ((i >> qubit) & 1) == result:
                new_state[i] = self.state[i] / np.sqrt(prob_0 if result == 0 else prob_1)
        
        self.state = new_state
        return result

# Quantum Gates
def hadamard_gate():
    return np.array([[1, 1], [1, -1]]) / np.sqrt(2)

def pauli_x_gate():
    return np.array([[0, 1], [1, 0]])

def pauli_z_gate():
    return np.array([[1, 0], [0, -1]])

# Quantum Algorithm Example: Grover's Search
def grovers_algorithm(num_qubits, target_item):
    qsim = QuantumSimulator(num_qubits)
    
    # Initialize superposition
    for i in range(num_qubits):
        qsim.apply_gate(hadamard_gate(), i)
    
    # Number of iterations
    num_iterations = int(np.pi / 4 * np.sqrt(2 ** num_qubits))
    
    for _ in range(num_iterations):
        # Oracle: flip phase of target state
        oracle_matrix = np.eye(qsim.num_states, dtype=complex)
        oracle_matrix[target_item, target_item] = -1
        
        # Apply oracle (simplified)
        qsim.state = oracle_matrix @ qsim.state
        
        # Diffusion operator
        # Apply Hadamard to all qubits
        for i in range(num_qubits):
            qsim.apply_gate(hadamard_gate(), i)
        
        # Flip phase of |0⟩ state
        qsim.state[0] *= -1
        
        # Apply Hadamard again
        for i in range(num_qubits):
            qsim.apply_gate(hadamard_gate(), i)
    
    return qsim

# Example: Search in 4-item database
grover_sim = grovers_algorithm(2, target_item=3)
print("Grover's Algorithm - Final state amplitudes:")
for i, amplitude in enumerate(grover_sim.state):
    probability = abs(amplitude) ** 2
    print(f"State |{i:02b}⟩: {probability:.4f}")
```

### Machine Learning Integration

```python
import numpy as np
from typing import List, Tuple

class NeuralNetwork:
    def __init__(self, layer_sizes: List[int]):
        self.layers = []
        for i in range(len(layer_sizes) - 1):
            layer = {
                'weights': np.random.randn(layer_sizes[i], layer_sizes[i+1]) * 0.1,
                'biases': np.zeros(layer_sizes[i+1])
            }
            self.layers.append(layer)
    
    def forward(self, x: np.ndarray) -> np.ndarray:
        activations = [x]
        
        for layer in self.layers:
            z = np.dot(activations[-1], layer['weights']) + layer['biases']
            a = self.sigmoid(z)
            activations.append(a)
        
        return activations
    
    def backward(self, x: np.ndarray, y: np.ndarray, learning_rate: float):
        m = x.shape[0]
        activations = self.forward(x)
        
        # Calculate error for output layer
        delta = activations[-1] - y
        
        # Backpropagate
        for i in reversed(range(len(self.layers))):
            # Calculate gradients
            dW = np.dot(activations[i].T, delta) / m
            db = np.sum(delta, axis=0) / m
            
            # Update parameters
            self.layers[i]['weights'] -= learning_rate * dW
            self.layers[i]['biases'] -= learning_rate * db
            
            # Calculate delta for previous layer
            if i > 0:
                delta = np.dot(delta, self.layers[i]['weights'].T) * \
                       self.sigmoid_derivative(activations[i])
    
    def sigmoid(self, z):
        return 1 / (1 + np.exp(-np.clip(z, -500, 500)))
    
    def sigmoid_derivative(self, a):
        return a * (1 - a)

# Edge Computing Example
class EdgeComputingNode:
    def __init__(self, node_id, processing_power, memory_capacity):
        self.node_id = node_id
        self.processing_power = processing_power
        self.memory_capacity = memory_capacity
        self.current_load = 0
        self.tasks = []
    
    def can_handle_task(self, task):
        required_resources = task['cpu'] + task['memory']
        available_resources = self.processing_power + self.memory_capacity - self.current_load
        return available_resources >= required_resources
    
    def add_task(self, task):
        if self.can_handle_task(task):
            self.tasks.append(task)
            self.current_load += task['cpu'] + task['memory']
            return True
        return False
    
    def process_tasks(self):
        completed = []
        for task in self.tasks[:]:
            # Simulate processing
            task['progress'] += self.processing_power * 0.1
            if task['progress'] >= task['total_work']:
                completed.append(task)
                self.tasks.remove(task)
                self.current_load -= task['cpu'] + task['memory']
        return completed

class EdgeOrchestrator:
    def __init__(self):
        self.nodes = []
        self.pending_tasks = []
    
    def add_node(self, node):
        self.nodes.append(node)
    
    def schedule_task(self, task):
        # Find best node using load balancing
        best_node = None
        min_load = float('inf')
        
        for node in self.nodes:
            if node.can_handle_task(task) and node.current_load < min_load:
                best_node = node
                min_load = node.current_load
        
        if best_node:
            best_node.add_task(task)
            print(f"Task {task['id']} assigned to node {best_node.node_id}")
        else:
            self.pending_tasks.append(task)
            print(f"Task {task['id']} queued - no available nodes")

# Example usage
edge_system = EdgeOrchestrator()

# Add edge nodes
edge_system.add_node(EdgeComputingNode("edge1", 100, 512))
edge_system.add_node(EdgeComputingNode("edge2", 80, 256))
edge_system.add_node(EdgeComputingNode("edge3", 120, 1024))

# Create tasks
tasks = [
    {'id': 1, 'cpu': 30, 'memory': 100, 'total_work': 1000, 'progress': 0},
    {'id': 2, 'cpu': 50, 'memory': 200, 'total_work': 1500, 'progress': 0},
    {'id': 3, 'cpu': 70, 'memory': 300, 'total_work': 2000, 'progress': 0}
]

# Schedule tasks
for task in tasks:
    edge_system.schedule_task(task)
```

---

## 🎯 Advanced System Design Patterns

### Microservices Architecture

```python
import asyncio
import aiohttp
import json
from datetime import datetime

class ServiceRegistry:
    def __init__(self):
        self.services = {}
    
    def register(self, service_name, host, port, health_check_url):
        self.services[service_name] = {
            'host': host,
            'port': port,
            'health_check_url': health_check_url,
            'last_seen': datetime.now(),
            'healthy': True
        }
    
    def discover(self, service_name):
        if service_name in self.services and self.services[service_name]['healthy']:
            service = self.services[service_name]
            return f"http://{service['host']}:{service['port']}"
        return None
    
    async def health_check(self):
        for service_name, service in self.services.items():
            try:
                async with aiohttp.ClientSession() as session:
                    async with session.get(service['health_check_url'], timeout=5) as response:
                        service['healthy'] = response.status == 200
                        service['last_seen'] = datetime.now()
            except:
                service['healthy'] = False

class APIGateway:
    def __init__(self, service_registry):
        self.service_registry = service_registry
        self.rate_limits = {}
    
    async def route_request(self, service_name, path, method='GET', data=None):
        # Rate limiting
        client_id = "default"  # In real implementation, extract from request
        if not self.check_rate_limit(client_id):
            return {"error": "Rate limit exceeded"}, 429
        
        # Service discovery
        service_url = self.service_registry.discover(service_name)
        if not service_url:
            return {"error": "Service unavailable"}, 503
        
        # Forward request
        try:
            async with aiohttp.ClientSession() as session:
                url = f"{service_url}{path}"
                async with session.request(method, url, json=data) as response:
                    result = await response.json()
                    return result, response.status
        except Exception as e:
            return {"error": str(e)}, 500
    
    def check_rate_limit(self, client_id, max_requests=100, window=60):
        now = datetime.now()
        
        if client_id not in self.rate_limits:
            self.rate_limits[client_id] = []
        
        # Remove old requests
        self.rate_limits[client_id] = [
            req_time for req_time in self.rate_limits[client_id]
            if (now - req_time).seconds < window
        ]
        
        if len(self.rate_limits[client_id]) >= max_requests:
            return False
        
        self.rate_limits[client_id].append(now)
        return True

# Circuit Breaker Pattern
class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=60, expected_exception=Exception):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.expected_exception = expected_exception
        
        self.failure_count = 0
        self.last_failure_time = None
        self.state = 'CLOSED'  # CLOSED, OPEN, HALF_OPEN
    
    def call(self, func, *args, **kwargs):
        if self.state == 'OPEN':
            if self._should_attempt_reset():
                self.state = 'HALF_OPEN'
            else:
                raise Exception("Circuit breaker is OPEN")
        
        try:
            result = func(*args, **kwargs)
            self._on_success()
            return result
        except self.expected_exception as e:
            self._on_failure()
            raise e
    
    def _should_attempt_reset(self):
        return (datetime.now() - self.last_failure_time).seconds >= self.recovery_timeout
    
    def _on_success(self):
        self.failure_count = 0
        self.state = 'CLOSED'
    
    def _on_failure(self):
        self.failure_count += 1
        self.last_failure_time = datetime.now()
        
        if self.failure_count >= self.failure_threshold:
            self.state = 'OPEN'

# Event Sourcing Example
class EventStore:
    def __init__(self):
        self.events = []
        self.snapshots = {}
    
    def append_event(self, aggregate_id, event_type, event_data, version):
        event = {
            'aggregate_id': aggregate_id,
            'event_type': event_type,
            'event_data': event_data,
            'version': version,
            'timestamp': datetime.now()
        }
        self.events.append(event)
    
    def get_events(self, aggregate_id, from_version=0):
        return [e for e in self.events 
                if e['aggregate_id'] == aggregate_id and e['version'] > from_version]
    
    def save_snapshot(self, aggregate_id, version, state):
        self.snapshots[aggregate_id] = {
            'version': version,
            'state': state,
            'timestamp': datetime.now()
        }
    
    def get_snapshot(self, aggregate_id):
        return self.snapshots.get(aggregate_id)

class BankAccount:
    def __init__(self, account_id):
        self.account_id = account_id
        self.balance = 0
        self.version = 0
        self.event_store = EventStore()
    
    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        
        self.balance += amount
        self.version += 1
        
        self.event_store.append_event(
            self.account_id,
            'MoneyDeposited',
            {'amount': amount, 'new_balance': self.balance},
            self.version
        )
    
    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("Withdrawal amount must be positive")
        if self.balance < amount:
            raise ValueError("Insufficient funds")
        
        self.balance -= amount
        self.version += 1
        
        self.event_store.append_event(
            self.account_id,
            'MoneyWithdrawn',
            {'amount': amount, 'new_balance': self.balance},
            self.version
        )
    
    @classmethod
    def from_events(cls, account_id, events):
        account = cls(account_id)
        
        for event in events:
            if event['event_type'] == 'MoneyDeposited':
                account.balance = event['event_data']['new_balance']
            elif event['event_type'] == 'MoneyWithdrawn':
                account.balance = event['event_data']['new_balance']
            account.version = event['version']
        
        return account

# Example usage
account = BankAccount("ACC-001")
account.deposit(1000)
account.withdraw(300)
account.deposit(500)

print(f"Final balance: ${account.balance}")
print(f"Version: {account.version}")

# Reconstruct from events
events = account.event_store.get_events("ACC-001")
reconstructed_account = BankAccount.from_events("ACC-001", events)
print(f"Reconstructed balance: ${reconstructed_account.balance}")
```

---

## 🎓 Summary & Future Directions

### Key Takeaways

| Area | Advanced Concepts | Practical Applications |
|------|------------------|------------------------|
| **Architecture** | • Superscalar CPUs<br>• NUMA systems<br>• Cache coherence | • High-performance computing<br>• Real-time systems |
| **Parallel Computing** | • Lock-free algorithms<br>• GPU programming<br>• Distributed computing | • Machine learning<br>• Scientific simulation |
| **Security** | • Cryptographic protocols<br>• Side-channel attacks<br>• Formal verification | • Blockchain<br>• Secure communications |
| **Performance** | • Cache optimization<br>• Memory hierarchies<br>• Vectorization | • Database engines<br>• Game engines |
| **Emerging Tech** | • Quantum computing<br>• Edge computing<br>• Neuromorphic chips | • AI acceleration<br>• IoT systems |

### 🚀 Career Paths

```
Advanced Computer Fundamentals
             │
    ┌────────┼────────┐
    │        │        │
System     Research  Product
Architect  Scientist Development
    │        │        │
    ▼        ▼        ▼
• Cloud     • ML/AI   • Software
  Infrastructure    • Quantum    Engineering
• Hardware  Computing • Technical
  Design   • Security  Leadership
• Performance        • DevOps
  Engineering
```

### 📚 Recommended Next Steps

1. **Hands-on Projects**
   - Implement a distributed system
   - Optimize performance-critical code
   - Build security tools
   - Experiment with quantum simulators

2. **Advanced Courses**
   - Computer Systems Engineering
   - Distributed Systems
   - Cryptography
   - Machine Learning Systems

3. **Industry Certifications**
   - Cloud Architecture (AWS, Azure, GCP)
   - Security (CISSP, CEH)
   - Performance Engineering
   - DevOps and SRE

4. **Research Areas**
   - Quantum-classical hybrid systems
   - Neuromorphic computing
   - Advanced cryptographic protocols
   - Post-quantum cryptography

The field of computer fundamentals continues to evolve rapidly with new architectures, paradigms, and applications emerging regularly. Staying current requires continuous learning and hands-on experimentation with cutting-edge technologies.
