Instalação da Ferramenta de TCC para o Moodle
=============================================

Máquina de Desenvolvimento
--------------------------

### Pré-requisitos <a name="prerequisito_dev"></a>

* Ajustar o shell para realizar o login: 
    * Ex. abrir o shel com o comando => bash --login
* Reabrir o Shell
* Ao menos um Weberver deve estar instalado. 
    * Para utilizar o Apache Webserver:
        * Digitar: ```which apache2```. 
            * Se não houver retorno deste comando, então o apache deve ser instalado. 
            * O retorno deve ser algo parecido como: ```/usr/sbin/apache2``` 
        * Para Instalar o Apache: ```sudo apt-get install apache2```
    * Para utilizar o Nginx, verificar na Web.
* Na maquina onde será instalado o TCC: ```sudo apt-get install openssh-server```

### Preparação <a name="preparacao_dev"></a>

Download do arquivo de instalação (com o repositório de instalação completo, **com o git instalado** na máquina)

```sh
git clone git@github.com:robertosilvino/moodle-tcc-instala.git
```

Download do arquivo de instalação (apenas o chef, sem o repositório de instalação completo, **sem o git instalado** na máquina)
    
```sh
cd ~
wget https://github.com/robertosilvino/moodle-tcc-instala/blob/master/chef-solo.tar.gz?raw=true -O chef-solo.tar.gz
```
    
Repositorio chef-solo (decompactar origem). **O texto entre colchete é opcional, deve ser utilizado caso o 
repositório tenha sido clonado.** Caso contrário o texto dentro dos colchetes deve ser suprimido. Isto vale para todos 
os comandos desta página.

```sh
cd ~[/moodle-tcc-instala] 
tar -xzvf chef-solo.tar.gz
```
        
Comando para gerar o arquivo com a chave criptográfica: **encrypted_data_bag_secret** 

```sh
cd ~[/moodle-tcc-instala/]chef_tcc/.chef
openssl rand -base64 1024 > encrypted_data_bag_secret
```

Geração do usuário de desenvolvimento. A linha abaixo irá cifrar todos os dados    
    
**Ob.: Antes de gerar o usuário de desenvolvimento ele deve ser configurado** 

```sh
cd ~[/moodle-tcc-instala]/chef_tcc/
knife data bag from file users data_bags/users/dev.json --secret-file .chef/encrypted_data_bag_secret
```

### Configurar a instalação <a name="configura_dev"></a>

Editar o arquivo **tcc_dev.json**

Abaixo serão descritas as seções do arquivo de configuração da instalação da máquina de desenvolvimento

