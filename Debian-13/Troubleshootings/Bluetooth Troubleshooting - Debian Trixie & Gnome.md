# Bluetooth Troubleshooting (Debian / GNOME)

Guia rápido de diagnóstico e correção de problemas comuns de Bluetooth em sistemas Linux baseados em Debian.

**Ambiente de referência:**

| Componente | Versão / Modelo |
|---|---|
| OS | Debian 13 (Trixie) |
| Desktop | GNOME |
| Stack Bluetooth | BlueZ |
| Hardware comum afetado | Intel AX201 |

---

## 1. Bluetooth não inicia (Intel AX201)

### Sintoma

O Bluetooth não ativa após boot do sistema. Comandos como:

```bash
bluetoothctl list
hciconfig
```

mostram o adaptador com problemas ou inexistente. Logs do kernel (`dmesg`) podem mostrar erros como:

```
Bluetooth: hci0: Device boot timeout
Bluetooth: hci0: Intel reset sent to retry FW download
Bluetooth: hci0: command 0xfc05 tx timeout
BD Address: 00:00:00:00:00:00
```

### Causa

Problema conhecido envolvendo:

- Driver `btusb`
- Firmware Intel Bluetooth
- Power management USB (autosuspend)

O autosuspend pode ocorrer antes do firmware terminar de carregar, causando falha de inicialização do controlador.

**Hardware frequentemente afetado:** Intel AX201

### Correção

Desabilitar autosuspend do driver `btusb`.

**1. Criar arquivo de configuração:**

```bash
sudo nano /etc/modprobe.d/btusb.conf
```

Conteúdo:

```
options btusb enable_autosuspend=n
```

**2. Atualizar initramfs:**

```bash
sudo update-initramfs -u
```

**3. Reiniciar o sistema:**

```bash
sudo reboot
```

### Verificação

Após reboot:

```bash
lsusb -t
```

O dispositivo deve aparecer com driver `btusb`, exemplo:

```
Port XX: Dev XX, Class=Wireless, Driver=btusb
```

Outro teste:

```bash
bluetoothctl list
```

Deve retornar o controller Bluetooth.

---

## 2. Bluetooth funciona no sistema mas não aparece no GNOME

### Sintoma

O Bluetooth funciona via terminal, mas:

- não aparece nas configurações do GNOME
- toggle de Bluetooth fica travado
- interface mostra Bluetooth desligado mesmo estando ativo

Exemplo: `bluetoothctl list` retorna o controlador normalmente, mas o GNOME não mostra o dispositivo.

### Causa

Problema entre GNOME, `gnome-bluetooth`, BlueZ e cache de estado do adaptador via D-Bus. O estado do dispositivo pode ficar inconsistente na interface gráfica.

### Diagnóstico

Verificar se o Bluetooth está ativo no sistema:

```bash
bluetoothctl list
```

Se retornar algo como `Controller XX:XX:XX:XX:XX:XX`, o Bluetooth está funcionando.

### Correção

#### Opção 1 — Correção rápida

Reiniciar o serviço Bluetooth e fazer logout/login no GNOME:

```bash
sudo systemctl restart bluetooth
```

#### Opção 2 — Limpar cache do GNOME Bluetooth

```bash
rm -rf ~/.cache/gnome-bluetooth
```

Após isso, fazer logout e login novamente.

#### Opção 3 — Verificar bloqueio por rfkill

Alguns laptops bloqueiam o Bluetooth via hardware. Verificar:

```bash
rfkill list
```

Se aparecer `Bluetooth: blocked`, desbloquear:

```bash
rfkill unblock bluetooth
```

---

## 3. Ferramenta recomendada para diagnóstico

A ferramenta mais confiável para gerenciar Bluetooth no Linux é o `bluetoothctl`.

**Fluxo comum de pareamento:**

```bash
bluetoothctl

power on
agent on
default-agent
scan on
pair DEVICE_MAC
connect DEVICE_MAC
trust DEVICE_MAC
```

---

## 4. Comando útil de diagnóstico

Para verificar erros do Bluetooth rapidamente:

```bash
dmesg | grep -i bluetooth
```

Erros como `Device boot timeout` normalmente indicam problemas de firmware ou power management.

---

## Resumo

| Problema | Solução |
|---|---|
| Bluetooth não inicia (Intel AX201) | Desabilitar autosuspend do `btusb` |
| Bluetooth não aparece no GNOME | Reiniciar bluetooth + limpar cache |
| Dispositivo bloqueado | `rfkill unblock bluetooth` |