## docker command to setup
docker run -d --name redis -p 6379:6379 redis

## docker command to get redis promot
docker exec -it redis redis-cli

## create and get key values
SET bike:1 "Process 134"
GET bike:1
