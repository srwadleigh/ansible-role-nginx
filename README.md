# ansible-roles-nginx

This role is designed to provide a reverse proxy in conjuction with another role based deployment such as a metrics stack

 - https://nginx.org/


## Task Configuration

```
- name: Setup nginx
  hosts: somehost
  become: true
  roles:
    - role: nginx
      nginx_sites:
        - name: lmer2-frontend
          url: lmer2.20c.com
          upstream: http://localhost:8001
          ssl: false
    - role: firewalld
      firewalld_services:
        - http
        - https
      firewalld_ports:
        - 8100/tcp # dashboard insecure mode
      firewalld_forwards:
        - port: 80
          to: 8080
        - port: 443
          to: 8443
```

`ssl` can have a few different meaningful values
-  undfefined will skip ssl entirely including its inclusion in the templates
-  true, lets encrypt will request a cert
-  false, lets encrypt will not request a cert but the template will assume a cert exists

`ssl_barrow` can be set to the url of an existing cert to piggy back on another existing cert


## Deployment and Removal


Sometimes you need to manually stop the running containers to get a clean run when re-deploying
Services must be stopped as the respecitve user or another means to aquire the correct user scope for systemd

```
systemctl --user stop container-nginx.service
```

Deploy

```
ansible-playbook -i hosts site.yml --tags=firewalld,nginx --limit=somehost
```

Remove

```
ansible-playbook -i hosts site.yml --tags=firewalld,nginx --extra-vars "container_state=absent firewall_action=remove" --limit=somehost
```
