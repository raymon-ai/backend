# Deployment

## Docker swarm
- Make sure Docker is in [Swarm mode](https://docs.docker.com/engine/swarm/), if not run `docker swarm init`

- Replace the value of `RAYMON_MGMT_API_CLIENT_ID` in `stack.yml` with your Client ID
- Create docker secrets:
    - `echo "<your Client secret>" | docker secret create raymon-mgmt-api-secret -`
    - `echo "your passwork" | docker secret create raymondb-password -`
- Run `docker stack deploy -c stack.yml raymon`


