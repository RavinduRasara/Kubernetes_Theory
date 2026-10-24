
&nbsp;

# 1\. Cluster Architecture

&nbsp;

![ClusterArchitecture.png](/2.Cluster-Architecture/Images/ClusterArchitecture.png)

&nbsp;

### The architectural concepts behind Kubernetes. -

A Kubernetes cluster consists of a **<span style="color: rgb(224, 62, 45);">control plane</span>** plus a set of **<span style="color: rgb(224, 62, 45);">worker machines, called nodes</span>**, that run containerized applications. **Every cluster needs <span style="color: rgb(224, 62, 45);">at least one worker node in order to run Pods.</span>**

The worker node(s) host the Pods that are the components of the application workload. The control plane manages the worker nodes and the Pods in the cluster. <span style="color: rgb(224, 62, 45);">**In production environments, the control plane usually runs across multiple <span style="color: rgb(224, 62, 45);">computers</span>**</span> **<span style="color: rgb(224, 62, 45);">and a cluster usually runs multiple nodes</span>**, providing fault-tolerance and high availability.

&nbsp;

> ### <span style="color: rgb(45, 194, 107);">**"In production environments, the control plane usually runs across multiple computers and the worker nodes also run on multiple computers, all together forming a cluster — providing fault-tolerance and high availability."**</span>

&nbsp;

> ## 📝 Write this in your notes:
> 
> ```
> 📌 PRODUCTION ENVIRONMENT:
> 
> - Control Plane  → runs on multiple computers
> - Worker Nodes   → run on multiple computers
> - Both together  → called a CLUSTER
> 
> This gives us:
>   ✅ Fault Tolerance   = if one computer dies, others keep working
>   ✅ High Availability = system is always up and running
> ```

&nbsp;

## 💡 Can there be multiple Control Planes in ONE cluster?

### ✅ YES! Absolutely!

In production, you usually have **3 control planes** in one cluster.

```
CLUSTER
│
├── Control Plane 1  ✅ (Active)
├── Control Plane 2  ✅ (Standby)
└── Control Plane 3  ✅ (Standby)
│
├── Worker Node 1
├── Worker Node 2
└── Worker Node 3
```

### Why 3 control planes?

- If Control Plane 1 **crashes** → Control Plane 2 takes over immediately
- Cluster **never goes down!**
- This is called **High Availability (HA) Setup**

&nbsp;

## 💡 Can there be multiple Clusters in ONE production system?

### ✅ YES! Very common in real companies!

Big companies run **many clusters** together!

```
PRODUCTION SYSTEM (e.g., Amazon)
│
├── Cluster 1 → Development environment
├── Cluster 2 → Testing environment  
├── Cluster 3 → Production environment
├── Cluster 4 → Europe region users
└── Cluster 5 → Asia region users
```

> ### Why multiple clusters?
> 
> | Reason | Simple Meaning |
> | --- | --- |
> | **Different environments** | Dev, Test, Production kept separate |
> | **Different regions** | US cluster, Europe cluster, Asia cluster |
> | **Security** | Sensitive apps isolated in own cluster |
> | **Team separation** | Team A has own cluster, Team B has own cluster |
> | **Risk management** | One cluster crashes → other clusters safe |

&nbsp;

# 2\. 🏗️ Architecture component

&nbsp;

```
CLUSTER
│
├── 1. CONTROL PLANE (Brain 🧠)
│       ├── kube-api-server
│       ├── etcd
│       ├── kube-scheduler
│       ├── kube-controller-manager
│       └── cloud-controller-manager
│
└── 2. WORKER NODES (Muscle 💪)
        ├── kubelet
        ├── kube-proxy
        └── CRI (Container Runtime Interface)
            └── Pods run here
```

&nbsp;

# 2.1 🌟 kube-api-server

**kube-api-server** is the **central communication hub** of Kubernetes. The **kube-api-server** is the **front door** of the entire Kubernetes cluster.

> Every single thing that happens in Kubernetes **goes through the API server first!**

- You want to create a pod? → talk to API server
- Scheduler wants to assign a pod? → talk to API server
- Kubelet wants to report status? → talk to API server
- etcd wants to store data? → talk to API server

**Everything talks to the API server. Nothing talks directly to each other!**

&nbsp;

### 💡 Key Responsibilities

| Job | Simple Meaning |
| --- | --- |
| **Authentication** | Are you allowed to access this cluster? |
| **Authorization** | Are you allowed to do THIS specific thing? |
| **The only entry point** | Nobody bypasses the API server |
| **Talks to etcd** | Only API server reads/writes to etcd directly |
| **Exposes REST API** | Everything communicates via HTTP requests |

### 🎯 Questions

**Q: What is kube-api-server?**

> kube-api-server is the **central communication hub** of Kubernetes. It is the **only entry point** to the cluster. Every component — scheduler, kubelet, controller manager — communicates only through the API server. It also handles **authentication and authorization.**

**Q: Can kubelet talk directly to etcd?**

> No! Nothing talks to etcd directly. **Only the API server** communicates with etcd. All other components go through the API server.

**Q: What happens when you run "kubectl" command?**

> When you run any kubectl command, it sends an **HTTP request to the kube-api-server.** The API server then authenticates, authorizes, and processes the request.

**Short Way -**

```
📌 KUBE-API-SERVER

🔑 Role: Front door / Central hub of K8s cluster

✅ Key Points:
  → ONLY entry point to the cluster
  → ALL components talk through API server
  → Handles Authentication (who are you?)
  → Handles Authorization (what can you do?)
  → ONLY component that talks to etcd directly
  → Exposes Kubernetes REST API

🏠 Real life = Hospital Reception Desk
   Nobody goes inside without reception approval!

💬 kubectl command
   → sends request to API server
   → API server processes it
   → forwards to right component
```

&nbsp;

&nbsp;

# 2.2 🌟 etcd

### 📖 Simple Explanation

**etcd** is the **database** of Kubernetes. It stores **everything** about your cluster.

> **<span style="color: rgb(45, 194, 107);">etcd is a distributed key-value store that acts as the database of Kubernetes. it stores the entire cluster state — all configurations, pod information, node status, and everything else</span>**

> Think of it as Kubernetes **memory / brain storage!**

Whatever happens in the cluster — etcd **remembers it!**

- How many pods are running? → stored in etcd
- Which node is healthy? → stored in etcd
- What is the desired state? → stored in etcd
- All configurations? → stored in etcd

### 💡 Real Life Example

Think of a **school register book:**

```
Teacher (API server) writes everything in register (etcd):
  → Student A is present ✅
  → Student B is absent ❌
  → Class has 30 students
  → Exam scheduled on Monday

If register is lost → teacher knows NOTHING!
Same way → if etcd is lost → K8s knows NOTHING!
```

&nbsp;

### 💡 Very Important Concept — Key/Value Store

etcd is not like a normal database (like MySQL). It stores data as **Key → Value** pairs. Very simple!

```
Key                          Value
─────────────────────────────────────────
"pod/myapp/status"      →   "running"
"node/node1/status"     →   "healthy"
"cluster/pod-count"     →   "10"
"deployment/myapp"      →   "3 replicas"
```

> Just like a **dictionary** — word (key) and its meaning (value)!

