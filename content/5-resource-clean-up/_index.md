---
title: "Clean Up Resources"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

#### **Clean Up Resources**

You can optionally clean up the resources that were created.

```python
# Optional: You can delete the agent after deleting the alias
# Also, make sure to disable it first
action_group_id = agent_action_group_response['agentActionGroup']['actionGroupId']
action_group_name = agent_action_group_response['agentActionGroup']['actionGroupName']

response = bedrock_agent_client.update_agent_action_group(
    agentId=agent_id,
    agentVersion='DRAFT',
    actionGroupId=action_group_id,
    actionGroupName=action_group_name,
    actionGroupState='DISABLED',
    parentActionGroupSignature='AMAZON.CodeInterpreter'
)

action_group_deletion = bedrock_agent_client.delete_agent_action_group(
    agentId=agent_id,
    agentVersion='DRAFT',
    actionGroupId=action_group_id
)

agent_deletion = bedrock_agent_client.delete_agent(
    agentId=agent_id
)

# Delete IAM Roles and Policies

for policy in [agent_bedrock_allow_policy_name]:
    iam_client.detach_role_policy(RoleName=agent_role_name, PolicyArn=f'arn:aws:iam::{account_id}:policy/{policy}')

for policy in [agent_bedrock_policy]:
    iam_client.delete_policy(
        PolicyArn=policy['Policy']['Arn']
)

iam_client.delete_role(
    RoleName=agent_role_name
) 
```

![clean-up](/images/5-resource-clean-up/image.png)