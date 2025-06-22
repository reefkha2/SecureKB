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

![SecureKB Architecture](securekb_architecture.png)

### 1. Data Ingestion & Processing Flow

1.1. **Document S3 Bucket**: Source repository for all documents to be ingested into the knowledge base

1.2. **Document Processing**: Cluster of components responsible for processing documents

1.3. **Transform Lambda**: Analyzes documents using Claude to:
   - Generate a concise summary of each document
   - Determine which departments should have access (public, sales, hr, finance, executive)
   - Assign an appropriate sensitivity level (public, internal, confidential, restricted)
   - Add metadata to the document for access control

1.4. **Parameter Store**: Stores configuration parameters for the transformation process

1.5. **Bedrock Knowledge Base**: Stores and indexes the processed documents with their metadata

### 2. User Interaction Flow

2.1. **Users**: End users with different roles (departments and clearance levels)

2.2. **Cognito Authentication**: Authenticates users and provides their role information (department and clearance level)

2.3. **Web Application**: User interface for interacting with the knowledge base

2.4. **API Gateway**: Exposes the query endpoint and forwards requests to the Lambda function

2.5. **Query Lambda**: Processes user queries by:
   - Retrieving relevant documents from the knowledge base
   - Filtering results based on the user's department and clearance level
   - Generating responses using Claude based on accessible documents
   - Providing appropriate access denied messages when users don't have sufficient privileges

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

## Making SecureKB Reusable

To make this solution easily reusable across different organizations and use cases, we can:

1. **Create a CloudFormation Template**: Package the entire solution as an AWS CloudFormation template that can be deployed with a few clicks

2. **Develop a CDK Construct**: Create an AWS CDK construct that encapsulates the SecureKB architecture, making it easy to integrate into existing CDK applications

3. **Build a Solution in AWS Solutions Library**: Submit the solution to the AWS Solutions Library with comprehensive documentation and deployment guides

4. **Create Configuration Parameters**: Make key aspects configurable:
   - Department names and hierarchy
   - Sensitivity levels and their hierarchy
   - Document classification prompts
   - Access control logic

5. **Provide Sample Documents and Templates**: Include sample documents and templates for different industries and use cases

6. **Create a Custom Web UI Package**: Package the web UI as a reusable component that can be easily customized and deployed

7. **Develop Integration Guides**: Create guides for integrating SecureKB with existing authentication systems, document repositories, and applications

8. **Build a CLI Tool**: Create a command-line tool for managing the SecureKB deployment, including adding/removing departments, updating sensitivity levels, and managing users

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
