---
title: AWS MCP Server Setup in Codex
date: 2026-08-24
updated: 2026-08-24
tags:
  - aws
  - mcp
  - codex
  - infrastructure
aliases:
  - AWS MCP in Codex
  - Codex AWS MCP Setup
---

# AWS MCP Server Setup in Codex

The **AWS MCP Server** is an AWS-managed remote Model Context Protocol server. It lets Codex search AWS documentation and call AWS APIs using the AWS identity authenticated for the MCP session.

This is different from the **AWS Knowledge MCP Server**:

| Server | Purpose | Authentication | Endpoint model |
| --- | --- | --- | --- |
| AWS MCP Server | Documentation plus authenticated AWS API access | AWS OAuth or AWS credentials through a SigV4 proxy | Regional endpoints |
| AWS Knowledge MCP Server | Public AWS documentation and regional-availability knowledge | None | Global endpoint: `https://knowledge-mcp.global.api.aws` |

> [!warning] API access can include writes
> The AWS MCP Server can expose create, update, and delete operations. An available MCP tool does not automatically mean the current identity is authorized to use it, but a broadly privileged AWS identity may allow consequential changes.

## Current AWS MCP endpoints

As of **2026-08-24**, AWS documents two regional endpoints:

| Endpoint region | Region code | URL |
| --- | --- | --- |
| US East (N. Virginia) | `us-east-1` | `https://aws-mcp.us-east-1.api.aws/mcp` |
| Europe (Frankfurt) | `eu-central-1` | `https://aws-mcp.eu-central-1.api.aws/mcp` |

> [!important] Endpoint region is not the workload region
> The endpoint region identifies where the managed MCP service is reached. It does **not** restrict AWS API calls to that region. Either endpoint can call supported AWS APIs in other regions when the authenticated identity has permission. For example, Codex can connect through the `us-east-1` MCP endpoint and inspect EC2 resources in `ap-northeast-1`.

Choose the endpoint closest to the client or required by organizational policy. Examples below use `us-east-1`; substitute the Frankfurt URL when appropriate.

## Primary setup: direct OAuth from Codex

Codex stores MCP configuration in `~/.codex/config.toml`. A trusted project may instead use `.codex/config.toml` for project-scoped configuration.

### Grant the OAuth sign-in permissions

Before starting the browser sign-in flow, the IAM user or role must be allowed to authorize AWS MCP OAuth access and create OAuth tokens. AWS provides the managed policy:

`arn:aws:iam::aws:policy/AWSMCPSignInOAuthAccessPolicy`

It grants the required actions:

- `signin:AuthorizeOAuth2Access`
- `signin:CreateOAuth2Token`

Attach it to an IAM role with:

```sh
aws iam attach-role-policy \
  --role-name MyAgentRole \
  --policy-arn arn:aws:iam::aws:policy/AWSMCPSignInOAuthAccessPolicy
```

Or attach it to an IAM user with:

```sh
aws iam attach-user-policy \
  --user-name MyAgentUser \
  --policy-arn arn:aws:iam::aws:policy/AWSMCPSignInOAuthAccessPolicy
```

Replace the example identity names with the intended least-privilege agent identity. An administrator may use a custom IAM policy granting the same two actions against the AWS MCP Server authorization resource instead of attaching the AWS-managed policy.

> [!important] OAuth permission is separate from workload permission
> `AWSMCPSignInOAuthAccessPolicy` permits the OAuth connection and token flow; it does not grant permission to list, create, modify, or delete AWS resources. The user or role still needs the relevant service permissions, such as `ec2:DescribeInstances`, and remains subject to permission boundaries, SCPs, resource policies, and explicit denies. AWS account root does not require this additional policy, but root credentials should not be used for routine MCP access.

Add the following server entry:

```toml
[mcp_servers.aws_mcp]
url = "https://aws-mcp.us-east-1.api.aws/mcp"
auth = "oauth"
default_tools_approval_mode = "writes"
startup_timeout_sec = 20
tool_timeout_sec = 120
enabled = true
```

`default_tools_approval_mode = "writes"` asks for approval when a tool is not marked read-only. It is a client-side safety control, not an IAM policy.

Authenticate the server:

```sh
codex mcp login aws_mcp
```

Codex opens the AWS sign-in flow. Sign in using the intended AWS identity, review the authorization request, and approve it. Codex supports OAuth Dynamic Client Registration when the authorization server advertises it.

Verify the registration:

```sh
codex mcp list
```

Inside the Codex terminal UI, use:

```text
/mcp
```

The server should appear as connected and authenticated. Start with a read-only request such as identifying the caller or listing VPCs in one explicitly named region.

> [!tip] Verify identity before inspecting resources
> Ask the MCP server to run the equivalent of `sts:GetCallerIdentity` before broader discovery. Compare the returned account and principal with the environment you intended to access. See [[aws-cli-references]] for related AWS CLI inspection commands.

## Alternative setup: SigV4 proxy and AWS profiles

Use `mcp-proxy-for-aws` when the workflow should use an existing AWS CLI profile, IAM role, or non-interactive AWS credentials. The proxy runs locally as an STDIO MCP server and signs requests to the remote AWS MCP endpoint.

Example Codex configuration:

```toml
[mcp_servers.aws_mcp_sigv4]
command = "uvx"
args = [
  "mcp-proxy-for-aws@latest",
  "https://aws-mcp.us-east-1.api.aws/mcp",
  "--profile",
  "agent-readonly",
  "--metadata",
  "AWS_REGION=ap-northeast-1"
]
default_tools_approval_mode = "writes"
startup_timeout_sec = 30
tool_timeout_sec = 120
enabled = true
```

In this example:

