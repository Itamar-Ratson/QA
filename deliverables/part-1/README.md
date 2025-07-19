# VPC Creation - Comprehensive Test Cases

##################################################
## Positive/Happy Path Scenarios

### TC-001
**Test Case ID:** TC-001  
**Test Case Description:** Create VPC with valid inputs and default settings  
**Pre-conditions:** User logged in, less than 5 VPCs in selected region  
**Test Steps:**
1. Navigate to VPC creation form
2. Enter VPC name: "test-vpc-01"
3. Enter CIDR block: "10.0.0.0/16"
4. Select region: "us-east-1"
5. Leave DNS options as default
6. Click "Create VPC"

**Expected Results:** VPC created successfully, confirmation message displayed, VPC appears in VPC list  
**Priority:** High  
**Category:** Functional

### TC-002
**Test Case ID:** TC-002  
**Test Case Description:** Create VPC with DNS resolution enabled  
**Pre-conditions:** User logged in, less than 5 VPCs in selected region  
**Test Steps:**
1. Navigate to VPC creation form
2. Enter VPC name: "dns-enabled-vpc"
3. Enter CIDR block: "172.16.0.0/16"
4. Select region: "us-west-2"
5. Enable "DNS resolution"
6. Click "Create VPC"

**Expected Results:** VPC created with DNS resolution enabled, instances can resolve domain names  
**Priority:** High  
**Category:** Functional

### TC-003
**Test Case ID:** TC-003  
**Test Case Description:** Create VPC with different private IP range (172.x.x.x)  
**Pre-conditions:** User logged in, less than 5 VPCs in selected region  
**Test Steps:**
1. Navigate to VPC creation form
2. Enter VPC name: "private-range-vpc"
3. Enter CIDR block: "172.16.0.0/16"
4. Select region: "us-west-1"
5. Leave DNS options as default
6. Click "Create VPC"

**Expected Results:** VPC created successfully with 172.16.0.0/16 private IP range  
**Priority:** High  
**Category:** Functional

##################################################
## Boundary Value Testing

### TC-004
**Test Case ID:** TC-004  
**Test Case Description:** Create VPC with minimum CIDR block size (/28)  
**Pre-conditions:** User logged in, less than 5 VPCs in selected region  
**Test Steps:**
1. Navigate to VPC creation form
2. Enter VPC name: "min-cidr-vpc"
3. Enter CIDR block: "10.0.0.0/28"
4. Select region: "us-east-1"
5. Click "Create VPC"

**Expected Results:** VPC created successfully with /28 CIDR block  
**Priority:** High  
**Category:** Functional

### TC-005
**Test Case ID:** TC-005  
**Test Case Description:** Create VPC with maximum CIDR block size (/16)  
**Pre-conditions:** User logged in, less than 5 VPCs in selected region  
**Test Steps:**
1. Navigate to VPC creation form
2. Enter VPC name: "max-cidr-vpc"
3. Enter CIDR block: "10.0.0.0/16"
4. Select region: "us-east-1"
5. Click "Create VPC"

**Expected Results:** VPC created successfully with /16 CIDR block  
**Priority:** High  
**Category:** Functional

### TC-006
**Test Case ID:** TC-006  
**Test Case Description:** Create 5th VPC in region (account limit)  
**Pre-conditions:** User logged in, 4 VPCs already exist in target region  
**Test Steps:**
1. Navigate to VPC creation form
2. Enter VPC name: "fifth-vpc"
3. Enter CIDR block: "10.5.0.0/16"
4. Select region with 4 existing VPCs
5. Click "Create VPC"

**Expected Results:** 5th VPC created successfully, reaching account limit  
**Priority:** High  
**Category:** Functional

##################################################
## Negative Testing Scenarios

### TC-007
**Test Case ID:** TC-007  
**Test Case Description:** Attempt to create VPC with CIDR block smaller than /28  
**Pre-conditions:** User logged in, less than 5 VPCs in selected region  
**Test Steps:**
1. Navigate to VPC creation form
2. Enter VPC name: "invalid-cidr-vpc"
3. Enter CIDR block: "10.0.0.0/29"
4. Select region: "us-east-1"
5. Click "Create VPC"

**Expected Results:** Error message displayed: "CIDR block must be between /16 and /28"  
**Priority:** High  
**Category:** Functional

