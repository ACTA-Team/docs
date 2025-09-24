# Examples and Use Cases

## Overview

This section provides practical examples and real-world use cases for the ACTA API. These examples demonstrate how to integrate the API into various applications and scenarios for managing verifiable credentials on the Stellar blockchain.

## Basic Usage Examples

### 1. Academic Credentials

#### Creating a University Degree Credential

```javascript
const axios = require('axios');

const API_BASE_URL = 'https://acta.up.railway.app/v1';

async function createAcademicCredential() {
  try {
    const credentialData = {
      data: {
        holder: "GXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
        issuer: "University of Technology",
        credentialType: "AcademicDegree",
        subject: "Computer Science Degree",
        issuanceDate: "2024-06-15",
        claims: {
          degree: "Bachelor of Science",
          major: "Computer Science",
          gpa: "3.8",
          graduationYear: "2024",
          honors: "Magna Cum Laude",
          studentId: "CS2024001"
        }
      },
      metadata: {
        issuer: "University of Technology",
        subject: "John Doe",
        expirationDate: "2029-06-15T23:59:59Z",
        category: "education"
      }
    };

    const response = await axios.post(`${API_BASE_URL}/v1/credentials`, credentialData);
    
    // Academic credential created successfully
    // Contract ID: ${response.data.data.contractId}
    // Hash: ${response.data.data.hash}
    // Transaction Hash: ${response.data.data.transactionHash}
    
    return response.data.data;
  } catch (error) {
    // Handle error appropriately in your application
    throw new Error(`Error creating academic credential: ${error.response?.data?.message || error.message}`);
  }
}

// Usage
createAcademicCredential()
  .then(credential => {
    // Credential created successfully
    return credential;
  })
  .catch(error => {
    // Handle credential creation failure
    throw error;
  });
```

### 2. Professional Certifications

#### Creating a Professional Certificate

```javascript
async function createProfessionalCertification() {
  const certificationData = {
    data: {
      holder: "GXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
      issuer: "Professional Certification Board",
      credentialType: "ProfessionalCertification",
      subject: "AWS Solutions Architect",
      issuanceDate: "2024-03-20",
      claims: {
        certificationName: "AWS Certified Solutions Architect - Professional",
        certificationId: "AWS-SAP-2024-001",
        level: "Professional",
        validUntil: "2027-03-20",
        score: "850/1000",
        specializations: ["Cloud Architecture", "Security", "Cost Optimization"]
      }
    },
    metadata: {
      issuer: "Amazon Web Services",
      subject: "Jane Smith",
      expirationDate: "2027-03-20T23:59:59Z",
      category: "professional"
    }
  };

  try {
    const response = await axios.post(`${API_BASE_URL}/v1/credentials`, certificationData);
    return response.data.data;
  } catch (error) {
    // Handle error appropriately in your application
    throw new Error(`Error creating certification: ${error.response?.data?.message || error.message}`);
  }
}
```

### 3. Identity Verification

#### Creating a Digital Identity Credential

```javascript
async function createIdentityCredential() {
  const identityData = {
    data: {
      holder: "GXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
      issuer: "Government Identity Authority",
      credentialType: "DigitalIdentity",
      subject: "Digital Identity Verification",
      issuanceDate: "2024-01-15",
      claims: {
        fullName: "John Doe",
        dateOfBirth: "1990-05-15",
        nationality: "US",
        documentNumber: "ID123456789",
        verificationLevel: "Level 3",
        biometricHash: "sha256:abcd1234...",
        address: {
          street: "123 Main St",
          city: "New York",
          state: "NY",
          zipCode: "10001",
          country: "US"
        }
      }
    },
    metadata: {
      issuer: "US Government",
      subject: "John Doe",
      expirationDate: "2034-01-15T23:59:59Z",
      category: "identity"
    }
  };

  try {
    const response = await axios.post(`${API_BASE_URL}/v1/credentials`, identityData);
    return response.data.data;
  } catch (error) {
    console.error('Error creating identity credential:', error.response?.data || error.message);
    throw error;
  }
}
```

