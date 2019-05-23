# A GitLab Runner Instance that Can Run Docker and docker-compose in a GitLab CI/CD Pipeline
To use Docker within a GitLab CI pipeline (e.g., to run `docker build` or `docker push`) you need a private GitLab Runner that uses an image containing Docker. The safest approach is to configure the runner so that it has access to the host's Docker socket. For additional safety and convenience, we can run the runner in its own container.

## How to Use
The terminology can be a bit confusing: you need a gitlab-runner instance which uses GitLab's own `gitlab/gitlab-runner` image. The gitlab-runner is responsible for running the individual runners that run the tasks in your CI/CD pipeline. The only thing special about this docker-compose file is that it makes it possible for the runner to access the host's Docker socket. Once the gitlab-runner is up and running, you'll need to create a `runner` via the command line or by editing the `config.toml` file in the `config` directory and restarting the gitlab-runner.

### Example Configuration
Below is an example of a `config.toml` file for a runner capable of running Docker and docker-compose commands in a GitLab CI/CD pipeline. The important parts: `executor = "docker"`, `image = "kmbn/docker-compose` (an image that includes Docker and docker-compose just for using in GitLab CI/CD pipelines), and `volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]` (for sharing Docker socket).

```
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "name of your runner"
  url = "url for your GitLab instance (https://gitlab.com/ if you use gitlab.com"
  token = "your project's token"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.docker]
    tls_verify = false
    image = "kmbn/docker-compose"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
    shm_size = 0
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
```

# Acknowledgements
The `docker-compose.yml` file is based on the one in Angristan's writeup at [Automatically build and push Docker images using GitLab CI](https://angristan.xyz/build-push-docker-images-gitlab-ci/)[https://angristan.xyz/)build-push-docker-images-gitlab-ci/]. The runner config above is based on the runner config described in that article as well. See [https://github.com/kmbn/docker-docker-compose](https://github.com/kmbn/docker-docker-compose) for the source of the image I suggest using in the runner.