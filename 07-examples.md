# Examples and Use Cases

## Overview

This section provides practical examples and real-world use cases for the ACTA API. Each example includes complete code samples, explanations, and best practices for implementation.

---

## **Basic Credential Operations**

### **Creating a Simple Credential**

Create a basic educational credential:

```javascript
// POST /api/credentials
const credentialData = {
  recipient: "GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  issuer: "GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  credentialType: "diploma",
  metadata: {
    institution: "University of Technology",
    degree: "Bachelor of Computer Science",
    graduationDate: "2024-05-15",
    gpa: "3.8"
  }
};

const response = await fetch('https://api.acta.build/credentials', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(credentialData)
});

const result = await response.json();
console.log('Credential created:', result);
```

**Expected Response:**
```json
{
  "success": true,
  "data": {
    "id": "cred_abc123def456",
    "transactionHash": "a1b2c3d4e5f6...",
    "stellarAccount": "GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "credentialHash": "sha256:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08",
    "timestamp": "2024-01-15T10:30:00Z",
    "status": "confirmed"
  }
}
```

### **Retrieving a Credential**

Get credential details by ID:

```javascript
// GET /api/credentials/:id
const credentialId = "cred_abc123def456";

const response = await fetch(`https://api.acta.build/credentials/${credentialId}`);
const credential = await response.json();

console.log('Credential details:', credential);
```

**Expected Response:**
```json
{
  "success": true,
  "data": {
    "id": "cred_abc123def456",
    "recipient": "GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "issuer": "GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "credentialType": "diploma",
    "metadata": {
      "institution": "University of Technology",
      "degree": "Bachelor of Computer Science",
      "graduationDate": "2024-05-15",
      "gpa": "3.8"
    },
    "transactionHash": "a1b2c3d4e5f6...",
    "stellarAccount": "GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "credentialHash": "sha256:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08",
    "timestamp": "2024-01-15T10:30:00Z",
    "status": "confirmed",
    "isValid": true
  }
}
```

---

## **Advanced Use Cases**

### **Professional Certification System**

Complete workflow for managing professional certifications:

```javascript
class CertificationManager {
  constructor(apiBaseUrl) {
    this.apiBaseUrl = apiBaseUrl;
  }

  // Issue a professional certification
  async issueCertification(certificationData) {
    const credential = {
      recipient: certificationData.professionalId,
      issuer: certificationData.organizationId,
      credentialType: "certification",
      metadata: {
        certificationName: certificationData.name,
        certifyingOrganization: certificationData.organization,
        issueDate: new Date().toISOString(),
        expiryDate: certificationData.expiryDate,
        certificationLevel: certificationData.level,
        skills: certificationData.skills,
        verificationCode: this.generateVerificationCode()
      }
    };

    const response = await fetch(`${this.apiBaseUrl}/api/credentials`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credential)
    });

    return await response.json();
  }

  // Verify certification validity
  async verifyCertification(credentialId) {
    const response = await fetch(`${this.apiBaseUrl}/api/credentials/${credentialId}/verify`);
    const verification = await response.json();

    if (verification.success && verification.data.isValid) {
      return {
        valid: true,
        certification: verification.data,
        verifiedAt: new Date().toISOString()
      };
    }

    return { valid: false, reason: verification.error || 'Invalid certification' };
  }

  // List all certifications for a professional
  async getProfessionalCertifications(professionalId) {
    const response = await fetch(
      `${this.apiBaseUrl}/api/credentials?recipient=${professionalId}&type=certification`
    );
    return await response.json();
  }

  generateVerificationCode() {
    return Math.random().toString(36).substring(2, 15) + 
           Math.random().toString(36).substring(2, 15);
  }
}

// Usage example
const certManager = new CertificationManager('https://api.acta.build');

// Issue AWS Solutions Architect certification
const awsCertification = await certManager.issueCertification({
  professionalId: "GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  organizationId: "GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  name: "AWS Certified Solutions Architect",
  organization: "Amazon Web Services",
  level: "Associate",
  expiryDate: "2027-01-15T00:00:00Z",
  skills: ["Cloud Architecture", "AWS Services", "Security", "Scalability"]
});

console.log('Certification issued:', awsCertification);
```

### **Academic Transcript System**

Manage complete academic transcripts:

```javascript
class TranscriptManager {
  constructor(apiBaseUrl) {
    this.apiBaseUrl = apiBaseUrl;
  }

