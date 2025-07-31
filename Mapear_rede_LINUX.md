# PASSOS
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

