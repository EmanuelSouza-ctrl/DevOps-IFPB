# Projeto DevOps - IFPB

**Aluno:** Emanuel Augusto Correia de Souza  
**Matrícula:** 20232380029  
**Disciplina:** Administração de Sistemas Abertos  
**Professor:** Leônidas Lima  
**Instituição:** Instituto Federal da Paraíba (IFPB) — Campus João Pessoa  
**Período:** 2026.2  
**Modalidade de desenvolvimento:** Individual

## 1. Objetivo

Provisionar e configurar automaticamente uma infraestrutura de rede virtualizada utilizando Vagrant, VirtualBox e Ansible, contemplando serviços de DHCP autoritativo, DNS autoritativo (direto e reverso), armazenamento LVM, compartilhamento NFS com montagem automática via Autofs, banco de dados MariaDB, servidor web Apache2 e host cliente com suporte a X11.

## 2. Arquitetura

A infraestrutura é composta por quatro máquinas virtuais Debian 12 (`debian/bookworm64`):

| Hostname (FQDN) | Função | Endereço IP | Atribuição | RAM |
|---|---|---|---|---:|
| `arq.emanuel.souza.devops` | Arquivos (LVM/NFS), DHCP e DNS | `192.168.56.129` | Estático no Vagrantfile | 512 MB |
| `db.emanuel.souza.devops` | Banco de dados MariaDB | `192.168.56.130` | DHCP (reserva por MAC) | 512 MB |
| `app.emanuel.souza.devops` | Servidor web Apache2 | `192.168.56.131` | DHCP (reserva por MAC) | 512 MB |
| `cli.emanuel.souza.devops` | Cliente Linux (Firefox/X11) | `192.168.56.50–100` | DHCP dinâmico | 1024 MB |

- **Rede privada:** `192.168.56.0/24`
- **Domínio:** `emanuel.souza.devops`
- **Servidor DNS:** `192.168.56.129` (encaminhadores `1.1.1.1` e `8.8.8.8`)
- **Servidor NTP / Fuso:** `pool.ntp.br` / `America/Recife`

## 3. Execução

Clone o repositório e inicie o provisionamento completo com um único comando:

```bash
git clone [https://github.com/EmanuelSouza-ctrl/DevOps-IFPB.git](https://github.com/EmanuelSouza-ctrl/DevOps-IFPB.git)
cd DevOps-IFPB
vagrant up
