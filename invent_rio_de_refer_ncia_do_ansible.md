# Arquivo de inventário de referência do projeto DevOps - IFPB
# Nota: Durante o provisionamento automático via `vagrant up`, o Vagrantfile gera
# dinamicamente uma cópia deste inventário em /tmp/hosts.ini dentro da VM `cli`,
# apontando para as chaves SSH temporárias criadas em .vagrant/machines/.

[todos]
arq ansible_host=192.168.56.129 ansible_user=vagrant
db  ansible_host=192.168.56.130 ansible_user=vagrant
app ansible_host=192.168.56.131 ansible_user=vagrant
cli ansible_connection=local

[servidores]
arq
db
app

[clientes]
db
app
cli