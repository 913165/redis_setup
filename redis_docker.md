
### docker command to run redis server
```
docker run -d --name redis -p 6379:6379 redis
```

### Connect with redis-cli 
```
docker exec -it redis redis-cli
```

### Store and retrieve data
```
SET bike:1 "Process 134"
GET bike:1
```
### Hash Example (Like Table Row)
HSET user:1 name "Pritesh" age 35 city "Mumbai"
HGETALL user:1
