# Part 4: API Test Scenarios - VPC Management API

## 1. CRUD Operations Testing

### Create VPC (POST /v1/vpcs)

#### API-001: Create VPC with Valid Data
**Endpoint:** POST /v1/vpcs  
**Test Data:**
```json
{
  "name": "test-vpc-001",
  "cidr_block": "10.0.0.0/16",
  "region": "us-east-1",
  "enable_dns_resolution": true,
  "enable_dns_hostnames": false
}
```
**Expected Response:** 201 Created
**Expected Payload:**
```json
{
  "id": "vpc-12345abc",
  "name": "test-vpc-001",
  "cidr_block": "10.0.0.0/16",
  "region": "us-east-1",
  "enable_dns_resolution": true,
  "enable_dns_hostnames": false,
  "state": "creating",
  "created_at": "2025-07-17T10:00:00Z"
}
```

#### API-002: Create VPC with Minimal Required Fields
**Endpoint:** POST /v1/vpcs  
**Test Data:**
```json
{
  "name": "minimal-vpc",
  "cidr_block": "172.16.0.0/16",
  "region": "us-west-2"
}
```
**Expected Response:** 201 Created

### Read VPC Operations (GET)

#### API-003: Get All VPCs
**Endpoint:** GET /v1/vpcs  
**Expected Response:** 200 OK
**Expected Payload:**
```json
{
  "vpcs": [
    {
      "id": "vpc-12345abc",
      "name": "test-vpc-001",
      "cidr_block": "10.0.0.0/16",
      "region": "us-east-1",
      "state": "available"
    }
  ],
  "count": 1
}
```

#### API-004: Get Specific VPC by ID
**Endpoint:** GET /v1/vpcs/{vpc_id}  
**Expected Response:** 200 OK
**Expected Payload:** Full VPC object with all details

#### API-005: Get Non-Existent VPC
**Endpoint:** GET /v1/vpcs/vpc-nonexistent  
**Expected Response:** 404 Not Found
**Expected Payload:**
```json
{
  "error": "VPC not found",
  "error_code": "VPC_NOT_FOUND"
}
```

### Update VPC (PUT /v1/vpcs/{vpc_id})

#### API-006: Update VPC Name
**Endpoint:** PUT /v1/vpcs/{vpc_id}  
**Test Data:**
```json
{
  "name": "updated-vpc-name",
  "enable_dns_resolution": false
}
```
**Expected Response:** 200 OK

#### API-007: Update Non-Existent VPC
**Endpoint:** PUT /v1/vpcs/vpc-nonexistent  
**Expected Response:** 404 Not Found

### Delete VPC (DELETE /v1/vpcs/{vpc_id})

#### API-008: Delete Existing VPC
**Endpoint:** DELETE /v1/vpcs/{vpc_id}  
**Expected Response:** 204 No Content

#### API-009: Delete Non-Existent VPC
**Endpoint:** DELETE /v1/vpcs/vpc-nonexistent  
**Expected Response:** 404 Not Found

---

## 2. Input Validation Testing

### CIDR Block Validation

#### API-010: Invalid CIDR - Outside Range
**Endpoint:** POST /v1/vpcs  
**Test Data:**
```json
{
  "name": "invalid-cidr-vpc",
  "cidr_block": "10.0.0.0/15",
  "region": "us-east-1"
}
```
**Expected Response:** 400 Bad Request
**Expected Error:** "CIDR block must be between /16 and /28"

#### API-011: Invalid CIDR - Malformed
**Endpoint:** POST /v1/vpcs  
**Test Data:**
```json
{
  "name": "malformed-cidr-vpc",
  "cidr_block": "10.0.0.0",
  "region": "us-east-1"
}
```
**Expected Response:** 400 Bad Request

#### API-012: Invalid CIDR - Public IP Range
**Endpoint:** POST /v1/vpcs  
**Test Data:**
```json
{
  "name": "public-ip-vpc",
  "cidr_block": "8.8.8.0/24",
  "region": "us-east-1"
}
```
**Expected Response:** 400 Bad Request
**Expected Error:** "Only private IP ranges allowed"

### Name Validation

#### API-013: Empty VPC Name
**Endpoint:** POST /v1/vpcs  
**Test Data:**
```json
{
  "name": "",
  "cidr_block": "10.0.0.0/16",
  "region": "us-east-1"
}
```
**Expected Response:** 400 Bad Request

#### API-014: VPC Name Too Long
**Endpoint:** POST /v1/vpcs  
**Test Data:**
```json
{
  "name": "a" * 256,
  "cidr_block": "10.0.0.0/16",
  "region": "us-east-1"
}
```
**Expected Response:** 400 Bad Request

#### API-015: Invalid Characters in Name
**Endpoint:** POST /v1/vpcs  
**Test Data:**
```json
{
  "name": "test@vpc#invalid",
  "cidr_block": "10.0.0.0/16",
  "region": "us-east-1"
}
```
**Expected Response:** 400 Bad Request

### Missing Required Fields

#### API-016: Missing Name Field
**Endpoint:** POST /v1/vpcs  
**Test Data:**
```json
{
  "cidr_block": "10.0.0.0/16",
  "region": "us-east-1"
}
```
**Expected Response:** 400 Bad Request

#### API-017: Missing CIDR Block
**Endpoint:** POST /v1/vpcs  
**Test Data:**
```json
{
  "name": "test-vpc",
  "region": "us-east-1"
}
```
**Expected Response:** 400 Bad Request

---

## 3. Authentication and Authorization Testing

