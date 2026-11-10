# SwitchOS for Virtualization

<table>
<tr>
<th width="50%">🇧🇷 Português</th>
<th width="50%">🇺🇸 English</th>
</tr>
<tr>
<td valign="top">

### O que é

Uma imagem baseada em Alpine Linux, criada para virtualizar o SwitchOS (SwOS) e facilitar o aprendizado, já que não existia uma imagem com essa finalidade.

A imagem possui **GUI** (interface web) e **CLI** (console), mesmo que o original não tenha CLI. A CLI existe para facilitar o uso e evitar problemas específicos.

</td>
<td valign="top">

### What it is

An Alpine Linux-based image created to virtualize SwitchOS (SwOS) and make learning easier, since no image existed for this purpose.

The image has both a **GUI** (web interface) and a **CLI** (console), even though the original has no CLI. The CLI is there to make things easier and to avoid specific problems.

</td>
</tr>
<tr>
<td valign="top">

### Aviso importante

- Este é um projeto **third party** (de terceiros) e **NÃO OFICIAL**. **A MikroTik não é responsável por este projeto**, mas temos **autorização explícita da MikroTik** para desenvolvê-lo e mantê-lo.
- **MikroTik** e **SwOS** são marcas da MikroTik.
- Como a imagem foi desenvolvida do zero, **pode haver bugs**. Ela está em constante melhoria, conforme os relatos dos usuários.
- Esta imagem é **somente para estudos** e **não deve ser usada em produção de forma nenhuma**.

</td>
<td valign="top">

### Important notice

- This is a **third-party** and **UNOFFICIAL** project. **MikroTik is not responsible for this project**, but we have **explicit authorization from MikroTik** to develop and maintain it.
- **MikroTik** and **SwOS** are trademarks of MikroTik.
- Since the image was built from scratch, **bugs may exist**. It is constantly being improved based on user reports.
- This image is **for study purposes only** and **must not be used in production in any way**.

</td>
</tr>
<tr>
<td valign="top">

### Créditos

