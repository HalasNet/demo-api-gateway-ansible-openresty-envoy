# Solution API Gateway OpenSource avec Git, Ansible, Docker et Openresty / Envoy

pour plus de détails : [http://blog.ebiznext.com/2019/04/05/apigateway-openresty-envoy-ansible/]()


## Pré-requis

* ansible
* VirtualBox
* Vagrant
  
## Préparation 

Le provisioning de la VM se fait avec ansible. La configuration  des ouvertures de ports, adresse ip, provisionning sont à modifier au niveau du fichier `platforms/vagrant/Vagrantfile`

Préparer la vm

```bash
$ cd platforms/vagrant
$ vagrant up --provision
```

Pour se connecter à la machine en ssh, toujours sous `platforms/vagrant`

```bash
$ vagrant ssh
```

Pour prendre en compte de nouvelles configurations après modification du `Vagrantfile`

```bash
$ vagrant reload --provision
```

## Démonstration

Pour lancer le déploiement des services

```bash
# Créer le fichier qui contiendra le vault password
echo 'changeit' > vault-password.txt
# Deployer `envoy` 
ansible-playbook playbooks/envoy/site.yml -i platforms/vagrant/vagrant-inventory.ini --vault-password-file vault-password.txt
# Deployer openresty
ansible-playbook playbooks/openresty/site.yml -i platforms/vagrant/vagrant-inventory.ini --vault-password-file vault-password.txt

```

Tester avec `envoy`

```bash
$ curl -k https://localhost:10000/er/latest | jq 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0*   
{
  "base": "EUR",
  "rates": {
    "BGN": 1.9558,
    "NZD": 1.6718,
    "ILS": 4.0346,
    "RUB": 73.0029,
    "CAD": 1.4984,
    "USD": 1.1277,
    "PHP": 58.682,
    "CHF": 1.127,
    "ZAR": 15.8215,
    "AUD": 1.5781,
    "JPY": 125.51,
    "TRY": 6.4056,
    "HKD": 8.844,
    "MYR": 4.622,
    "THB": 35.804,
    "HRK": 7.4342,
    "NOK": 9.619,
    "IDR": 15937.78,
    "DKK": 7.465,
    "CZK": 25.618,
    "HUF": 321.5,
    "GBP": 0.86335,
    "MXN": 21.3479,
    "KRW": 1284.96,
    "ISK": 133.8,
    "SGD": 1.5255,
    "BRL": 4.3471,
    "PLN": 4.287,
    "INR": 78.188,
    "RON": 4.761,
    "CNY": 7.5688,
    "SEK": 10.425
  },
  "date": "2019-04-09"
}
```

des url démo ont été mis en place pour `openresty`

```nginx
    # Openbar echo
    location /echo {
        default_type text/plain;
        echo "->$remote_addr\n";
    }

    # Rate limited echo
    location /recho {
        limit_req zone=req_zone nodelay;
        default_type text/plain;
        echo "->$remote_addr\n";
    }

    # Basic Auth Echo
    location /acho {
        auth_basic "Free Speech, not really !";
        auth_basic_user_file /etc/nginx/conf.d/auth/htpasswd;
        client_body_buffer_size 32k;
        default_type text/plain;
        echo ">>>>$remote_addr\n";
    }

    # Basic Auth Rate limited echo
    location /racho {
        limit_req zone=req_zone nodelay;
        auth_basic "Free Speech, not really !";
        auth_basic_user_file /etc/nginx/conf.d/auth/htpasswd;
        client_body_buffer_size 32k;
        default_type text/plain;
        echo ">>>>$remote_addr\n";
    }
```

Quelques test avec `curl`

```bash
$ curl -k https://localhost:4443/echo
->10.0.2.2

$ curl -I -k https://localhost:4443/acho 2>/dev/null | head -n 1
HTTP/1.1 401 Unauthorized

$ curl -k -u janedoe:letmein https://localhost:4443/acho
>>>>10.0.2.2

# curl toutes les 10 secondes
$ for i in {1..6}; do curl -I -k https://localhost:4443/recho 2>/dev/null | head -n 1 ; sleep 10s; done
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK

# curl toutes les 2 secondes
$ for i in {1..6}; do curl -I -k https://localhost:4443/recho 2>/dev/null | head -n 1 ; sleep 2s; done
HTTP/1.1 200 OK
HTTP/1.1 503 Service Temporarily Unavailable
HTTP/1.1 503 Service Temporarily Unavailable
HTTP/1.1 503 Service Temporarily Unavailable
HTTP/1.1 503 Service Temporarily Unavailable
HTTP/1.1 200 OK

$ curl -k -u janedoe:letmein https://localhost:4443/er/latest | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   458    0   458    0     0   4099      0 --:--:-- --:--:-- --:--:--  4126
{
  "base": "EUR",
  "rates": {
    "BGN": 1.9558,
    "NZD": 1.6718,
    "ILS": 4.0346,
    "RUB": 73.0029,
    "CAD": 1.4984,
    "USD": 1.1277,
    "PHP": 58.682,
    "CHF": 1.127,
    "ZAR": 15.8215,
    "AUD": 1.5781,
    "JPY": 125.51,
    "TRY": 6.4056,
    "HKD": 8.844,
    "MYR": 4.622,
    "THB": 35.804,
    "HRK": 7.4342,
    "NOK": 9.619,
    "IDR": 15937.78,
    "DKK": 7.465,
    "CZK": 25.618,
    "HUF": 321.5,
    "GBP": 0.86335,
    "MXN": 21.3479,
    "KRW": 1284.96,
    "ISK": 133.8,
    "SGD": 1.5255,
    "BRL": 4.3471,
    "PLN": 4.287,
    "INR": 78.188,
    "RON": 4.761,
    "CNY": 7.5688,
    "SEK": 10.425
  },
  "date": "2019-04-09"
}
```
