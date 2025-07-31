# PASSOS
## PARA CLIENTE LINUX

1. Criar nova pasta
2. Acessar o servidor via ssh
  ssh usuario@IP
3. Compartilhar via Samba (SMB) (para Windows e Linux)
  * Instale no servidor:
    - sudo apt install samba
  * Edite o smb.conf:
    - sudo nano /etc/samba/smb.conf
  * Adicione ao final:
    ```
    [nome_da_pasta]
    path = /home/user/nome_da_pasta
    read only = no
    guest ok = yes"
    ```
  * Depois:
    - sudo systemctl restart smbd
   
