Como Desinstalar Completamente o Nginx no Ubuntu/Debian
Este guia explica como remover completamente o Nginx, incluindo todos os arquivos de configuração e diretórios residuais, usando o apt.

📋 Pré-requisitos
Acesso de superusuário (sudo)

Terminal aberto

🔄 Passo a Passo
1. Parar o serviço Nginx
bash
sudo systemctl stop nginx
sudo systemctl disable nginx
2. Remover completamente o Nginx
bash
sudo apt purge nginx nginx-common nginx-core
3. Remover dependências não utilizadas
bash
sudo apt autoremove
4. Remover diretórios residuais (opcional)
bash
sudo rm -rf /etc/nginx
sudo rm -rf /var/www/html
sudo rm -rf /var/log/nginx
sudo rm -rf /usr/share/nginx
5. Verificar remoção completa
bash
# Verificar pacotes remanescentes
dpkg -l | grep nginx

# Verificar serviços remanescentes
systemctl list-unit-files | grep nginx
6. Limpar cache do apt
bash
sudo apt autoclean
⚠️ Avisos Importantes
Backup importante: Faça backup de qualquer arquivo de configuração ou site que deseje preservar antes de prosseguir

Cuidado com rm -rf: Os comandos de remoção de diretórios são irreversíveis

Verifique dependências: Algumas aplicações podem depender do Nginx

💡 Comando Único (para usuários experientes)
bash
sudo systemctl stop nginx && sudo systemctl disable nginx && sudo apt purge nginx nginx-common nginx-core && sudo apt autoremove && sudo apt autoclean
🔍 Pós-desinstalação
Após a desinstalação, verifique se todas as portas relacionadas ao Nginx (80, 443) estão livres:

bash
sudo netstat -tulpn | grep :80
sudo netstat -tulpn | grep :443
Se precisar reinstalar o Nginx posteriormente, use:

bash
sudo apt install nginx
