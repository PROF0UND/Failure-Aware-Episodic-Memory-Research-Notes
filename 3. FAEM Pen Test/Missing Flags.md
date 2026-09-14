1. Invalid password not being flagged as a failure.
```shell
── step 7/15 ──────────────────────────────
[shell] [unknown] curl -s -X POST -d "username=admin' OR '1'='1' #&password=password" http://localhost:1337
[output]
{"message":"Invalid user or password"}
```