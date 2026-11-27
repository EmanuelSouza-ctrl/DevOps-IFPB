# Projeto DevOps - IFPB

**Aluno:** Emanuel Augusto Correia de Souza  
**Matrícula:** 20232380029  
**Disciplina:** Administração de Sistemas Abertos  
**Professor:** Leônidas Lima  
**Instituição:** Instituto Federal da Paraíba (IFPB) — Campus João Pessoa  
**Período:** 2026.2  
**Modalidade de desenvolvimento:** Individual

---

## 1. Objetivo

Provisionar e configurar automaticamente uma infraestrutura de rede virtualizada utilizando **Vagrant**, **VirtualBox** e **Ansible** (Infraestrutura como Código), contemplando serviços essenciais de administração de sistemas Linux:

- Servidor DHCP autoritativo com reservas por endereço MAC;
- Servidor DNS autoritativo (zonas direta e reversa) com encaminhamento externo;
- Gerenciamento de volumes lógicos com LVM;
- Compartilhamento de arquivos via NFS com mapeamento de usuário dedicado e montagem sob demanda via Autofs;
- Servidor de banco de dados MariaDB;
- Servidor Web Apache2;
- Política de segurança SSH, controle de privilégios (`sudo`), sincronização de horário (NTP) e suporte gráfico X11.

---

## 2. Arquitetura da Infraestrutura

O ambiente é composto por quatro máquinas virtuais baseadas no **Debian 12** (`debian/bookworm64`), interligadas em uma rede privada Host-Only (`192.168.56.0/24`):

| Hostname (FQDN) | Função | Endereço IP | Atribuição de IP | RAM |
| :--- | :--- | :--- | :--- | :---: |
| `arq.emanuel.souza.devops` | Arquivos (LVM/NFS), DHCP e DNS | `192.168.56.129` | Estático no `Vagrantfile` | 512 MB |
| `db.emanuel.souza.devops` | Banco de dados MariaDB | `192.168.56.130` | DHCP (reserva MAC `08:00:27:00:00:30`) | 512 MB |
| `app.emanuel.souza.devops` | Servidor Web Apache2 | `192.168.56.131` | DHCP (reserva MAC `08:00:27:00:00:31`) | 512 MB |
| `cli.emanuel.souza.devops` | Cliente Linux (Firefox/X11) e Controlador Ansible | `192.168.56.50–100` | DHCP dinâmico | 1024 MB |

### Parâmetros Globais de Rede e Sistema
- **Rede Privada:** `192.168.56.0/24` (Gateway: `192.168.56.1`)
- **Domínio Local:** `emanuel.souza.devops`
- **Servidor DNS:** `192.168.56.129` (Forwarders: `1.1.1.1` e `8.8.8.8`)
- **Servidor NTP / Fuso Horário:** `pool.ntp.br` / `America/Recife`

---

## 3. Estrutura do Repositório

```text
DevOps-IFPB/
├── Vagrantfile                        # Definição das 4 VMs, gatilho DHCP, discos e orquestração
├── ansible.cfg                        # Configurações padrão do Ansible (escalonamento e SSH)
├── hosts.ini                          # Modelo de inventário dos hosts gerenciados
├── site.yml                           # Playbook principal que importa todas as etapas
├── playbooks/
│   ├── configuracao_basica.yml        # Atualização do sistema, Chrony (NTP), timezone e grupo ifpb
│   ├── usuarios_ssh.yml               # Usuário emanuel, chaves ed25519, sudo e hardening SSH
│   ├── servidor_arq.yml               # LVM (30GB VG / 15GB LV), NFS, DHCP e DNS (BIND9)
│   ├── servidor_db.yml               # Instalação do MariaDB e montagem NFS via Autofs
│   ├── servidor_app.yml               # Instalação do Apache2, página HTML e montagem NFS via Autofs
│   └── cliente_cli.yml               # Firefox ESR, Xauth, X11 Forwarding e montagem NFS via Autofs
└── README.md                          # Documentação do projeto