  // Create a complete academic transcript
  async createTranscript(studentData, courses) {
    const transcriptCredential = {
      recipient: studentData.stellarId,
      issuer: studentData.institutionId,
      credentialType: "transcript",
      metadata: {
        studentInfo: {
          name: studentData.name,
          studentId: studentData.id,
          program: studentData.program,
          admissionDate: studentData.admissionDate,
          graduationDate: studentData.graduationDate
        },
        institution: {
          name: studentData.institutionName,
          address: studentData.institutionAddress,
          accreditation: studentData.accreditation
        },
        academicRecord: {
          courses: courses,
          totalCredits: courses.reduce((sum, course) => sum + course.credits, 0),
          gpa: this.calculateGPA(courses),
          honors: studentData.honors || []
        },
        issueDate: new Date().toISOString(),
        transcriptId: this.generateTranscriptId()
      }
    };

    const response = await fetch(`${this.apiBaseUrl}/api/credentials`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(transcriptCredential)
    });

    return await response.json();
  }

  // Add a single course to existing transcript
  async addCourseToTranscript(transcriptId, courseData) {
    // First, get the existing transcript
    const existingResponse = await fetch(`${this.apiBaseUrl}/api/credentials/${transcriptId}`);
    const existing = await existingResponse.json();

    if (!existing.success) {
      throw new Error('Transcript not found');
    }

    // Update the transcript with new course
    const updatedCourses = [...existing.data.metadata.academicRecord.courses, courseData];
    const updatedMetadata = {
      ...existing.data.metadata,
      academicRecord: {
        ...existing.data.metadata.academicRecord,
        courses: updatedCourses,
        totalCredits: updatedCourses.reduce((sum, course) => sum + course.credits, 0),
        gpa: this.calculateGPA(updatedCourses)
      }
    };

    // Create new credential with updated data
    const updatedCredential = {
      recipient: existing.data.recipient,
      issuer: existing.data.issuer,
      credentialType: "transcript",
      metadata: updatedMetadata
    };

    const response = await fetch(`${this.apiBaseUrl}/api/credentials`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(updatedCredential)
    });

    return await response.json();
  }

  calculateGPA(courses) {
    const gradePoints = {
      'A+': 4.0, 'A': 4.0, 'A-': 3.7,
      'B+': 3.3, 'B': 3.0, 'B-': 2.7,
      'C+': 2.3, 'C': 2.0, 'C-': 1.7,
      'D+': 1.3, 'D': 1.0, 'F': 0.0
    };

    let totalPoints = 0;
    let totalCredits = 0;

    courses.forEach(course => {
      const points = gradePoints[course.grade] || 0;
      totalPoints += points * course.credits;
      totalCredits += course.credits;
    });

    return totalCredits > 0 ? (totalPoints / totalCredits).toFixed(2) : '0.00';
  }

  generateTranscriptId() {
    return 'TRANS-' + Date.now() + '-' + Math.random().toString(36).substring(2, 8).toUpperCase();
  }
}

// Usage example
const transcriptManager = new TranscriptManager('https://api.acta.build');

const studentData = {
  stellarId: "GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  institutionId: "GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  name: "Jane Smith",
  id: "STU2024001",
  program: "Bachelor of Science in Computer Science",
  admissionDate: "2020-09-01",
  graduationDate: "2024-05-15",
  institutionName: "Tech University",
  institutionAddress: "123 University Ave, Tech City, TC 12345",
  accreditation: "ABET Accredited",
  honors: ["Magna Cum Laude", "Dean's List"]
};

const courses = [
  {
    courseCode: "CS101",
    courseName: "Introduction to Programming",
    credits: 3,
    grade: "A",
    semester: "Fall 2020"
  },
  {
    courseCode: "CS201",
    courseName: "Data Structures",
    credits: 4,
    grade: "A-",
    semester: "Spring 2021"
  },
  {
    courseCode: "CS301",
    courseName: "Algorithms",
    credits: 4,
    grade: "B+",
    semester: "Fall 2021"
  }
];

const transcript = await transcriptManager.createTranscript(studentData, courses);
console.log('Transcript created:', transcript);
```

---

## **Integration Examples**

### **React Frontend Integration**

Complete React component for credential management:

```jsx
import React, { useState, useEffect } from 'react';