## Advanced Use Cases

### 1. Credential Verification System

#### Complete Verification Workflow

```javascript
class CredentialVerifier {
  constructor(apiBaseUrl) {
    this.apiBaseUrl = apiBaseUrl;
  }

  async verifyCredential(contractId) {
    try {
      // Step 1: Retrieve credential
      const response = await axios.get(`${this.apiBaseUrl}/v1/credentials/${contractId}`);
      const credential = response.data.data;

      // Step 2: Check status
      if (credential.status !== 'Active') {
        return {
          valid: false,
          reason: `Credential status is ${credential.status}`,
          credential
        };
      }

      // Step 3: Verify on blockchain (additional verification)
      const blockchainVerification = await this.verifyOnBlockchain(contractId);
      
      return {
        valid: blockchainVerification.valid,
        reason: blockchainVerification.reason,
        credential,
        verifiedAt: new Date().toISOString()
      };

    } catch (error) {
      if (error.response?.status === 404) {
        return {
          valid: false,
          reason: 'Credential not found',
          error: error.response.data
        };
      }
      throw error;
    }
  }

  async verifyOnBlockchain(contractId) {
    // This would implement additional blockchain verification
    // For now, we'll simulate the verification
    return {
      valid: true,
      reason: 'Credential verified on Stellar blockchain'
    };
  }

  async verifyByHash(hash) {
    try {
      const response = await axios.get(`${this.apiBaseUrl}/v1/credentials/hash/${hash}`);
      return await this.verifyCredential(response.data.data.contractId);
    } catch (error) {
      if (error.response?.status === 404) {
        return {
          valid: false,
          reason: 'Credential not found by hash'
        };
      }
      throw error;
    }
  }
}

// Usage
const verifier = new CredentialVerifier('https://acta.up.railway.app/v1');

async function verifyExample() {
  const contractId = 'CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ';
  
  const result = await verifier.verifyCredential(contractId);
  
  if (result.valid) {
    console.log('✅ Credential is valid');
    console.log('Verified at:', result.verifiedAt);
  } else {
    console.log('❌ Credential is invalid');
    console.log('Reason:', result.reason);
  }
}
```

### 2. Batch Credential Management

#### Creating Multiple Credentials

```javascript
class BatchCredentialManager {
  constructor(apiBaseUrl) {
    this.apiBaseUrl = apiBaseUrl;
  }

  async createBatchCredentials(credentialsData) {
    const results = [];
    const errors = [];

    for (let i = 0; i < credentialsData.length; i++) {
      try {
        console.log(`Creating credential ${i + 1}/${credentialsData.length}...`);
        
        const response = await axios.post(`${this.apiBaseUrl}/v1/credentials`, credentialsData[i]);
        results.push({
          index: i,
          success: true,
          data: response.data.data
        });

        // Add delay to avoid rate limiting
        await this.delay(1000);

      } catch (error) {
        errors.push({
          index: i,
          success: false,
          error: error.response?.data || error.message,
          data: credentialsData[i]
        });
      }
    }

    return {
      successful: results,
      failed: errors,
      summary: {
        total: credentialsData.length,
        successful: results.length,
        failed: errors.length
      }
    };
  }

  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Example: Creating multiple student credentials
async function createStudentBatch() {
  const students = [
    {
      name: "Alice Johnson",
      studentId: "STU001",
      degree: "Computer Science",
      gpa: "3.9"
    },
    {
      name: "Bob Smith",
      studentId: "STU002", 
      degree: "Mathematics",
      gpa: "3.7"
    },
    {
      name: "Carol Davis",
      studentId: "STU003",
      degree: "Physics",
      gpa: "3.8"
    }
  ];

  const credentialsData = students.map(student => ({
    data: {
      holder: "GXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX", // Would be unique per student
      issuer: "University of Technology",
      credentialType: "AcademicDegree",
      subject: `${student.degree} Degree`,
      issuanceDate: "2024-06-15",
      claims: {
        degree: "Bachelor of Science",
        major: student.degree,
        gpa: student.gpa,
        graduationYear: "2024",
        studentId: student.studentId
      }
    },
    metadata: {
      issuer: "University of Technology",
      subject: student.name,
      expirationDate: "2029-06-15T23:59:59Z",
      category: "education"
    }
  }));

  const manager = new BatchCredentialManager('https://acta.up.railway.app/v1');
  const results = await manager.createBatchCredentials(credentialsData);

  console.log('Batch creation results:');
  console.log(`✅ Successful: ${results.summary.successful}`);
  console.log(`❌ Failed: ${results.summary.failed}`);
  
  if (results.failed.length > 0) {
    console.log('Failed credentials:');
    results.failed.forEach(failure => {
      console.log(`- Index ${failure.index}: ${failure.error}`);
    });
  }

  return results;
}
```

