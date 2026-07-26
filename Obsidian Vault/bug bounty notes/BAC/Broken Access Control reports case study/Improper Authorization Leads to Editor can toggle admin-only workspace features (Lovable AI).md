## 🔗 Here is the link of the report
	https://hackerone.com/reports/3371448
## Summary:

An account with the Editor role can successfully call the API endpoint `/workspaces/<WORKSPACE_ID>/tool-preferences/ai_gateway/enable`that disables workspace-wide admin-only feature (Lovable AI). The API does not enforce server-side role checks for this action, allowing a vertical privilege escalation (Broken Access Control).

## Steps To Reproduce:

Preconditions

- Two accounts are members of the same workspace `Victim-Workspace`:
    
    - Account A (Owner/Admin) - Owner role in `Victim-Workspace`.
    - Account B (Editor) - Editor role in the same `Victim-Workspace`.

1. Sign in as Account A (Admin) and toggle the AI feature off inside the workspace.
    
2. Capture this endpoint (Responsible for Disabling AI Feature).
3. **Code**•198 Bytes
```code

POST /workspaces/<WORKSPACE_ID>/tool-preferences/ai_gateway/enable HTTP/2 
Host: lovable-api.com 
Authorization: Bearer <OWNER-TOKEN> 
Content-Type: application/json 

{"approval_preference":"disable"}
```
Note: <WORKSPACE_ID> is the workspace id for Victim-Workspace.
4. Modify request to use Editor JWT Token

- Replace the Authorization header with Account B (Editor) JWT:

**Code**•196 Bytes
```code

POST /workspaces/<WORKSPACE_ID>/tool-preferences/ai_gateway/enable HTTP/2
Host: lovable-api.com
Authorization: Bearer <EDITOR_JWT>
Content-Type: application/json

{"approval_preference":"disable"}

```
5. When the request is sent again with the Editor's JWT , the request succeeds and the Lovable AI setting is switched off for the workspace.
#### impact : - 
Based on the Lovable AI documentation, this feature powers all the AI-driven parts of a workspace like prompt integrations, content generation, and other model-based actions.

If an Editor disables it, all those AI features stop working across the workspace, breaking functionality for every member. Since only admins are supposed to control this setting, the bug lets an unauthorized user disrupt how the workspace operates and remove core features that teams rely on.