- `us-east-1` is the AWS MCP endpoint region.
- `ap-northeast-1` is the default workload region passed as MCP metadata.
- `agent-readonly` is a placeholder profile name and should be replaced with a dedicated least-privilege profile.
- `uvx` downloads and runs the proxy package, so pin a reviewed version instead of `@latest` when reproducibility is required.

For headless OAuth, AWS also supports obtaining a short-lived OAuth token with IAM credentials. Prefer this for controlled automation rather than copying interactive tokens between machines.

## Authorization model

MCP registration establishes a connection; it does not grant unrestricted AWS access. Effective authorization is determined by the intersection of applicable controls:

- IAM identity policies attached to the user or role
- IAM permission boundaries
- AWS Organizations service control policies
- Resource-based policies
- Session policies and tags
- Region, source-network, MFA, and other policy conditions
- Explicit denies, which override allows

Use a dedicated agent identity with least privilege. Separate read-only discovery from exceptional operational roles, and require approval before assuming a role that can modify infrastructure.

> [!danger] Do not store credentials in the note or MCP configuration
> Never place access keys, session tokens, OAuth tokens, account-specific secrets, or production credentials in this file or directly in `config.toml`. Use AWS profiles, IAM Identity Center, environment-variable references, or the supported OAuth credential store.

## Recommended operating model

Use the tools according to a clear ownership boundary:

| Activity | Preferred tool |
| --- | --- |
| Inventory and configuration discovery | AWS MCP Server |
| Troubleshooting and read-only verification | AWS MCP Server |
| AWS documentation and service availability research | AWS MCP Server or AWS Knowledge MCP Server |
| Infrastructure creation and persistent changes | Terraform |
| Reviewing proposed infrastructure changes | `terraform plan` |
| Confirming deployed state | AWS MCP Server after `terraform apply` |

Avoid changing Terraform-managed resources directly through MCP or the AWS Console. Out-of-band changes create drift between configuration and deployed state. Reserve direct changes for documented emergency procedures and reconcile them into Terraform afterward.

## Troubleshooting

### OAuth login does not open or complete

- Confirm the IAM user or role has `signin:AuthorizeOAuth2Access` and `signin:CreateOAuth2Token`, normally through `AWSMCPSignInOAuthAccessPolicy` or an equivalent custom policy.
- Confirm the MCP URL uses HTTPS and ends in `/mcp`.
- Run `codex mcp list` and confirm the server name matches `aws_mcp`.
- Retry `codex mcp login aws_mcp` from a machine that can open the browser callback.
- Check whether a corporate proxy, browser policy, or callback restriction blocks OAuth.
- For a headless environment, use the supported IAM-based token flow or SigV4 proxy instead of interactive login.

### Credentials are expired

- Reauthenticate the OAuth server with `codex mcp login aws_mcp`.
- For a proxy-backed profile, renew the AWS CLI or IAM Identity Center session.
- Verify the caller identity again before continuing.

### Endpoint connection fails

- Use one of the documented endpoint regions: `us-east-1` or `eu-central-1`.
- Ensure DNS and outbound HTTPS access to the selected `api.aws` hostname are allowed.
- Try the alternate documented endpoint when policy and latency requirements permit.

### An AWS API call returns `AccessDenied`

- Identify the exact AWS action and resource ARN in the error.
- Review identity policies, permission boundaries, SCPs, resource policies, and conditions.
- Do not solve the problem by attaching broad administrator permissions.
- Add only the minimum action and resource scope required by the intended workflow.

### Resources appear in the wrong region or are missing

- Distinguish the MCP endpoint region from the workload region.
- State the target AWS region explicitly in each request when accuracy matters.
- When using the SigV4 proxy, check the `AWS_REGION` metadata value.
- For account-wide inventory, enumerate enabled regions and query each region separately.

### An opt-in region cannot be queried

- Check whether the region is enabled for the AWS account.
- Confirm that the service and required API are available in that region.
- Verify that IAM or organizational policies do not deny the region.
- Do not infer that an empty result means the region was queried successfully; inspect tool errors as well.

## Safe verification checklist

- [ ] The MCP endpoint is an official `aws-mcp.<region>.api.aws/mcp` URL.
- [ ] The authenticated AWS account and principal are the intended ones.
- [ ] The workload region is stated explicitly.
- [ ] The agent identity follows least privilege.
- [ ] Write-capable tools require approval.
- [ ] No credentials or tokens appear in the note or configuration.
- [ ] Initial validation uses read-only AWS APIs.
- [ ] Terraform remains the source of truth for managed infrastructure.

## Official references

- [AWS MCP Server endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/aws-mcp.html)
- [Setting up the AWS MCP Server — Agent Toolkit for AWS](https://docs.aws.amazon.com/agent-toolkit/latest/userguide/getting-started-aws-mcp-server.html)
- [OAuth 2.1 authentication for AWS MCP Server](https://docs.aws.amazon.com/agent-toolkit/latest/userguide/oauth-authentication.html)
- [AWS MCP Server OAuth permissions — AWS Sign-In](https://docs.aws.amazon.com/signin/latest/userguide/aws-mcp-server.html)
- [AWS MCP Server is generally available](https://aws.amazon.com/blogs/aws/the-aws-mcp-server-is-now-generally-available/)
- [Introducing OAuth Support for AWS MCP Server](https://aws.amazon.com/blogs/security/introducing-oauth-support-for-aws-mcp-server/)
- [Secure AI agent access patterns using MCP](https://aws.amazon.com/blogs/security/secure-ai-agent-access-patterns-to-aws-resources-using-model-context-protocol/)
- [Codex Model Context Protocol configuration](https://developers.openai.com/codex/mcp)
