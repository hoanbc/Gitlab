### Upgrade gitlab on old server
```
sudo wget https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/8/gitlab-ce-16.8.1-ce.0.el8.x86_64.rpm/download.rpm -O gitlab-ce-16.8.1-ce.0.el8.x86_64.rpm
dnf localinstall gitlab-ce-16.8.1-ce.0.el8.x86_64.rpm
gitlab-ctl status
```
### Install gitlab on new server (rhel8)
```
sudo yum install wget -y
sudo mkdir /tmp/gitlab
sudo cd /tmp/gitlab
sudo wget https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/8/gitlab-ce-16.8.1-ce.0.el8.x86_64.rpm/download.rpm -O gitlab-ce-16.8.1-ce.0.el8.x86_64.rpm
sudo wget https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey/gitlab-gitlab-ce-3D645A26AB9FBD22.pub.gpg -O gpg-gitlab
sudo rpm --import gpg-gitlab
sudo EXTERNAL_URL="https://gitlab.devops.id.vn" dnf localinstall gitlab-ce-16.8.1-ce.0.el8.x86_64.rpm
sudo gitlab-ctl reconfigure
```
### Backup gitlab on old server
```
sudo gitlab-rake gitlab:backup:create
sudo ls -la /var/opt/gitlab/backups
sudo scp -P 20022 /etc/gitlab/gitlab.rb devops@10.200.12.218:/tmp/gitlab
sudo scp -P 20022 /etc/gitlab/gitlab-secrets.json devops@10.200.12.218:/tmp/gitlab
sudo scp -P 20022 /var/opt/gitlab/backups/1706929558_2024_02_03_16.8.1_gitlab_backup.tar devops@10.200.12.218:/tmp/gitlab
sudo scp -R -P 20022 /etc/gitlab/trusted-certs devops@10.200.12.218:/tmp/gitlab
```
### Restore gitlab on new server
```
sudo systemctl disable --now fapolicyd
sudo mv /tmp/gitlab/gitlab.rb /etc/gitlab/gitlab.rb
sudo mv /tmp/gitlab/trusted-certs /etc/gitlab/trusted-certs

sudo mv /tmp/gitlab/1706929558_2024_02_03_16.8.1_gitlab_backup.tar /var/opt/gitlab/backups/
sudo chown git:git /var/opt/gitlab/backups/1706929558_2024_02_03_16.8.1_gitlab_backup.tar
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq
sudo gitlab-backup restore BACKUP=1706929558_2024_02_03_16.8.1
sudo gitlab-ctl restart
sudo gitlab-rake gitlab:check SANITIZE=true
sudo gitlab-rake gitlab:artifacts:check
sudo gitlab-rake gitlab:lfs:check
sudo gitlab-rake gitlab:uploads:check
Note: If have any function show error 500, execute "sudo gitlab-ctl reconfigure" for fix that
```
