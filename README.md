# Solution API Gateway OpenSource avec Git, Ansible, Docker et Openresty / Envoy

pour plus de détails : [http://blog.ebiznext.com/2019/04/...]


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
