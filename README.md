# SecureKB: Role-Based Access Control for Amazon Bedrock Knowledge Base

## Problem Statement

Organizations need to securely manage and share knowledge across different departments while ensuring sensitive information is only accessible to authorized personnel. Traditional knowledge management systems often lack granular access controls, leading to either oversharing of sensitive information or creating information silos that hinder collaboration.

Key challenges include:

1. **Access Control**: Ensuring users can only access information appropriate for their role and clearance level
2. **Content Classification**: Automatically determining the sensitivity and departmental relevance of documents
3. **Contextual Retrieval**: Providing relevant context for document chunks to improve search quality
4. **Secure Integration**: Building a secure end-to-end solution from document ingestion to user queries

## Solution: SecureKB

SecureKB is a secure knowledge management solution built on Amazon Bedrock Knowledge Base with role-based access control. It uses AI to automatically classify documents by department and sensitivity level, then enforces access controls when users query the knowledge base.

### Key Features

- **AI-Powered Document Classification**: Automatically determines which departments should have access to documents and their sensitivity level
- **Role-Based Access Control**: Filters knowledge base results based on user's department and clearance level
- **Contextual Retrieval**: Generates summaries for documents to improve search quality
- **Secure Authentication**: Uses Amazon Cognito for user authentication and role management
- **Transparent Access Messaging**: Clearly explains to users why certain information is restricted and what access level is required

## Architecture

![SecureKB Architecture](securekb_architecture_horizontal.png)

### Data Ingestion Flow

**1. Amazon S3 Data Source → Transform Lambda**
- Documents are uploaded to an Amazon S3 bucket
- This triggers the Transform Lambda function
- The S3 bucket serves as the source repository for all documents to be ingested into the knowledge base

**2. Transform Lambda → Bedrock Knowledge Base**
- The Transform Lambda function processes each document using Claude to:
  - Generate a concise summary of the document (maximum 50 words)
  - Determine which departments should have access (public, sales, hr, finance, executive)
  - Assign an appropriate sensitivity level (public, internal, confidential, restricted)
- The Lambda function adds this metadata to the document
- The processed document is then ingested into the Bedrock Knowledge Base
- No chunking is performed, preserving the full document content

### User Interaction Flow

**3. Users → Cognito Authentication**
- Users access the SecureKB web application
- They authenticate using Amazon Cognito
- Cognito verifies their credentials and provides authentication tokens
- User attributes (department and clearance level) are stored in Cognito and included in the tokens

**4. Cognito Authentication → API Gateway**
- After successful authentication, the web application receives tokens containing user attributes
- When a user submits a query, the tokens are sent with the request to the API Gateway
- The API Gateway validates the tokens using a Cognito authorizer

**5. API Gateway → Query Lambda**
- The API Gateway forwards the authenticated request to the Query Lambda function
- The request includes the user's query and authentication tokens

**6. Query Lambda → Bedrock Knowledge Base**
- The Query Lambda function extracts user role information (department and clearance level) from the Cognito tokens
- It queries the Bedrock Knowledge Base using the retrieve API
- It filters the results based on the user's department and clearance level
- It uses only the accessible documents to generate a response using Claude
- If the user doesn't have access to any relevant documents, it provides a detailed access denied message

## Implementation Details

### Document Classification

Documents are classified using Claude to determine:

- **Departments**: Which departments should have access (public, sales, hr, finance, executive)
- **Sensitivity Level**: Required clearance level to access (public, internal, confidential, restricted)

This classification is stored as metadata with each document in the knowledge base.

### Access Control Logic

When a user queries the knowledge base:

1. The system retrieves all potentially relevant documents
2. It filters out documents that the user doesn't have access to based on:
   - Department match: User's department must match one of the document's allowed departments
   - Clearance level: User's clearance level must be equal to or higher than the document's sensitivity level
3. It uses only the accessible documents to generate a response
4. If the user doesn't have access to any relevant documents, it provides a detailed access denied message

### Access Denied Messaging

When a user doesn't have access to information, the system provides a clear message explaining:

- That information exists but is restricted based on their access level
- Which departments have access to the information
- What clearance level is required
- The user's current access level
- How to request elevated access privileges

### Source Code

The source code for the Lambda functions is available in the repository:

- **Transform Lambda**: Located in `src/lambda/transform_processor/index.py`
- **Query Lambda**: Located in `src/lambda/query_processor/index.py`

These Lambda functions implement the core functionality of the SecureKB solution, including document processing, metadata extraction, and access control enforcement.

## Getting Started

To deploy SecureKB in your environment:

1. Clone this repository
2. Configure your AWS credentials
3. Update the configuration in `securekb_config.json`
4. Run the deployment scripts:
   ```bash
   ./create_s3_bucket.sh
   ./deploy_transform_lambda.sh
   ./deploy_lambda_api.sh
   ```
5. Create a Bedrock Knowledge Base in the AWS Console
6. Configure the Knowledge Base to use the transformation Lambda function
7. Upload your documents to the S3 bucket
8. Start the web interface:
   ```bash
   ./start_web_demo.sh
   ```

## Conclusion

SecureKB provides a secure, intelligent solution for knowledge management with role-based access control. By leveraging Amazon Bedrock and Claude, it automatically classifies documents and enforces access controls, ensuring users only see information appropriate for their role while providing clear explanations when access is restricted.

This solution can be easily customized and deployed across different organizations and use cases, making it a versatile tool for secure knowledge management.
