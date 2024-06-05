Here is a Jira story for the task:

---

**Title:** Remove Permissions for IBM Contractors in OpenShift MFTS Cluster

**Description:**
As part of our security and compliance process, we need to remove permissions for IBM contractors who are no longer working on the MFTS contract. This involves updating the user groups in the Git repository to ensure ArgoCD can sync these changes. The process is managed through GitOps using ArgoCD.

**Acceptance Criteria:**
1. Identify all IBM contractors who no longer work on the MFTS contract.
2. Remove the identified users from the relevant user groups in the Git repository.
3. Ensure the changes are committed and pushed to the repository.
4. Verify ArgoCD syncs the changes to remove the permissions in OpenShift.
5. Confirm that the users no longer have access to the MFTS cluster.

**Tasks:**
1. Review the list of IBM contractors and identify those no longer with the MFTS contract.
2. Access the Git repository containing the user group definitions.
3. Remove the identified users from the appropriate user groups.
4. Commit and push the changes to the repository.
5. Monitor ArgoCD to ensure it syncs the changes.
6. Verify the removal of permissions in OpenShift.

**Assumptions:**
- The list of IBM contractors is up to date.
- Appropriate permissions to access and modify the Git repository are available.
- ArgoCD is properly configured to sync changes from the Git repository to OpenShift.

**Dependencies:**
- Updated list of IBM contractors.
- Access to the Git repository and ArgoCD dashboard.

**Priority:** Medium

**Story Points:** 3

**Reporter:** [Your Name]

**Assignee:** [Assignee's Name]

---

This Jira story outlines the necessary steps and criteria for removing permissions for the specified users from the OpenShift MFTS cluster.
