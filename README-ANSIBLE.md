# 📦 Playbook Ansible - Backup Diário com Rotação Automática

## 📋 Descrição

Este playbook Ansible automatiza o backup diário do **Ansible AWX/Tower** e implementa uma rotação inteligente de backups, mantendo apenas os **3 backups mais recentes** no diretório especificado.

## 🎯 Funcionalidades

✅ Executa o comando de backup: `./setup.sh -b -e backup_dir=/root/bkp`  
✅ Monitora o diretório de backups automaticamente  
✅ Remove backups antigos mantendo apenas os 3 mais recentes  
✅ Logging de execuções  
✅ Validações de segurança (verifica existência de diretórios e scripts)  
✅ Relatório detalhado de cada execução  

---

## 📁 Estrutura do Projeto

```
.
├── ansible-backup-playbook.yml   # Playbook principal
├── inventory.ini                 # Inventário de hosts
├── group_vars/
│   └── all.yml                   # Variáveis configuráveis
└── README-ANSIBLE.md             # Esta documentação
```

---

## ⚙️ Configuração

### 1. Variáveis Configuráveis

Edite o arquivo `group_vars/all.yml` para ajustar às suas necessidades:

```yaml
backup_dir: "/root/bkp"              # Diretório de destino dos backups
setup_script_path: "./setup.sh"      # Caminho do script setup.sh
max_backups_to_keep: 3               # Quantidade de backups a manter
```

### 2. Inventário

Ajuste o arquivo `inventory.ini` conforme seu ambiente:

**Para execução local (no próprio servidor AWX/Tower):**
```ini
[ansible_server]
localhost ansible_connection=local
```

**Para execução remota:**
```ini
[ansible_server]
ansible-awx.example.com ansible_user=root ansible_ssh_private_key_file=/path/to/key
```

---

## 🚀 Como Usar

### Execução Manual

```bash
# Executar o playbook
ansible-playbook -i inventory.ini ansible-backup-playbook.yml

# Com verbosidade (para debug)
ansible-playbook -i inventory.ini ansible-backup-playbook.yml -vvv

# Usando variáveis customizadas
ansible-playbook -i inventory.ini ansible-backup-playbook.yml \
  -e "backup_dir=/opt/backups" \
  -e "max_backups_to_keep=5"
```

---

## 📅 Agendamento no Ansible AWX/Tower

### Passo 1: Criar Projeto

1. Acesse **AWX/Tower → Projects**
2. Clique em **Add** (➕)
3. Preencha:
   - **Name:** `Backup Ansible Diário`
   - **SCM Type:** `Git`
   - **SCM URL:** `https://github.com/seu-usuario/seu-repositorio.git`
   - **SCM Branch:** `main` (ou sua branch principal)
4. Marque: ☑️ **Update Revision on Launch**
5. Clique em **Save**

### Passo 2: Criar Inventário

1. Acesse **Inventories → Add → Add Inventory**
2. Preencha:
   - **Name:** `Ansible Server`
3. Em **Hosts**, adicione:
   - **Host Name:** `localhost`
   - **Variables:**
     ```yaml
     ansible_connection: local
     ansible_python_interpreter: /usr/bin/python3
     ```

### Passo 3: Criar Template de Job

1. Acesse **Templates → Add → Add Job Template**
2. Preencha:
   - **Name:** `Backup Diário do Ansible`
   - **Job Type:** `Run`
   - **Inventory:** `Ansible Server`
   - **Project:** `Backup Ansible Diário`
   - **Playbook:** `ansible-backup-playbook.yml`
   - **Credentials:** Selecione as credenciais apropriadas
3. **Options:**
   - ☑️ Enable Privilege Escalation (become: yes)
4. Clique em **Save**

### Passo 4: Criar Schedule (Agendamento Diário)