### 3. Credential Status Management

#### Automated Status Updates

```javascript
class CredentialStatusManager {
  constructor(apiBaseUrl) {
    this.apiBaseUrl = apiBaseUrl;
  }

  async updateStatus(contractId, newStatus, reason = '') {
    try {
      const response = await axios.patch(
        `${this.apiBaseUrl}/v1/credentials/${contractId}/status`,
        { status: newStatus }
      );

      console.log(`✅ Credential ${contractId} status updated to ${newStatus}`);
      
      // Log the status change
      await this.logStatusChange(contractId, newStatus, reason);
      
      return response.data;
    } catch (error) {
      console.error(`❌ Failed to update credential ${contractId}:`, error.response?.data || error.message);
      throw error;
    }
  }

  async revokeCredential(contractId, reason) {
    return await this.updateStatus(contractId, 'Revoked', reason);
  }

  async suspendCredential(contractId, reason) {
    return await this.updateStatus(contractId, 'Suspended', reason);
  }

  async reactivateCredential(contractId, reason) {
    return await this.updateStatus(contractId, 'Active', reason);
  }

  async logStatusChange(contractId, status, reason) {
    const logEntry = {
      contractId,
      status,
      reason,
      timestamp: new Date().toISOString(),
      action: 'status_change'
    };

    // In a real application, this would write to a database or logging service
    console.log('Status change logged:', logEntry);
  }

  async bulkStatusUpdate(updates) {
    const results = [];
    
    for (const update of updates) {
      try {
        await this.updateStatus(update.contractId, update.status, update.reason);
        results.push({
          contractId: update.contractId,
          success: true,
          status: update.status
        });
      } catch (error) {
        results.push({
          contractId: update.contractId,
          success: false,
          error: error.message
        });
      }
    }

    return results;
  }
}

// Example usage
async function manageCredentialStatuses() {
  const manager = new CredentialStatusManager('https://acta.up.railway.app/v1');

  // Revoke a credential
  await manager.revokeCredential(
    'CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ',
    'Student expelled for academic misconduct'
  );

  // Suspend multiple credentials
  const suspensions = [
    {
      contractId: 'CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ',
      status: 'Suspended',
      reason: 'Under investigation'
    },
    {
      contractId: 'CB3J7CBYOH8FIT5EG4KGYPRL4MTO7KVMOWK4HNIXURBYJY6XXQ3WBFVJR',
      status: 'Suspended', 
      reason: 'Pending verification'
    }
  ];

  const results = await manager.bulkStatusUpdate(suspensions);
  console.log('Bulk update results:', results);
}
```

## Integration Examples

### 1. Express.js Middleware

#### Credential Verification Middleware