**Obs.: O arquivo de configuração poderá ser alterado. Sempre que for alterado deve ser executada 
a [instalação](#instala_dev) novamente. Dessa forma a instalação da ferramenta se adequará aos novos parâmetros.**

#### run_list

    "run_list": [
    "role[ubuntu]",
    "recipe[latex]",
    "role[dev]",
    "recipe[rvm::system]",
    "recipe[tcc_dev_env]",
    "recipe[imagemagick]",
    "recipe[imagemagick::devel]",
    "recipe[imagemagick::rmagick]",
    "recipe[tcc_unasus]",
    "recipe[tcc_unasus::mysql]",
    "recipe[tcc_unasus::tcc]",
    "recipe[tcc_dev_env::after_install]"
    ]

#### tcc_dev_env
    
    "tcc_dev_env": {
      "user": "dev",          // Usuário de desenvolvimento, contido no data_bags/users/dev.json.example
      "ruby_flavor": "ruby",  // Tipo de ruby que será utilizado para a instalação na máquina de desenvolvimento
      "ruby_version": "2.2.1"
    }

#### latex

    "latex": {
      "version": 2015
    }

#### mysql

    "mysql": {
    //  "socket": "/var/run/mysqld/mysqld.sock", // quando o mysql estiver pré-instalado
      "socket": "/var/run/mysql-default/mysqld.sock", // quando instalar o mysql pelo instalador
      "bind_address": "127.0.0.1",
      "data_dir": "/var/lib/mysql"
    }

#### rvm 

    "rvm": {
      "gpg": { // Chave para a instalação do RVM
        "keyserver": "hkp://keys.gnupg.net:80",
        "homedir": "/root"
      },
      "gpg_key": "409B6B1796C275462A1703113804BB82D39DC0E3",
      "installer_flags": "stable --autolibs=packages"
    }
    
#### unasus:tccs
    
    "unasus": {
      "tccs": [
        {
          "name": "my_tcc_local",
          "user_deploy": "dev", // Usuário de desenvolvimento, contido no data_bags/users/dev.json.example
          "config_dir": "/home/dev/workspace/tcc",
          "server_name": "", // deve ser vazio para desenvolvimento
          //"server_name": "tcc.unasus.ufxx.br", // se for servidor deve conter o nome do site
          ...
          
##### Database

          ...
          "create_db_and_user": true,
          "dbgrant_hosts": [ // IPs que terão acesso à base de dados
            "localhost",     // Acesso local
            "127.0.0.1",     // Acesso local
            "192.168.0.2"    // Caso alguma outra máquina na rede tenha acesso 
          ],

          // A linha abaixo servirá para restaurar uma base de dados pré-existente do sistema de TCC, 
          // para o caso de uma instalação rápida em outro servidor. Esta redundância pode ser utilizada 
          // para balanceamento de carga, pelo NGinx, por exemplo.
          
          // "database_download_site": "http://192.168.0.2/tcc_unasus-backup.sql.gz",
          "dbadapter": "mysql2",
          "dbhost": "127.0.0.1",
          "dbname": "tcc_unasus",
          "dbuser": "tcc_unasus",
          "dbpass": "tcc_unasus",
          "dbsocket": false,
          ...

#### Reporsitório

          ...  
          "web_root_dir": "/home/dev/workspace/tcc/current",
          "git_auto_update": false, // Não é aconselhada a atualização automática do repositório
          "git_repo": "git@github.com:UFSC/moodle-tcc.git",
          "git_revision": "master", 
          "root_https": false,      // Caso o webserver forneça https. False para desenvolvimento. 
          ...  

#### Servidor de Email

          ...  
          "delivery_method": "file", //"smtp",
          "smtp_address": "",        //"smtp.sistemas.ufxx.br",
          "smtp_port": "",           //"25",
          "smtp_authentication": "", //"login",
          "smtp_user_name": "",      //"unasus",
          "smtp_password": "",       //"unasus123",
          ...  

#### Configuração do LTI do TCC

          ...  
          "tcc_moodle_url": "http://unasus2.local",
          "tcc_moodle_token":  "2398954343ji34343iuy43g43432095x", // Token
          "tcc_consumer_key": "consumer_key",                      // chave para autenticação no LTI do Moodle
          "tcc_consumer_secret": "consumer_secret",                // segredo da chave para autenticação no LTI do Moodle 
          "tcc_notification_email ": "unasus@sistemas.ufxx.br"     // email para o envio
        }
      ]
    }

Para



### Instalar na máquina de desenvolvimento <a name="instala_dev"></a>

Execução da instalação (padrão)
    
```sh
sudo su
cd ~[/moodle-tcc-instala]/chef-tcc/
./instala_tcc.sh -t dev -L ~/chef-tcc/instala.log
```
  
A execução da instalação, comando acima, irá instalar tanto o **chef-solo** quanto a ferramenta de TCC e suas 
dependências.

Para verificar os comandos de instalação possíveis digitar:

```sh
./instala_tcc.sh --help
```    
A seguinte tela com os comandos será apresentada:    

    Instala o ambiente da ferramenta de TCC para o Moodle
    $ ./instala_tcc.sh [opções]

    Opções:
    -h, --help                      Apresenta este manual
    -d, --debug                     Debug da instalação do clef-solo (padrao: Não)
    -s, --silently                  Sem apresentar mensagens (padrao: Não)
    -L, --logfile                   Define a localização do arquivo de log

    -t, --tipo [dev,srv]            Define o tipo de instacao, que pode ser:
                                        - {dev}: Ambiente de Desenvolvimento
                                        - {srv}: Ambiente de Produção (Servidor)

    -a, --atualiza                  Atualiza a instalação do chef-solo (padrao: Não)

    Examplos:
    $ ./instala_tcc.sh -t dev -a
    $ ./instala_tcc.sh -t dev -L ~/chef-tcc/instala.log
    $ ./instala_tcc.sh -t srv

Máquina de Produção
-------------------

Geração da senha de deploy. A linha abaixo irá cifrar todos os dados    
    
```sh
cd ~[/moodle-tcc-instala]/chef_tcc/
knife data bag from file users data_bags/users/deploy.json --secret-file .chef/encrypted_data_bag_secret
```

Preparação do Moodle
--------------------

Dependências do wstcc: (atualizar https://gitlab.setic.ufsc.br/tcc-unasus/local-wstcc/tree/master, version.php) 

* local/relationship // para agrupar estudantes e seus orientadores
* local/tutores      // rotinas para pegar orientadores, tutores e estudantes
* local/ufsc         // para pegar categoria correta conforme relationship de grupo de orientação

### Criar usuário do Webservice do TCC

* Criar usuário: Moodle_Config_User.png
    * Nome: Webservice UNASUS TCC
    * Metodo de autenticação: Autenticação do webservice
 
### Criar um serviço externo para o Webservice do TCC

* Minha página inicial / Administração do site / Plugins / Serviços da Web / Serviços externos
    * Adicionar: Figura
        * Nome: TCC Services
        * Ativado: True
        * Apenas usuários autorizados: True
        * Pode baixar arquivos: True
        * Pode fazer upload de arquivos: False
        * Capacidades: 
            * mod/assign:grade
            * mod/assign:view
            * moodle/course:view
            * moodle/site:accessallgroups
            * webservice/rest:use
    * Usuários Autorizados: Usuário criado no passo anterior (Webservice UNASUS TCC)
    
### Criar token para o usuário do Webservice (Webservice UNASUS TCC)

* Minha página inicial / Administração do site / Plugins / Serviços da Web / Gerenciar tokens 
    * Adicionar: (figura)
        * Identificação de usuário / id de usuário: Webservice UNASUS TCC (Criado anteriormente)
        * Serviço: TCC Services (criado no passo anterior)
    

### Configurar     
    
Atividade de TCC no Moodle
--------------------------

Manual do Coordenador de AVEA, na seção "Configuração da atividade de TCC". O arquivo pode ser baixado em: link



