# 🧩 DevOps Engineer Diploma – Containers with Docker  
## 🧪 Lab 01: Docker Network Configuration and Service Isolation  

### 🎯 Lab Objective  
This lab demonstrates how to design **isolated and secure Docker network environments** to host containerized applications while ensuring:
- Custom Bridge Network creation and management  
- Controlled IP Address allocation  
- Internal Service Discovery (DNS-based communication)  
- Multi-network container architecture (Frontend & Backend separation)  
- Network isolation and connectivity testing  

---

## 🧱 Lab Sections  
### Part 1: HR Application Stack (Single Custom Network)
In this part, we create a **dedicated bridge network** for a Human Resources (HR) application running on a single Docker host.  
The goal is to enable container-to-container communication using names instead of IPs, while avoiding conflicts with the corporate VPN (172.17.0.0/16).

---

### ⚙️ Requirements  
| Component | Details |
|------------|----------|
| **Network Type** | Custom Bridge |
| **Network Name** | `hr-app-net` |
| **Subnet** | `192.168.20.0/24` |
| **Gateway** | `192.168.20.1` |
| **Containers** | `nginx-server` (frontend) & `alpine-tester` (testing client) |

---

### 🧰 Steps to Execute  

#### 1️⃣ Create a Custom Bridge Network  
```bash
docker network create \
  --driver bridge \
  --subnet 192.168.20.0/24 \
  --gateway 192.168.20.1 \
  hr-app-net
````

#### 2️⃣ Verify Network Configuration

```bash
docker network inspect hr-app-net
```

#### 3️⃣ Run Containers in the Network

```bash
# Run NGINX container
docker run -d --name nginx-server --network hr-app-net nginx

# Run Alpine container for testing
docker run -it --name alpine-tester --network hr-app-net alpine sh
```

#### 4️⃣ Verify IP Allocation

```bash
docker inspect nginx-server | grep "IPAddress"
docker inspect alpine-tester | grep "IPAddress"
```

✅ Both containers should receive IPs within `192.168.20.x`.

#### 5️⃣ Test Service Discovery

Inside `alpine-tester`:

```bash
ping nginx-server
```

✅ Expected Result: Successful ping → proves internal DNS resolution works.

---

### 🧾 Expected Outcome

* Both containers are assigned IPs from the `192.168.20.0/24` range.
* Ping from `alpine-tester` → `nginx-server` succeeds using the container name.
* Confirms **service discovery** and **network isolation** within the subnet.

---

## 🧩 Part 2: Multi-Network Architecture (Frontend & Backend Isolation)

This part demonstrates a **multi-homed container setup**, where an NGINX Load Balancer bridges communication between two isolated networks:

* **frontend-net** (client traffic)
* **backend-net** (internal service communication)

---

### ⚙️ Requirements

| Network  | Name           | Subnet        |
| -------- | -------------- | ------------- |
| Frontend | `frontend-net` | `10.1.1.0/24` |
| Backend  | `backend-net`  | `10.1.2.0/24` |

| Container         | Description                | Connected Network(s)       |
| ----------------- | -------------------------- | -------------------------- |
| **nginx-lb**      | Acts as Load Balancer      | frontend-net + backend-net |
| **client-tester** | Simulated client container | frontend-net only          |
| **backend-db**    | Simulated backend service  | backend-net only           |

---

### 🧰 Steps to Execute

#### 1️⃣ Create Networks

```bash
docker network create --driver bridge --subnet 10.1.1.0/24 frontend-net
docker network create --driver bridge --subnet 10.1.2.0/24 backend-net
```

#### 2️⃣ Deploy Containers with Isolation

```bash
docker run -d --name backend-db --network backend-net alpine sleep 1000
docker run -it --name client-tester --network frontend-net alpine sh
```

#### 3️⃣ Deploy Multi-Homed NGINX Load Balancer

```bash
docker run -d --name nginx-lb \
  --network frontend-net \
  nginx

