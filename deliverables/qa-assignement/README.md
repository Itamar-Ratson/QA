# Part 1: VPC Creation Test Cases

## Positive Scenarios

**TC-001: Create VPC with Valid Inputs**
- Pre-conditions: User logged in, <5 VPCs in region
- Steps: Enter name "test-vpc-01", CIDR "10.0.0.0/16", region "us-east-1", click Create
- Expected: VPC created successfully, appears in list
- Priority: High | Category: Functional

**TC-002: Create VPC with DNS Resolution**
- Steps: Enable DNS resolution during creation
- Expected: VPC created with DNS enabled
- Priority: High | Category: Functional

## Boundary Testing

**TC-003: Minimum CIDR Block (/28)**
- Steps: Create VPC with "10.0.0.0/28"
- Expected: Success
- Priority: High | Category: Functional

**TC-004: Maximum CIDR Block (/16)**
- Steps: Create VPC with "10.0.0.0/16"
- Expected: Success
- Priority: High | Category: Functional

**TC-005: Account Limit (5th VPC)**
- Pre-conditions: 4 existing VPCs
- Expected: 5th VPC creates successfully
- Priority: High | Category: Functional

## Negative Testing

**TC-006: Invalid CIDR Size**
- Steps: Try "10.0.0.0/29"
- Expected: Error "CIDR must be between /16 and /28"
- Priority: High | Category: Functional

**TC-007: Overlapping CIDR**
- Pre-conditions: Existing VPC with 10.0.0.0/16
- Steps: Create new VPC with "10.0.1.0/24"
- Expected: Error "CIDR overlaps"
- Priority: High | Category: Functional

**TC-008: Exceed Account Limit**
- Pre-conditions: 5 existing VPCs
- Expected: Error "Account limit exceeded"
- Priority: High | Category: Functional

## Edge Cases

**TC-009: Special Characters in Name**
- Steps: Name with valid chars "test-vpc_01"
- Expected: Success
- Priority: Medium | Category: Functional

**TC-010: Concurrent Creation**
- Steps: Two users create VPCs simultaneously
- Expected: Both succeed without conflicts
- Priority: High | Category: Integration

---

# Part 2: Bug Reports

## Bug VPC-001: Intermittent VPC Creation Failure

**Summary:** VPC creation with 10.0.0.0/16 fails randomly in us-east-1
**Severity:** HIGH | **Priority:** HIGH

**Steps to Reproduce:**
1. Create VPC with CIDR 10.0.0.0/16 in us-east-1
2. Observe intermittent "Creation failed - please try again" error

**Expected:** Consistent success or specific error message
**Actual:** Random failures, more frequent during peak hours

**Root Cause Analysis:**
- Likely race condition or database locks during peak load
- Missing detailed error logging

**Recommendations:**
- Add retry mechanism
- Implement detailed error messages
- Load test before production

## Bug VPC-002: DNS Resolution Not Working

**Summary:** DNS fails despite being enabled during VPC creation
**Severity:** CRITICAL | **Priority:** CRITICAL

**Steps to Reproduce:**
1. Create VPC with DNS resolution enabled in us-west-2
2. Launch instances
3. DNS queries fail (nslookup google.com)

**Root Cause Analysis:**
- DNS resolver service not properly configured
- Missing DHCP options or route table entries

**Recommendations:**
- Verify DNS resolver service status
- Add automated DNS validation tests

## Bug VPC-003: CIDR Validation Inconsistency

**Summary:** UI accepts 10.0.0.0/15 but backend rejects it
**Severity:** MEDIUM | **Priority:** MEDIUM

**Steps to Reproduce:**
1. Enter CIDR 10.0.0.0/15 in UI (no error)
2. Submit form
3. Backend validation fails

**Root Cause Analysis:**
- Frontend and backend validation logic mismatch
- Frontend allows /15 when only /16-/28 valid

**Recommendations:**
- Sync validation rules between frontend and backend
- Add comprehensive boundary tests

---

# Part 3: 3-Tier Application Testing

## Test Strategy

**Objective:** Validate 3-tier web app deployment with proper network isolation

**Scope:**
- VPC (10.0.0.0/16) in us-east-1
- 3 subnets: Public (ALB), Private (App), Database
- Security group isolation verification
- End-to-end connectivity testing

**Key Scenarios:**
1. Infrastructure provisioning
2. Network connectivity (Internet→ALB→App→DB)
3. Security isolation (no unauthorized access)
4. Basic load testing (50 concurrent users)

**Success Criteria:**
- All components deploy without errors
- Proper tier isolation enforced
- Response times <2 seconds
- No security breaches

