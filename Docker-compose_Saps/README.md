# Docker-compose_Saps

* Importante! É preciso mudar as portas (de 5432 para 5433) e a senha (de @rrebol para arrebol) nos seguintes arquivos de configurações do arrebol:

  ```/arrebol/src/main/resources/application.properties```

## Dockerfile-catalog
### Execução
1. Crie uma nova imagem do catalog usando a nova Dockerfile:

      1.1. Build a imagem usando:

          sudo docker build -f Dockerfile-catalog -t catalog:v5 .
          
     
## Dockerfile-arrebol
### Execução
1. Crie uma nova imagem do arrebol usando a nova Dockerfile:

      1.1. Build a imagem usando:

         sudo docker build -f Dockerfile-arrebol -t arrebol:v5 .


* Após a criação das imagem rode o docker-compose:

## Docker-Compose
### Execução
1. Para executar o docker-compose execute os seguintes passos:

      1.1 Suba usando comando:

         sudo docker compose up

   
   * Atenção: Nem todos os containers irão subir mas não se preocupe.


2. É necessário configurar o catalog da seguinte forma:

      2.2 Entre no container do catalog:v5 usando o seguinte comando:

         sudo docker exec <CONTAINER_ID> -it bash
        
  
  * Para achar o ``` <CONTAINER_ID> ``` do catalog:v5 use o comando: 

          sudo docker ps

          
      2.3 Dentro do container execute os seguintes comandos:

          pg_createcluster 12 main --start

    * Isso irá criar o cluster do postgres
     
      2.3.1 Em seguinta execute:

          su postgres
          export arrebol_db_user=arrebol_db_user
          export arrebol_db_passwd=arrebol
          export arrebol_db_name=arrebol
          psql -c "CREATE USER $arrebol_db_user WITH PASSWORD '$arrebol_db_passwd';"
          psql -c "CREATE DATABASE $arrebol_db_name OWNER $arrebol_db_user;"
          psql -c "ALTER USER $arrebol_db_user PASSWORD '$arrebol_db_passwd';"

          export catalog_user=catalog_user
          export catalog_passwd=catalog_passwd
          export catalog_db_name=catalog_db_name
          psql -c "CREATE USER $catalog_user WITH PASSWORD '$catalog_passwd';"
          psql -c "CREATE DATABASE $catalog_db_name OWNER $catalog_user;"
          psql -c "GRANT ALL PRIVILEGES ON DATABASE $catalog_db_name TO $catalog_user;"
          
3. Em uma nova aba repita os passos 2 e 2.2, e depois de subir o dispatcher execute o script do diretorio root **/scripts/fetch_landsat_data.sh** (ele demora um pouco).
 cd scripts
 bash fetch_landsat_data.sh

 
4. Para tudo funcionar corretamente é necessário configurar o dispatcher da seguinte forma:

     4.1 Entre no container do dispatcher:v4 usando o seguinte comando:
     
     ```
     sudo docker exec <CONTAINER_ID> -it bash
     ```
    
  
  * Para achar o ``` <CONTAINER_ID> ``` do dispatcher:v4 use o comando: 

    ```
     sudo docker ps
    ```
    
     
     4.2 Dentro do container execute os seguintes comandos:
     
     ```
      pip3 install gdal
      pip3 install shapely
      pip install ogr
      pip install --upgrade --force-reinstall ogr
     ```
