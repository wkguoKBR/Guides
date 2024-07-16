# Expose LeapfrogAI API Guide (Tuesday, July 16 2024)

## Introduction

This guide provides instructions to expose LeapfrogAI API to your local server.  Note that work is being done to provide long-lived API keys and address a couple pain-points that will be mentioned. This writing references documentation written by Justin Law from Defense Unicorns, which can be found in the *Documents* folder of the repository.

## Requirements
> 1) You have cloned the [LeapfrogAI Github Repository](https://github.com/defenseunicorns/leapfrogai.git)
> 2) UDS Core has been previously deployed (ensure that `uds deploy k3d-core-slim-dev:0.22.2` has ran successfully)
> 
> 3) Your local environment is clear of prior deployment artifacts:
>     ```
>     uds zarf tools clear-cache && rm -rf ~/.uds-cache && rm -rf /tmp/zarf-*
>     docker system prune -a -f && docker volume prune -f
>     ```

**ATTENTION:** It can be a challenging, frustrating, and long-process to setup your environment for proper API deployment. There are multiple areas where errors may occur when running commands (specifically at step 3), and the best solution if you're at a big roadblock is to either perform a full clean-up/re-clone the LeapfrogAI repo and start from scratch.

Run the following commands in order to perform a clean-up or fresh install of your deployments.

```
k3d cluster delete uds  # kills a running uds cluster
uds zarf tools clear-cache # clears the Zarf tool cache
rm -rf ~/.uds-cache # clears the UDS cache
docker system prune -a -f # removes all hanging containers and images
docker volume prune -f # removes all hanging container volumes
```

## Instructions

1) Change directories using the following command: `cd uds-bundles/latest/cpu`

2) Open the `uds-config.yaml` with vim (ex. vim uds-config.yaml) and insert the line `leapfrogai-api.expose_openapi_schema: true` under the `leapfrogai-api` section.

3) Create and deploy the LeapfrogAI 0.8.0 UDS bundle:
   ```
   uds create .
   uds deploy uds-bundle-leapfrogai-*.tar.zst --confirm
   ```
4) Obtain the Supabase JWT Secret:
   ```
   uds zarf tools kubectl get secret -n leapfrogai supabase-bootstrap-jwt -o json | jq '.data | map_values(@base64d)'
   ```
5) Generate a Supabase user and obtain an access token:
   - `<anon-key>` is the Supabase JWT Secret
   - `<email>` is the user's email
   - `<password>` is the user's password
   - `<confirmPassword>` is the user's password again
   <br/><br/>
   ```
   curl 'https://supabase-kong.uds.dev/auth/v1/signup' \
    -H "apikey: <anon-key>" \
    -H "Content-Type: application/json" \
    -H "Authorization": f"Bearer <anon-key>" \
    -d '{ "email": "<email>", "password": "<password>", "confirmPassword": "<confirmPassword>"}'
   ```
7) Test the LeapfrogAI API (CPU Deployment):
   - `<api-key>` is the access token given in the previous step
   <br/><br/>
   ```
   curl -l 'https://leapfrogai-api.uds.dev/openai/v1/chat/completions' \
    -H 'Authorization: Bearer <api-key>' \
    -H 'Content-Type: application/json' \
    -d '{
        "model": "llama-cpp-python",
        "messages": [
            {
                "role": "system",
                "content": "Your name is Doug, and you are a helpful AI assistant created by a company named Defense Unicorns."
            },
            {
                "role": "user",
                "content": "What is your name?"
            }
        ],
        "stream": false,
        "temperature": 0.8,
        "max_tokens": 1024
    }'
   ```
  
   The first entry in the `messages` array defines the chatbot assistant. The second is the user query, in this case asking `"What is your name?"` as the value of the `"content"` key. Recognize the difference      in roles between the two messages.

   Here is the response I obtained to the above query:
   ```
   {"id":"",
    "object":"chat.completion",
    "created":0,
    "model":"llama-cpp-python",
    "choices":[
       {"index":0,
        "message":{"role":"assistant",
        "content":" My name is Doug, and I am an AI assistant created by the company Defense Unicorns."},
        "finish_reason":"FinishReason.STOP"
       }
    ],
    "usage":{"prompt_tokens":41,"completion_tokens":23,"total_tokens":64}
   }
   ```
   
9) After a successful deployment, explore the schema at https://leapfrogai-api.uds.dev/docs

## Common Issues

1) **curl command in Step 7 (Instructions) gives `{"detail":"Token has expired or is not valid. Generate a new token"}`**

   The API key is only valid for approximately an hour, requiring the need to generate a new one. However, the problem here is that the curl command we originally used in Step 5 to do so is only valid when generating new users. To generate a new access token as an existing user, run the following command:

    ```
   curl -X POST 'https://supabase-kong.uds.dev/auth/v1/token?grant_type=password' \
    -H "apikey: <anon-key>" \
    -H "Content-Type: application/json" \
    -d '{ "email": "<email>", "password": "<password>"}'
   ```
  
2) **curl command in Step 1 (Common Issues) gives `{"message":"Unauthorized", "request_id":"..."}`**

   Your provided <anon-key> is likely no longer valid. Generate a new one with the command in Step 4 (Instructions).
   
3) **curl command in Step 1 (Common Issues) gives `{"message":"Duplicate API key found","request_id":"..."}`**

   This occurs when you attempt to generate a new access token as an existing user, but your pre-existing one is still valid for use. Simply continue using this key for your queries.
   
4) **curl command in Step 1 (Common Issues) gives `{"error":"invalid_grant","error_description":"Invalid login credentials"}`**

   A plausible reason for this error is that your provided email/password is inputted incorrectly or no longer is recorded. Double check you inputted your email + password wrong and if you still get the same error, create a new user with a different email.