## Test Plan

### Phase 1: VPC Setup
```bash
# Create VPC with Terraform
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  enable_dns_support = true
}
```
**Validation:** VPC created in <30 seconds

### Phase 2: Subnet Creation
- Public: 10.0.1.0/24 (with IGW)
- Private: 10.0.2.0/24 (with NAT)
- Database: 10.0.3.0/24 (isolated)
**Validation:** Subnets in different AZs

### Phase 3: Security Groups
- ALB: Allow 80 from internet
- App: Allow 8080 from ALB only
- DB: Allow 3306 from App only
**Validation:** Tier isolation working

### Phase 4: Application Deployment
- ALB in public subnet
- 2 EC2 instances in private subnet
- RDS MySQL in database subnet
**Validation:** Health checks pass

### Phase 5: Testing
```bash
# Test public access
curl http://$ALB_DNS
# Test load balancing
for i in {1..10}; do curl http://$ALB_DNS; done
# Verify isolation (should fail)
mysql -h $DB_ENDPOINT # from internet
```

**Testing Considerations:**
- Network connectivity between tiers
- Security isolation verification
- Performance baselines
- Integration with monitoring

---

# Part 4: API Testing

## Key Test Scenarios

### CRUD Operations

**POST /v1/vpcs - Create VPC**
```json
{
  "name": "test-vpc",
  "cidr_block": "10.0.0.0/16",
  "region": "us-east-1"
}
```
Expected: 201 Created

**GET /v1/vpcs - List VPCs**
Expected: 200 OK with VPC array

**GET /v1/vpcs/{id} - Get Specific VPC**
Expected: 200 OK or 404 Not Found

**DELETE /v1/vpcs/{id} - Delete VPC**
Expected: 204 No Content

### Input Validation
- Invalid CIDR: 400 Bad Request
- Missing fields: 400 Bad Request
- Name >255 chars: 400 Bad Request

### Authentication
- No token: 401 Unauthorized
- Invalid token: 401 Unauthorized
- Insufficient permissions: 403 Forbidden

### Error Handling
- Overlapping CIDR: 409 Conflict
- Account limit exceeded: 429 Too Many Requests
- Invalid JSON: 400 Bad Request

### Rate Limiting
- Test 100 requests/minute threshold
- Verify headers: X-RateLimit-Limit, X-RateLimit-Remaining
- Test recovery after limit

### Concurrent Requests
- 10 simultaneous creates with different names
- Verify no race conditions
- Test read-write consistency

## Implementation Plan

**Tools:** Postman for manual testing, Newman for CI/CD

**Test Data:**
```json
{
  "valid_vpc": {
    "name": "test-vpc",
    "cidr_block": "10.0.0.0/16",
    "region": "us-east-1"
  }
}
```

**Performance Baselines:**
- GET: <200ms
- POST: <500ms
- Error rate: <1%
- 50 concurrent users supported

**Test Environment Setup:**
- Clean staging environment with all existing VPCs deleted
- Test user accounts: admin (full access), read-only, write-only
- Valid authentication tokens for each permission level
- Base URL configured for staging API endpoint

**Test Execution Approach:**
1. **Setup Phase:** Configure Postman collections with environment variables
2. **Sequential Testing:** Run CRUD operations first, then validation tests
3. **Concurrent Testing:** Execute simultaneous requests using Newman
4. **Cleanup:** Reset environment state between test runs

**Expected Response Codes:**
- **Success:** 200 (GET/PUT), 201 (POST), 204 (DELETE)
- **Client Errors:** 400 (validation), 401 (auth), 403 (permissions), 404 (not found), 409 (conflict), 429 (rate limit)
- **Server Errors:** 500 (internal error), 503 (service unavailable)

**Sample Error Response:**
```json
{
  "error": "CIDR block must be between /16 and /28",
  "error_code": "INVALID_CIDR_RANGE",
  "timestamp": "2025-07-17T10:00:00Z"
}
```

**Load Testing Setup:**
- Use k6 or Apache Bench for performance testing
- Baseline: 100 requests/minute sustained load
- Stress test: Gradually increase to 200 requests/minute
- Monitor response times and error rates during load

**CI/CD Integration:**
```bash
# Run API tests in pipeline
newman run vpc-api-tests.json \
  --environment staging.json \
  --reporters cli,junit \
  --reporter-junit-export results.xml
```

**Test Reporting:**
- Pass/fail status for each test scenario
- Response time metrics and SLA compliance
- Error rate analysis and trending
- Security validation results (auth/authorization tests)
