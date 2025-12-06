# Web Application Firewall Lab (NGINX + ModSecurity + CRS)

This project shows how to run a Web Application Firewall (WAF) using:
* NGINX
* ModSecurity
* OWASP Core Rule Set (CRS)
* Docker Compose

A simple backend application (`hashicorp/http-echo`) is used to test safe and malicious requests.

## 📌 Features

* WAF protects the backend from:
   * SQL Injection (SQLi)
   * Cross‑Site Scripting (XSS)
   * Remote Code Execution (RCE)
* Safe requests reach the backend
* Malicious requests are blocked with HTTP 403
* Easy to run with Docker Compose
* No changes were made to original config files

## 📁 Project Structure

```
project/
 ├─ docker-compose.yml
 └─ waf/
      ├─ modsecurity.conf
      ├─ modsecurity-crs.conf
      └─ conf.d/
           └─ proxy.conf
```

## 🏗️ Architecture

```
[Browser]
     ↓
[WAF: NGINX + ModSecurity + CRS]
     ↓
[Backend: hashicorp/http-echo]
```

* WAF listens on port **8080**
* Backend listens on port **5678**

## 🚀 Getting Started

### 1. Start the environment

```bash
docker compose up -d
```

### 2. Check running containers

```bash
docker ps
```

You should see:
* `owasp/modsecurity-crs:nginx`
* `hashicorp/http-echo`

**📸 Screenshot placeholder #1 – Running containers**

### 3. Test WAF is responding

Visit:

```
http://localhost:8080
```

Expected output:

```
hello from app
```

**📸 Screenshot placeholder #2 – Safe request success**

## 🧪 Test Cases

### ✅ Test Case 1 – Safe Request

**Request:**

```
http://localhost:8080/?status=ok
```

**Expected:**
* 200 OK
* `hello from app`


---

### ❌ Test Case 2 – SQL Injection

**Request:**

```
http://localhost:8080/?id=1' OR '1'='1
```

**Expected:**
* 403 Forbidden
* Blocked by CRS (rules 942100, 942130)

---

### ❌ Test Case 3 – XSS Attack

**Request:**

```
http://localhost:8080/?q=<script>alert(1)</script>
```

**Expected:**
* 403 Forbidden
* Blocked by CRS (rules 941100, 941120, 941210)

---

### ❌ Test Case 4 – Remote Command Execution (RCE)

**Requests:**

```
http://localhost:8080/?cmd=;cat /etc/passwd
http://localhost:8080/?cmd=$(id)
```

**Expected:**
* 403 Forbidden
* Blocked by RCE rules


---

### ❌ Test Case 5 – POST Attack

**Command:**

```bash
curl -X POST -d "username=admin' --" http://localhost:8080
```

**Expected:**
* 403 Forbidden
* SQLi blocked


---

## 📜 Logs and Validation

To see WAF logs:

```bash
docker exec -it <waf_container> sh
tail -f /var/log/nginx/access.log
```

Example blocked log entry:

```
"403" "-" ModSecurity ...
```

---

## 📄 Config File Descriptions

### docker-compose.yml
* Starts the backend and WAF services
* Maps ports 8080 (WAF) and 5678 (backend)
* Mounts WAF configuration files

### proxy.conf
* NGINX reverse proxy file
* Forwards all traffic from WAF to backend
* Adds standard proxy headers

### modsecurity.conf
* Main ModSecurity configuration
* Enables WAF engine
* Controls logging, request body checks, and rule behavior

### modsecurity-crs.conf
* Loads OWASP Core Rule Set
* Includes rules to block SQLi, XSS, RCE, and many other attacks

---

## ✅ Conclusion

This lab shows that:
* Safe traffic reaches the backend
* SQLi, XSS, and RCE attacks are blocked
* ModSecurity + CRS work correctly inside Docker
* The setup runs without changing your original files
* The WAF protects the backend effectively

---

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

## 👤 Author

Pedram Faramarz