### 💡 Key Responsibilities

| Job | Simple Meaning |
| --- | --- |
| **Stores cluster state** | Everything about cluster is saved here |
| **Source of truth** | K8s always checks etcd for real information |
| **Only API server talks to it** | No other component accesses etcd directly |
| **Distributed storage** | Runs on multiple computers for safety |

### 💡 Very Important — Desired State vs Actual State

This is a **key Kubernetes concept** — etcd stores both:

```
DESIRED STATE (what you WANT)
→ "I want 3 pods running"
→ stored in etcd

ACTUAL STATE (what is HAPPENING)
→ "Currently 2 pods are running"
→ also stored in etcd

K8s always tries to match actual state = desired state!
→ sees 2 pods but wants 3 → creates 1 more pod!
```

### 💡etcd Backup — Very Important in Production!

```
etcd data lost = entire cluster information lost!

So in production:
→ etcd is backed up regularly
→ etcd runs on multiple computers (HA setup)
→ If etcd crashes → restore from backup
```

&nbsp;

### 🎯 Questions

**Q: What is etcd in Kubernetes?**

> etcd is a **distributed key-value store** that acts as the **database of Kubernetes.** It stores the entire cluster state — all configurations, pod information, node status, and everything else. It is the **single source of truth** for the cluster.

**Q: What type of data does etcd store?**

> etcd stores data as **key-value pairs.** It stores the **desired state** (what you want) and **actual state** (what is running) of the entire cluster.

**Q: Who can directly access etcd?**

> Only the **kube-api-server** can directly read and write to etcd. No other component accesses etcd directly.

**Q: Why is etcd backup important?**

> Because etcd contains the **entire cluster state.** If etcd data is lost, Kubernetes loses all information about the cluster. That is why in production, etcd is **regularly backed up** and runs on **multiple nodes** for high availability.

**Q: What happens if etcd goes down?**

> If etcd goes down, the **API server cannot read or write** cluster state. The cluster becomes unmanageable. Existing pods may keep running but **no new changes** can be made to the cluster.

&nbsp;

```
📌 ETCD

🔑 Role: Database / Memory of Kubernetes cluster

✅ Key Points:
  → Stores EVERYTHING about the cluster
  → Key-Value store (not like MySQL)
  → ONLY API server talks to etcd directly
  → Single source of truth for K8s
  → Stores DESIRED state + ACTUAL state

📦 What it stores:
  → Pod information
  → Node status
  → Configurations
  → Cluster state

⚠️ Very Important:
  → etcd backup is CRITICAL in production
  → etcd runs on multiple nodes (HA)
  → etcd lost = cluster info lost!

🏠 Real life = School register book
   Everything is written there
   Lose it = lose all information!

🔑 Key-Value example:
  "pod/myapp/status" → "running"
  "node/node1"       → "healthy"
```

&nbsp;

## 💡 API Server + etcd Together

```
You run: kubectl get pods
         ↓
   kube-api-server  (receives request)
         ↓
      etcd          (hey etcd, what pods are running?)
         ↓
   kube-api-server  (gets answer from etcd)
         ↓
   shows you the result ✅
```

&nbsp;

&nbsp;

# 2.3 🌟 kube-scheduler

kube-scheduler is the component that decides which worker node a new pod should be placed on

> It does NOT create the pod. It just **decides the best node** for the pod to run on!

Simple one line:

> **"Scheduler = decides WHERE to place the pod"**

### 💡 Real Life Example

Think of a **hotel receptionist** assigning rooms:

```
Guest arrives at hotel (new pod needs to run)
         ↓
Receptionist checks all rooms (scheduler checks all nodes)
  → Room 1 = already FULL ❌
  → Room 2 = has AC problem ❌
  → Room 3 = clean, available, perfect ✅
         ↓
Receptionist assigns Room 3 to guest!
(Scheduler assigns Node 3 to pod!)
```

> Receptionist doesn't BUILD the room. Just **decides the best room!** Same way — scheduler doesn't CREATE the pod. Just **decides the best node!**

### 💡 How does Scheduler decide which node?

Scheduler checks **2 things:**

#### Step 1 — Filtering ❌

> Remove nodes that are **NOT suitable**

```
Checks:
  → Does node have enough CPU?
  → Does node have enough Memory?
  → Is node healthy?
  → Any special rules? (taints/tolerations)

Nodes that FAIL these checks → removed from list
```

#### Step 2 — Scoring ✅It stores the \*\*desired state\*\* (what you want) and \*\*actual state\*\*

> From remaining nodes, **pick the BEST one**

```
Remaining nodes get a SCORE:
  → Node 1 = 70 points
  → Node 2 = 90 points  ← WINNER! 🏆
  → Node 3 = 60 points

Pod goes to highest score node!
```

### 💡 Simple Flow

```
New Pod needs to be placed
         ↓
Scheduler gets info from API Server
         ↓
Step 1: FILTERING
→ removes unsuitable nodes
         ↓
Step 2: SCORING
→ picks best node from remaining
         ↓
Tells API Server the decision
         ↓
API Server saves decision in etcd
         ↓
Kubelet on that node creates the pod ✅
```

### 💡 Important Points

| Point | Simple Meaning |
| --- | --- |
| **Scheduler only decides** | Does NOT create the pod itself |
| **Talks via API server** | Never directly talks to nodes |
| **Checks resources** | CPU, memory, disk before deciding |
| **Can be customized** | You can write your own scheduler! |

### 💡 Interview Questions

**Q: What is kube-scheduler?**

> kube-scheduler is the component that **decides which worker node** a new pod should be placed on. It does not create the pod — it only makes the **placement decision** based on available resources and node conditions.

**Q: How does kube-scheduler select a node?**

> It works in 2 steps:
> 
> 1.  **Filtering** — removes nodes that don't have enough CPU, memory, or are unhealthy
> 2.  **Scoring** — gives remaining nodes a score and picks the **highest scoring node**

**Q: Does scheduler directly talk to worker nodes?**

> No! Scheduler communicates only through the **kube-api-server.** It gets node information from API server and sends its decision back to API server.

**Q: What happens after scheduler picks a node?**

> Scheduler tells the **API server** its decision. API server saves it in **etcd.** Then the **kubelet** on that worker node picks it up and actually creates the pod.

### 💡 Notes

```
📌 KUBE-SCHEDULER

🔑 Role: Decides WHICH node a pod should run on

✅ Key Points:
  → Does NOT create pods
  → Only makes PLACEMENT decision
  → Talks only through API server
  → Never directly contacts worker nodes

📋 How it picks a node (2 steps):
  Step 1: FILTERING
    → Removes nodes with not enough CPU
    → Removes nodes with not enough Memory
    → Removes unhealthy nodes

  Step 2: SCORING
    → Gives score to remaining nodes
    → Picks HIGHEST score node
    → Pod goes to that node!

🔗 Flow:
  New pod needed
    → Scheduler checks nodes (via API server)
    → Filters bad nodes
    → Scores good nodes
    → Tells API server the winner
    → API server saves in etcd
    → Kubelet creates the pod ✅

🏠 Real life = Hotel receptionist
   Assigns best available room to guest
   Does not BUILD the room!
```

### 💡 How API Server + etcd + Scheduler work together

