Instalação da Ferramenta de TCC para o Moodle
=============================================

Máquina de Desenvolvimento
--------------------------

### Pré-requisitos <a name="prerequisito_dev"></a>

* Ajustar o shell para realizar o login: Ex. abrir o shel com o comando => bash --login
* Reabrir o Shell
* Ao menos um Weberver deve estar instalado. 
    * Para utilizar o Apache Webserver:
        * Digitar: ```which apache2```. Se não houver retorno deste comando, então o apache deve ser instalado. O retorno deve ser algo parecido como: ```/usr/sbin/apache2``` 
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

Geração do usuário de deploy. A linha abaixo irá cifrar todos os dados    
    
**Ob.: Antes de gerar o usuário de deploy ele deve ser configurado** 

```sh
cd ~[/moodle-tcc-instala]/chef_tcc/
knife data bag from file users data_bags/users/deploy.json --secret-file .chef/encrypted_data_bag_secret
```

### Configurar a instalação <a name="configura_dev"></a>

Editar o arquivo **tcc_dev.json**

Abaixo serão descritas as seções do arquivo de configuração da instalação da máquina de desenvolvimento

**Obs.: O arquivo de configuração poderá ser alterado. Sempre que for alterado deve ser executada 
a [instalação](#instala_dev) novamente. Dessa forma a instalação da ferramenta se adequará aos novos parâmetros.**

#### run_list

    "run_list": [
      "role[ubuntu]",
      "role[deploy]",
      "recipe[rvm]",
      "recipe[rvm::system]",
      "recipe[latex]",
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
      "user": "rsc",
      "ruby_flavor": "ruby",
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
      "gpg": {
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
          "name2": "my_tcc_local",
          "user_deploy": "rsc",
          "config_dir": "/home/rsc/workspace/tcc",
          "server_name": "", // deve ser vazio para desenvvolvimento
          //"server_name": "tcc.unasus.ufxx.br", // se for servidor deve conter o nome do site
          ...
          
##### Database

          ...
          "create_db_and_user": true,
          "dbgrant_hosts": [
            "localhost",
            "127.0.0.1",
            "150.162.242.121",
            "192.168.25.41",
            "192.168.3.103",
            "150.162.66.136",
            "150.162.9.59"
          ],
          "xdatabase_download_site": "http://192.168.25.41/tcc_unasus-20150819.sql.gz",
          "xdatabase_download_site": "http://192.168.3.103/tcc_unasus-20150819.sql.gz",
          "dbadapter": "mysql2",
          "dbhost": "127.0.0.1",
          "dbname": "tcc_unasus",
          "dbuser": "tcc_unasus",
          "dbpass": "tcc_unasus",
          "dbsocket": false,
          ...

#### Reporsitório

          ...  
          "web_root_dir": "/home/rsc/workspace/tcc/current",
          "git_auto_update": false,
          "git_repo": "git@gitlab.setic.ufsc.br:tcc-unasus/sistema-tcc.git",
          "git_revision": "master",
          "root_https": false,
          ...  

#### Servidor de Email

          ...  
          "delivery_method": "file", //"smtp",
          "smtp_address": "", //"smtp.sistemas.ufxx.br",
          "smtp_port": "", //"25",
          "smtp_authentication": "", //"login",
          "smtp_user_name": "", //"unasus",
          "smtp_password": "", //"unasus123",
          ...  

#### Conexão com o moodle

          ...  
          "moodle_dbadapter": "mysql2",
          "moodle_dbhost": "192.168.25.41",
          "moodle_dbname": "moodle_unasus2",
          "moodle_dbuser": "root",
          "moodle_dbpass": "",
          "moodle_dbport": "3306",
          "moodle_dbsocket": false,
          ...  

#### Configuração do LTI do TCC

          ...  
          "tcc_moodle_url": "http://unasus2.local",
          "tcc_moodle_token":  "2398954343ji34343iuy43g43432095x",
          "tcc_consumer_key": "consumer_key",
          "tcc_consumer_secret": "consumer_secret",
          "tcc_notification_email ": "unasus@sistemas.ufxx.br"
        }
      ]
    }



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


