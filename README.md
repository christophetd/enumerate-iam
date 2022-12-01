## Enumerate IAM permissions

Found a set of AWS credentials and have no idea which permissions it might have?

```console
$ ./enumerate-iam.py --access-key AKIA... --secret-key StF0q...
2019-05-10 15:57:58,447 - 21345 - [INFO] Starting permission enumeration for access-key-id "AKIA..."
2019-05-10 15:58:01,532 - 21345 - [INFO] Run for the hills, get_account_authorization_details worked!
2019-05-10 15:58:01,537 - 21345 - [INFO] -- {
    "RoleDetailList": [
        {
            "Tags": [], 
            "AssumeRolePolicyDocument": {
                "Version": "2008-10-17", 
                "Statement": [
                    {
...
2019-05-10 15:58:26,709 - 21345 - [INFO] -- gamelift.list_builds() worked!
2019-05-10 15:58:26,850 - 21345 - [INFO] -- cloudformation.list_stack_sets() worked!
2019-05-10 15:58:26,982 - 21345 - [INFO] -- directconnect.describe_locations() worked!
2019-05-10 15:58:27,021 - 21345 - [INFO] -- gamelift.describe_matchmaking_rule_sets() worked!
2019-05-10 15:58:27,311 - 21345 - [INFO] -- sqs.list_queues() worked!
```

Now you do!

`enumerate-iam.py` tries to brute force all API calls allowed by the IAM policy.
The calls performed by this tool are all non-destructive (only get* and list*
calls are performed).

If you don't provide credentials through the CLI arguments, enumerate-iam will attempt to read them from your current environment (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`).

## Installation

```
git clone git@github.com:andresriancho/enumerate-iam.git
cd enumerate-iam/
pip install -r requirements.txt
```

## Library

This software was written to be easy to integrate with other tools, just import
the main function and provide the required arguments:

```python
from enumerate_iam.main import enumerate_iam

enumerate_iam(access_key,
              secret_key,
              session_token,
              region)
```

The output will contain all the enumerated permission information in a python
dictionary.

## Other tools

Before writing `enumerate-iam.py` I tried a few that performed the same task.
Decided to write my own because the others:

 * Did not check for all API calls
 * Where painfully slow when adding more API calls to the list
 * Did not return the permissions in a programmatic way

## Updating the API calls

The API calls to be performed during permission enumeration are stored in
`enumerate_iam/bruteforce_tests.py`, a Python dict() which is generated by
`enumerate_iam/generate_bruteforce_tests.py` using the API documentation
available in the `aws-sdk-js` library. 

AWS releases new services every quarter, to make sure that this tool is
finding all the existing permissions run:

```console
cd enumerate_iam/
git clone https://github.com/aws/aws-sdk-js.git
python generate_bruteforce_tests.py
rm -rf aws-sdk-js
```

## Related tools

This tool was released as part of the [Internet-Scale Analysis of AWS Cognito Security](https://www.blackhat.com/us-19/briefings/schedule/?hootPostID=4abc475398765919352042ac015752e6#internet-scale-analysis-of-aws-cognito-security-15829)
research. During this research the [cc-lambda](https://github.com/andresriancho/cc-lambda) tool
was also used to extract information from the Common Crawl data.

## Initial code

The initial code was released in [this gist](https://gist.github.com/darkarnium/1df59865f503355ef30672168063da4e)
and improved in multiple ways:

 * Complete refactoring
 * Results returned in a programmatic way
 * Threads
 * Improved logging
 * Increased API call coverage
 * Export as a library