const CredentialManager = () => {
  const [credentials, setCredentials] = useState([]);
  const [loading, setLoading] = useState(false);
  const [newCredential, setNewCredential] = useState({
    recipient: '',
    credentialType: 'certificate',
    metadata: {}
  });

  const API_BASE_URL = 'https://api.acta.build';

  // Fetch all credentials
  const fetchCredentials = async () => {
    setLoading(true);
    try {
      const response = await fetch(`${API_BASE_URL}/credentials`);
      const data = await response.json();
      if (data.success) {
        setCredentials(data.data);
      }
    } catch (error) {
      console.error('Error fetching credentials:', error);
    } finally {
      setLoading(false);
    }
  };

  // Create new credential
  const createCredential = async (e) => {
    e.preventDefault();
    setLoading(true);

    try {
      const response = await fetch(`${API_BASE_URL}/credentials`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(newCredential),
      });

      const data = await response.json();
      if (data.success) {
        setCredentials([...credentials, data.data]);
        setNewCredential({
          recipient: '',
          credentialType: 'certificate',
          metadata: {}
        });
        alert('Credential created successfully!');
      } else {
        alert('Error creating credential: ' + data.error);
      }
    } catch (error) {
      console.error('Error creating credential:', error);
      alert('Error creating credential');
    } finally {
      setLoading(false);
    }
  };

  // Verify credential
  const verifyCredential = async (credentialId) => {
    try {
      const response = await fetch(`${API_BASE_URL}/credentials/${credentialId}/verify`);
      const data = await response.json();
      
      if (data.success && data.data.isValid) {
        alert('Credential is valid!');
      } else {
        alert('Credential is invalid or expired');
      }
    } catch (error) {
      console.error('Error verifying credential:', error);
      alert('Error verifying credential');
    }
  };

  useEffect(() => {
    fetchCredentials();
  }, []);

  return (
    <div className="credential-manager">
      <h2>ACTA Credential Manager</h2>
      
      {/* Create Credential Form */}
      <div className="create-credential">
        <h3>Create New Credential</h3>
        <form onSubmit={createCredential}>
          <div>
            <label>Recipient Stellar ID:</label>
            <input
              type="text"
              value={newCredential.recipient}
              onChange={(e) => setNewCredential({
                ...newCredential,
                recipient: e.target.value
              })}
              placeholder="GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
              required
            />
          </div>
          
          <div>
            <label>Credential Type:</label>
            <select
              value={newCredential.credentialType}
              onChange={(e) => setNewCredential({
                ...newCredential,
                credentialType: e.target.value
              })}
            >
              <option value="certificate">Certificate</option>
              <option value="diploma">Diploma</option>
              <option value="license">License</option>
              <option value="badge">Badge</option>
            </select>
          </div>
          
          <button type="submit" disabled={loading}>
            {loading ? 'Creating...' : 'Create Credential'}
          </button>
        </form>
      </div>

      {/* Credentials List */}
      <div className="credentials-list">
        <h3>Existing Credentials</h3>
        {loading ? (
          <p>Loading credentials...</p>
        ) : (
          <div className="credentials-grid">
            {credentials.map((credential) => (
              <div key={credential.id} className="credential-card">
                <h4>{credential.credentialType}</h4>
                <p><strong>ID:</strong> {credential.id}</p>
                <p><strong>Recipient:</strong> {credential.recipient.substring(0, 10)}...</p>
                <p><strong>Status:</strong> {credential.status}</p>
                <p><strong>Created:</strong> {new Date(credential.timestamp).toLocaleDateString()}</p>
                
                <div className="credential-actions">
                  <button onClick={() => verifyCredential(credential.id)}>
                    Verify
                  </button>
                  <button onClick={() => window.open(`https://stellar.expert/explorer/testnet/tx/${credential.transactionHash}`, '_blank')}>
                    View on Stellar
                  </button>
                </div>
              </div>
            ))}
          </div>
        )}
      </div>
    </div>
  );
};

export default CredentialManager;
```

### **Node.js Backend Integration**

Express.js middleware for credential verification:

```javascript
const express = require('express');
const axios = require('axios');

class ACTAIntegration {
  constructor(apiBaseUrl) {
    this.apiBaseUrl = apiBaseUrl;
  }

