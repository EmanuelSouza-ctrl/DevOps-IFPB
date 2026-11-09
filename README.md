# Projeto DevOps - IFPB

**Aluno:** Emanuel Augusto Correia de Souza  
**Matrícula:** 20232380029  
**Instituição:** Instituto Federal da Paraíba (IFPB)  
**Período:** 2026.2  
**Modalidade de desenvolvimento:** Individual

## 1. Objetivo

Construir uma infraestrutura de rede virtualizada utilizando Vagrant, VirtualBox e Ansible, com servidores de arquivos, banco de dados, aplicação web e uma máquina cliente.

O projeto utiliza automação para instalação, configuração e gerenciamento dos serviços.

O trabalho foi desenvolvido individualmente, com autorização do professor responsável pela disciplina. Todas as etapas de planejamento, implementação, configuração, automação, documentação e testes foram realizadas pelo próprio aluno.

## 2. Arquitetura

A infraestrutura possui quatro máquinas virtuais Debian 12 (Bookworm):

| Máquina | Função | Endereço IP | RAM |
|---|---|---|---:|
| ARQ | Arquivos, DHCP, DNS e NFS | 192.168.56.129 | 512 MB |
| DB | Banco de dados MariaDB | 192.168.56.130 | 512 MB |
| APP | Servidor web Apache2 | 192.168.56.131 | 512 MB |
| CLI | Cliente Linux | 192.168.56.50 (DHCP) | 1024 MB |

Rede privada: `192.168.56.0/24`  
Domínio: `devops.local`  
Servidor DNS: `192.168.56.129`  
Servidor NTP: `pool.ntp.br`  
Fuso horário: `America/Recife`

### 2.1. Desenvolvimento individual e adaptações

O projeto foi realizado individualmente, mediante autorização do professor responsável pela disciplina.

Como a atividade foi originalmente planejada para uma equipe, foram necessárias adaptações na identificação da infraestrutura, especialmente na definição dos endereços IP que seriam calculados a partir das matrículas dos integrantes.

Foi utilizada a matrícula do único integrante, preservando a organização da rede e a separação das funções entre as quatro máquinas virtuais.

As adaptações foram realizadas para manter os requisitos técnicos e funcionais da atividade, sem comprometer a implementação dos serviços solicitados.

## 3. Serviços implementados

### ARQ
- Servidor DHCP com intervalo dinâmico e reservas para DB e APP.
- Servidor DNS autoritativo com zona direta e reversa.
- Encaminhamento DNS externo para `1.1.1.1` e `8.8.8.8`.
- Gerenciamento de armazenamento com LVM.
- Volume lógico de 15 GB montado em `/dados`.
- Servidor NFS exportando `/dados/nfs`.
- Usuário e grupo específicos para controle de acesso ao compartilhamento.

### DB
- MariaDB Server e cliente.
- Autofs para montagem automática do compartilhamento NFS em `/var/nfs`.

### APP
- Apache2.
- Página inicial personalizada com identificação do projeto e do integrante.
- Autofs para montagem automática do compartilhamento NFS em `/var/nfs`.

### CLI
- Firefox ESR.
- Xauth e suporte a encaminhamento gráfico X11 por SSH.
- Autofs para montagem automática do compartilhamento NFS em `/var/nfs`.

### Configuração comum
- Atualização dos pacotes.
- Chrony utilizando `pool.ntp.br`.
- Fuso horário `America/Recife`.
- Grupo `ifpb` e usuário do integrante.
- Autenticação SSH por chave pública.
- Bloqueio de login SSH como root.
- Banner de acesso.
- Permissão sudo para o grupo `ifpb`.

## 4. Estrutura do projeto

```text
devops-ansible/
├── Vagrantfile
├── ansible.cfg
├── README.md
├── site.yml
├── inventory/
│   └── hosts.ini
├── files/
│   └── chave_emanuel.pub
└── playbooks/
    ├── configuracao_basica.yml
    ├── usuarios_ssh.yml
    ├── servidor_arq.yml
    ├── servidor_db.yml
    ├── servidor_app.yml
    └── cliente_cli.yml
```

## 5. Requisitos

- Windows ou Linux como sistema hospedeiro.
- Oracle VirtualBox.
- Vagrant.
- Debian 12 (Bookworm) como sistema das máquinas virtuais.
- Ansible instalado no controlador.
- Conectividade de rede entre o controlador e as máquinas virtuais.

## 6. Execução

### Provisionamento com Vagrant

No computador hospedeiro, entre na pasta que contém o Vagrantfile e execute:

```bash
vagrant validate
vagrant up
vagrant status
```

### Configuração com Ansible

No controlador Ansible, entre na pasta do projeto e verifique a conectividade:

```bash
ansible todos -m ping
```

Verifique a sintaxe dos playbooks:

```bash
ansible-playbook site.yml --syntax-check
```

Execute a configuração completa:

```bash
ansible-playbook site.yml -K
```

A opção `-K` solicita a senha para elevação de privilégios.

## 7. Testes realizados

Foram realizados testes de funcionamento dos seguintes componentes:

- Conectividade Ansible com as quatro máquinas.
- Resolução DNS direta, reversa e externa.
- Distribuição de endereços pelo DHCP.
- Montagem do volume lógico `/dados`.
- Exportação NFS pelo servidor ARQ.
- Montagem NFS automática em DB, APP e CLI.
- Inicialização do MariaDB.
- Inicialização do Apache2 e resposta HTTP 200.
- Instalação do Firefox ESR e Xauth.
- Encaminhamento gráfico X11 por SSH.
- Configuração de fuso horário e segurança SSH.

## 8. Considerações finais

O projeto demonstra a utilização de virtualização e automação de infraestrutura para implantação de serviços de rede em ambiente Linux.

A utilização do Vagrant permite provisionar as máquinas virtuais, enquanto o Ansible automatiza a configuração dos serviços e dos componentes de segurança.

Todo o desenvolvimento foi realizado individualmente pelo aluno identificado neste documento, desde a preparação do ambiente até a execução e validação dos serviços, conforme autorização do professor responsável pela disciplina.
