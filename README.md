# Docker-Saps
* Verifique de estar usando Ubuntu 16.04 ou 18.04

## Dockerfile-catalog
### Execução
1. Para executar o Dockerfile execute os seguinte comandos:

     1.1. Build a imagem usando:
      ```
      sudo docker build -f Dockerfile-catalog -t catalog:v4 .
      ```

     1.2. Rode a imagem usando:
      ```
      sudo docker run --net=host -it catalog:v4 bash
      ```

     1.2.1 Dentro do bash do catalog execute:
      ```
      pg_createcluster 12 main --start
      ```

Logo após configure o catalog e o arrebol:
 ```
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
 ```
      
      
* ATENCÃO
Apenas depois de subir o dispatcher execute o script **/scripts/fetch_landsat_data.sh** que está localizado no diretório root (ele demora um pouco):
 
```
cd scripts
bash fetch_landsat_data.sh
```
 
Para configurar a network do catalog, faça o seguinte:
* Crie uma subnet da network fora para a vm
```
sudo docker network create --driver=bridge --subnet=192.168.0.0/16 catalog
```

* Pegue o IP da interface da vm pra criar a conexão
```
inspect catalog
```

* Faça a configuraçãp com a outra vm com a network
```
sudo ip route add <IP_do_catalog> via <IP_do_catalog>
```

* Conexão com o network
```
sudo docker network create --driver=bridge --subnet=192.168.0.0/16 --gateway=192.168.0.1 catalog
```

Refaça os pontos 1.2 em diante usando o novo comando:
```
sudo docker run --net=catalog -p 5432:5432 -it catalog:v4 bash
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
      ```
      sudo docker build -f Dockerfile-archiver -t archiver:v4 .
      ```
      
      
        
     7.2. Rode a imagem usando:
      ```
      sudo docker run --net=host archiver:v4
      ```

* Para configurar o NFS dentro do container siga os seguintes passos:
Entre no container do archiver:v4 usando o seguinte comando:
```
sudo docker exec -it <CONTAINER_ID> bash
```

Para achar o ``` <CONTAINER_ID> ``` do archiver:v4 use o comando: 
```
sudo docker ps
```

Dentro do container execute os seguintes comandos:
```
apt-get update
apt-get install -y nfs-kernel-server
mkdir -p /nfs/
echo /nfs *(rw,fsid=0,insecure,no_subtree_check,async,no_root_squash) >> /etc/exports 
exportfs -arvf

su -                                
service nfs-kernel-server start

su -
service nfs-kernel-server restart 
```
        
## Dockerfile-dispatcher
### Execução
1. Configure o arquivo **/config/dispatcher.conf** de acordo com os outros componentes
   * Exemplo (nfs): [dispatcher.conf](cilasmarques/saps-docs/blob/main/confs/dispatcher/clean/dispatcher.conf)
   
    1.1. Configure o arquivo **/scripts/get_wrs.py** usando o esse get_wrs.py: [get_wrs.py](./get_wrs.py)
    
    1.2. Configure o arquivo **/src/main/java/saps/dispatcher/utils/RegionUtil.java** (linha 27) para:
   ``` 
   new ProcessBuilder("python3.9", "./scripts/get_wrs.py", latitude, longitude).start();
   ```
        

2. Para executar o Dockerfile execute os seguinte comandos:

     2.1. Build a imagem usando:
      ```
      sudo docker build -f Dockerfile-dispatcher -t dispatcher:v4 .
      ```
     
     2.2. Rode a imagem usando:
      ```
      sudo docker run --net=host dispatcher:v4
      ```
          
          
 3. Para tudo funcionar corretamente é necessário configurar o dispatcher da seguinte forma:

      3.1 Entre no container do dispatcher:v4 usando o seguinte comando:
      ```
      sudo docker exec -it <CONTAINER_ID> bash
      ```
  
  * Para achar o ``` <CONTAINER_ID> ``` do dispatcher:v4 use o comando: 
      ```
      sudo docker ps
      ```
     
      3.2 Dentro do container execute os seguintes comandos:
      ```
      pip3 install gdal
      pip3 install shapely
      pip install ogr
      pip install --upgrade --force-reinstall ogr
      ```
           
           
        
## Dockerfile-scheduler
### Execução
1. Configure o arquivo **/config/scheduler.conf** de acordo com os outros componentes
   * Exemplo (nfs): [scheduler.conf](./confs/scheduler/clean/scheduler.conf) 
   
2. Para executar o Dockerfile execute os seguinte comandos:

     2.1. Build a imagem usando:
      ```
      sudo docker build -f Dockerfile-scheduler -t scheduler:v4 .
      ```
     
        
     2.2. Rode a imagem usando:
      ```
      sudo docker run --net=host scheduler:v4
      ```
     
        
## Dockerfile-dashboard
### Execução:
1. Configure o host e as portas em [**/backend.config**](./confs/dashboard/clean/backend.config)

2. Configure a urlSapsService em [**/public/dashboardApp.js**](./confs/dashboard/clean/dashboardApp.js) (Linha 52)

3. Para executar o Dockerfile execute os seguinte comandos:

     3.1. Build a imagem usando:
      ```
      sudo docker build -f Dockerfile-dashboard -t dashboard:v4 .
      ```
     
        
     3.2. Rode a imagem usando:
      ```
      sudo docker run --net=host dashboard:v4
      ```
     
        
4. Para verificar se esta funcionando acesse o endereço IP configurado no dashboard usando:
   
 ```
 login: admin_email
 senha: admin_password
 ```
   
## Dockerfile-arrebol
### Execução
1. Configure os arquivos **src/main/resources/application.properties** e **src/main/resources/arrebol.json** de acordo com os outros componentes
   * Exemplo: [application.properties](./confs/arrebol/clean/application.properties) 
   * Exemplo: [arrebol.json](./confs/arrebol/clean/arrebol.json)
   
2. Para executar o Dockerfile execute os seguinte comandos:
 
     2.1. Build a imagem usando:
      ```
      sudo docker build -f Dockerfile-arrebol -t arrebol:v4 .]
      ```
     
        
     2.2. Rode a imagem usando:
      ```
      sudo docker run --net=host arrebol:v4
      ```
     
        
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
   