# Attach it to backend network as well
docker network connect backend-net nginx-lb
```

#### 4️⃣ Verify Multi-Network Connection

```bash
docker inspect nginx-lb | grep "IPAddress"
```

✅ You should see two IPs:

* One from `10.1.1.x` (frontend-net)
* One from `10.1.2.x` (backend-net)

---

### 🧪 Diagnostic Tests

#### Test 1: Isolation Validation

From `client-tester`:

```bash
ping backend-db
```

❌ Expected Result: Ping fails → proves frontend and backend are **isolated**.

#### Test 2: Connectivity via Load Balancer

From `nginx-lb`:

```bash
docker exec -it nginx-lb ping backend-db
```

✅ Expected Result: Ping succeeds → Load Balancer bridges both networks.

---

### 🧾 Expected Outcome

* `nginx-lb` container gets two IP addresses (one from each network).
* Direct communication between frontend and backend containers is blocked.
* Only the Load Balancer (nginx-lb) can communicate with both sides — ensuring **network segmentation** and **security**.

---

## 🧠 Key Learnings

✅ Understand how Docker handles networking and IP allocation.
✅ Learn to isolate application tiers using multiple networks.
✅ Practice internal service discovery using container names (DNS).
✅ Implement secure, production-like multi-network architectures.
✅ Validate and troubleshoot network communication using `docker inspect` and `ping`.

---
```
<img width="1366" height="768" alt="Screenshot (472)" src="https://github.com/user-attachments/assets/7c5468e8-4872-49ee-b367-ea82ba7a2d6f" />
<img width="1366" height="768" alt="Screenshot (471)" src="https://github.com/user-attachments/assets/fe2c1d10-c375-4b09-bc72-ca1f93a74468" />
<img width="1366" height="768" alt="Screenshot (470)" src="https://github.com/user-attachments/assets/1d64ff51-1bc5-491f-8087-53f9b0eb3b0a" />
<img width="1366" height="768" alt="Screenshot (469)" src="https://github.com/user-attachments/assets/44d158c2-3469-4a87-8a64-05671fdea891" />
<img width="1366" height="768" alt="Screenshot (468)" src="https://github.com/user-attachments/assets/f25f3f5f-1a70-4e38-b12d-0aadafa13a3b" />
<img width="1366" height="768" alt="Screenshot (467)" src="https://github.com/user-attachments/assets/29b5eb7c-a807-4d84-bc57-e05651cf20b5" />
<img width="1366" height="768" alt="Screenshot (466)" src="https://github.com/user-attachments/assets/d045f568-7652-4db4-85c6-52b8cf68e339" />
<img width="1366" height="768" alt="Screenshot (465)" src="https://github.com/user-attachments/assets/48bdce0e-961b-4d9f-ab2b-930e91ed4608" />
<img width="1366" height="768" alt="Screenshot (464)" src="https://github.com/user-attachments/assets/206325ea-0663-4265-9bb3-e6110f3f6282" />
<img width="1366" height="768" alt="Screenshot (463)" src="https://github.com/user-attachments/assets/b4772855-e6d2-43d1-856f-73fdfe610e12" />
<img width="1366" height="768" alt="Screenshot (462)" src="https://github.com/user-attachments/assets/08b77d19-851f-4f5a-b6f4-a0de2f3c60b7" />
<img width="1366" height="768" alt="Screenshot (461)" src="https://github.com/user-attachments/assets/52c80953-1d9b-459a-bed3-7d9010217274" />
<img width="1366" height="768" alt="Screenshot (460)" src="https://github.com/user-attachments/assets/63612bec-bf06-43c9-908a-288b9a1559a4" />
<img width="1366" height="768" alt="Screenshot (459)" src="https://github.com/user-attachments/assets/1e149c7c-0fa5-4947-a4fa-0507b32be45f" />
<img width="1366" height="768" alt="Screenshot (458)" src="https://github.com/user-attachments/assets/6ad5af4f-e77c-4e74-ab9f-77af15bf31c7" />


--
