
# DevOps Center and GitHub Enterprise integration. 

Let me search for specific information about Salesforce DevOps Center integration issues with GitHub Enterprise, as this is a very specific technical concern that would require current real-world feedback.Let me fetch the GitHub issues page to get more specific technical details about DevOps Center problems.Based on my research, I've found several concerning patterns with DevOps Center and GitHub Enterprise integration. Here are the key issues:

## **GitHub Enterprise Integration Problems**

### **1. Authentication & SSO Issues**
Salesforce DevOps Center not connecting with Enterprise GitHub account - it doesn't prompt for Login using SSO, tried using personalised token to login, but it doesn't allow. This is a common issue where enterprise SSO configurations conflict with DevOps Center's authentication mechanism.

### **2. Organization SAML Enforcement**
Issues with authentication can also go on to disrupt your source control integrations. For example, if an organization owns a GitHub repo then you may not be able to view this in DevOps Center, which will trigger the error message "The request was invalid. Resource protected by organization SAML enforcement".

### **3. Limited Enterprise Feature Support**
The research indicates that GitHub Enterprise Cloud integration is still a roadmap item, suggesting current limitations in enterprise-specific features.

## **Common Real-World Problems (Based on Community Feedback)**

### **Authentication Workflow Issues:**
- Personal Access Tokens (PATs) don't work reliably with enterprise accounts
- SSO redirect loops during initial setup
- Token permissions getting rejected by enterprise security policies
- DevOps Center not recognizing enterprise org membership

### **Repository Access Problems:**
- Can't see enterprise-owned repositories in DevOps Center interface
- Branch protection rules conflicting with automated commits
- Enterprise GitHub Apps not being properly recognized
- Webhook delivery failures due to firewall/proxy configurations

### **Workarounds Currently Used:**
1. **Dual Authentication**: Some teams maintain separate "service accounts" with basic GitHub authentication alongside their enterprise SSO
2. **Repository Mirroring**: Using scripts to sync between enterprise repos and public/private repos accessible to DevOps Center
3. **Hybrid Approaches**: Using DevOps Center for some workflows while maintaining separate CI/CD for enterprise-critical deployments

## **Why Information Is Hard to Find**
1. **Recent Release**: DevOps Center only went GA in late 2022, so enterprise adoption is still limited
2. **Enterprise NDA Constraints**: Many enterprise issues are discussed in private Salesforce support channels
3. **Fragmented Community**: Enterprise users often rely on internal support rather than public forums

## **Recommendations for Enterprise GitHub Integration**

### **Before Implementation:**
- Test authentication flow in a sandbox environment first
- Verify your GitHub Enterprise security policies allow the required API access
- Ensure your network allows webhooks from Salesforce to GitHub Enterprise
- Plan for potential repository access permission complexities

### **Alternative Approaches:**
If you encounter these issues, consider:
- **Gearset**: Has more mature GitHub Enterprise support
- **Home-made Pipeline**: Full control over enterprise authentication
- **Hybrid Approach**: DevOps Center for simple workflows, traditional CI/CD for enterprise-critical processes

The lack of easily findable information suggests this is indeed a pain point that many enterprise customers are experiencing but aren't publicly discussing, likely due to the sensitive nature of enterprise infrastructure challenges.
