Instalação da Ferramenta de TCC para o Moodle
=============================================

Máquina de Desenvolvimento
--------------------------

Instalação do chef-solo

    sudo curl -L https://www.opscode.com/chef/install.sh | sudo bash
    chef-solo -v # para verificar a instalação
    
Repositorio chef-solo (decompactar origem)

    cd ~/moodle-tcc-instala
    tar -xzvf chef-solo.tar.gz
        
Comando para gerar o arquivo com a chave criptográfica: **encrypted_data_bag_secret** 

    cd ~/moodle-tcc-instala/.chef
    openssl rand -base64 1024 > encrypted_data_bag_secret

Geração da senha de deploy. A linha abaixo irá cifrar todos os dados    
    
    cd ~/moodle-tcc-instala
    knife data bag from file users data_bags/users/deploy.json --secret-file .chef/encrypted_data_bag_secret
    
Execução do chef-solo
    
    sudo su
    cd ~/moodle-tcc-instala
    chef-solo -c solo.rb -j tcc.json -l info

Máquina de Produção
-------------------