#### API-018: Request Without Authentication
**Endpoint:** POST /v1/vpcs (no auth headers)  
**Expected Response:** 401 Unauthorized
**Expected Payload:**
```json
{
  "error": "Authentication required",
  "error_code": "AUTH_REQUIRED"
}
```

#### API-019: Request with Invalid Token
**Endpoint:** POST /v1/vpcs  
**Headers:** Authorization: Bearer invalid_token  
**Expected Response:** 401 Unauthorized

#### API-020: Request with Expired Token
**Endpoint:** POST /v1/vpcs  
**Headers:** Authorization: Bearer expired_token  
**Expected Response:** 401 Unauthorized

#### API-021: Insufficient Permissions
**Endpoint:** DELETE /v1/vpcs/{vpc_id} (using read-only token)  
**Expected Response:** 403 Forbidden
**Expected Payload:**
```json
{
  "error": "Insufficient permissions",
  "error_code": "INSUFFICIENT_PERMISSIONS"
}
```

---

## 4. Error Handling Testing

#### API-022: Overlapping CIDR Block
**Endpoint:** POST /v1/vpcs  
**Test Data:**
```json
{
  "name": "overlapping-vpc",
  "cidr_block": "10.0.0.0/16",
  "region": "us-east-1"
}
```
**Prerequisite:** VPC with 10.0.0.0/16 already exists in us-east-1  
**Expected Response:** 409 Conflict
**Expected Error:** "CIDR block overlaps with existing VPC"

#### API-023: Account Limit Exceeded
**Endpoint:** POST /v1/vpcs  
**Prerequisite:** 5 VPCs already exist in target region  
**Expected Response:** 429 Too Many Requests
**Expected Error:** "Account limit of 5 VPCs per region exceeded"

#### API-024: Invalid JSON Payload
**Endpoint:** POST /v1/vpcs  
**Test Data:** `{"name": "test", "cidr_block": }`  
**Expected Response:** 400 Bad Request
**Expected Error:** "Invalid JSON format"

#### API-025: Unsupported HTTP Method
**Endpoint:** PATCH /v1/vpcs  
**Expected Response:** 405 Method Not Allowed

#### API-026: Invalid Content-Type
**Endpoint:** POST /v1/vpcs  
**Headers:** Content-Type: text/plain  
**Expected Response:** 415 Unsupported Media Type

---

## 5. Rate Limiting Testing

#### API-027: Rate Limit Threshold Test
**Test Steps:**
1. Send 100 requests/minute to POST /v1/vpcs
2. Monitor response times and success rates
3. Verify rate limiting kicks in appropriately

**Expected Behavior:**
- Requests under limit: 201/200 responses
- Requests over limit: 429 Too Many Requests
- Rate limit headers present

#### API-028: Rate Limit Headers Validation
**Expected Headers:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1690454400
```

#### API-029: Rate Limit Recovery Test
**Test Steps:**
1. Trigger rate limit
2. Wait for reset period
3. Verify normal operation resumes

---

## 6. Concurrent Request Handling

#### API-030: Concurrent VPC Creation
**Test Steps:**
1. Send 10 simultaneous POST requests with different VPC names
2. Verify all succeed with unique IDs
3. Check for race conditions

**Expected Results:**
- All requests return 201 Created
- Each VPC gets unique ID
- No duplicate names/CIDRs created

#### API-031: Concurrent Overlapping CIDR Test
**Test Steps:**
1. Send 5 simultaneous POST requests with same CIDR block
2. Verify only one succeeds
3. Others return 409 Conflict

**Expected Results:**
- Exactly one 201 Created response
- Four 409 Conflict responses
- No data corruption

#### API-032: Read-Write Concurrency
**Test Steps:**
1. Start continuous GET /v1/vpcs requests
2. Simultaneously execute POST/PUT/DELETE operations
3. Verify read consistency

**Expected Results:**
- GET requests remain responsive
- No stale data returned
- Write operations complete successfully

---

########################################################################################################################

# Test Implementation Plan - VPC Management API

## Testing Tools/Framework Recommendation

**Primary Tool:** Postman
- Easy to use GUI interface
- Built-in test scripts
- Collection organization
- Environment variables

**For CI/CD:** Newman (Postman CLI)
```bash
newman run vpc-tests.json --environment staging.json
```

**Alternative:** curl + bash scripts for simple automation

## Test Data Setup Requirements

**Clean Environment:**
- Delete all existing VPCs before testing
- Have 3 test user accounts: admin, read-only, write-only

**Test Data:**
```json
{
  "valid_vpc": {
    "name": "test-vpc",
    "cidr_block": "10.0.0.0/16", 
    "region": "us-east-1"
  },
  "invalid_vpc": {
    "name": "",
    "cidr_block": "10.0.0.0/15",
    "region": "invalid-region"
  }
}
```

## Expected Response Codes and Payloads

**Success:**
- 200 OK: GET, PUT operations
- 201 Created: POST operations  
- 204 No Content: DELETE operations

**Errors:**
- 400: Bad request (validation errors)
- 401: No authentication
- 403: Insufficient permissions
- 404: VPC not found
- 409: CIDR overlap
- 429: Rate limit exceeded

**Sample Response:**
```json
{
  "id": "vpc-12345",
  "name": "test-vpc",
  "cidr_block": "10.0.0.0/16",
  "region": "us-east-1",
  "state": "available"
}
```

## Performance Baseline Suggestions

**Response Times:**
- GET requests: < 200ms
- POST requests: < 500ms
- PUT/DELETE: < 300ms

**Load Targets:**
- 100 requests/minute sustained
- 50 concurrent users
- Error rate < 1%

**Tools:** k6 or Apache Bench for basic load testing
