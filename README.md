To specify a user with which to run AWS Systems Manager (SSM) commands, you typically use the `runAs` parameter in the SSM document if it supports this option. However, not all AWS-provided SSM documents directly support a `runAs` parameter. You might need to create a custom SSM document if you want to use a specific parameter name like `runAsDefaultUser`. 

Here's how you can define a custom SSM document that specifies running commands as the "ubuntu" user and then how to execute an SSM command using this custom document:

### Step 1: Create a Custom SSM Document

First, you need to create a custom SSM document that includes a step to run shell commands as a specified user. Here’s an example JSON for a simple custom SSM document that runs shell commands using a specified user:

```json
{
  "schemaVersion": "2.2",
  "description": "Run shell scripts or commands with a specified default user",
  "parameters": {
    "commands": {
      "type": "StringList",
      "description": "The commands to run."
    },
    "runAsDefaultUser": {
      "type": "String",
      "default": "ubuntu",
      "description": "User to run the commands as."
    }
  },
  "mainSteps": [
    {
      "action": "aws:runShellScript",
      "name": "runShellCommands",
      "inputs": {
        "runCommand": "{{ commands }}",
        "runAs": "{{ runAsDefaultUser }}"
      }
    }
  ]
}
```

You would save this document in AWS Systems Manager with a unique name, such as `CustomRunShellScriptWithUser`.

### Step 2: Use the AWS CLI to Create the Document

You can use the AWS CLI to create this document:

```bash
aws ssm create-document \
    --content file://path_to_your_document.json \
    --name "CustomRunShellScriptWithUser" \
    --document-type "Command" \
    --document-format "JSON"
```

Replace `path_to_your_document.json` with the path to the JSON file you have prepared.

### Step 3: Send a Command Using the Custom Document

Once the document is created, you can use it to send commands via AWS SSM that run as the `ubuntu` user:

```bash
aws ssm send-command \
    --document-name "CustomRunShellScriptWithUser" \
    --targets "Key=instanceids,Values=i-1234567890abcdef0" \
    --parameters '{"commands":["echo Hello, World!"]}' \
    --timeout-seconds 600 \
    --max-concurrency "50" \
    --max-errors "0" \
    --region us-east-1
```

This command uses the custom document to execute `echo Hello, World!` on the specified EC2 instance as the `ubuntu` user.

### Conclusion

This setup allows you to specify exactly how your commands are executed, including under which user account, using AWS SSM. Always ensure that the `ubuntu` user exists on the target machine and that it has the necessary permissions to execute the commands you are sending. Also, adjust the `--targets` parameter as needed to target specific instances or groups of instances.
