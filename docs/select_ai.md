# Select AI With Autonomous DB

**Select AI** is the simplest way to get answers about your business. All you have to do is use your **natural language** in order to just ask a question. The Autonomous DB manages the entire query process to produce your answer.

## Select AI Benefits

- **Simple**
    - Easily build generative AI into new or existing applications
- **Future-Enabled**
    - Choose from an array of LLMs
    - Pick the model that is best suited to your business
- **Secure**
    - Rely on the same DB security that protects the data
    - Data will not be sent to the LLM provider

## Select AI: SQL

Select AI **translates natural language into Oracle SQL language** using an LLM.

![Select AI](../imgs/select_ai.png)

## Sample code

Drop the AI profile (if already exists).

```
begin
    DBMS_CLOUD_API.DROP_PROFILE(
        profile_name => 'GENAI',
        force        => true
    );
end;
```

Create the AI profile.

```
begin
    DBMS_CLOUD_AI.CREATE_PROFILE(
        profile_name => 'GENAI',
        attributes => '{
            "provider": "OCI",
            "credential_name": "OCI_CRED",
            "object_list": [
                {"owner": "MOVIESTREAM", "name": "movies"},
                {"owner": "MOVIESTREAM", "name": "streams"},
                {"owner": "MOVIESTREAM", "name": "actors"}
            ],
            "region": "eu-milan-1"
        }'     
    );
end;
```

Set the AI profile for this session.

```
begin
    DBMS_CLOUD_AI.SET_PROFILE(profile_name => 'GENAI')
end;
```

Ask your question.

```
SELECT DBMS_CLOUD_AI.GENERATE(
    profile_name => 'GENAI',
    action       => 'chat',
    prompt       => 'What is Oracle Autonomous DB?')
```