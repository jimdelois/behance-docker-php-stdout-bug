To reproduce:
```bash
git clone git@github.com:jimdelois/behance-docker-php-stdout-bug
cd behance-docker-php-stdout-bug
docker-compose up -d
```
Enter the Docker Machine Name for your local
```bash
DOCKER_MACHINE_NAME=""
```
Send a request to the container:
```bash
curl -XGET http://$(docker-machine ip $DOCKER_MACHINE_NAME):$(docker port badboy 8080 | awk '{split($0,a,":"); print a[2]}')
```

Examine the Docker logging by examining the raw container logs from the docker host
(e.g., `docker-machine ssh $DOCKER_MACHINE_NAME`, thence `tail -F /var/lib/docker/containers/$CONTAINER_ID/$CONTAINER_ID-json.log`):
- Actual: `{"log":"I want a Bad Boy Sandwich\n","stream":"stderr","time":"2020-11-19T06:18:08.672226893Z"}`
- Expected: `{"log":"I want a Bad Boy Sandwich\n","stream":"stdout","time":"2020-11-19T06:18:08.672226893Z"}`

Notes:
- Raw PHP stream explicitly set to `php://stdout`; Loggers (e.g., Monolog) in production applications typically use this stream
- The unexpected outcome is that containers are sending "normal" PHP output to STDERR, which affects metrics in AWS, FluentD routing, Sumo queries, etc.
