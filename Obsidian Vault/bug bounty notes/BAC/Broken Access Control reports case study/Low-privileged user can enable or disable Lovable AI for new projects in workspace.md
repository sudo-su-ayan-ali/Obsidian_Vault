## 🔗 Here a link of the report :- 
	https://hackerone.com/reports/3369843
## Summary :-
	On lovable.dev, low-privileged user can enable or disable Lovable AI for new projects in workspace. During testing, I found improper authorization vulnerability that allows low privileged user to modify Lovable AI settings. Here's the steps to reproduce.

## Steps To Reproduce:

1. Get the endpoint to enable and disable Lovable AI for new projects in your own workspace
2. Now, replay that endpoint on another workspace with low-privileged role

**Code**•211 Bytes

<!-- To enable --> DELETE /workspaces/<workspace_id>/tool-preferences/ai_gateway/enable <!-- To disable --> POST /workspaces/<workspace_id>/tool-preferences/ai_gateway/enable { "approval_preference":"disable" }

