# podman_systemd_container

This role installs systemd unit files to run containers using podman

## Requirements

This Role has been tested on Red Hat Enterprise Linux 8. The requirement is that the podman package has been installed.

## Role Variables

| **Variable**         | **Description**                                                                                                                                                           | **Required** | **Default value** | **Example value**                                                          |
| -------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :----------- | :---------------- | :------------------------------------------------------------------------- |
| image                | Container Image to start. If it doesn't exist, it will try to pull it on start.                                                                                           | yes          |                   | registry.redhat.io/rhel8/mariadb-103                                       |
| description          | A description of the service.                                                                                                                                             | yes          |                   | MariaDB service                                                            |
| env                  | A dictionary of environment variables which will be passed to the container.                                                                                              | no           | {}                | { 'MYSQL_USER': 'user', 'MYSQL_PASSWORD': 'pass', 'MYSQL_DATABASE': 'db' } |
| ports                | A list of port mappings which will be passed to the container.                                                                                                            | no           | []                | [ '3306:3306' ]                                                            |
| volumes              | A list of volume mappings which will be passed to the container.                                                                                                          | no           | []                | [ '/var/dbservice/db:/var/lib/mysql/data:Z' ]                              |
| container_restart    | Systemd restart policy of container. See man 5 systemd.service for all possibilities.                                                                                     | no           | on-failure        | always                                                                     |
| healthcheck          | A command which will be executed inside the container to run a health check. The healthcheck will report health if the exit code of the command is 0.                     | no           |                   | /usr/bin/mysql -h localhost -u root -e "show databases;" \|\| exit 1       |
| healthcheck_interval | Interval for the healthcheck command to execute periodically. Time unit suffixes can be used (s, m, h). This option only applies if a healthcheck is specified.           | no           | 30s               | 20s                                                                        |
| health_retries       | The number of retries allowed before a healthcheck is considered to be unhealthy. This option only applies if a healthcheck is specified.                                 | no           | 3                 | 5                                                                          |
| health_start_period  | The initialization time needed for a container to bootstrap. The value can be expressed in time format like 2m3s. This option only applies if a healthcheck is specified. | no           | 0s                | 2m3s                                                                       |
| depends_on           | Specifies the name of a systemd service on which this service will depend on. This will be rendered using the After= and Requires= options in the systemd unit file.      | no           |                   | NetworkManager                                                             |

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - include_role:
        name: podman_systemd_container
      vars:
        container_name: dbservice-mariadb
        image: registry.redhat.io/rhel8/mariadb-103
        description: MariaDB service
        env:
          MYSQL_USER: user
          MYSQL_PASSWORD: pass
          MYSQL_DATABASE: db
        ports:
          - '3306:3306'
          - '8080:80'
        volumes:
          - /var/dbservice/db:/var/lib/mysql/data:Z
        healthcheck: '/usr/bin/mysql -h localhost -u root -e "show databases;" || exit 1'
        healthcheck_interval: 10s

    - include_role:
        name: podman_systemd_container
      vars:
        container_name: dbservice-pma
        image: phpmyadmin/phpmyadmin
        description: phpMyAdmin service
        network: container:dbservice-mariadb
        env:
          PMA_HOST: 127.0.0.1
        healthcheck: 'curl http://localhost || exit 1'
        healthcheck_interval: 10s
        depends_on: dbservice-mariadb