```
You run: kubectl create pod
         ↓
   kube-api-server  (receives request, saves in etcd)
         ↓
   kube-scheduler   (hey! new pod needs a home!)
         ↓
   checks all nodes → filters → scores
         ↓
   tells API server → "put it on Node 2!"
         ↓
   API server saves decision in etcd
         ↓
   kubelet on Node 2 → creates the pod ✅
```

&nbsp;

```
You run: kubectl create pod (I want 3 pods!)
              ↓
   kube-api-server receives request
              ↓
   1st SAVE to etcd ← DESIRED STATE
   "User WANTS 3 pods running"
   (but pods not created yet!)
              ↓
   kube-scheduler checks nodes
   filters → scores → picks Node 2
              ↓
   tells API server the decision
              ↓
   kubelet creates the pod on Node 2
              ↓
   2nd SAVE to etcd ← ACTUAL STATE
   "3 pods are NOW actually running on Node 2"
```

&nbsp;

## 💡Questions

**Q: What is desired state and actual state in Kubernetes?**

> Desired state is **what the user wants** (e.g., 3 pods running) and actual state is **what is currently happening** (e.g., 2 pods running). Both are stored in etcd. Kubernetes constantly compares them and automatically fixes any difference.

**Q: What is a Reconciliation Loop?**

> It is the process where Kubernetes **constantly compares** desired state vs actual state and **automatically takes action** to make them match. For example if desired state is 3 pods but only 2 are running, K8s creates 1 more pod automatically.

## 💡 Add this to your Notes

```
📌 DESIRED STATE vs ACTUAL STATE

1st Save to etcd = DESIRED STATE
  → What user WANTS
  → Saved when request first comes in
  → Example: "I want 3 pods"

2nd Save to etcd = ACTUAL STATE
  → What is ACTUALLY running
  → Saved after pod is created
  → Example: "3 pods are now running"

🔄 RECONCILIATION LOOP:
  → K8s constantly compares both
  → If not matching → K8s fixes it automatically!
  → This is the CORE concept of Kubernetes!

🏠 Real life = Thermostat
  Set 22°C (desired) vs current 26°C (actual)
  → AC turns on automatically to match!
```

&nbsp;

&nbsp;

# 2.4 kube-controller-manager

**kube-controller-manager** is the component that **constantly watches the cluster** and makes sure the **desired state always matches the actual state!**

> Remember the reconciliation loop we just discussed? 🔄 **kube-controller-manager is the one doing that job!**

Simple one line:

> **"Controller Manager = the person who constantly watches and fixes things!"**

Same way kube-controller-manager:

```
  → Constantly watches the cluster
  → Pod crashed? → creates new one
  → Less replicas than desired? → adds more
  → Node went down? → reschedules pods
Never stops watching!
Never stops fixing!
```

&nbsp;

### 🔑 Important Concept — What is a Controller?

> Controller manager is actually a **collection of many controllers** running together!

Each controller is responsible for **one specific job:**

| Controller | Job |
| --- | --- |
| **Node Controller** | Watches nodes — if node dies, takes action |
| **Replication Controller** | Makes sure correct number of pods always running |
| **Deployment Controller** | Manages deployments and updates |
| **Service Controller** | Manages services in the cluster |
| **Job Controller** | Watches one-time jobs |

&nbsp;

> kube-controller-manager  
>         │  
>         ├── Node Controller  
>         ├── Replication Controller  
>         ├── Deployment Controller  
>         ├── Service Controller  
>         └── Job Controller

### 💡 How does it work?

```
Controller Manager constantly does this loop:

STEP 1: Check desired state from etcd (through API server)
        "User wants 3 pods"
              ↓
STEP 2: Check actual state from etcd ()
        "Only 2 pods running"
              ↓
STEP 3: Compare both
        "2 ≠ 3 → NOT MATCHING!"
              ↓
STEP 4: Take action
        "Create 1 more pod!"
              ↓
STEP 5: Go back to STEP 1 and repeat forever! 🔄
```

> This is the **Reconciliation Loop** in action!

### 💡 Important Points

| Point | Simple Meaning |
| --- | --- |
| **Never stops working** | Constantly running in background |
| **Talks via API server** | Never directly touches etcd |
| **Many controllers inside** | Each handles specific job |
| **Works with scheduler** | Controller decides WHAT to do, scheduler decides WHERE |

&nbsp;

### 💡 Controller Manager vs Scheduler — Simple Difference

This is a **common interview confusion!**

```
kube-controller-manager:
  → Notices "1 pod is missing!"
  → Decides "we need to create 1 more pod"
  → Tells API server

kube-scheduler:
  → Gets the new pod request
  → Decides "put it on Node 2!"
  → Tells API server

Controller Manager = WHAT needs to be done
Scheduler          = WHERE it needs to be done
```

&nbsp;

### 💡 Questions

**Q: What is kube-controller-manager?**

> kube-controller-manager is a component that **runs a collection of controllers.** It constantly watches the cluster state via the API server and ensures the **actual state always matches the desired state.** It is the component that implements the **reconciliation loop.**

**Q: What is a controller in Kubernetes?**

> A controller is a **control loop** that watches the cluster state and takes action to move the actual state towards the desired state. For example, the replication controller ensures the correct number of pod replicas are always running.

**Q: What is the difference between controller manager and scheduler?**

> Controller manager decides **WHAT needs to be done** — for example, a pod is missing so a new one needs to be created. The scheduler decides **WHERE to run it** — which specific node the new pod should go on.

**Q: If a node goes down, which component notices it first?**

> The **Node Controller** inside kube-controller-manager notices it. It then takes action — marking the node as unhealthy and rescheduling the pods that were on that node.

### 💡 Notes

```
📌 KUBE-CONTROLLER-MANAGER

🔑 Role: Watches cluster and fixes differences
         between desired state and actual state

✅ Key Points:
  → Runs MANY controllers inside it
  → Constantly watches via API server
  → Never talks directly to etcd
  → Implements the RECONCILIATION LOOP
  → Never stops running!

🎮 Controllers inside it:
  → Node Controller        = watches nodes
  → Replication Controller = watches pod count
  → Deployment Controller  = watches deployments
  → Service Controller     = watches services
  → Job Controller         = watches jobs

🔄 How it works (Reconciliation Loop):
  1. Check desired state (from etcd through API server API server)
  2. Check actual state (from etcd through API server)
  3. Compare both
  4. If different → take action to fix!
  5. Repeat forever!

⚖️ Controller Manager vs Scheduler:
  Controller Manager = WHAT needs to be done
  Scheduler          = WHERE it needs to be done

🏠 Real life = Security guard
   Constantly watches and fixes problems!
```

&nbsp;

### 💡 How all 4 components work together

```
You run: kubectl create 3 pods
              ↓
   kube-api-server  → saves DESIRED STATE in etcd
   "User wants 3 pods"
              ↓
   kube-controller-manager  → notices new request
   "Need to create 3 pods!" (WHAT)
              ↓
   kube-scheduler  → picks best nodes
   "Pod 1 → Node1, Pod 2 → Node2" (WHERE)
              ↓
   kubelet on each node → creates pods
              ↓
   kube-api-server  → saves ACTUAL STATE in etcd
   "3 pods are now running"
              ↓
   kube-controller-manager  → checks again 🔄
   "Desired 3 = Actual 3 ✅ All good!"
```