```javascript
const axios = require('axios');

function createCredentialVerificationMiddleware(apiBaseUrl) {
  return async (req, res, next) => {
    try {
      const contractId = req.headers['x-credential-id'];
      
      if (!contractId) {
        return res.status(401).json({
          error: 'Missing credential ID in headers'
        });
      }

      // Verify the credential
      const response = await axios.get(`${apiBaseUrl}/v1/credentials/${contractId}`);
      const credential = response.data.data;

      if (credential.status !== 'Active') {
        return res.status(403).json({
          error: 'Credential is not active',
          status: credential.status
        });
      }

      // Add credential info to request
      req.credential = credential;
      next();

    } catch (error) {
      if (error.response?.status === 404) {
        return res.status(404).json({
          error: 'Credential not found'
        });
      }

      console.error('Credential verification error:', error);
      return res.status(500).json({
        error: 'Failed to verify credential'
      });
    }
  };
}

// Usage in Express app
const express = require('express');
const app = express();

const verifyCredential = createCredentialVerificationMiddleware('https://acta.up.railway.app/v1');

app.get('/protected-resource', verifyCredential, (req, res) => {
  res.json({
    message: 'Access granted',
    credential: req.credential
  });
});
```

### 2. React Frontend Integration

#### Credential Display Component

```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const CredentialViewer = ({ contractId }) => {
  const [credential, setCredential] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchCredential = async () => {
      try {
        setLoading(true);
        const response = await axios.get(`/api/v1/credentials/${contractId}`);
        setCredential(response.data.data);
      } catch (err) {
        setError(err.response?.data?.error || 'Failed to fetch credential');
      } finally {
        setLoading(false);
      }
    };

    if (contractId) {
      fetchCredential();
    }
  }, [contractId]);

  if (loading) return <div className="loading">Loading credential...</div>;
  if (error) return <div className="error">Error: {error}</div>;
  if (!credential) return <div>No credential found</div>;

  const getStatusColor = (status) => {
    switch (status) {
      case 'Active': return 'green';
      case 'Revoked': return 'red';
      case 'Suspended': return 'orange';
      default: return 'gray';
    }
  };

  return (
    <div className="credential-card">
      <div className="credential-header">
        <h3>Verifiable Credential</h3>
        <span 
          className="status-badge" 
          style={{ backgroundColor: getStatusColor(credential.status) }}
        >
          {credential.status}
        </span>
      </div>
      
      <div className="credential-details">
        <div className="detail-row">
          <strong>Contract ID:</strong>
          <code>{credential.contractId}</code>
        </div>
        <div className="detail-row">
          <strong>Hash:</strong>
          <code>{credential.hash}</code>
        </div>
        <div className="detail-row">
          <strong>Created:</strong>
          {new Date(credential.createdAt).toLocaleDateString()}
        </div>
      </div>

      <div className="credential-actions">
        <button onClick={() => window.open(`https://stellar.expert/explorer/testnet/contract/${credential.contractId}`)}>
          View on Stellar
        </button>
      </div>
    </div>
  );
};

// Credential Creation Form
const CredentialForm = ({ onCredentialCreated }) => {
  const [formData, setFormData] = useState({
    holder: '',
    credentialType: '',
    subject: '',
    claims: {}
  });
  const [submitting, setSubmitting] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setSubmitting(true);

    try {
      const response = await axios.post('/api/v1/credentials', {
        data: formData
      });

      onCredentialCreated(response.data.data);
      setFormData({ holder: '', credentialType: '', subject: '', claims: {} });
    } catch (error) {
      alert('Failed to create credential: ' + (error.response?.data?.error || error.message));
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="credential-form">
      <div className="form-group">
        <label>Holder Address:</label>
        <input
          type="text"
          value={formData.holder}
          onChange={(e) => setFormData({...formData, holder: e.target.value})}
          placeholder="GXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
          required
        />
      </div>

      <div className="form-group">
        <label>Credential Type:</label>
        <select
          value={formData.credentialType}
          onChange={(e) => setFormData({...formData, credentialType: e.target.value})}
          required
        >
          <option value="">Select type...</option>
          <option value="AcademicDegree">Academic Degree</option>
          <option value="ProfessionalCertification">Professional Certification</option>
          <option value="DigitalIdentity">Digital Identity</option>
        </select>
      </div>

      <div className="form-group">
        <label>Subject:</label>
        <input
          type="text"
          value={formData.subject}
          onChange={(e) => setFormData({...formData, subject: e.target.value})}
          placeholder="e.g., Computer Science Degree"
          required
        />
      </div>

      <button type="submit" disabled={submitting}>
        {submitting ? 'Creating...' : 'Create Credential'}
      </button>
    </form>
  );
};

