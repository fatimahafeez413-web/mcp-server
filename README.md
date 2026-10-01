## My Deployment Notes

This is a fork of [ajayamal/mcp-server](https://github.com/ajayamal/mcp-server). 
I deployed this server and connected it to Claude.ai as a live custom MCP connector.

Demo video:[Watch it here](https://www.loom.com/share/e87dff5653b047c392846b5939c793b1)

What I did:
- Deployed the server to Railway
- Configured it to run in remote/HTTP mode instead of local-only mode
- Debugged and fixed deployment issues (start command, environment variables, port/domain configuration)
- Connected it to Claude.ai as a custom MCP connector
- Verified it works end-to-end by asking Claude live weather questions, which it answered using this server's `get_forecast` and `get_alerts` tools