&nbsp;

&nbsp;

# 2.5 Cloud-controller-manager

### Simple Explanation

**cloud-controller-manager** is the component that **connects your Kubernetes cluster to your cloud provider** (like AWS, Google Cloud, Azure).

> Simple one line: **"Cloud Controller Manager = bridge between K8s and Cloud Provider"**

Look at your diagram — you can see the **dotted line** going from cloud-controller-manager all the way to **CLOUD PROVIDER API** on the right side! That dotted line = the connection/bridge!

💡 Why do we need it?

When you run Kubernetes on a cloud (AWS, GCP, Azure) — you need cloud specific things:

```
☁️ Examples of cloud specific things:

→ Need a Load Balancer?
  → AWS has its own load balancer (ELB)
  → GCP has its own load balancer
  → Azure has its own load balancer

→ Need Storage?
  → AWS has EBS (Elastic Block Store)
  → GCP has Persistent Disk
  → Azure has Azure Disk

→ Need to add more nodes?
  → Need to talk to AWS/GCP/Azure to create new VMs
```

> Kubernetes itself doesn't know how to talk to AWS or GCP! **cloud-controller-manager handles all that communication!**

### 💡What does it control?

It has **3 main controllers** inside it:

| Controller | Job |
| --- | --- |
| **Node Controller** | Checks cloud if a node is deleted |
| **Route Controller** | Sets up network routes in cloud |
| **Service Controller** | Creates/manages cloud load balancers |

&nbsp;

### 💡 Very Important Point

```
On-premise K8s (your own servers):
→ cloud-controller-manager NOT needed!
→ You manage your own hardware

Cloud K8s (AWS/GCP/Azure):
→ cloud-controller-manager IS needed!
→ It talks to cloud provider API
```

This is why in your diagram the line to cloud provider API is **dotted** — it means it is **optional!** Only needed when running on cloud!💡 Cloud-controller-manager vs kube-controller-manager

Common interview confusion! Simple difference:

```
kube-controller-manager:
→ manages INTERNAL cluster things
→ pods, nodes, deployments
→ works everywhere (cloud or on-premise)

cloud-controller-manager:
→ manages EXTERNAL cloud things
→ load balancers, cloud storage, cloud VMs
→ only works with cloud providers
```

&nbsp;

### 💡 Interview Questions

**Q: What is cloud-controller-manager?**

> cloud-controller-manager is the component that acts as a **bridge between Kubernetes and the cloud provider** (AWS, GCP, Azure). It handles all cloud specific operations like managing load balancers, cloud storage, and cloud nodes through the **cloud provider API.**

**Q: Do you always need cloud-controller-manager?**

> No! It is only needed when running Kubernetes **on a cloud provider.** If you are running K8s on your own servers (on-premise), cloud-controller-manager is not needed.

**Q: What is the difference between kube-controller-manager and cloud-controller-manager?**

> kube-controller-manager manages **internal cluster resources** like pods and deployments and works everywhere. cloud-controller-manager manages **cloud specific resources** like AWS load balancers, cloud storage, and cloud VMs and only works when K8s is running on a cloud provider.

**Q: Which cloud providers does cloud-controller-manager support?**

> It supports all major cloud providers — **AWS, Google Cloud (GCP), and Microsoft Azure.** Each cloud provider has their own implementation of the cloud-controller-manager.

&nbsp;

### 💡 Notes

```
📌 CLOUD-CONTROLLER-MANAGER

🔑 Role: Bridge between K8s and Cloud Provider
         (AWS / GCP / Azure)

✅ Key Points:
  → Connects K8s to cloud provider API
  → OPTIONAL - only needed on cloud
  → Not needed for on-premise setup
  → Shown as DOTTED line in architecture diagram
    (dotted = optional connection)

☁️ 3 Controllers inside it:
  → Node Controller    = checks cloud for deleted nodes
  → Route Controller   = sets up cloud network routes
  → Service Controller = manages cloud load balancers

☁️ Cloud specific things it manages:
  → Load Balancers (AWS ELB, GCP LB, Azure LB)
  → Cloud Storage (AWS EBS, GCP Disk, Azure Disk)
  → Cloud VMs (adding/removing nodes)

⚖️ vs kube-controller-manager:
  kube-controller-manager  = manages INTERNAL things
                             (pods, nodes, deployments)
                             works EVERYWHERE

  cloud-controller-manager = manages EXTERNAL cloud things
                             (load balancers, cloud storage)
                             works only on CLOUD
```

&nbsp;

## 💡Complete Control Plane Picture

```
CONTROL PLANE
│
├── kube-api-server
│     → Front door of cluster
│     → Everything talks through it
│
├── etcd
│     → Database of cluster
│     → Stores desired + actual state
│
├── kube-scheduler
│     → Decides WHERE pods run
│     → Filters + scores nodes
│
├── kube-controller-manager
│     → Watches + fixes cluster (WHAT)
│     → Runs reconciliation loop
│
└── cloud-controller-manager
      → Talks to cloud provider
      → Manages cloud resources
      → Optional (only for cloud)
```

&nbsp;

### Control Plane Components — Quick Interview Summary

| Component | One Line Answer |
| --- | --- |
| **kube-api-server** | Front door — everything goes through it |
| **etcd** | Database — stores entire cluster state |
| **kube-scheduler** | Decides WHERE to place pods |
| **kube-controller-manager** | Watches and fixes cluster state |
| **cloud-controller-manager** | Bridge between K8s and cloud provider |

&nbsp;

&nbsp;

# 2.6 👷 kubelet

📖 Simple Explanation

**kubelet** is an **agent** that runs on **every worker node** and makes sure the containers are running properly on that node.

> Simple one line: **"kubelet = manager of each worker node"**

### 🔑 Key Responsibilities

| Job | Simple Meaning |
| --- | --- |
| **Registers the node** | Tells API server "hey I exist!" |
| **Creates pods** | When scheduler assigns pod → kubelet creates it |
| **Watches pod health** | Constantly checks if pods are running |
| **Reports to API server** | Sends node and pod status regularly |
| **Kills unhealthy pods** | Removes pods that are not healthy |

&nbsp;

💡 How kubelet works — Step by Step

```
STEP 1: kubelet starts on worker node
              ↓
STEP 2: registers itself with API server
        "Hey! Node 2 is ready!"
              ↓
STEP 3: API server assigns a pod to Node 2
        (after scheduler decision)
              ↓
STEP 4: kubelet receives the instruction
        "Create this pod on your node!"
              ↓
STEP 5: kubelet talks to CRI
        (Container Runtime Interface)
        "Hey CRI, create this container!"
              ↓
STEP 6: Container is created and running ✅
              ↓
STEP 7: kubelet constantly watches the pod
        "Is it still running? Is it healthy?"
              ↓
STEP 8: kubelet reports status to API server
        "Pod is running fine!" ✅
        or
        "Pod has crashed!" ❌
              ↓
STEP 9: If crashed → controller manager notices
        → creates new pod → back to STEP 3!
```

💡 Very Important Points

#### Point 1 — kubelet does NOT manage containers directly!