Criado por **[Matheus Salvador](https://github.com/SalvadorJr33)** e **[Redes Brasil](https://www.redesbrasil.com)**.

</td>
<td valign="top">

### Credits

Created by **[Matheus Salvador](https://github.com/SalvadorJr33)** and **[Redes Brasil](https://www.redesbrasil.com)**.

</td>
</tr>
</table>

---

## Sumário / Table of Contents

- 🇧🇷 **Português**
  - [Antes de começar](#antes-de-começar)
  - [EVE-NG Legacy (2.0.3)](#eve-ng-legacy-203)
  - [EVE-NG 6](#eve-ng-6)
  - [Como usar o switch na topologia](#como-usar-o-switch-na-topologia)
  - [Se der problema](#se-der-problema)
- 🇺🇸 **English**
  - [Before you start](#before-you-start)
  - [EVE-NG Legacy (2.0.3) install](#eve-ng-legacy-203-install)
  - [EVE-NG 6 install](#eve-ng-6-install)
  - [Using the switch in a topology](#using-the-switch-in-a-topology)
  - [Troubleshooting](#troubleshooting)

---

# 🇧🇷 Português

## Antes de começar

Você precisa de:

- Acesso SSH ao servidor EVE-NG, como `root`.
- O arquivo da imagem: `virtioa.qcow2`.

Nos comandos abaixo, troque `IP-DO-SERVIDOR` pelo IP do seu EVE-NG e `PORTA-SSH` pela porta SSH dele (normalmente `22`).

Os passos 1 a 3 são iguais no Legacy e no EVE-NG 6. O que muda é a criação do template.

### Passo 1: copiar a imagem para o servidor

Crie a pasta da imagem. O nome precisa estar em minúsculas, com hífen e a versão no final:

```bash
mkdir -p /opt/unetlab/addons/qemu/mk-switchos-1.0
```

Do seu computador, envie a imagem. O arquivo **precisa** se chamar `virtioa.qcow2`:

```bash
scp -P PORTA-SSH virtioa.qcow2 root@IP-DO-SERVIDOR:/opt/unetlab/addons/qemu/mk-switchos-1.0/virtioa.qcow2
```

Se a imagem já estiver no servidor, use `cp` no lugar do `scp`.

### Passo 2: ajustar as permissões

```bash
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
ls -lh /opt/unetlab/addons/qemu/mk-switchos-1.0/
```

Deve aparecer o `virtioa.qcow2`, com cerca de 78 MB.

> **Usa NFS para compartilhar as imagens?** Então **não rode o `fixpermissions`**: ele altera as permissões de todas as imagens da pasta e pode causar problemas no compartilhamento. Ajuste só este arquivo:
>
> ```bash
> chmod 644 /opt/unetlab/addons/qemu/mk-switchos-1.0/virtioa.qcow2
> chown root:root /opt/unetlab/addons/qemu/mk-switchos-1.0/virtioa.qcow2
> ```

## EVE-NG Legacy (2.0.3)

### Passo 3: criar o template

```bash
cat > /opt/unetlab/html/templates/mk-switchos.php << 'EOT'
<?php
# vim: syntax=php tabstop=4 softtabstop=0 noexpandtab laststatus=1 ruler

$p['type'] = 'qemu';
$p['name'] = 'SwitchOS for Virtualization';
$p['cpulimit'] = 1;
$p['icon'] = 'Switch.png';
$p['cpu'] = 1;
$p['ram'] = 256;
$p['ethernet'] = 8;
$p['console'] = 'telnet';
$p['qemu_arch'] = 'x86_64';
$p['qemu_nic'] = 'virtio-net-pci';
$p['qemu_options'] = '-machine type=pc,accel=kvm -serial mon:stdio -nographic';
?>
EOT
php -l /opt/unetlab/html/templates/mk-switchos.php
```

Deve aparecer: `No syntax errors detected`.

### Passo 4: registrar o template no init.php

No Legacy esse passo é obrigatório. Faça backup e abra o arquivo:

```bash
cp /opt/unetlab/html/includes/init.php /opt/unetlab/html/includes/init.php.backup
nano /opt/unetlab/html/includes/init.php
```

Procure a linha do `mikrotik` (`Ctrl+W`, digite `mikrotik`, `Enter`) e **adicione uma linha logo abaixo dela**:

```php
    'mikrotik'    =>  'MikroTik RouterOS',
    'mk-switchos' =>  'SwitchOS for Virtualization',
```

- O texto entre aspas (`mk-switchos`) deve ser igual ao nome do arquivo `.php`, sem o `.php`.
- Termine a linha com vírgula.

Salve com `Ctrl+O`, `Enter`, `Ctrl+X`, e valide:

```bash
php -l /opt/unetlab/html/includes/init.php
```

Se der erro de sintaxe, restaure o backup e refaça o passo com mais atenção:

```bash
cp /opt/unetlab/html/includes/init.php.backup /opt/unetlab/html/includes/init.php
```

## EVE-NG 6

### Passo 3: criar o template

No EVE-NG 6 o template é um arquivo `.yml` e **não é preciso editar o `init.php`**.

Descubra o processador do servidor:

```bash
lscpu | grep -i vendor
```

- `AuthenticAMD`: use a pasta `amd`.
- `GenuineIntel`: use a pasta `intel`.

Crie o template na pasta correspondente (o exemplo usa `amd`; troque por `intel` se for o caso):

```bash
cat > /opt/unetlab/html/templates/amd/mk-switchos.yml << 'EOT'
---
type: qemu
description: SwitchOS for Virtualization
name: SwitchOS
cpulimit: 1
icon: Switch.png
cpu: 1
ram: 256
ethernet: 8
console: telnet
qemu_arch: x86_64
qemu_nic: virtio-net-pci
qemu_options: -machine type=pc,accel=kvm -serial mon:stdio -nographic
EOT
ls -l /opt/unetlab/html/templates/amd/mk-switchos.yml
```

Cuidado com o YAML: indente com espaços (nunca Tab) e sempre deixe um espaço depois dos dois pontos (`ram: 256`, e não `ram:256`).

## Como usar o switch na topologia

1. Abra o EVE-NG no navegador e faça login.
2. Abra um laboratório e clique em **Add Node** (ícone `+`).
3. Procure por **SwitchOS** e selecione a versão **1.0**.
4. Clique em **Add**.

Pontos importantes:

- **A interface `e0` é reservada** e não é uma porta do switch. Deixe a `e0` desconectada e comece a ligar os cabos pela `e1`, que é a `Port1` da tela do SwOS.
- A quantidade de portas é detectada automaticamente pelas interfaces do node. Com `ethernet: 8`, o switch fica com 7 portas (`e1` a `e7`). Aumente esse número no template se precisar de mais.
- A gerência é feita **por uma das portas** (in-band), como num switch real. Para abrir a interface web, ligue uma porta a um objeto **Cloud/pnet** e acesse o IP que o switch pegar por DHCP.
- Usuário padrão: `admin`, com a senha em branco.

Personalização do template:

| Parâmetro | Valores comuns | O que faz |
|---|---|---|
| `name` | qualquer texto | Nome do node na topologia |
| `ram` | 256, 512, 1024 | Memória em MB. 256 é suficiente |
| `cpu` | 1, 2 | Quantidade de CPUs |
| `ethernet` | 8, 16, 26 | Quantidade de interfaces (lembre da `e0` reservada) |
| `console` | `telnet` | Console do node |
| `icon` | `Switch.png` | Ícone na topologia |

## Se der problema

| Sintoma | O que fazer |
|---|---|
| O template não aparece na lista | Limpe o cache do navegador (`Ctrl+Shift+R`). No Legacy, reinicie o Apache: `systemctl restart apache2` |
| O template aparece em cinza | O EVE-NG não achou a pasta da imagem. Confira com `ls /opt/unetlab/addons/qemu/ \| grep mk-switchos`: deve aparecer `mk-switchos-1.0`, com hífen e versão |
| Erro de sintaxe no PHP (Legacy) | Restaure o backup do `init.php` e refaça o passo 4 |
| O node não inicia | Confirme que o arquivo se chama exatamente `virtioa.qcow2` e ajuste as permissões de novo (passo 2) |

Checklist final, antes de testar:

- [ ] A pasta `/opt/unetlab/addons/qemu/mk-switchos-1.0/` existe.
- [ ] O `virtioa.qcow2` está dentro dela.
- [ ] As permissões foram ajustadas (`fixpermissions`, ou `chmod`/`chown` se usar NFS).
- [ ] O template foi criado (`.php` no Legacy, `.yml` no EVE-NG 6).
- [ ] Só no Legacy: a linha foi adicionada no `init.php` e a sintaxe está ok.

---

# 🇺🇸 English

## Before you start

You need:

- SSH access to the EVE-NG server, as `root`.
- The image file: `virtioa.qcow2`.

In the commands below, replace `SERVER-IP` with your EVE-NG IP and `SSH-PORT` with its SSH port (usually `22`).

Steps 1 to 3 are the same on Legacy and EVE-NG 6. Only the template creation differs.

### Step 1: copy the image to the server

Create the image folder. The name must be lowercase, with a hyphen and the version at the end:

```bash
mkdir -p /opt/unetlab/addons/qemu/mk-switchos-1.0
```

From your computer, upload the image. The file **must** be named `virtioa.qcow2`:

```bash
scp -P SSH-PORT virtioa.qcow2 root@SERVER-IP:/opt/unetlab/addons/qemu/mk-switchos-1.0/virtioa.qcow2
```

If the image is already on the server, use `cp` instead of `scp`.

### Step 2: fix the permissions

```bash
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
ls -lh /opt/unetlab/addons/qemu/mk-switchos-1.0/
```

You should see `virtioa.qcow2`, about 78 MB.

> **Sharing images over NFS?** Then **do not run `fixpermissions`**: it changes the permissions of every image in the folder and can break the share. Fix only this file instead:
>
> ```bash
> chmod 644 /opt/unetlab/addons/qemu/mk-switchos-1.0/virtioa.qcow2
> chown root:root /opt/unetlab/addons/qemu/mk-switchos-1.0/virtioa.qcow2
> ```

## EVE-NG Legacy (2.0.3) install

### Step 3: create the template

```bash
cat > /opt/unetlab/html/templates/mk-switchos.php << 'EOT'
<?php
# vim: syntax=php tabstop=4 softtabstop=0 noexpandtab laststatus=1 ruler

$p['type'] = 'qemu';
$p['name'] = 'SwitchOS for Virtualization';
$p['cpulimit'] = 1;
$p['icon'] = 'Switch.png';
$p['cpu'] = 1;
$p['ram'] = 256;
$p['ethernet'] = 8;
$p['console'] = 'telnet';
$p['qemu_arch'] = 'x86_64';
$p['qemu_nic'] = 'virtio-net-pci';
$p['qemu_options'] = '-machine type=pc,accel=kvm -serial mon:stdio -nographic';
?>
EOT
php -l /opt/unetlab/html/templates/mk-switchos.php
```

You should see: `No syntax errors detected`.

### Step 4: register the template in init.php

On Legacy this step is mandatory. Back up the file and open it:

```bash
cp /opt/unetlab/html/includes/init.php /opt/unetlab/html/includes/init.php.backup
nano /opt/unetlab/html/includes/init.php
```

Find the `mikrotik` line (`Ctrl+W`, type `mikrotik`, `Enter`) and **add one line right below it**:

```php
    'mikrotik'    =>  'MikroTik RouterOS',
    'mk-switchos' =>  'SwitchOS for Virtualization',
```

- The quoted text (`mk-switchos`) must match the `.php` file name, without the `.php`.
- End the line with a comma.

Save with `Ctrl+O`, `Enter`, `Ctrl+X`, then validate:

```bash
php -l /opt/unetlab/html/includes/init.php
```

If you get a syntax error, restore the backup and redo the step more carefully:

```bash
cp /opt/unetlab/html/includes/init.php.backup /opt/unetlab/html/includes/init.php
```

## EVE-NG 6 install

### Step 3: create the template

On EVE-NG 6 the template is a `.yml` file and you **do not need to edit `init.php`**.

Check the server CPU vendor:

```bash
lscpu | grep -i vendor
```

- `AuthenticAMD`: use the `amd` folder.
- `GenuineIntel`: use the `intel` folder.

Create the template in the matching folder (the example uses `amd`; use `intel` if needed):

```bash
cat > /opt/unetlab/html/templates/amd/mk-switchos.yml << 'EOT'
---
type: qemu
description: SwitchOS for Virtualization
name: SwitchOS
cpulimit: 1
icon: Switch.png
cpu: 1
ram: 256
ethernet: 8
console: telnet
qemu_arch: x86_64
qemu_nic: virtio-net-pci
qemu_options: -machine type=pc,accel=kvm -serial mon:stdio -nographic
EOT
ls -l /opt/unetlab/html/templates/amd/mk-switchos.yml
```

Watch out for YAML: indent with spaces (never Tab) and always leave a space after the colon (`ram: 256`, not `ram:256`).

## Using the switch in a topology

1. Open EVE-NG in your browser and log in.
2. Open a lab and click **Add Node** (`+` icon).
3. Search for **SwitchOS** and select version **1.0**.
4. Click **Add**.

Important points:

- **Interface `e0` is reserved** and is not a switch port. Leave `e0` disconnected and start cabling from `e1`, which is `Port1` on the SwOS screen.
- The number of ports is detected automatically from the node's interfaces. With `ethernet: 8`, the switch has 7 ports (`e1` to `e7`). Increase that number in the template if you need more.
- Management is **in-band** (through one of the ports), like a real switch. To open the web interface, connect a port to a **Cloud/pnet** object and browse to the IP the switch gets via DHCP.
- Default user: `admin`, with a blank password.

Template options:

| Parameter | Common values | What it does |
|---|---|---|
| `name` | any text | Node name in the topology |
| `ram` | 256, 512, 1024 | Memory in MB. 256 is enough |
| `cpu` | 1, 2 | Number of CPUs |
| `ethernet` | 8, 16, 26 | Number of interfaces (remember the reserved `e0`) |
| `console` | `telnet` | Node console |
| `icon` | `Switch.png` | Icon in the topology |

## Troubleshooting

| Symptom | What to do |
|---|---|
| Template does not show in the list | Clear the browser cache (`Ctrl+Shift+R`). On Legacy, restart Apache: `systemctl restart apache2` |
| Template shows in gray | EVE-NG could not find the image folder. Check with `ls /opt/unetlab/addons/qemu/ \| grep mk-switchos`: you should see `mk-switchos-1.0`, with hyphen and version |
| PHP syntax error (Legacy) | Restore the `init.php` backup and redo step 4 |
| Node does not start | Make sure the file is named exactly `virtioa.qcow2` and fix the permissions again (step 2) |

Final checklist, before testing:

- [ ] The folder `/opt/unetlab/addons/qemu/mk-switchos-1.0/` exists.
- [ ] `virtioa.qcow2` is inside it.
- [ ] Permissions were fixed (`fixpermissions`, or `chmod`/`chown` if you use NFS).
- [ ] The template was created (`.php` on Legacy, `.yml` on EVE-NG 6).
- [ ] Legacy only: the line was added to `init.php` and the syntax is OK.
