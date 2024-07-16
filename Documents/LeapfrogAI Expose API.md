---
created: 20240703 16:31
updated: 20240703 16:31
Author: Justin Law
---
# Exposing LeapfrogAI's API

> ***Note:***
> LeapfrogAI's API is currently being refactored as part of our migration to a monorepo. The API is not currently accessible out of the box by users. Long-lived authentication and API keys are on the roadmap for an upcoming release.

Pre-requisites:  
- UDS Core has been deployed within the cluster (e.g., `uds deploy k3d-core-slim-dev:0.22.2` has already completed successfully)
- You have pre-configured your host for the LeapfrogAI GPU or CPU deployment
- You have cleaned your local environment of previous deployment artifacts:

`uds zarf tools clear-cache && rm -rf ~/.uds-cache && rm -rf /tmp/zarf-*`
`docker system prune -a -f && docker volume prune -f`

1. Inside of the `uds-config.yaml` being used to deploy the LeapfrogAI 0.8.0 UDS bundle, add the `leapfrogai-api.expose_openapi_schema: true` so that the section in the YAML looks like this:  

```
leapfrogai-api:
    hosted_domain: "uds.dev"
    expose_openapi_schema: true
```

2. Create and deploy the LeapfrogAI 0.8.0 UDS bundle:  

`cd uds-bundles/dev/cpu`
`uds create .`
`uds deploy uds-bundle-leapfrogai*.tar.zst`

3. Get the Supabase JWT Secret:  

`uds zarf tools kubectl get secret -n leapfrogai supabase-bootstrap-jwt -o json | jq '.data | map_values(@base64d)'`

4. Generate a Supabase user:  

- `<anon-key>` should be replaced with the Supabase JWT Secret
- `<email>` should be replaced with the desired user email
- `<password>` should be replaced with the desired user's password
- `<confirmPassword>` should be replaced with the desired user's password, again

```
curl -X POST '[https://supabase-kong.uds.dev/auth/v1/signup](https://supabase-kong.uds.dev/auth/v1/signup)' \
  -H "apikey: <anon-key>" \
  -H "Content-Type: application/json" \
  -H "Authorization": f"Bearer <anon_key>" \
  -d '{ "email": "<email>", "password": "<password>", "confirmPassword": "<confirmPassword>"}'
```


- The API key is the large key, `access_token`, that is logged as a result of the above POST command

5. Test the LeapfrogAI API (CPU):  

- `<api-key>` should be replaced with the API key from the previous step

```
curl -l '[https://leapfrogai-api.uds.dev/openai/v1/chat/completions](https://leapfrogai-api.uds.dev/openai/v1/chat/completions)' \
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

6. Once deployed and tested, you can view the schema at https://leapfrogai-api.uds.dev/docs