```
kubelet → talks to → CRI (Container Runtime)
                          ↓
                    CRI creates container
                    (Docker, containerd etc.)

kubelet tells WHAT to do
CRI actually DOES it!
```

&nbsp;

#### Point 2 — kubelet is like a heartbeat!

```
Every few seconds kubelet sends message to API server:
"Node is alive! ✅ Pods are running! ✅"

If API server stops getting this message:
→ Node Controller notices!
→ "Node is dead!" ❌
→ Pods get rescheduled to other nodes!
```

> This regular message is called **Node Heartbeat!**

### 💡 kubelet vs kube-controller-manager — Simple Difference

```
kube-controller-manager:
→ runs on CONTROL PLANE
→ watches cluster from TOP level
→ decides WHAT action to take

kubelet:
→ runs on WORKER NODE
→ watches its OWN node only
→ actually DOES the action on that node
```

### 💡 kubelet vs kube-controller-manager — Simple Difference

```
kube-controller-manager:
→ runs on CONTROL PLANE
→ watches cluster from TOP level
→ decides WHAT action to take

kubelet:
→ runs on WORKER NODE
→ watches its OWN node only
→ actually DOES the action on that node
```

&nbsp;

### 💡 Questions

**Q: What is kubelet?**

> kubelet is an **agent that runs on every worker node.** It receives instructions from the API server, creates and manages pods on its node through the CRI, and constantly reports the node and pod status back to the API server.

**Q: Does kubelet run on the master node?**

> Normally kubelet runs on **worker nodes only.** But technically it can run on master nodes too if master nodes are also used to run pods (which is not recommended in production).

**Q: What is a Node Heartbeat?**

> kubelet regularly sends a **status message to the API server** to say the node is alive and healthy. If the API server stops receiving this heartbeat, the node is marked as unhealthy and its pods are rescheduled to other nodes.

**Q: What happens when kubelet receives a pod assignment?**

> kubelet receives the pod specification from the API server, then talks to the **CRI (Container Runtime Interface)** to actually create the container. It then monitors the container and reports its status back to the API server.

**Q: What is the difference between kubelet and kube-controller-manager?**

> kube-controller-manager runs on the **control plane** and watches the entire cluster from a top level, deciding what actions need to be taken. kubelet runs on each **worker node** and actually executes those actions on its specific node.

&nbsp;

### 💡 Notes

```
📌 KUBELET

🔑 Role: Agent on every worker node
         Manages pods on its node

✅ Key Points:
  → Runs on EVERY worker node
  → Registers node with API server
  → Receives pod instructions from API server
  → Talks to CRI to create containers
  → Constantly watches pod health
  → Sends regular heartbeat to API server
  → Reports pod and node status

💓 NODE HEARTBEAT:
  → kubelet sends regular "I am alive!" message
  → API server stops receiving it?
  → Node marked as unhealthy!
  → Pods rescheduled to other nodes!

🔗 kubelet flow:
  API server assigns pod
        ↓
  kubelet receives instruction
        ↓
  kubelet tells CRI to create container
        ↓
  CRI creates container
        ↓
  kubelet watches container health
        ↓
  kubelet reports status to API server

⚖️ vs kube-controller-manager:
  kube-controller-manager = control plane
                            watches WHOLE cluster
                            decides WHAT to do

  kubelet                 = worker node
                            watches OWN node only
                            actually DOES the action

```

**How everything works together so far**

```
You: "I want 3 pods!"
         ↓
kube-api-server    → saves DESIRED STATE in etcd
         ↓
kube-controller-manager → "need to create 3 pods!" (WHAT)
         ↓
kube-scheduler     → "Pod goes to Node 2!" (WHERE)
         ↓
kube-api-server    → tells kubelet on Node 2
         ↓
kubelet on Node 2  → tells CRI → creates pod ✅
         ↓
kubelet            → reports back to API server
         ↓
kube-api-server    → saves ACTUAL STATE in etcd
         ↓
kube-controller-manager → checks 🔄
"Desired 3 = Actual 3 ✅ All good!"
```

&nbsp;

&nbsp;

# 2.7 🌐 kube-proxy

**kube-proxy** is a **network agent** that runs on **every worker node** and manages **network communication** between pods — inside the cluster and from outside world.

> Simple one line: **"kube-proxy = network manager of each worker node"**

&nbsp;

### 💡 Why do we need kube-proxy?

In Kubernetes — pods are **constantly created and destroyed.**

```
Problem:
  → Pod on Node 1 wants to talk to Pod on Node 2
  → But Pod IP addresses CHANGE every time!
  → How do you find the right pod?
  → How does traffic reach the correct pod?

Solution:
  → kube-proxy! 🎯
```

> kube-proxy makes sure **network traffic always reaches the right pod** — even when pods change!

Same way:

```
Pod A wants to talk to Pod B
           ↓
kube-proxy checks routing rules
           ↓
Forwards traffic to correct pod ✅

Even if Pod B is recreated with new IP
→ kube-proxy updates rules
→ Traffic still reaches Pod B!
```

### 💡 How kube-proxy works

kube-proxy uses something called **IPTables rules** (Linux networking rules):

```
STEP 1: New pod or service created in cluster
              ↓
STEP 2: API server notifies kube-proxy
              ↓
STEP 3: kube-proxy updates IPTables rules
        "Traffic for Service A → go to Pod X"
              ↓
STEP 4: When traffic comes in
        kube-proxy routes it to correct pod
              ↓
STEP 5: Pod destroyed → new pod created
        kube-proxy updates rules again! 🔄
```

### 💡 Key Responsibilities

| Job | Simple Meaning |
| --- | --- |
| **Network routing** | Routes traffic to correct pod |
| **Load balancing** | Spreads traffic among multiple pods |
| **Maintains IPTables** | Keeps network rules updated |
| **Service to Pod mapping** | Connects services to correct pods |

### 💡 Very Important Concept — Service & kube-proxy

This is **key for interviews!**

```
Problem:
  Pod IP changes every time pod restarts!
  
Solution:
  Kubernetes SERVICE = fixed address for pods
  
  Service IP never changes!
  Even if pods behind it change!

kube-proxy job:
  → Maps SERVICE (fixed IP) to correct PODS
  → Traffic hits service → kube-proxy → correct pod
```

Simple example:

```
Your app pods:
  Pod 1 = IP 10.0.0.1  ← can change!
  Pod 2 = IP 10.0.0.2  ← can change!
  Pod 3 = IP 10.0.0.3  ← can change!

Service = IP 10.100.0.1  ← NEVER changes!

User hits → Service IP (10.100.0.1)
                  ↓
            kube-proxy
                  ↓
         routes to Pod 1, 2, or 3 ✅
```

&nbsp;

### 💡 kubelet vs kube-proxy — Simple Difference

Both run on every worker node! But different jobs:

```
kubelet:
→ Manages PODS on the node
→ Creates/destroys containers
→ Watches pod health
→ Talks to CRI

kube-proxy:
→ Manages NETWORK on the node
→ Routes traffic between pods
→ Maintains network rules
→ Talks to IPTables
```

> kubelet = **pod manager** of the node
> 
> kube-proxy = **network manager** of the node

### 💡Questions

**Q: What is kube-proxy?**

