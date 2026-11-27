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
│   ├── servidor_db.yml                # Instalação do MariaDB e montagem NFS via Autofs
│   ├── servidor_app.yml               # Instalação do Apache2, página HTML e montagem NFS via Autofs
│   └── cliente_cli.yml                # Firefox ESR, Xauth, X11 Forwarding e montagem NFS via Autofs
└── README.md                          # Documentação do projeto
```

---

## 4. Funcionamento dos Serviços

1. **Orquestração e Segurança Sem Chaves Estáticas (`Vagrantfile`):**
   - Antes do boot, um gatilho (`trigger.before :up`) desativa o servidor DHCP nativo da interface *Host-Only* do VirtualBox para evitar conflitos com o servidor `arq`.
   - Para manter o repositório livre de chaves privadas, o Vagrant gera chaves SSH exclusivas na pasta local `.vagrant/` (ignorada pelo Git). A máquina `cli` coleta essas chaves no provisionamento inicial, gera o inventário dinâmico `/tmp/hosts.ini` e executa todos os playbooks via `ansible_local`.

2. **Controle de Acesso e SSH (`usuarios_ssh.yml`):**
   - Cria o usuário `emanuel` pertencente ao grupo `ifpb` com permissão administrativa (`sudo` sem senha).
   - Gera um par de chaves `ed25519` no host `cli` e distribui a chave pública para todas as máquinas.
   - Bloqueia login de `root` e autenticação por senha no SSH, restringindo o acesso aos grupos `vagrant` e `ifpb` e exibindo o banner obrigatório.

3. **Armazenamento LVM e Compartilhamento NFS (`servidor_arq.yml`):**
   - Agrupa três discos virtuais de 10 GB (`/dev/sdb`, `/dev/sdc`, `/dev/sdd`) no Volume Group `dados` (30 GB) e cria o Logical Volume `ifpb` de 15 GB formatado em `ext4`, montado persistentemente em `/dados`.
   - Exporta `/dados/nfs` com gravação síncrona (`sync`) e mapeamento de todos os clientes remotos (`all_squash`) para o usuário de sistema `nfs-ifpb` (UID/GID `2000`, sem shell interativo).
   - As máquinas `db`, `app` e `cli` utilizam o serviço `autofs` para montar automaticamente esse compartilhamento em `/var/nfs` sob demanda.

4. **Serviços de Rede (`servidor_arq.yml` e `site.yml`):**
   - **DHCP (`isc-dhcp-server`):** Atua de forma autoritativa na interface `eth1`, entregando endereços dinâmicos entre `.50` e `.100` (lease padrão de `180s` e máximo de `3600s`) e reservas fixas por MAC para `db` (`.130`) e `app` (`.131`).
   - **DNS (`bind9`):** Gerencia as zonas direta (`emanuel.souza.devops`) e reversa (`56.168.192.in-addr.arpa`), validando sintaticamente os arquivos antes de iniciar o serviço.

---

## 5. Como Executar

### Pré-requisitos
- [Oracle VM VirtualBox](https://www.virtualbox.org/)
- [HashiCorp Vagrant](https://www.vagrantup.com/)
- Git

### Passo a passo

1. Clone o repositório e entre na pasta do projeto:
   ```bash
   git clone https://github.com/EmanuelSouza-ctrl/DevOps-IFPB.git
   cd DevOps-IFPB
   ```

2. Inicie e provisione toda a infraestrutura:
   ```bash
   vagrant up
   ```
   *(Dica para ambientes Windows: caso prefira evitar disputa de disco no boot simultâneo, execute sequencialmente `vagrant up arq`, `vagrant up db`, `vagrant up app` e `vagrant up cli`.)*

3. *(Opcional)* Para resolver os nomes `.emanuel.souza.devops` diretamente na máquina hospedeira (Windows/Linux), configure o DNS da placa *VirtualBox Host-Only* para `192.168.56.129` ou adicione ao arquivo `hosts` do sistema operacional:
   ```text
   192.168.56.129  arq.emanuel.souza.devops
   192.168.56.130  db.emanuel.souza.devops
   192.168.56.131  app.emanuel.souza.devops
   ```

---

## 6. Validação dos Serviços

Acesse a máquina cliente (`cli`):
```bash
vagrant ssh cli
```

Dentro do terminal do `cli`, execute os testes abaixo:

```bash
# 1. Testar resolução DNS (direta e reversa)
host arq.emanuel.souza.devops
host db.emanuel.souza.devops
host app.emanuel.souza.devops
host 192.168.56.131

# 2. Testar SSH com banner e autenticação por chave usando o usuário emanuel
sudo su - emanuel -c "ssh -o StrictHostKeyChecking=no app.emanuel.souza.devops 'hostname && uptime'"

# 3. Testar montagem automática NFS via Autofs
ls -la /var/nfs
touch /var/nfs/teste_ifpb.txt && ls -l /var/nfs/teste_ifpb.txt

# 4. Testar servidor Web Apache2
curl -I http://app.emanuel.souza.devops
```

Para abrir a página web graficamente a partir do `cli` (requer servidor X11 ativo no hospedeiro e conexão com `vagrant ssh cli -- -X`):
```bash
firefox-esr http://app.emanuel.souza.devops &
```