### TC-008
**Test Case ID:** TC-008  
**Test Case Description:** Attempt to create VPC with overlapping CIDR block  
**Pre-conditions:** User logged in, existing VPC with CIDR 10.0.0.0/16 in region  
**Test Steps:**
1. Navigate to VPC creation form
2. Enter VPC name: "overlapping-vpc"
3. Enter CIDR block: "10.0.1.0/24"
4. Select same region as existing VPC
5. Click "Create VPC"

**Expected Results:** Error message displayed: "CIDR block overlaps with existing VPC"  
**Priority:** High  
**Category:** Functional

### TC-009
**Test Case ID:** TC-009  
**Test Case Description:** Attempt to create 6th VPC in region (exceed limit)  
**Pre-conditions:** User logged in, 5 VPCs already exist in target region  
**Test Steps:**
1. Navigate to VPC creation form
2. Enter VPC name: "sixth-vpc"
3. Enter CIDR block: "10.6.0.0/16"
4. Select region with 5 existing VPCs
5. Click "Create VPC"

**Expected Results:** Error message displayed: "Account limit of 5 VPCs per region exceeded"  
**Priority:** High  
**Category:** Functional

##################################################
## Edge Cases

### TC-010
**Test Case ID:** TC-010  
**Test Case Description:** Create VPC with network interruption during creation  
**Pre-conditions:** User logged in, less than 5 VPCs in selected region  
**Test Steps:**
1. Navigate to VPC creation form
2. Enter valid VPC details
3. Click "Create VPC"
4. Simulate network interruption during creation
5. Restore network connection

**Expected Results:** System handles interruption gracefully, either completes creation or shows appropriate error with retry option  
**Priority:** Medium  
**Category:** Functional

### TC-011
**Test Case ID:** TC-011  
**Test Case Description:** Create VPC with special characters in name (valid ones)  
**Pre-conditions:** User logged in, less than 5 VPCs in selected region  
**Test Steps:**
1. Navigate to VPC creation form
2. Enter VPC name: "test-vpc_with-valid_chars"
3. Enter CIDR block: "10.0.0.0/16"
4. Select region: "us-east-1"
5. Click "Create VPC"

**Expected Results:** VPC created successfully with hyphens and underscores in name  
**Priority:** Medium  
**Category:** Functional

### TC-012
**Test Case ID:** TC-012  
**Test Case Description:** Create VPC with all three private IP ranges  
**Pre-conditions:** User logged in, less than 5 VPCs in selected region  
**Test Steps:**
1. Create VPC with CIDR: "10.0.0.0/16"
2. Create VPC with CIDR: "172.16.0.0/16"
3. Create VPC with CIDR: "192.168.0.0/16"
4. All in same region

**Expected Results:** All three VPCs created successfully using different private IP ranges  
**Priority:** Medium  
**Category:** Functional

##################################################
## Integration Scenarios

### TC-013
**Test Case ID:** TC-013  
**Test Case Description:** Concurrent VPC creation by multiple users  
**Pre-conditions:** Multiple users logged in, less than 5 VPCs in selected region  
**Test Steps:**
1. User A starts VPC creation with CIDR: "10.1.0.0/16"
2. User B simultaneously starts VPC creation with CIDR: "10.2.0.0/16"
3. Both users submit forms at same time
4. Monitor creation process

**Expected Results:** Both VPCs created successfully without conflicts  
**Priority:** High  
**Category:** Integration

### TC-014
**Test Case ID:** TC-014  
**Test Case Description:** Create VPC across different regions  
**Pre-conditions:** User logged in, access to multiple regions  
**Test Steps:**
1. Create VPC in us-east-1 with CIDR: "10.0.0.0/16"
2. Create VPC in us-west-2 with CIDR: "10.0.0.0/16"
3. Verify both VPCs exist independently

**Expected Results:** Same CIDR blocks allowed in different regions  
**Priority:** Medium  
**Category:** Integration

### TC-015
**Test Case ID:** TC-015  
**Test Case Description:** VPC creation with subnet and security group integration  
**Pre-conditions:** User logged in, less than 5 VPCs in selected region  
**Test Steps:**
1. Create VPC with CIDR: "10.0.0.0/16"
2. Create subnet within VPC with CIDR: "10.0.1.0/24"
3. Create security group associated with VPC
4. Verify all components reference the same VPC ID

**Expected Results:** Subnet and security group successfully created and associated with VPC  
**Priority:** High  
**Category:** Integration
