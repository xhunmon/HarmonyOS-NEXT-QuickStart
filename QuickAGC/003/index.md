# Complete Guide to Domain Registration and Network Compliance

For HarmonyOS Next apps involving network services (e.g., APIs, cloud storage, user registration), domain name registration and ICP filing are mandatory. This guide details the filing process, compliance requirements, common issues, and official resources to help developers pass network compliance reviews.

## 1. Necessity of Domain Name Filing

### HTTPS Configuration Example
Configure HTTPS in Nginx for secure communication:
```nginx
server {
  listen 443 ssl;
  server_name repo.example.com;  // Replace with filed domain

  ssl_certificate /path/to/ssl/server.crt;  // SSL certificate path
  ssl_certificate_key /path/to/ssl/server.key;  // Private key path

  // Security headers
  add_header Content-Security-Policy "default-src 'self'";
  add_header X-Content-Type-Options nosniff;

  location / {
    proxy_pass   // Forward to app service
  }
}
```  

### Network Requests with Axios
Pseudocode for API requests:
```typescript
axios.create({
  baseURL: API.base_url,
  timeout: 5000,
  connectTimeout: 60000,
  headers: {}
}).post<string, AxiosResponse<MagicResp<T>>, D>(url, data)
  .then((response: AxiosResponse<MagicResp<T>>) => {
    // Handle successful response
  })
  .catch((error: Error) => {
    // Handle errors
  });
```  

### Network Permission Declaration
Declare permissions in `module.json5`:
```json
"requestPermissions": [
  {
    "name": "ohos.permission.INTERNET"
  },
  {
    "name": "ohos.permission.GET_NETWORK_INFO"
  }
]
```  

**Key Requirements**:
- **Network services** (APIs, cloud storage, etc.) require filed domains.
- **Standalone offline apps** (no internet) are exempt.
- **Legal mandate**: Unfiled domains are unusable in Chinese mainland apps.

## 2. Domain Registration Process

1. **Choose a registrar** (e.g., Huawei Cloud, Alibaba Cloud).
2. **Select a concise domain** aligned with your app’s branding.
3. **Complete real-name verification** and submit registration.
4. **Manage domains** via the registrar’s backend.


## 3. ICP Filing Process

1. Log in to the registrar’s backend and navigate to "ICP Filing".
2. Submit:
    - Filing entity info (enterprise/individual)
    - Website details
    - Responsible person information
3. Upload documents:
    - Business license (enterprises)
    - ID card (individuals)
    - Commitment letter
4. Await registrar初审 → Local bureau review (7-20 working days).
5. **Display filing number** prominently in app/website footers post-approval.


## 4. Common Compliance Issues

| Issue | Consequence | Solution |  
|-------|-------------|----------|  
| **Unfiled domains** | App rejection | Complete filing before submission |  
| **Inconsistent entity/app info** | Filing rejection | Ensure entity = developer info |  
| **Missing filing number display** | Compliance failure | Add to app/website footer |  
| **Blurry/incomplete documents** | Processing delays | Submit clear, valid materials |  

## 5. Official References
- Huawei Cloud ICP Filing
- Huawei Developer Alliance
- HarmonyOS Next Docs

## 6. Conclusion
Domain filing and network compliance are critical for HarmonyOS Next app launches. Proactively register domains, complete filings, and ensure all network services comply. Consult registrar/regulatory documentation or developer communities if issues arise to guarantee smooth approval and secure deployment. 