> kube-proxy is a **network agent** that runs on every worker node. It maintains network rules (IPTables) on the node and is responsible for **routing traffic to the correct pods.** It enables communication between pods across different nodes and from outside the cluster.

**Q: Why do we need kube-proxy if we have kubelet?**

> kubelet manages **pod lifecycle** — creating and monitoring containers. kube-proxy manages **network communication** — routing traffic to the correct pods. They do completely different jobs. Both are needed on every worker node.

**Q: How does kube-proxy handle pod IP changes?**

> Pod IPs change every time a pod restarts. kube-proxy solves this by mapping a **Service** (which has a fixed IP) to the current pods. When pods change, kube-proxy **updates the IPTables rules** automatically so traffic always reaches the right pod.

**Q: What is IPTables in context of kube-proxy?**

> IPTables is a **Linux networking tool** that defines rules for how network traffic should be routed. kube-proxy uses IPTables to set up and maintain rules that direct traffic to the correct pods in the cluster.

**Q: Does kube-proxy do load balancing?**

> Yes! When multiple pods are running for the same service, kube-proxy **distributes the traffic** among all healthy pods — acting as a basic load balancer.

### 💡 **Notes**

```
📌 KUBE-PROXY

🔑 Role: Network agent on every worker node
         Routes traffic to correct pods

✅ Key Points:
  → Runs on EVERY worker node
  → Manages network rules (IPTables)
  → Routes traffic to correct pods
  → Does basic load balancing
  → Updates rules when pods change
  → Notified by API server about changes

🌐 Why needed?
  → Pod IPs change constantly!
  → kube-proxy maps SERVICE (fixed IP)
    to correct PODS (changing IPs)
  → Traffic always reaches right pod!

🔗 kube-proxy flow:
  New pod/service created
        ↓
  API server notifies kube-proxy
        ↓
  kube-proxy updates IPTables rules
        ↓
  Traffic comes in
        ↓
  kube-proxy routes to correct pod ✅

⚖️ kubelet vs kube-proxy:
  kubelet     = POD manager of node
                creates/destroys containers
                talks to CRI

  kube-proxy  = NETWORK manager of node
                routes traffic between pods
                talks to IPTables


```

&nbsp;

### 💡More about KUBE-PROXY networking.

### First — Why does Pod IP change?

Simple reason:

```
Pod is like a TEMPORARY worker!

Pod created   → gets IP: 10.0.0.1
Pod crashes   → IP 10.0.0.1 is GONE!
New pod created → gets NEW IP: 10.0.0.5

Every time pod restarts → NEW IP address!
K8s does not keep same IP for pods!
```

> Pod IP is like a **temporary phone number** — every time you get a new SIM, number changes!

### 💡 What is a SERVICE?

Think of a **real development example:**

You built a website:

```
Website = Frontend (what users see)
Backend = Your Node.js API app
Database = MySQL
```

Your **Node.js API** is running in 3 pods:

```
Pod 1 = IP 10.0.0.1  (Node.js app)
Pod 2 = IP 10.0.0.2  (Node.js app)
Pod 3 = IP 10.0.0.3  (Node.js app)
```

Now your **Frontend** wants to call the API.

```
❌ Problem without Service:

Frontend calls → 10.0.0.1 (Pod 1)
Pod 1 crashes!
Pod 1 gets new IP → 10.0.0.4

Frontend still calling → 10.0.0.1
NOBODY is there! ❌
Frontend is broken!
```

### 💡 Solution — Kubernetes SERVICE

> A **Service** is like a **permanent reception desk** that always sits in front of your pods!

```
SERVICE = fixed address that NEVER changes
        = always points to your pods
        = even when pods change!
```

```
✅ With Service:

Frontend calls → SERVICE IP (10.100.0.1) ← NEVER changes!
                      ↓
                  kube-proxy
                      ↓
              routes to Pod 1, 2, or 3
                  (whichever is healthy!)

Pod 1 crashes? → Service still there!
New Pod 4 created? → Service finds it!
Frontend never breaks! ✅
```

&nbsp;

### 💡 Complete Picture — Real Dev Example

```
YOUR PRODUCTION SETUP:

[User visits website]
        ↓
[Frontend pod]
        ↓
calls API at → SERVICE IP 10.100.0.1
        ↓
[kube-proxy on node]
checks IPTables rules:
"10.100.0.1 → go to Node.js pods"
        ↓
routes to → Pod 1 ✅
         or Pod 2 ✅
         or Pod 3 ✅
(whichever is running and healthy!)
        ↓
[Node.js API pod responds]
        ↓
[Frontend gets response] ✅

Pod crashes and restarts with new IP?
→ kube-proxy updates rules automatically!
→ Service IP stays same!
→ Everything keeps working! ✅
```

&nbsp;

### 💡 Notes — Add this to kube-proxy notes

```
📌 WHAT IS A SERVICE?

🔑 Service = Permanent fixed address
             that sits in front of pods

WHY pods need a Service:
  → Pod IP changes every restart
  → Other apps cannot rely on pod IP
  → Service IP NEVER changes!
  → Always points to correct pods!

🌐 How it works:
  User/App → Service (fixed IP)
                ↓
           kube-proxy
                ↓
           correct pod ✅

Real dev example:
  Node.js API pods = change IPs constantly
  Service in front = always same IP
  Frontend always calls Service IP
  Never breaks! ✅

🔑 Simple one line:
  Service = stable phone number
  Pods    = the people who answer
  Even if people change →
  phone number stays same!
```

&nbsp;

### 💡 Question

**Q: What is a Kubernetes Service and why is it needed?**

> A Service is a **permanent fixed IP address** that sits in front of pods. Since pod IPs change every time a pod restarts, other applications cannot directly rely on pod IPs. A Service provides a **stable address that never changes** and kube-proxy routes traffic from the Service to the correct running pods automatically.

**Q: Where is a Kubernetes Service stored?**

> A Service is a **Kubernetes object stored in etcd.** kube-proxy reads the Service and Pod information from the API server and writes **routing rules into IPTables** on each worker node. IPTables is part of the **Linux OS** on the node — kube-proxy uses it but does not contain it.

&nbsp;

### 🔑 Where is IPTables located?

```
IPTables is NOT inside kube-proxy!

IPTables is part of LINUX KERNEL
→ It lives in the OPERATING SYSTEM
→ of each worker node!

kube-proxy USES IPTables
→ kube-proxy WRITES rules INTO IPTables
→ kube-proxy does NOT contain IPTables!
```

> Think of it like this: IPTables = a **whiteboard on the wall** (Linux OS) kube-proxy = the person who **writes rules on the whiteboard**

### 🔑 where Kubernetes Service located !

```
KUBERNETES SERVICE
→ Service is NOT inside IPTables
→ Service is NOT inside kube-proxy

→ Service is a K8s OBJECT
→ It is defined in K8s cluster
→ Its information is stored in ETCD!
```

> Service is like a **rule you create in Kubernetes.** Just like pods and deployments — Service is also a Kubernetes object stored in **etcd!**

### 🔑 Then what is inside IPTables?

IPTables stores the **ROUTING RULES** — not the Service itself!

