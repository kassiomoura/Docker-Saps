# Docker-Saps
* Verifique de estar usando Ubuntu 16.04 ou 18.04

## Dockerfile-catalog
### Execução
1. Para executar o Dockerfile execute os seguinte comandos:

1.1. Build a imagem usando:
     
     
     sudo docker build -f Dockerfile-catalog -t catalog:v4 .
     
     
1.2. Rode a imagem usando:
     
     
     sudo docker run --net=host -it catalog:v4 bash 
     
     
1.2.1 Dentro do bash do catalog execute:
     
      
      pg_createcluster 12 main --start
      
      
      Logo após configure o catalog e o arrebol:
      
      
      su postgres
      
      
      
      export catalog_user=catalog_user
      export catalog_passwd=catalog_passwd
      export catalog_db_name=catalog_db_name
      psql -c "CREATE USER $catalog_user WITH PASSWORD '$catalog_passwd';"
      psql -c "CREATE DATABASE $catalog_db_name OWNER $catalog_user;"
      psql -c "GRANT ALL PRIVILEGES ON DATABASE $catalog_db_name TO $catalog_user;"
      export arrebol_db_user=arrebol_db_user
      export arrebol_db_passwd=@rrebol
      export arrebol_db_name=arrebol 
      psql -c "CREATE USER $arrebol_db_user WITH PASSWORD '$arrebol_db_passwd';"
      psql -c "CREATE DATABASE $arrebol_db_name OWNER $arrebol_db_user;"
      psql -c "ALTER USER $arrebol_db_user PASSWORD '$arrebol_db_passwd';"
      exit
      
      
* Por fim execute o script **/scripts/fetch_landsat_data.sh** (ele demora um pouco
 
 ```
 cd scripts
 bash fetch_landsat_data.sh
 ```
 
## IMPORTANTE
* Para não derrubar o catalog execute o comando ```Ctrl + P``` seguido de ```Ctrl + Q``` no terminal

## Dockerfile-archiver
### Execução
1. Modifique o arquivo sites-available/default-ssl.conf
   
   ```
   sudo vim /etc/apache2/sites-available/default-ssl.conf
   ```
   
2. Mude o DocumentRoot para o diretorio do nfs (default = /nfs)
   
   ```
   DocumentRoot $nfs_server_folder_path 
   # Exemplo: DocumentRoot /nfs
   ```
   
3. Modifique o arquivo sites-available/000-default.conf
   
   ```
   sudo vim /etc/apache2/sites-available/000-default.conf
   ```
   
4. Mude o DocumentRoot e adicione as linhas em sequencia
   
   ```
   DocumentRoot $nfs_server_folder_path 
   # Exemplo: DocumentRoot /nfs
           Options +Indexes
           <Directory $nfs_server_folder_path>
           # Exemplo: <Directory /nfs>
                   Options Indexes FollowSymLinks
                   AllowOverride None
                   Require all granted
           </Directory>
   ```
   
5. Modifique o arquivo sites-available/000-default.conf
  
   ```
   sudo vim /etc/apache2/apache2.conf
   ```
   
6. Mude o FilesMatch 
   
   ```
   <FilesMatch ".+\.(txt|TXT|nc|NC|tif|TIF|tiff|TIFF|csv|CSV|log|LOG|metadata)$">
           ForceType application/octet-stream
           Header set Content-Disposition attachment
   </FilesMatch>
   ```
   
7. Para executar o Dockerfile execute os seguinte comandos:
 
7.1. Build a imagem usando:
      
      
      sudo docker build -f Dockerfile-archiver -t archiver:v4 .
      
        
7.2. Rode a imagem usando:

     
     sudo docker run --net=host archiver:v4
     
        
## Dockerfile-dispatcher
### Execução
1. Configure o arquivo **/config/dispatcher.conf** de acordo com os outros componentes
   * Exemplo (nfs): [dispatcher.conf](./confs/dispatcher/clean/dispatcher.conf) 
   
2. Para executar o Dockerfile execute os seguinte comandos:

2.1. Build a imagem usando:
     
     
     sudo docker build -f Dockerfile-dispatcher -t dispatcher:v4 .
     
        
2.2. Rode a imagem usando:
     
     
     sudo docker run --net=host dispatcher:v4
     
        
## Dockerfile-scheduler
### Execução
1. Configure o arquivo **/config/scheduler.conf** de acordo com os outros componentes
   * Exemplo (nfs): [scheduler.conf](./confs/scheduler/clean/scheduler.conf) 
   
2. Para executar o Dockerfile execute os seguinte comandos:

2.1. Build a imagem usando:
     
     
     sudo docker build -f Dockerfile-scheduler -t scheduler:v4 .
     
        
2.2. Rode a imagem usando:
     
     
     sudo docker run --net=host scheduler:v4
     
        
## Dockerfile-dashboard
### Execução:
1. Configure o host e as portas em [**/backend.config**](./confs/dashboard/clean/backend.config)

2. Configure a urlSapsService em [**/public/dashboardApp.js**](./confs/dashboard/clean/dashboardApp.js) (Linha 52)

3. Para executar o Dockerfile execute os seguinte comandos:

3.1. Build a imagem usando:
     
     
     sudo docker build -f Dockerfile-dashboard -t dashboard:v4 .
     
        
3.2. Rode a imagem usando:
     
     
     sudo docker run --net=host dashboard:v4
     
        
4. Para verificar se esta funcionando acesse o endereço IP configurado no dashboard usando:
   
   ```
   login: admin_saps
   senha: admin_password
   ```
   
## Dockerfile-arrebol
### Execução
1. Configure os arquivos **src/main/resources/application.properties** e **src/main/resources/arrebol.json** de acordo com os outros componentes
   * Exemplo: [application.properties](./confs/arrebol/clean/application.properties) 
   * Exemplo: [arrebol.json](./confs/arrebol/clean/arrebol.json)
   
2. Para executar o Dockerfile execute os seguinte comandos:
 
2.1. Build a imagem usando:
     
     
     sudo docker build -f Dockerfile-arrebol -t arrebol:v4 .
     
        
2.2. Rode a imagem usando:
     
     
     sudo docker run --net=host arrebol:v4
     
        
3. Após a execução do arrebol, são criadas as tabelas no bd, com isso é preciso adicionar as seguintes constraints:
   
   ```
   psql -h localhost -p 5432 arrebol arrebol_db_user
   ALTER TABLE task_spec_commands DROP CONSTRAINT fk7j4vqu34tq49sh0hltl02wtlv;
   ALTER TABLE task_spec_commands ADD CONSTRAINT commands_id_fk FOREIGN KEY (commands_id) REFERENCES command(id) ON DELETE CASCADE;

   ALTER TABLE task_spec_commands DROP CONSTRAINT fk9y8pgyqjodor03p8983w1mwnq;
   ALTER TABLE task_spec_commands ADD CONSTRAINT task_spec_id_fk FOREIGN KEY (task_spec_id) REFERENCES task_spec(id) ON DELETE CASCADE;

   ALTER TABLE task_spec_requirements DROP CONSTRAINT fkrxke07njv364ypn1i8b2p6grm;
   ALTER TABLE task_spec_requirements ADD CONSTRAINT task_spec_id_fk FOREIGN KEY (task_spec_id) REFERENCES task_spec(id) ON DELETE CASCADE;

   ALTER TABLE task DROP CONSTRAINT fk303yjlm5m2en8gknk80nkd27p; 
   ALTER TABLE task ADD CONSTRAINT task_spec_id_fk FOREIGN KEY (task_spec_id) REFERENCES task_spec(id) ON DELETE CASCADE;
   ```
   
