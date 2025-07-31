# PASSOS
## PARA O SERVIDOR

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
   
🖥️ Etapa 2: Configurar clientes
🪟 Em máquinas Windows:
Mapear a pasta da rede:

No Explorador de Arquivos → Clique com o direito em “Este Computador” → “Mapear unidade de rede”

Ex: \\192.168.0.100\automacao → Montar como Z:\

Criar um script .bat:

bat
Copiar
Editar
@echo off
cd /d Z:\ # pasta mapeada do servidor
call C:\AUTOMACAO\venv\Scripts\activate.bat
python main.py
pause
🐧 Em máquinas Linux:
Monte a pasta via NFS ou SMB:

bash
Copiar
Editar
sudo mount -t cifs //192.168.0.100/automacao /mnt/automacao -o guest
Depois, crie um script .sh:

bash
Copiar
Editar
#!/bin/bash
cd /mnt/automacao
source ~/automacao/venv/bin/activate
python3 main.py
Dê permissão de execução:

bash
Copiar
Editar
chmod +x rodar_automacao.sh