  // Middleware to verify credentials
  verifyCredentialMiddleware() {
    return async (req, res, next) => {
      try {
        const credentialId = req.headers['x-credential-id'];
        
        if (!credentialId) {
          return res.status(401).json({
            error: 'Credential ID required in X-Credential-ID header'
          });
        }

        const verification = await this.verifyCredential(credentialId);
        
        if (verification.isValid) {
          req.credential = verification;
          next();
        } else {
          res.status(403).json({
            error: 'Invalid or expired credential'
          });
        }
      } catch (error) {
        res.status(500).json({
          error: 'Credential verification failed'
        });
      }
    };
  }

  // Verify a credential
  async verifyCredential(credentialId) {
    try {
      const response = await axios.get(`${this.apiBaseUrl}/credentials/${credentialId}/verify`);
      return response.data.data;
    } catch (error) {
      throw new Error('Verification request failed');
    }
  }

  // Create a credential
  async createCredential(credentialData) {
    try {
      const response = await axios.post(`${this.apiBaseUrl}/credentials`, credentialData);
      return response.data.data;
    } catch (error) {
      throw new Error('Credential creation failed');
    }
  }

  // Get credentials for a recipient
  async getCredentialsForRecipient(recipientId) {
    try {
      const response = await axios.get(`${this.apiBaseUrl}/credentials?recipient=${recipientId}`);
      return response.data.data;
    } catch (error) {
      throw new Error('Failed to fetch credentials');
    }
  }
}

// Usage in Express app
const app = express();
const acta = new ACTAIntegration('https://api.acta.build');

app.use(express.json());

// Protected route that requires valid credential
app.get('/protected-resource', acta.verifyCredentialMiddleware(), (req, res) => {
  res.json({
    message: 'Access granted!',
    credential: req.credential,
    data: 'This is protected data'
  });
});

