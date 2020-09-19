

如下
```bash
sudo docker run --detach \
--hostname gitlab.local-test.com \
--publish 443:443 --publish 80:80 --publish 22:22 \
--name gitlab \
--restart always \
--volume /Users/eos/my-docs/env/dockers/gitlab/config:/etc/gitlab \
--volume /Users/eos/my-docs/env/dockers/gitlab/logs:/var/log/gitlab \
--volume /Users/eos/my-docs/env/dockers/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce:latest
```