1. Dentro do **Job Template** criado, vá para a aba **Schedules**
2. Clique em **Add** (➕)
3. Preencha:
   - **Name:** `Backup Diário - 02:00 AM`
   - **Start Date/Time:** Data e hora de início
   - **Local Time Zone:** Seu fuso horário
   - **Repeat Frequency:** `Daily`
   - **Run Every:** `1 Day`
   - **Run On:** Todos os dias
   - **Run At:** `02:00:00` (ou horário desejado)
4. Clique em **Save**

---

## 📊 Exemplo de Execução

```
PLAY [Backup Diário do Ansible com Rotação de Arquivos] *********************

TASK [Verificar se o diretório de backup existe] *****************************
ok: [localhost]

TASK [Executar backup do Ansible] ********************************************
changed: [localhost]

TASK [Listar todos os backups no diretório] **********************************
ok: [localhost]

TASK [Remover backups excedentes] ********************************************
changed: [localhost] => (item=/root/bkp/backup-20250101.tar.gz)

TASK [Exibir resumo final] ***************************************************
ok: [localhost] => {
    "msg": [
        "============================================",
        "RESUMO DO BACKUP",
        "============================================",
        "Diretório de backup: /root/bkp",
        "Backups mantidos: 3",
        "Backups atuais no diretório: 3",
        "Backups removidos: 1",
        "Status: Backup concluído com sucesso",
        "============================================"
    ]
}

PLAY RECAP *******************************************************************
localhost                  : ok=10   changed=2    unreachable=0    failed=0
```

---

## 🔍 Monitoramento e Logs

O playbook gera logs em: `/var/log/ansible-backup.log`

```bash
# Ver logs de execução
tail -f /var/log/ansible-backup.log

# Exemplo de saída:
# 2025-01-15T02:00:05 - Backup executado. Backups mantidos: 3
# 2025-01-16T02:00:03 - Backup executado. Backups mantidos: 3
```

---

## 🛠️ Troubleshooting

### Problema: Script setup.sh não encontrado

**Solução:** Ajuste a variável `setup_script_path` em `group_vars/all.yml` com o caminho completo:

```yaml
setup_script_path: "/opt/ansible-tower/setup.sh"
```

### Problema: Permissões negadas

**Solução:** Certifique-se de que o playbook está rodando com privilégios elevados (`become: yes`) e que o usuário tem permissões no diretório de backup.

### Problema: Backups não estão sendo removidos

**Solução:** Verifique se há mais de 3 backups no diretório e se os arquivos têm permissões de escrita.

---

## 🔐 Segurança

- ✅ Validação de existência do script antes da execução
- ✅ Validação do diretório de backup
- ✅ Tratamento de erros e falhas
- ✅ Logs de auditoria

---

## 📝 Customizações Avançadas

### Notificação por Email

Adicione ao final do playbook:

```yaml
- name: Enviar notificação por email
  ansible.builtin.mail:
    host: smtp.example.com
    port: 587
    username: "ansible@example.com"
    password: "{{ email_password }}"
    to: "admin@example.com"
    subject: "Backup Ansible - {{ ansible_date_time.date }}"
    body: "Backup executado com sucesso. Backups mantidos: {{ remaining_backups.matched }}"
  when: backup_result.rc == 0
```

### Manter Backups por Tempo (dias)

Substitua a task de remoção por:

```yaml
- name: Remover backups com mais de 7 dias
  ansible.builtin.find:
    paths: "{{ backup_dir }}"
    age: "7d"
    age_stamp: mtime
  register: old_backups

- name: Deletar backups antigos
  ansible.builtin.file:
    path: "{{ item.path }}"
    state: absent
  loop: "{{ old_backups.files }}"
```

---

## 🤝 Contribuindo

Sinta-se à vontade para abrir issues ou pull requests no GitHub!

---

## 📄 Licença

MIT License

---

## 📧 Suporte

Para dúvidas ou suporte, consulte a documentação oficial do Ansible:
- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible AWX/Tower Documentation](https://docs.ansible.com/automation-controller/)

---

**Desenvolvido com ❤️ para automação de backups do Ansible**
