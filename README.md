# License
See [LICENSE.md](LICENSE.md)

# Deployment

## Docker swarm
- Make sure Docker is in [Swarm mode](https://docs.docker.com/engine/swarm/), if not run `docker swarm init`

- Replace the value of `RAYMON_MGMT_API_CLIENT_ID` in `stack.yml` with your Client ID
- Replace the value of `API_URL` in the `ui` service to where the api of this deployment will be available. 
- Create docker secrets:
    - `echo "<your client secret>" | docker secret create raymon-mgmt-api-secret -`
    - `echo "your password" | docker secret create raymondb-password -`
- Run `docker stack deploy -c stack.yml raymon`