```
etcd stores:
→ Service definition
→ Service IP (10.100.0.1)
→ Which pods belong to this service

IPTables stores:
→ "Traffic coming for 10.100.0.1
   → forward to Pod 10.0.0.1
   OR Pod 10.0.0.2
   OR Pod 10.0.0.3"
```

> IPTables = just the **traffic routing rules!** Not the Service itself!

### 🔑 Are pod IPs also saved in IPTables?

```
✅ YES! Partially correct!

IPTables has rules like:

"Service IP 10.100.0.1
    → Pod 1 IP 10.0.0.1  ✅
    → Pod 2 IP 10.0.0.2  ✅
    → Pod 3 IP 10.0.0.3  ✅"

So YES — pod IPs are referenced
inside IPTables rules!
```

### **💡 Clear Picture**

```
ETCD
→ stores Service object
→ stores Service IP (10.100.0.1)
→ stores pod information
        ↓
API server notifies kube-proxy
about Service and Pod changes
        ↓
KUBE-PROXY
→ reads Service + Pod info
→ writes ROUTING RULES
        ↓
IPTABLES (Linux OS of worker node)
→ stores routing rules
→ "Service IP → which pods"
→ pod IPs referenced here
        ↓
Traffic comes in
→ IPTables routes to correct pod ✅
```

&nbsp;

### **💡 Simple Summary**

| Thing | Where it lives | Simple meaning |
| --- | --- | --- |
| **Service object** | etcd | K8s object — defined in cluster |
| **Service IP** | etcd + IPTables rules | Fixed IP stored in etcd, used in IPTables |
| **IPTables** | Linux OS of worker node | Routing rules whiteboard |
| **kube-proxy** | Worker node | Writes rules onto IPTables |
| **Pod IPs** | etcd + IPTables rules | Referenced in routing rules |

&nbsp;

&nbsp;

## 2.8 CRI (Container Runtime Interface)

* * *

### 💡Simple Explanation

**CRI** is the software that **actually runs the containers** on the worker node.

> Simple one line: **"CRI = the engine that actually creates and runs containers"**

Remember we said kubelet tells WHAT to create? **CRI is the one that ACTUALLY does it!**

```
kubelet → "please create this container!"
              ↓
           CRI
              ↓
      container is created ✅
```

### 💡 Wait — Isn't that Docker's job?

Great question! This is a **very common confusion!**

```
Before Kubernetes grew big:
→ Docker was the ONLY way to run containers
→ K8s only supported Docker

But then other container tools came:
→ containerd
→ CRI-O
→ podman

K8s needed to support ALL of them!
So K8s created CRI —
a STANDARD INTERFACE that works
with ANY container runtime! ✅
```

> 💡CRI is like a **universal remote control** that works with ANY TV brand!

### 💡 Real Life Example

Think of a **power socket standard:**

```
In Sri Lanka — Type D socket standard

Any plug that follows Type D standard
→ works in any Sri Lankan socket!

Hairdryer ✅
Phone charger ✅
Laptop charger ✅

Same way:

CRI = standard interface

Any container runtime that follows CRI standard
→ works with Kubernetes!

containerd ✅
CRI-O ✅
Docker ✅
```

### 💡 Popular CRI Implementations

| CRI Runtime | Simple Meaning |
| --- | --- |
| **containerd** | Most popular today — lightweight, fast |
| **CRI-O** | Lightweight — made specifically for K8s |
| **Docker** | Old default — still works but heavy |

```
Old days (before K8s 1.24):
→ Docker was default CRI

Today (K8s 1.24+):
→ containerd is default CRI
→ Docker removed as default!
```

> ⚠️ Important interview point! **Docker was removed as default CRI in K8s version 1.24!**

### 💡 How CRI works — Step by Step

```
STEP 1: Scheduler assigns pod to Node 2
              ↓
STEP 2: API server tells kubelet on Node 2
        "create this pod!"
              ↓
STEP 3: kubelet reads pod specification
        "needs nginx container,
         512MB memory, 1 CPU"
              ↓
STEP 4: kubelet calls CRI
        "hey CRI! create this container!"
              ↓
STEP 5: CRI pulls container image
        from registry (Docker Hub etc.)
              ↓
STEP 6: CRI creates and starts container
              ↓
STEP 7: Container is running! ✅
              ↓
STEP 8: kubelet monitors it
        reports status to API server
```

### 💡 Important Points

| Point | Simple Meaning |
| --- | --- |
| **CRI is an interface** | A standard — not one specific tool |
| **Lives on worker node** | Runs on every worker node |
| **kubelet talks to CRI** | kubelet uses CRI to create containers |
| **Pulls images** | Downloads container images from registry |
| **containerd is default** | Most clusters use containerd today |

### 💡 Docker vs containerd — Simple Difference

```
Docker:
→ Big tool with many features
→ Build images ✅
→ Run containers ✅
→ Docker CLI ✅
→ Docker compose ✅
→ Heavy — too much for K8s!

containerd:
→ Small lightweight tool
→ Only runs containers ✅
→ No extra features
→ Fast and efficient
→ Perfect for K8s!
```

> K8s only needs to RUN containers — not build them! So containerd is perfect! ✅

### 💡 Interview Questions

**Q: What is CRI in Kubernetes?**

> CRI stands for **Container Runtime Interface.** It is a standard interface that allows Kubernetes to work with any container runtime — like containerd, CRI-O, or Docker. kubelet communicates with the container runtime through CRI to create and manage containers.

**Q: Why was CRI created?**

> Initially K8s only supported Docker. As other container runtimes emerged, K8s created CRI as a **standard interface** so that any container runtime that follows the CRI standard can work with Kubernetes — making K8s runtime independent.

**Q: What is the default CRI in modern Kubernetes?**

> **containerd** is the default CRI in modern Kubernetes. Docker was removed as the default container runtime in **Kubernetes version 1.24.**

**Q: What is the difference between Docker and containerd?**

> Docker is a **full featured tool** that can build images, run containers, and more — making it heavy for K8s. containerd is a **lightweight runtime** that only focuses on running containers, making it faster and more efficient for Kubernetes.

**Q: Which component talks to CRI?**

> **kubelet** talks to CRI. When kubelet receives a pod assignment, it calls CRI to actually create and run the container on the node.

### 💡Notes

```
📌 CRI (CONTAINER RUNTIME INTERFACE)

🔑 Role: Actually creates and runs containers
         on worker nodes

✅ Key Points:
  → CRI = standard interface for container runtimes
  → Lives on every worker node
  → kubelet talks to CRI
  → CRI actually creates containers
  → Pulls images from registry
  → Any runtime following CRI standard works with K8s!

📦 Popular CRI runtimes:
  → containerd  = default today (lightweight, fast)
  → CRI-O       = made for K8s (lightweight)
  → Docker      = old default (heavy, removed in 1.24)

⚠️ Important:
  → Docker removed as default CRI in K8s v1.24!
  → containerd is now default!

⚖️ Docker vs containerd:
  Docker      = full featured (build + run + CLI)
                heavy — too much for K8s
  containerd  = only runs containers
                lightweight — perfect for K8s!

🔗 Flow:
  kubelet receives pod instruction
        ↓
  kubelet calls CRI
        ↓
  CRI pulls image from registry
        ↓
  CRI creates and starts container ✅
```

&nbsp;