export { CredentialViewer, CredentialForm };
```

### 3. Python Integration

#### Python SDK Example

```python
import requests
import json
import hashlib
from datetime import datetime, timezone
from typing import Dict, Any, Optional

class ACTAClient:
    def __init__(self, base_url: str = "https://acta.up.railway.app/v1"):
        self.base_url = base_url.rstrip('/')
        self.session = requests.Session()
        self.session.headers.update({
            'Content-Type': 'application/json'
        })

    def create_credential(self, data: Dict[str, Any], metadata: Optional[Dict[str, Any]] = None) -> Dict[str, Any]:
        """Create a new verifiable credential."""
        payload = {"data": data}
        if metadata:
            payload["metadata"] = metadata

        response = self.session.post(f"{self.base_url}/v1/credentials", json=payload)
        response.raise_for_status()
        return response.json()

    def get_credential(self, contract_id: str) -> Dict[str, Any]:
        """Get credential by contract ID."""
        response = self.session.get(f"{self.base_url}/v1/credentials/{contract_id}")
        response.raise_for_status()
        return response.json()

    def get_credential_by_hash(self, hash_value: str) -> Dict[str, Any]:
        """Get credential by hash."""
        response = self.session.get(f"{self.base_url}/v1/credentials/hash/{hash_value}")
        response.raise_for_status()
        return response.json()

    def update_credential_status(self, contract_id: str, status: str) -> Dict[str, Any]:
        """Update credential status."""
        payload = {"status": status}
        response = self.session.patch(f"{self.base_url}/v1/credentials/{contract_id}/status", json=payload)
        response.raise_for_status()
        return response.json()

    def verify_credential(self, contract_id: str) -> Dict[str, Any]:
        """Verify a credential's validity."""
        try:
            credential = self.get_credential(contract_id)
            
            return {
                "valid": credential["data"]["status"] == "Active",
                "status": credential["data"]["status"],
                "contract_id": contract_id,
                "verified_at": datetime.now(timezone.utc).isoformat()
            }
        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 404:
                return {
                    "valid": False,
                    "error": "Credential not found",
                    "contract_id": contract_id
                }
            raise

# Example usage
def main():
    client = ACTAClient()

    # Create an academic credential
    academic_data = {
        "holder": "GXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
        "issuer": "Python University",
        "credentialType": "AcademicDegree",
        "subject": "Data Science Degree",
        "issuanceDate": "2024-01-15",
        "claims": {
            "degree": "Master of Science",
            "major": "Data Science",
            "gpa": "3.9",
            "graduationYear": "2024"
        }
    }

    metadata = {
        "issuer": "Python University",
        "subject": "Alice Python",
        "expirationDate": "2029-01-15T23:59:59Z",
        "category": "education"
    }

    try:
        # Create credential
        result = client.create_credential(academic_data, metadata)
        print(f"✅ Credential created: {result['data']['contractId']}")

        # Verify credential
        verification = client.verify_credential(result['data']['contractId'])
        print(f"✅ Credential valid: {verification['valid']}")

        # Update status
        client.update_credential_status(result['data']['contractId'], "Suspended")
        print("✅ Credential suspended")

        # Verify again
        verification = client.verify_credential(result['data']['contractId'])
        print(f"✅ Credential valid after suspension: {verification['valid']}")

    except requests.exceptions.RequestException as e:
        print(f"❌ Error: {e}")

if __name__ == "__main__":
    main()