// Issue a new credential
app.post('/issue-credential', async (req, res) => {
  try {
    const credential = await acta.createCredential(req.body);
    res.json({ success: true, credential });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Get user's credentials
app.get('/my-credentials/:stellarId', async (req, res) => {
  try {
    const credentials = await acta.getCredentialsForRecipient(req.params.stellarId);
    res.json({ success: true, credentials });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

---

## **Real-World Scenarios**

### **University Degree Verification System**

Complete system for universities to issue and verify degrees:

```javascript
class UniversityCredentialSystem {
  constructor(apiBaseUrl, universityId) {
    this.apiBaseUrl = apiBaseUrl;
    this.universityId = universityId;
  }

  // Issue a degree
  async issueDegree(graduateData) {
    const degreeCredential = {
      recipient: graduateData.stellarId,
      issuer: this.universityId,
      credentialType: "degree",
      metadata: {
        degree: {
          title: graduateData.degreeTitle,
          level: graduateData.level, // "Bachelor", "Master", "Doctorate"
          field: graduateData.field,
          specialization: graduateData.specialization
        },
        student: {
          name: graduateData.name,
          studentId: graduateData.studentId,
          admissionDate: graduateData.admissionDate,
          graduationDate: graduateData.graduationDate
        },
        university: {
          name: "Tech University",
          address: "123 University Ave, Tech City",
          accreditation: "Regional Accreditation Board",
          registrarSignature: await this.generateRegistrarSignature(graduateData)
        },
        academic: {
          gpa: graduateData.gpa,
          honors: graduateData.honors,
          thesis: graduateData.thesis
        }
      }
    };

    const response = await fetch(`${this.apiBaseUrl}/credentials`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(degreeCredential)
    });

    return await response.json();
  }

  // Verify degree for employers
  async verifyDegreeForEmployer(credentialId, employerInfo) {
    const verification = await fetch(`${this.apiBaseUrl}/credentials/${credentialId}/verify`);
    const result = await verification.json();

    if (result.success && result.data.isValid) {
      // Log verification request for audit
      await this.logVerificationRequest(credentialId, employerInfo);
      
      return {
        verified: true,
        degree: result.data.metadata.degree,
        graduate: result.data.metadata.student,
        university: result.data.metadata.university,
        verificationDate: new Date().toISOString()
      };
    }

    return { verified: false };
  }

  async generateRegistrarSignature(graduateData) {
    // In a real implementation, this would be a cryptographic signature
    return `REG-${Date.now()}-${graduateData.studentId}`;
  }

  async logVerificationRequest(credentialId, employerInfo) {
    // Log verification for audit trail
    console.log(`Degree verification requested for ${credentialId} by ${employerInfo.company}`);
  }
}

// Usage
const university = new UniversityCredentialSystem(
  'https://api.acta.build',
  'GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'
);

// Issue a Master's degree
const mastersDegree = await university.issueDegree({
  stellarId: "GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  name: "John Doe",
  studentId: "MS2024001",
  degreeTitle: "Master of Science in Computer Science",
  level: "Master",
  field: "Computer Science",
  specialization: "Artificial Intelligence",
  admissionDate: "2022-09-01",
  graduationDate: "2024-05-15",
  gpa: "3.9",
  honors: ["Summa Cum Laude"],
  thesis: "Machine Learning Applications in Healthcare"
});

console.log('Degree issued:', mastersDegree);
```

### **Professional License Management**

System for managing professional licenses with expiration:

```javascript
class ProfessionalLicenseManager {
  constructor(apiBaseUrl, licensingBodyId) {
    this.apiBaseUrl = apiBaseUrl;
    this.licensingBodyId = licensingBodyId;
  }

  // Issue a professional license
  async issueLicense(licenseData) {
    const licenseCredential = {
      recipient: licenseData.professionalId,
      issuer: this.licensingBodyId,
      credentialType: "license",
      metadata: {
        license: {
          type: licenseData.licenseType,
          number: licenseData.licenseNumber,
          issueDate: new Date().toISOString(),
          expiryDate: licenseData.expiryDate,
          status: "active"
        },
        professional: {
          name: licenseData.name,
          professionalId: licenseData.professionalId,
          specializations: licenseData.specializations
        },
        licensingBody: {
          name: licenseData.licensingBodyName,
          jurisdiction: licenseData.jurisdiction,
          contactInfo: licenseData.contactInfo
        },
        requirements: {
          education: licenseData.educationRequirements,
          experience: licenseData.experienceRequirements,
          examinations: licenseData.examinations
        }
      }
    };

    const response = await fetch(`${this.apiBaseUrl}/credentials`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(licenseCredential)
    });

    return await response.json();
  }

  // Check license status and expiry
  async checkLicenseStatus(credentialId) {
    const response = await fetch(`${this.apiBaseUrl}/credentials/${credentialId}`);
    const credential = await response.json();

    if (!credential.success) {
      return { valid: false, reason: 'License not found' };
    }

    const expiryDate = new Date(credential.data.metadata.license.expiryDate);
    const now = new Date();

    if (expiryDate < now) {
      return {
        valid: false,
        reason: 'License expired',
        expiredOn: expiryDate.toISOString()
      };
    }

    const daysUntilExpiry = Math.ceil((expiryDate - now) / (1000 * 60 * 60 * 24));

    return {
      valid: true,
      license: credential.data.metadata.license,
      professional: credential.data.metadata.professional,
      daysUntilExpiry,
      renewalRequired: daysUntilExpiry <= 30
    };
  }

  // Renew a license
  async renewLicense(originalCredentialId, renewalData) {
    // Get original license
    const originalResponse = await fetch(`${this.apiBaseUrl}/credentials/${originalCredentialId}`);
    const original = await originalResponse.json();

    if (!original.success) {
      throw new Error('Original license not found');
    }

    // Create renewed license
    const renewedLicense = {
      ...original.data,
      metadata: {
        ...original.data.metadata,
        license: {
          ...original.data.metadata.license,
          issueDate: new Date().toISOString(),
          expiryDate: renewalData.newExpiryDate,
          renewalHistory: [
            ...(original.data.metadata.license.renewalHistory || []),
            {
              previousExpiry: original.data.metadata.license.expiryDate,
              renewedOn: new Date().toISOString(),
              renewalFee: renewalData.fee
            }
          ]
        }
      }
    };

    delete renewedLicense.id;
    delete renewedLicense.transactionHash;
    delete renewedLicense.timestamp;

    const response = await fetch(`${this.apiBaseUrl}/credentials`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(renewedLicense)
    });

    return await response.json();
  }
}

// Usage
const licenseManager = new ProfessionalLicenseManager(
  'https://api.acta.build',
  'GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'
);

// Issue a medical license
const medicalLicense = await licenseManager.issueLicense({
  professionalId: "GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  name: "Dr. Sarah Johnson",
  licenseType: "Medical Practice License",
  licenseNumber: "MD-2024-001234",
  expiryDate: "2026-12-31T23:59:59Z",
  specializations: ["Internal Medicine", "Cardiology"],
  licensingBodyName: "State Medical Board",
  jurisdiction: "California, USA",
  contactInfo: "contact@medicalboard.ca.gov",
  educationRequirements: ["MD from accredited institution"],
  experienceRequirements: ["3 years residency"],
  examinations: ["USMLE Step 1", "USMLE Step 2", "USMLE Step 3"]
});

console.log('Medical license issued:', medicalLicense);

// Check license status
const licenseStatus = await licenseManager.checkLicenseStatus(medicalLicense.data.id);
console.log('License status:', licenseStatus);
```

---

## **Error Handling Examples**

### **Comprehensive Error Handling**

```javascript
class ACTAErrorHandler {
  static async handleAPICall(apiCall) {
    try {
      const response = await apiCall();
      
      if (!response.ok) {
        const errorData = await response.json();
        throw new ACTAError(errorData.error, response.status, errorData);
      }
      
      return await response.json();
    } catch (error) {
      if (error instanceof ACTAError) {
        throw error;
      }
      
      // Network or other errors
      throw new ACTAError('Network error or service unavailable', 500, { originalError: error.message });
    }
  }

  static handleError(error) {
    console.error('ACTA API Error:', error);
    
    switch (error.status) {
      case 400:
        return 'Invalid request data. Please check your input.';
      case 401:
        return 'Authentication required. Please provide valid credentials.';
      case 403:
        return 'Access denied. You do not have permission for this operation.';
      case 404:
        return 'Credential not found.';
      case 429:
        return 'Too many requests. Please try again later.';
      case 500:
        return 'Server error. Please try again later.';
      default:
        return 'An unexpected error occurred.';
    }
  }
}

class ACTAError extends Error {
  constructor(message, status, data) {
    super(message);
    this.name = 'ACTAError';
    this.status = status;
    this.data = data;
  }
}

// Usage with error handling
async function createCredentialWithErrorHandling(credentialData) {
  try {
    const result = await ACTAErrorHandler.handleAPICall(async () => {
      return fetch('https://api.acta.build/credentials', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentialData)
      });
    });
    
    console.log('Credential created successfully:', result);
    return result;
  } catch (error) {
    const userMessage = ACTAErrorHandler.handleError(error);
    console.error('User-friendly error:', userMessage);
    throw new Error(userMessage);
  }
}
```

---

## **Performance Optimization**

### **Batch Operations**

```javascript
class BatchCredentialManager {
  constructor(apiBaseUrl, batchSize = 10) {
    this.apiBaseUrl = apiBaseUrl;
    this.batchSize = batchSize;
  }

  // Create multiple credentials in batches
  async createCredentialsBatch(credentialsData) {
    const results = [];
    const errors = [];

    for (let i = 0; i < credentialsData.length; i += this.batchSize) {
      const batch = credentialsData.slice(i, i + this.batchSize);
      
      const batchPromises = batch.map(async (credentialData, index) => {
        try {
          const response = await fetch(`${this.apiBaseUrl}/credentials`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(credentialData)
          });
          
          const result = await response.json();
          return { index: i + index, success: true, data: result };
        } catch (error) {
          return { index: i + index, success: false, error: error.message };
        }
      });

      const batchResults = await Promise.allSettled(batchPromises);
      
      batchResults.forEach(result => {
        if (result.status === 'fulfilled') {
          if (result.value.success) {
            results.push(result.value);
          } else {
            errors.push(result.value);
          }
        } else {
          errors.push({ error: result.reason.message });
        }
      });

      // Add delay between batches to avoid rate limiting
      if (i + this.batchSize < credentialsData.length) {
        await new Promise(resolve => setTimeout(resolve, 1000));
      }
    }

    return { results, errors, total: credentialsData.length };
  }
}

// Usage
const batchManager = new BatchCredentialManager('https://api.acta.build');

const credentialsToCreate = [
  {
    recipient: "GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    credentialType: "certificate",
    metadata: { course: "JavaScript Fundamentals" }
  },
  {
    recipient: "GDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    credentialType: "certificate",
    metadata: { course: "React Development" }
  }
  // ... more credentials
];

const batchResult = await batchManager.createCredentialsBatch(credentialsToCreate);
console.log(`Created ${batchResult.results.length} credentials, ${batchResult.errors.length} errors`);
```

---

*Next: Learn about [Deployment](./08-deployment.md) to deploy your ACTA API integration.*