### 💡 What is Container Runtime?

Simple answer:

> **Container Runtime = software that actually creates and runs containers**

```
You have a container image
(like a blueprint)
        ↓
Container Runtime takes that image
        ↓
Creates a running container from it!
```

Think of it like:

```
Container Image  = a recipe 📄
Container Runtime = the chef who cooks it! 👨‍🍳
```

&nbsp;

### 💡 Docker Diagram

```
Docker CLI
→ Just the command tool
→ What YOU type (docker run, docker build)
           to
Docker Engine (dockerd)
→ Brain of Docker
→ High level manager
→ Receives CLI commands
           to
containerd
→ Mid level manager
→ Manages container lifecycle
→ pulls images, starts containers
           to
runc
→ Lowest level
→ Actually creates the container
→ Talks directly to Linux kernel

Linux Kernel
→ namespaces + cgroups
→ Actually isolates the container
```

&nbsp;

### 💡 Two different levels to understand

### In Docker world:

```
Docker Engine (dockerd)
        ↓
    containerd        ← manages containers
        ↓
      runc             ← actually runs containers
        ↓
   containers
```

### In Kubernetes world:

```
kubelet
   ↓
containerd (CRI)  ← same containerd!
   ↓
runc              ← same runc!
   ↓
containers
```

### 💡 The KEY difference

```
IN DOCKER:
Docker Engine → controls containerd
Docker Engine is the BOSS of containerd

IN KUBERNETES:
Docker Engine is REMOVED!
kubelet directly talks to containerd!
kubelet becomes the NEW BOSS!

containerd and runc stay the same!
Only the boss changed!
```

### 💡 Docker vs Kubernetes

```
DOCKER WORLD:
You
 ↓ docker run nginx
Docker CLI
 ↓
Docker Engine (dockerd)  ← extra layer!
 ↓
containerd               ← manages container
 ↓
runc                     ← runs container
 ↓
container ✅

KUBERNETES WORLD:
You
 ↓ kubectl create pod
API server
 ↓
kubelet                  ← replaces Docker Engine!
 ↓
containerd (CRI)         ← same containerd!
 ↓
runc                     ← same runc!
 ↓
container ✅
```

### 💡Interview Questions

**Q: What is a container runtime?**

> A container runtime is the **software responsible for actually creating and running containers** on a node. It takes a container image and creates a running container from it.

**Q: Is containerd in Docker the same as containerd in Kubernetes?**

> Yes! It is the **exact same containerd.** In Docker, containerd is managed by the Docker Engine. In Kubernetes, Docker Engine is removed and **kubelet directly manages containerd** through the CRI standard.

**Q: Why was Docker removed as default runtime in K8s 1.24?**

> Docker was removed because it did not directly implement the **CRI standard.** K8s had to use a special compatibility layer called **dockershim** to talk to Docker. This was extra overhead. containerd directly implements CRI — so K8s removed Docker and uses containerd directly for better performance.

&nbsp;

### 🎯 In Docker world there are **2 levels of runtime:**

💡 Level 1 — High Level Runtime

```
containerd = HIGH LEVEL container runtime

Job:
→ pulls images
→ manages container lifecycle
→ manages storage
→ manages networking
→ then passes to runc
```

💡Level 2 — Low Level Runtime

```
runc = LOW LEVEL container runtime

Job:
→ actually creates the container
→ talks directly to Linux kernel
→ uses namespaces + cgroups
→ starts the process inside container
```

💡 Simple Difference

```
containerd = manager of containers
runc       = actually BUILDS the container

containerd says → "hey runc! create this container!"
runc says       → "yes sir!" → talks to kernel → done!
```

&nbsp;

**Q: What is the container runtime in Docker?**

> Docker uses **two levels of runtime:**
> 
> 1.  **containerd** — high level runtime that manages container lifecycle
> 2.  **runc** — low level runtime that actually creates and runs containers by talking directly to the Linux kernel
> 
> containerd manages the process and delegates the actual container creation to runc.

### 🔗 Your Docker Diagram — Runtime highlighted

```
Docker CLI          ← NOT runtime (just a tool)
      ↓
Docker Engine       ← NOT runtime (manager/brain)
      ↓
containerd          ← ✅ HIGH LEVEL RUNTIME
      ↓
runc                ← ✅ LOW LEVEL RUNTIME
      ↓
containers
      ↓
Linux Kernel
```

&nbsp;

&nbsp;

# 3\. Complete Short Note

### 🎉 Complete Architecture — Everything Together!

```
YOU
 ↓ kubectl command
 ↓
CONTROL PLANE
│
├── kube-api-server    ← receives your request
│         ↓
├── etcd               ← saves desired state
│         ↓
├── kube-controller-manager  ← notices new request
│         ↓                    decides WHAT to do
├── kube-scheduler     ← decides WHERE to run
│         ↓
│   tells API server → saves in etcd
│
WORKER NODE (chosen by scheduler)
│
├── kubelet            ← gets instruction from API server
│         ↓
├── CRI                ← creates the container
│         ↓
└── Pod running! ✅
│
├── kube-proxy         ← makes sure network traffic
                         reaches the pod correctly!
```

&nbsp;

### 🎉 Complete Architecture Picture — All Components!

```
CLUSTER
│
├── CONTROL PLANE (Brain 🧠)
│   │
│   ├── kube-api-server
│   │     → Front door of cluster
│   │     → Everything talks through it
│   │
│   ├── etcd
│   │     → Database of cluster
│   │     → Stores desired + actual state
│   │
│   ├── kube-scheduler
│   │     → Decides WHERE pods run
│   │     → Filters + scores nodes
│   │
│   ├── kube-controller-manager
│   │     → Watches + fixes cluster (WHAT)
│   │     → Runs reconciliation loop
│   │
│   └── cloud-controller-manager
│         → Bridge between K8s and cloud
│         → Optional (only for cloud)
│
└── WORKER NODES (Muscle 💪)
    │
    ├── kubelet
    │     → Pod manager of node
    │     → Creates pods via CRI
    │     → Sends heartbeat to API server
    │
    ├── kube-proxy
    │     → Network manager of node
    │     → Routes traffic to correct pods
    │     → Maintains IPTables rules
    │
    └── CRI (Container Runtime Interface)
          → Actually runs the containers
          → eg: containerd, Docker
```

### 💡 All Components — Quick Interview Table

| Component | Where | One Line |
| --- | --- | --- |
| **kube-api-server** | Control Plane | Front door — everything goes through it |
| **etcd** | Control Plane | Database — stores entire cluster state |
| **kube-scheduler** | Control Plane | Decides WHERE to place pods |
| **kube-controller-manager** | Control Plane | Watches and fixes cluster state |
| **cloud-controller-manager** | Control Plane | Bridge between K8s and cloud |
| **kubelet** | Worker Node | Pod manager of each node |
| **kube-proxy** | Worker Node | Network manager of each node |
| **CRI** | Worker Node | Actually runs the containers |

&nbsp;

## 🏆 Congratulations!

You have completed the **entire Kubernetes Architecture!** 🎉

```
✅ What is Kubernetes
✅ Why Kubernetes (Docker problems)
✅ Cluster Architecture
✅ All Control Plane components
✅ All Worker Node components
```