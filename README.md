# Dependencies

## RHEL

> ref: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/building_running_and_managing_containers/index#getting-container-tools_building-running-and-managing-containers

### Install packages

```shell
sudo yum module install -y container-tools
```

### Install podman-docker (optional):

```shell
sudo yum install -y podman-docker
sudo touch /etc/containers/nodocker
```

### Configure subuid subgid

```shell
echo "$(id -u -n):165537:65536" | sudo tee -a /etc/subuid
echo "$(id -u -n):165537:65536" | sudo tee -a /etc/subgid
```

### Allow users to expose services below 1024 (optional)

```shell
sudo sh -c 'echo 80 > /proc/sys/net/ipv4/ip_unprivileged_port_start'
```

## Ansible

Install the follwing roles as dependency

```shell
ansible-galaxy collection install -r requirements.yml
```

# Using the role

```shell
ansible-playbook -i hosts.ini dbservice.yaml
```
