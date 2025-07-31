# INSTALAÇÃO E CONFIGURAÇÃO DO SAMBA
1. Instalar
```
sudo apt update
sudo apt install samba -y
```
2. Configurar
```
sudo nano /etc/samba/smb.conf
```
* Adicione no final:
      ```
      [nome_da_pasta]
      path = /home/seu_usuario/automacao_dejem
      browseable = yes
      writable = yes
      guest ok = yes
      create mask = 0775
      directory mask = 0775
      ```
3. Permissões
      ```
      sudo chmod -R 775 /home/seu_usuario/nome_da_pasta
      sudo chown -R nobody:nogroup /home/seu_usuario/nome_da_pasta
      ```
4. Reiniciar
      ```
      sudo systemctl restart smbd
      ```      
5. Verificar o IP
      ```
      ip a
      ```
*deve conter isto:
      ```
      inet IP_servidor/24
      ```
# VERIFICAÇÕES
1. Verifique se o compartilhamento realmente existe: No servidor, rode:
      ```      
      testparm -s
      ```
* Esse comando mostra os compartilhamentos disponíveis. Procure por algo como:
      -[nome_da_pasta]
        path = /home/bombeiros/COBOM_server/site/gestao_dejem/auto_dejem
        
- Se não aparecer, é porque o compartilhamento não está configurado corretamente no /etc/samba/smb.conf.

2. Verifique se a pasta está visível na rede:
* Na máquina cliente, tente listar os compartilhamentos disponíveis:
      ```
      smbclient -L //10.44.133.44 -N
      ```
** Saída esperada:
      Sharename       Type      Comment
      ---------       ----      -------
      auto_dejem      Disk
      ...

-Se auto_dejem não aparecer, algo está errado com a configuração do Samba.


# ACESSANDO EM OUTRAS MÁQUINAS
## ETAPA 1 - CONFIGURAR O SERVIDOR

1. Criar nova pasta
2. Acessar o servidor via ssh
      ```
      ssh usuario@IP
      ```
4. Compartilhar via Samba (SMB) (para Windows e Linux)
  * Instale no servidor:
      ```
      sudo apt install samba
      ```
  * Edite o smb.conf:
      ```
      sudo nano /etc/samba/smb.conf
      ```
  * Adicione ao final:
      ```
      [nome_da_pasta]
      path = /home/user/nome_da_pasta
      read only = no
      guest ok = yes"
      ```
  * Depois:
      ```
      sudo systemctl restart smbd
      ```
   
## ETAPA 2 - CONFIGURAR CLIENTES
### EM MÁQUINAS WINDOWS

* No Explorador de Arquivos, Clique com o direito em “Este Computador” → “Mapear unidade de rede”

Ex: \\192.168.0.100\nome_da_pasta => Montar como Z:\

### EM MÁQUINAS LINUX
* Monte a pasta via NFS ou SMB:
```
sudo mount -t cifs //192.168.0.100/nome_da_pasta /mnt/nome_da_pasta_criada -o guest
```