```

## Real-World Scenarios

### 1. University Degree Verification System

A complete system for managing and verifying university degrees:

```javascript
class UniversityCredentialSystem {
  constructor(apiBaseUrl, universityId) {
    this.apiBaseUrl = apiBaseUrl;
    this.universityId = universityId;
  }

  async issueDegreeCertificate(studentData) {
    const credentialData = {
      data: {
        holder: studentData.stellarAddress,
        issuer: this.universityId,
        credentialType: "AcademicDegree",
        subject: `${studentData.degree} in ${studentData.major}`,
        issuanceDate: new Date().toISOString().split('T')[0],
        claims: {
          degree: studentData.degree,
          major: studentData.major,
          gpa: studentData.gpa,
          graduationYear: studentData.graduationYear,
          studentId: studentData.studentId,
          honors: studentData.honors,
          thesis: studentData.thesis
        }
      },
      metadata: {
        issuer: "University of Technology",
        subject: studentData.fullName,
        expirationDate: null, // Degrees don't expire
        category: "education"
      }
    };

    const response = await axios.post(`${this.apiBaseUrl}/v1/credentials`, credentialData);
    
    // Store in university database
    await this.storeInUniversityDatabase(studentData.studentId, response.data.data);
    
    return response.data.data;
  }

  async verifyDegreeForEmployer(contractId, employerRequest) {
    try {
      const credential = await axios.get(`${this.apiBaseUrl}/v1/credentials/${contractId}`);
      
      return {
        verified: credential.data.data.status === 'Active',
        degree: credential.data.data.claims?.degree,
        major: credential.data.data.claims?.major,
        graduationYear: credential.data.data.claims?.graduationYear,
        gpa: employerRequest.includeGPA ? credential.data.data.claims?.gpa : null,
        verificationDate: new Date().toISOString()
      };
    } catch (error) {
      return {
        verified: false,
        error: 'Credential not found or invalid'
      };
    }
  }

  async storeInUniversityDatabase(studentId, credentialData) {
    // Implementation would store in university's database
    console.log(`Stored credential for student ${studentId}:`, credentialData.contractId);
  }
}
```

### 2. Professional Certification Tracking

```javascript
class CertificationTracker {
  constructor(apiBaseUrl) {
    this.apiBaseUrl = apiBaseUrl;
  }

  async trackCertificationExpiry() {
    // This would typically query a database of issued certifications
    const certifications = await this.getCertificationsNearExpiry();
    
    for (const cert of certifications) {
      if (this.isExpired(cert.expirationDate)) {
        await this.expireCertification(cert.contractId);
      } else if (this.isNearExpiry(cert.expirationDate)) {
        await this.notifyRenewal(cert);
      }
    }
  }

  async expireCertification(contractId) {
    await axios.patch(`${this.apiBaseUrl}/v1/credentials/${contractId}/status`, {
      status: 'Revoked'
    });
    console.log(`Certification ${contractId} expired and revoked`);
  }

  isExpired(expirationDate) {
    return new Date(expirationDate) < new Date();
  }

  isNearExpiry(expirationDate, daysThreshold = 30) {
    const expiry = new Date(expirationDate);
    const threshold = new Date();
    threshold.setDate(threshold.getDate() + daysThreshold);
    return expiry <= threshold;
  }

  async getCertificationsNearExpiry() {
    // Mock data - in real implementation, this would query a database
    return [
      {
        contractId: 'CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ',
        expirationDate: '2024-02-15T23:59:59Z',
        holderEmail: 'john@example.com'
      }
    ];
  }

  async notifyRenewal(certification) {
    console.log(`Sending renewal notification for ${certification.contractId}`);
    // Implementation would send email/notification
  }
}
```

These examples demonstrate the versatility and power of the ACTA API for managing verifiable credentials across various industries and use cases. The API's blockchain-based approach ensures security, immutability, and global verifiability of credentials.

---

*Next: Learn about [Deployment and Production Setup](./08-deployment.md) to deploy your API to production environments.*